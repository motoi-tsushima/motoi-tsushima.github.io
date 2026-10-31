---
title: "SnowStack.EncodingProbe 1.3.0 解説 — CSV の読み書きと、文字を失わない書き込み"
layout: single
classes: wide
permalink: /encodingprobe_1_3_0/
author_profile: true
---
（リリース日） document update

SnowStack.EncodingProbe のバージョン 1.3.0 で行った変更の解説です。

1.3.0 の主な変更は、以下のとおりです。

| 対象 | 変更の内容 |
| :---- | :---- |
| PowerShell コマンドレット | CSV の読み書きを行う `Import-ProbedCsv` と `Export-ProbedCsv` を追加しました |
| PowerShell コマンドレット | `Resolve-Encoding` などに、先頭の N 行だけで判定する `-DetectionLineCount` を追加しました |
| PowerShell コマンドレット | **書き込み先で表現できない文字を `?` にして黙って書くのをやめ、エラーにしてファイルを元のままにします** |
| PowerShell コマンドレット | **読み込んでいるファイルへの上書きを、ハードリンクや逆向きの指定でも検出し、中身を失わなくなりました。** 1.2.0 と標準のコマンドは、この場合に中身を失います。Linux / macOS では、ハードリンクの別名と、大文字・小文字だけが違う名前を検出しません（[既知の制限](#linux--macos-の既知の制限)の D2・D5） |
| クラスライブラリ（NuGet パッケージ） | BOM 無しの UTF-16 / UTF-32 の判定など、判定処理の不具合を 3 件直しました。公開 API は変えていません |

1.3.0 には、1.2.0 までと結果が変わる「挙動の変更」がいくつかあります。更新する前に、「[1.2.0 からの挙動の変更](#120-からの挙動の変更)」と「[更新するときの注意](#更新するときの注意)」をご覧ください。

この記事は、1.1.0 で導入した「統一語彙」（`utf8NoBOM` のような文字エンコーディングの名前の体系）と、1.2.0 で追加した `Out-ProbedFile` を前提にしています。それぞれ、以下の記事で解説しています。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

[SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)

インストール方法は、以下の記事をご覧ください。

[SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)

（この記事に掲載した実行結果は、すべて実機で採取したものです。採取日は 2026年10月31日、環境は日本語の Windows 11 上の PowerShell 7.6.9 と Windows PowerShell 5.1.26100 で、特に断らない限り両ホストで同じ結果になります。エラーのメッセージは PowerShell 7.x の日本語の表示を載せ、パスは `...\` と略しています。バイト列の確認には、1.1.0 の記事で紹介した `Show-Bytes` 関数を使っています。Linux / macOS の挙動は、GitHub の README の記載によるもので、この記事のために実機では確かめていません）

## 1.3.0 で追加したコマンド

| コマンド | 役割 | 標準の対応物 |
| :---- | :---- | :---- |
| `Import-ProbedCsv` | 文字エンコーディングを判定して CSV ファイルを読み込む | `Import-Csv` |
| `Export-ProbedCsv` | 文字エンコーディング・BOM・改行を指定して CSV ファイルに書き出す | `Export-Csv` |

あわせて、`Resolve-Encoding`・`Import-ProbedCsv`・`Export-ProbedCsv` に、判定に使う先頭の行数を指定する `-DetectionLineCount` を追加しました。

CSV の解析と文字列化は、PowerShell 7.x の `Import-Csv` / `Export-Csv` から移植した処理で行います。ホストの `Import-Csv` / `Export-Csv` は呼びません。そのため、**Windows PowerShell 5.1 でも PowerShell 7.x と同じ結果になります。** 書き出しは、バイト単位まで同じファイルになります（日付・数値の書式については後述します）。

## Import-ProbedCsv — 判定して CSV を読む

### 基本の使い方

Shift_JIS の CSV を、文字エンコーディングを指定せずに読んでみます。

```
# Shift_JIS の CSV を作る
Set-ProbedContent .\sjis.csv -Encoding shift_jis -Value '名前,都市','佐藤,札幌','鈴木,大阪'

# 文字エンコーディングを判定して読む
Import-ProbedCsv .\sjis.csv
```

```
名前 都市
---- ----
佐藤 札幌
鈴木 大阪
```

同じファイルを標準の `Import-Csv` で `-Encoding` を付けずに読むと、両ホストとも文字化けします。

```
Import-Csv .\sjis.csv
```

```
���O �s�s
---- ----
���� �D�y
���  ���
```

EUC-JP（改行 LF）の CSV も、同じく指定せずに読めます。Windows PowerShell 5.1 の標準の `Import-Csv` は、`-Encoding` が固定の列挙型なので、EUC-JP を指定する方法がありません。

文字エンコーディングが分かっている場合は、`-Encoding` で明示できます。値は `Get-ProbedContent` と同じ統一語彙・WebName・コードページです。

```
Import-ProbedCsv .\sjis.csv -Encoding shift_jis
```

### Windows PowerShell 5.1 の Import-Csv で起きる問題が、起きません

Windows PowerShell 5.1 の標準の `Import-Csv` は、**改行が CR だけの CSV を読むと、エラーも警告も出さずに 0 件を返します。** データが黙って失われます。

```
# 改行を CR にして CSV を作る
Set-ProbedContent .\cr.csv -Encoding utf8NoBOM -LineBreak Cr -Value 'Name,City','Sato,Sapporo','Suzuki,Osaka'

@(Import-Csv .\cr.csv).Count        # 5.1 では 0、7.x では 2
@(Import-ProbedCsv .\cr.csv).Count  # 両ホストとも 2
```

ほかにも、5.1 の標準とは次の点が違います。どれも 7.x の `Import-Csv` に合わせたものです。

| 項目 | 標準 5.1 | 標準 7.x | `Import-ProbedCsv` |
| :---- | :---- | :---- | :---- |
| CR だけの改行 | **0 件（エラーも警告もない）** | 読める | 読める |
| W3C 拡張ログ形式（`#Fields:`） | 未対応 | 対応 | 対応 |
| `#TYPE` の型名の順序 | `CSV:Foo.Bar` が先 | `Foo.Bar` が先 | `Foo.Bar` が先 |
| <code>Get-ChildItem &#124; Import-Csv</code> | エラー | 読める | 読める |

### 複数のファイルと、名前に [ ] があるファイル

`Get-ChildItem` から渡したり、ワイルドカードで複数のファイルを指定したりすると、一つずつ判定して順に読みます。ファイルごとに文字エンコーディングが違っていてもかまいません。

```
# UTF-8 と Shift_JIS の CSV が混ざったフォルダーを、まとめて読む
Get-ChildItem .\sub\*.csv | Import-ProbedCsv

# ワイルドカードでも読める（標準の Import-Csv は、複数に一致するとエラー）
Import-ProbedCsv .\sub\*.csv
```

`Get-ChildItem` から渡したファイルは、名前に `[ ]` があっても、そのファイルそのものを読みます。`a[1].csv` を渡して、`a1.csv` を読んでしまうことはありません（この点は 1.3.0 で、ほかのコマンドも直しました。後述します）。

```
Get-ChildItem -LiteralPath '.\sub\a[1].csv' | Import-ProbedCsv
```

標準の `Import-Csv` と違い、存在しないファイルや、列名が重複したファイルは非終了エラーにして、残りのファイルを読み続けます。ワイルドカードがどのファイルにも一致しない場合は、`Get-ProbedContent` と同じく、エラーにせず何も読みません。

### -Culture と -UseCulture は、別のものです

`Import-ProbedCsv` と `Export-ProbedCsv` には、名前の似た二つのパラメータがあります。

| パラメータ | 意味 |
| :---- | :---- |
| `-Culture` | **文字エンコーディングの判定**に使うカルチャー（`Resolve-Encoding` などと同じ） |
| `-UseCulture` | **区切り文字**に、現在のカルチャーのリスト区切り文字を使う（標準の `Import-Csv` と同じ） |

たとえば、日本語環境で韓国語の CSV を読む場合は、`-Culture ko-KR` を指定します。区切り文字は変わりません。

```
# 韓国語（CP949）の CSV を、日本語環境で読む
Import-ProbedCsv .\korean.csv -Culture ko-KR
```

```
이름 도시
---- ----
김   서울
```

`-Culture` を付けないと、日本語環境では EUC-JP と判定されて文字化けします。`-Culture` については、以下の記事で解説しています。

[-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)

### パラメータ一覧

| パラメータ | 説明 |
| :---- | :---- |
| `-Path` | 読み込むファイル。ワイルドカード対応。パイプラインから受け取れる |
| `-LiteralPath` | 読み込むファイル。ワイルドカード非対応。別名 `PSPath`・`LP`・`FullName` |
| `-Delimiter` | 区切り文字。省略時は `,` |
| `-UseCulture` | 区切り文字に、現在のカルチャーのリスト区切り文字を使う |
| `-Header` | 列名。指定すると、ファイルの 1 行目もデータとして読む（標準と同じ） |
| `-Encoding` | 文字エンコーディング。既定は `Auto`（判定する） |
| `-DetectionLineCount` | 判定に使う先頭の行数（後述） |
| `-Culture` | 判定に使うカルチャー |
| `-Strategy` | 判定方式 |

`-Encoding` を明示した場合は判定を行わないので、`-DetectionLineCount`・`-Culture`・`-Strategy` は意味を持ちません。

## Export-ProbedCsv — 文字エンコーディングを指定して CSV を書く

### 5.1 でも 7.x でも、同じバイト列になります

日本語を含むオブジェクトを、`-Encoding` を付けずに書き出してみます。

```
$rows = @(
  [pscustomobject]@{ 名前 = '佐藤'; 都市 = '札幌'; 点数 = 85 },
  [pscustomobject]@{ 名前 = '鈴木'; 都市 = '大阪'; 点数 = 92 }
)

$rows | Export-ProbedCsv .\out.csv
Show-Bytes .\out.csv
```

```
22 E5 90 8D E5 89 8D 22 2C 22 E9 83 BD E5 B8 82 22 2C 22 E7 82 B9 E6 95 B0 22 0D 0A
22 E4 BD 90 E8 97 A4 22 2C 22 E6 9C AD E5 B9 8C 22 2C 22 38 35 22 0D 0A
22 E9 88 B4 E6 9C A8 22 2C 22 E5 A4 A7 E9 98 AA 22 2C 22 39 32 22 0D 0A
```

（見やすいように、行ごとに折り返しています）

`-Encoding` を省略すると、**両ホストとも BOM 無しの UTF-8（`utf8NoBOM`）**で書きます。両ホストで書いたファイルの SHA-256 も一致しました。

同じことを、Windows PowerShell 5.1 の標準の `Export-Csv` で行うと、こうなります。

```
$rows | Export-Csv .\std.csv
Get-Content .\std.csv
```

```
#TYPE System.Management.Automation.PSCustomObject
"??","??","??"
"??","??","85"
"??","??","92"
```

5.1 の標準の `Export-Csv` は、既定で ASCII で書くので、**日本語がすべて `?`（`3F`）になります。** 元の文字は戻りません。また、既定で `#TYPE` の行を書きます。

`Export-ProbedCsv` は、7.x の標準と同じく、既定で `#TYPE` の行を書きません（`-IncludeTypeInformation` で書けます）。5.1 向けに書かれたスクリプトの `-NoTypeInformation` は、受け付けるだけで何もしません。

### 7.x にしかない機能も、5.1 で使えます

`-UseQuotes`（引用符の付け方）・`-QuoteFields`（引用符で囲む列）・`-NoHeader`（ヘッダー行を書かない）は、標準では PowerShell 7.x にしかありません。`Export-ProbedCsv` では、5.1 でも使えます。

```
# 必要な値だけ引用符で囲む
$rows | Export-ProbedCsv .\q.csv -UseQuotes AsNeeded
```

```
名前,都市,点数
佐藤,札幌,85
鈴木,大阪,92
```

辞書（ハッシュテーブル）を渡した場合は、7.x の標準と同じく、キーを列として書きます（5.1 の標準は、辞書オブジェクトそのもののプロパティを書きます）。

### -Append は、追記先の文字エンコーディングと改行を引き継ぎます

`-Append` で `-Encoding` を省略すると、`Out-ProbedFile -Append` と同じく、追記先の文字エンコーディングと改行を判定して引き継ぎます。`-Encoding` を明示した場合は、バイト列比較の整合性検査を行います。

標準の `Export-Csv -Append` は、追記先と違う文字エンコーディングや改行で書いて、ファイルの中で混ざることがあります。

### 最後の行に改行が無くても、行がつながりません

最後の行に改行が無い CSV に、標準の `Export-Csv -Append` で追記すると、**新しい行が前の行につながってしまいます。**

```
# 最後の行に改行が無い CSV
Set-ProbedContent .\t3.csv -Encoding utf8NoBOM -NoNewline -Value "Name,City`r`nSato,Sapporo"

[pscustomobject]@{ Name = 'Suzuki'; City = 'Osaka' } |
    Export-Csv .\t3.csv -Append -Encoding ascii
Get-Content .\t3.csv
```

```
Name,City
Sato,Sapporo"Suzuki","Osaka"
```

`Export-ProbedCsv -Append` は、追記先の最後の行に改行が無ければ、**改行を補ってから追記します。**

```
[pscustomobject]@{ Name = 'Suzuki'; City = 'Osaka' } |
    Export-ProbedCsv .\t4.csv -Append
Get-ProbedContent .\t4.csv
```

```
Name,City
Sato,Sapporo
"Suzuki","Osaka"
```

Shift_JIS の CSV でも同じです。補う改行と追記するレコードは、追記先の文字エンコーディング（Shift_JIS）と改行（CR-LF）で書きます。

```
# 最後の行に改行が無い、Shift_JIS の CSV
Set-ProbedContent .\tail.csv -Encoding shift_jis -NoNewline -Value "名前,都市`r`n佐藤,札幌"

[pscustomobject]@{ 名前 = '鈴木'; 都市 = '大阪' } | Export-ProbedCsv .\tail.csv -Append
Show-Bytes .\tail.csv
```

| | バイト列 |
| :---- | :---- |
| 追記前 | `96 BC 91 4F 2C 93 73 8E 73 0D 0A 8D B2 93 A1 2C 8E 44 96 79` |
| 追記後 | `96 BC 91 4F 2C 93 73 8E 73 0D 0A 8D B2 93 A1 2C 8E 44 96 79 0D 0A 22 97 E9 96 D8 22 2C 22 91 E5 8D E3 22 0D 0A` |

ただし、追記先の最後の値の引用符が閉じていない場合（`1,"2` で終わっているなど）は、補った改行と追記したレコードが、その引用符の中に入ってしまいます。警告もエラーも出ません。標準の `Export-Csv -Append` も同じ結果になります。

### 読み込んだ CSV に書き戻すと、止まります

標準の `Import-Csv` と `Export-Csv` で、読み込んだファイルにそのまま書き戻すと、**ファイルは 0 バイトになります。**

```
Import-Csv .\rt2.csv | Export-Csv .\rt2.csv -NoTypeInformation
(Get-Item .\rt2.csv).Length   # → 0
```

`Export-ProbedCsv` は、ファイルを開くのを最初の行を書く直前まで遅らせているので、同じファイルであることを検出して止まれます。

```
Import-ProbedCsv .\rt.csv | Export-ProbedCsv .\rt.csv
```

```
'...\rt.csv' は読み取り中のため、書き込み先に指定できません。
同一のファイルを1つのパイプラインで読み書きすると、読み終える前にファイルが切り詰められます。
$text = Get-ProbedContent <path> -Raw のように、いったん変数に受けてください。
```

エラー ID は `SamePathRoundTrip` です。rt.csv は元のまま残ります。

### 日付・数値の書式は、ホストで違うことがあります

日付や数値の値は、標準の `Export-Csv` と同じく、現在のカルチャーの書式で文字列にします。この書式はホストの .NET（5.1 は .NET Framework、7.x は .NET）で決まるため、両ホストで違うことがあります。

- ko-KR・zh-TW・zh-HK・ar-SA などのカルチャーでは、日付・数値の書式が両ホストで違うことがあります
- `0.1 + 0.2` は、5.1 では `0.3`、7.x では `0.30000000000000004` になります
- `-UseCulture` の区切り文字も、ms-ID・fa-AF・es-BZ など一部のカルチャーでは両ホストで違います（5.1 は `;`、7.x は `,`）

両ホストで同じ CSV が必要な場合は、書式を指定して文字列にした値（`ToString('G15', [cultureinfo]::InvariantCulture)` など）を渡し、区切り文字は `-Delimiter` で明示してください。

### パラメータ一覧

| パラメータ | 説明 |
| :---- | :---- |
| `-InputObject` | 書き出すオブジェクト。パイプラインから受け取る |
| `-Path` / `-LiteralPath` | 出力先（どちらか一方を指定する） |
| `-Delimiter` / `-UseCulture` | 区切り文字（標準と同じ） |
| `-Encoding` | 文字エンコーディング。省略すると `utf8NoBOM`（追記では追記先から継承）。**位置パラメータではありません**（標準の位置 1 は `-Delimiter` のため） |
| `-EncodingFrom` | 文字エンコーディングを参照ファイルから継承する |
| `-LineBreak` | 各行の後ろに付ける改行（`Auto` / `CrLf` / `Lf` / `Cr`） |
| `-Append` | 追記する |
| `-AllowEncodingChange` | 追記の整合性検査を行わない |
| `-Force` | 読み取り専用のファイルにも書く。`-Append` で列が足りないオブジェクトも、その列を空にして書く（標準と同じ二つの意味） |
| `-NoClobber` | 出力先があればエラーにする |
| `-NoHeader` | ヘッダー行を書かない |
| `-IncludeTypeInformation` | `#TYPE` の行を書く |
| `-NoTypeInformation` | 何もしない（受け付けるだけ） |
| `-UseQuotes` | 引用符の付け方（`Never` / `Always` / `AsNeeded`）。既定は `Always` |
| `-QuoteFields` | 引用符で囲む列名 |
| `-DetectionLineCount` | 追記先・出力先・参照ファイルの判定に使う先頭の行数（後述） |
| `-Culture` / `-Strategy` | 判定に使うカルチャーと判定方式 |
| `-WhatIf` / `-Confirm` | 実行せずに確認する |

エラーはすべて終了エラーです（`Out-ProbedFile` と同じです）。

## -DetectionLineCount — 先頭の N 行で判定する

### なぜ必要なのか

このモジュールの判定は、ファイル全体を対象にしています。正確に判定するためですが、巨大なファイルでは時間とメモリを使います。

`-DetectionLineCount` を指定すると、**先頭の N 行だけで判定します。** 対象は `Resolve-Encoding`・`Import-ProbedCsv`・`Export-ProbedCsv` の 3 つです。

```
# 巨大な CSV を、先頭 1000 行で判定して読む
Import-ProbedCsv .\huge.csv -DetectionLineCount 1000
```

N は**物理的な行の数**です。改行（CR-LF / LF / CR）で数え、CSV のレコードの数ではありません。引用符の中の改行も 1 行と数えます。0 以下の値はエラーになります。

### Resolve-Encoding は、先頭 N 行の判定結果をそのまま返します

先頭が英数字だけで、後半に日本語がある Shift_JIS の CSV を作って試します。

```
# 先頭 2001 行は英数字、最後の 2 行が日本語の Shift_JIS の CSV
$lines = @('Id,Name') + (1..2000 | ForEach-Object { "$_,user$_" }) + @('2001,佐藤','2002,鈴木')
Set-ProbedContent .\big.csv -Encoding shift_jis -Value $lines

Resolve-Encoding .\big.csv
Resolve-Encoding .\big.csv -DetectionLineCount 100
```

| 指定 | CodePage | EncodingWebName |
| :---- | :---- | :---- |
| なし（ファイル全体で判定） | `932` | `shift_jis` |
| `-DetectionLineCount 100` | `20127` | `us-ascii` |

先頭 100 行は英数字だけなので、`us-ascii` と報告します。`Resolve-Encoding -DetectionLineCount` は「先頭 N 行が何に見えるか」を報告するコマンドで、後半にだけ現れる文字は結果に反映されません。改行の種類（`LineBreak`）も、先頭 N 行だけで判定した結果です。

### Import-ProbedCsv と Export-ProbedCsv は、使う前にファイル全体を検証します

`Import-ProbedCsv` と `Export-ProbedCsv` では、先頭 N 行が英数字だけなら、BOM 無しの UTF-8 とみなします。そして、**使う前にファイル全体を厳密に検証します。**

同じファイルを `-DetectionLineCount 100` で読んでみます。

```
Import-ProbedCsv .\big.csv -DetectionLineCount 100
```

```
'...\big.csv' には、先頭 100 行から判定した文字エンコーディング 'utf8NoBOM' として
不正なバイト列がある（バイト位置 27800: 8D）ため、処理しませんでした。
-DetectionLineCount を増やすか、-Encoding で文字エンコーディングを指定してください。
```

エラー ID は `DetectionVerificationFailed` です。**レコードは 1 件も出力しません。** 判定を誤ったまま読んで、後半を文字化けさせることはありません。

`Export-ProbedCsv -Append -DetectionLineCount 100` で追記しようとした場合も、同じエラーで止まり、ファイルには何も書きません。先頭だけで UTF-8 と判定して、Shift_JIS のファイルに UTF-8 の行を足してしまうことはありません。

この場合は、メッセージのとおり N を増やすか、`-Encoding` で明示します。

```
# 日本語の行まで含めて判定する
Import-ProbedCsv .\big.csv -DetectionLineCount 3000

# 文字エンコーディングを明示する（判定も検証も行わない）
Import-ProbedCsv .\big.csv -Encoding shift_jis
```

### 時間とメモリ

`-DetectionLineCount` を指定すると、判定に使うメモリは N 行分で済みます。ただし、検証のために**ファイル全体を 2 回読みます**（検証と、本来の読み込み）。検証のメモリはファイルの大きさによらず一定です。

速さが必要な場合は、`-Encoding` で文字エンコーディングを明示してください。判定も検証も行いません。

改行の無い巨大なファイル（1 行に詰めた JSON など）は、`-DetectionLineCount` を指定してもファイル全体を判定に使うので、大きすぎれば判定できません。

### 既知の限界

次のようなファイルは、`-DetectionLineCount` を指定すると、改行の形（1 バイト・UTF-16・UTF-32）を誤るか、決められないことがあります。`Import-ProbedCsv` / `Export-ProbedCsv` は多くの場合、ファイル全体の検証で止まりますが、確実ではありません。

- BOM 無しの UTF-16 で、特殊な文字（U+0D00・U+0A00）が多いもの、先頭の 64 KiB に英数字も改行も無いもの、ひらがなだけでできているもの
- 各行に `00` が 1 バイトずつある、Shift_JIS などの日本語のファイル
- U+0000 の文字を含む BOM 無しの UTF-16 / UTF-32

心当たりがあれば、`-DetectionLineCount` を外すか、`-Encoding` で文字エンコーディングを指定してください。詳しい条件は、GitHub の README の「[`-DetectionLineCount` の改善と既知の限界](https://github.com/motoi-tsushima/SnowStack.EncodingProbe/blob/v1.3.0/README.md#-detectionlinecount-の改善と既知の限界powershell-モジュール)」にあります。

## 1.2.0 からの挙動の変更

ここからは、既存のコマンドとクラスライブラリで、**1.2.0 までと結果が変わる点**です。

### 表現できない文字は、エラーにしてファイルを元のままにします

1.2.0 までの書き込みのコマンドは、書き込み先の文字エンコーディングで表現できない文字を、**エラーも警告も出さずに `?`（`3F`）にして書いていました。** Windows PowerShell 5.1 では、似た文字に置き換える（ベストフィット。`é` → `e`）場合もありました。

1.3.0 では、`Set-ProbedContent`・`Add-ProbedContent`・`Out-ProbedFile`・`Export-ProbedCsv` の 4 つのコマンドが、**表現できない文字をエラーにして、ファイルを元のままにします。**

```
# Shift_JIS のファイルに、ハングルを追記しようとする
Set-ProbedContent .\sj.txt -Encoding shift_jis -Value '日本語'
Add-ProbedContent .\sj.txt -Value '안녕'
```

```
文字 U+C548 '안' は文字エンコーディング 'shift_jis' で表現できないため、
'...\sj.txt' に書き込みませんでした。ファイルは変更していません。
```

エラー ID は `UnrepresentableCharacter` で、1.2.0 の `Convert-ProbedContent` と同じです。メッセージには、最初に表現できなかった文字の符号位置と、文字エンコーディングの名前が示されます。sj.txt は `93 FA 96 7B 8C EA 0D 0A`（「日本語」と改行）のまま残ります。

- 途中まで書いていた場合も、書き込みの前の状態に戻します
- 文字エンコーディングを継承した場合も、`-Encoding` で明示した場合も、`-EncodingFrom` の場合も同じです。`-Encoding` に `System.Text.Encoding` のインスタンスを渡した場合も、そのインスタンスの置き換えの設定は使いません
- `Set-ProbedContent` / `Add-ProbedContent` は非終了エラーで、ほかのファイルへの書き込みは続けます。`Out-ProbedFile` / `Export-ProbedCsv` は終了エラーです
- **表現できない文字を置き換えて書く方法（スイッチ）はありません。** 置き換えてよい場合は、文字列を先に編集してください。または、UTF-8 などの Unicode で書いてください

`?` で書かれたファイルからは、元の文字を復元できません。1.2.0 までのスクリプトで、文字が `?` になるのを承知で書いていた場合は、1.3.0 ではエラーで止まるようになります。

### 継承した us-ascii は、BOM 無しの UTF-8 とみなします

英数字だけで始まるログや CSV は、判定すると `us-ascii` になります。1.2.0 までは、ここに日本語を追記すると、`us-ascii` を引き継いで日本語を `?` にしていました。

1.3.0 では、**書き込み先・追記先・`-EncodingFrom` の参照先から引き継いだ文字エンコーディングが `us-ascii` の場合は、BOM 無しの UTF-8 とみなして書きます。**

```
# 英数字だけのログ
Set-ProbedContent .\log.txt -Encoding ascii -LineBreak Lf -Value 'start'
(Resolve-Encoding .\log.txt).EncodingWebName   # → us-ascii

# 日本語を追記する
Add-ProbedContent .\log.txt -Value '日本語'
Show-Bytes .\log.txt
```

```
73 74 61 72 74 0A E6 97 A5 E6 9C AC E8 AA 9E 0A
```

「日本語」が UTF-8（`E6 97 A5 ...`）で書かれ、改行も LF を引き継いでいます。英数字だけのファイルは UTF-8 としても正しいので、ファイルは壊れません。英数字だけを書く場合のバイト列は、1.2.0 と変わりません。

あわせて、`us-ascii` のファイルに `-Encoding utf8NoBOM`（または `utf8BOM`）を明示して日本語を追記すると、1.2.0 までは整合性検査（`EncodingChangeOnAppend`）で拒否していましたが、1.3.0 では許可します。

`-Encoding ascii` を明示した場合は、置き換えません。日本語を書くと `UnrepresentableCharacter` になります。

### 追記の整合性検査で止まったら、ファイルを元に戻します

1.2.0 までは、追記の整合性検査（`EncodingChangeOnAppend`）で途中の行が拒否されると、その手前の行までが書かれて止まっていました。

1.3.0 では、`Add-ProbedContent`・`Out-ProbedFile -Append`・`Export-ProbedCsv -Append` のどれでも、**その呼び出しで書いた分を取り消して、追記の前の状態に戻します。**

```
# UTF-8 のファイルに、Shift_JIS で 2 行追記しようとする（1 行目は ASCII なので検査を通る）
Set-ProbedContent .\u8.txt -Encoding utf8NoBOM -LineBreak Lf -Value '日本語'
Add-ProbedContent .\u8.txt -Value 'ABC','あ' -Encoding shift_jis
Show-Bytes .\u8.txt   # → E6 97 A5 E6 9C AC E8 AA 9E 0A（ABC も書かれていない）
```

「エラーで止まったら、そのファイルには、その呼び出しで書いた分を残さない」に、書き込みのコマンドの規則をそろえました。

### 読み込み中のファイルへの上書きで、中身を失いません

1.1.0 から、同じファイルを一つのパイプラインで読み書きすると `SamePathRoundTrip` で止まります。しかし 1.2.0 までは、同じファイルかどうかを**パスの文字列**で比べていました。そのため、ハードリンク・シンボリックリンク・ジャンクション・`subst`・UNC の管理共有などで書き方を変えると検出をすり抜け、読み込み中のファイルが切り詰められていました。

1.3.0 では、**ファイルの実体**で比べます。

```
# r.txt のハードリンク h.txt を作る
New-Item -ItemType HardLink -Path .\h.txt -Target .\r.txt

# r.txt を読んで、h.txt に書く（中身は同じファイル）
Get-ProbedContent .\r.txt | Out-ProbedFile .\h.txt
```

```
'...\h.txt' は読み取り中のため、書き込み先に指定できません。
同一のファイルを1つのパイプラインで読み書きすると、読み終える前にファイルが切り詰められます。
$text = Get-ProbedContent <path> -Raw のように、いったん変数に受けてください。
```

r.txt は元のまま残ります。`Set-ProbedContent` に書き換えても同じです。

同じことを標準のコマンドで行うと、中身を失います。

```
Get-Content .\s.txt | Out-File .\sh.txt   # sh.txt は s.txt のハードリンク
(Get-Item .\s.txt).Length                 # → 0
```

逆向きの場合も検出します。書き込み側が先に開いたファイルを、同じパイプラインの読み込み側が後から読もうとした場合です。

```
# a.txt と z.txt を読んで、z.txt に書く
Get-ProbedContent .\a.txt, .\z.txt | Out-ProbedFile .\z.txt
```

```
'...\z.txt' は同じパイプラインで書き込み中のため、読み込みませんでした。
書き込みは取り消し、ファイルは元のままです。
書き込み先を別のファイルにするか、いったん変数に受けてから書いてください。
```

z.txt は元の内容のまま残ります。1.2.0 では、この書き方で z.txt の元の内容が消えていました。

**Linux / macOS では、ハードリンクと、大文字・小文字だけが違う名前（`t.txt` と `T.TXT`）を、同じファイルと判定しません。** 後述の既知の制限（D2・D5）をご覧ください。

### 上書きの書き方が変わりました

1.3.0 では、上書き（`Set-ProbedContent`・`Out-ProbedFile`・`Export-ProbedCsv`・`Convert-ProbedContent`）の書き方を、ファイルの大きさで使い分けます。

| 場合 | 書き方 |
| :---- | :---- |
| 書く内容と元のファイルが、どちらも 1 MiB 以内 | 元のファイルの中身をメモリに写してから、元のファイルに直接書く。失敗したら写しから書き戻す |
| 1 MiB を超える | 同じフォルダーの一時ファイルに書いてから、元のファイルと置き換える |

どちらの場合も、失敗したときは元の内容に戻ります。そのほか、次の点が変わりました。

- **シンボリックリンクに上書きすると、リンクの先の実体を書き換え、リンクはそのまま残します。** 1.2.0 では、リンクがただのファイルに置き換わることがありました
- **Windows では、ハードリンクを切りません。** 1 MiB を超えるハードリンクと、フォルダーにファイルを作る権限が無い場合は、元のファイルを一時フォルダーに写してから直接書きます（大きなファイルでは、写す時間がかかります）
- 書き込みに成功したファイルには、Archive 属性が付きます。隠し・システム属性のファイルにも `-Force` なしで書け、代替データストリーム（`Zone.Identifier` など）を保ちます。標準の `Out-File` / `Export-Csv` は、隠し・システム属性のファイルに書けず、代替データストリームを消すので、この点は標準と違います
- 置き換えに使う一時ファイルの名前は `.~probed<8 桁の 16 進>.probedtmp` です。名前の長いファイルにも書けるようになりました
- 1 MiB 以内のファイルを書いている最中に、プロセスが強制終了される（タスク マネージャーで終了する・電源が切れるなど）と、ファイルが書きかけで残ることがあります

**Linux / macOS では、1 MiB を超える上書きの結果が 1.2.0 から変わりました。** 1.2.0 の `Set-ProbedContent`・`Out-ProbedFile` は、大きさによらず元のファイルに直接書いていました。1.3.0 は 1 MiB を超えると一時ファイルで置き換えるので、**ハードリンクが切れ、ACL と拡張属性が消えます。** アクセス権（モード）と macOS の隠しファイルのフラグ（hidden）は保ちます。1 MiB 以内の上書きと追記は、元のファイルに直接書くので、どれも保ちます。

その代わり、Linux / macOS でも、ハードリンクの別名や大文字・小文字だけが違う名前で、読み込み中のファイルに上書きしても、中身を失わなくなりました（1.2.0 は読み込み中のファイルを切り詰めていました）。

`Convert-ProbedContent` は、Linux / macOS でも 1.2.0 より良くなりました。1.2.0 は大きさによらずファイルを置き換えていたので、小さなファイルでもハードリンクが切れ、ACL・拡張属性・モード・hidden フラグを失っていました。1.3.0 は、1 MiB 以内ならすべて保ちます。

### Get-ChildItem から渡したファイルの名前に [ ] があっても、そのファイルを読み書きします

1.2.0 までは、`Get-ChildItem` などから渡したファイルの `FullName` が `-Path` に入り、ワイルドカードとして展開されていました。名前に `[ ]` があるファイル `a[1].txt` を渡すと、`a1.txt` を読み書きし、一致するファイルが無ければ黙って何もしませんでした。

1.3.0 では、別名 `FullName` を `-Path` から `-LiteralPath` に移しました。対象は `Get-ProbedContent`・`Import-ProbedCsv`・`Set-ProbedContent`・`Add-ProbedContent`・`Convert-ProbedContent` です。

```
# a[1].txt と a1.txt がある
Get-ChildItem -LiteralPath '.\a[1].txt' | Set-ProbedContent -Value 'updated'
Get-ProbedContent -LiteralPath '.\a[1].txt'   # → updated（a1.txt は変わらない）
```

文字列をパイプラインで渡した場合は、これまでどおり `-Path`（ワイルドカード）です。

また、`Set-ProbedContent` / `Add-ProbedContent -Path` のワイルドカードがどのファイルにも一致しないときは、非終了エラー `ItemNotFound` にします（標準の `Set-Content` / `Add-Content` と同じです）。1.2.0 までは、何も書かずにエラーも出しませんでした。

```
Set-ProbedContent .\nomatch*.txt -Value 'x'
```

```
'.\nomatch*.txt' に一致するファイルが無いため、書き込みませんでした。
名前にワイルドカード文字を含むファイルを作成するには -LiteralPath を使用してください。
```

読み込み（`Get-ProbedContent`・`Import-ProbedCsv`）は、これまでどおりエラーにしません。

### ansi / oem と Get-EncodingProbePlatformInfo の Locale

**`-Encoding ansi` / `oem` は、Windows では「システムのロケール」（「Unicode 対応ではないプログラムの言語」）の ANSI / OEM コードページになりました。** 5.1 の `Default` / `Oem`、7.x の標準の `oem` と同じ値です。対象はすべての `-Encoding` と `ConvertTo-DotNetEncoding` です。

1.2.0 までは、「地域」の設定の「形式」（スレッドのカルチャー）の値でした。そのため、日本語の Windows で形式を「英語（米国）」にしていると、`ansi` が 1252 になっていました。

`Get-EncodingProbePlatformInfo` の `Locale` の `AnsiCodePage` / `OemCodePage` も、Windows では同じくシステムのロケールの値です。

```
(Get-EncodingProbePlatformInfo).Locale
```

```
CurrentCulture : ja-JP
AnsiCodePage   : 932
OemCodePage    : 932
IsAnsiOemSame  : True
```

`CurrentCulture` は、今までどおりスレッドのカルチャーの名前です。

注意点が三つあります。

- **標準の PowerShell 7.x の `-Encoding ansi` は、プロセスの開始時の「形式」の ANSI コードページです。** 「形式」をシステムのロケールと違う値にしている環境では、本モジュールの `ansi` と違います（日本語の Windows で形式が英語（米国）なら、標準は 1252、本モジュールは 932）
- システムの ANSI / OEM が 65001（「ベータ: ワールドワイド言語サポートで Unicode UTF-8 を使用」）の場合は、BOM 無しの UTF-8 として扱います
- `-Encoding 0` は、エラーにします（1.2.0 までは、実行環境の既定の文字エンコーディングになっていました）

Linux / macOS には ANSI / OEM コードページの考え方が無いので、今までどおり、現在のカルチャー（環境変数 `LC_ALL`・`LANG` などから決まる）の値です。実測した値の表は、GitHub の README の「[Linux / macOS での ansi / oem](https://github.com/motoi-tsushima/SnowStack.EncodingProbe/blob/v1.3.0/README.md#linux--macos-での-ansi--oem)」にあります。

### -Culture に受け付けるカルチャー名

`-Culture`（クラスライブラリでは `EncodingDetectorOptions.Culture`）に受け付けるカルチャー名を、.NET で作れるかどうかではなく、**名前の形（BCP 47）**で決めるようにしました。1.2.0 までは、Windows PowerShell 5.1 と PowerShell 7.x で受け付ける名前が違っていました。

| 名前 | 1.2.0 | 1.3.0 |
| :---- | :---- | :---- |
| `C`・`root` | 5.1 ではエラー、7.x では判定に進む | 両ホストでエラー（`InvalidCulture`） |
| `POSIX`・`c-JP` など | ホストによる | 両ホストでエラー |
| `art-lojban`・`zh-yue`・`und-u-ca-gregory` など | どちらかのホストだけが受け付ける | 両ホストで受け付ける |
| `' ko-KR'`（前後に空白） | エラー | 空白を除いて `ko-KR` として受け付ける |

```
Resolve-Encoding .\k.txt -Culture C
```

```
'C' はカルチャー名として解釈できません。'ja-JP' や 'ko-KR' のようなカルチャー名を指定するか、
-Culture を省略して実行環境のカルチャーを使用してください。
```

言語の直後の拡張言語は、それを言語として扱います。`zh-yue` は広東語（香港の扱い）として判定します。

`Resolve-Encoding` の、解釈できない `-Strategy` / `-Culture` のエラーも、ほかのコマンドと同じ `InvalidStrategy` / `InvalidCulture`（5 言語のメッセージ）になりました。1.2.0 までは、.NET の英語のメッセージでした。

### 判定の修正（クラスライブラリ）

クラスライブラリの判定処理の不具合を 3 件直しました。`Resolve-Encoding`・`Get-ProbedContent`・`Import-ProbedCsv` など、判定を使うすべてのコマンドに効きます。

**1. BOM 無しの UTF-16 / UTF-32 で、英数字だけのファイルを判定できるようになりました。**

1.2.0 までは `us-ascii` と判定され、読み込むと各文字の間に `U+0000` が混じっていました（1.0.0 から存在した不具合です）。

```
Set-ProbedContent .\u16.txt -Encoding unicodeNoBOM -Value 'Hello','World'
Resolve-Encoding .\u16.txt
```

```
CodePage EncodingWebName   Bom LineBreak
-------- ---------------   --- ---------
    1200 utf-16          False      CrLf
```

ただし、英数字の中に、2 バイトとも `0x80` 未満になる文字（「上」U+4E0A、「字」、「あ」、「、」など）が混じる BOM 無しの UTF-16 / UTF-32 は、従来どおり `us-ascii` と判定されます。`Import-ProbedCsv` / `Export-ProbedCsv` では、`-DetectionLineCount` を指定すると正しく読み書きできます。ほとんどの日本語の文字のように `0x80` 以上のバイトになる文字を 1 文字でも含むファイルは、この限界に当たりません。

**2. UTF-16 / UTF-32 のファイルの改行コード（`LineBreak`）を、1 文字分の単位で数えるようにしました。**

「上」は UTF-16LE で `0A 4E` になり、1 バイト目が LF（`0A`）と同じです。1.2.0 までは 1 バイトずつ数えていたため、「上」「《」「不」のような文字があると、CR-LF のファイルを改行が混在している（`LfAndCrAndCrLf` など）と報告していました。

```
Set-ProbedContent .\u16c.txt -Encoding unicodeBOM -LineBreak CrLf -Value '上の段','不明《注》'
(Resolve-Encoding .\u16c.txt).LineBreak   # → CrLf
```

この値は、改行を引き継ぐ書き込み（`-EncodingFrom`、`-Encoding` 省略の追記など）にも使われます。1.2.0 までは、改行が CR のファイルに LF を追記することがありました。

**3. 短い EUC-JP の日本語を、日本語の環境で Shift_JIS と判定しなくなりました。**

EUC-JP と Shift_JIS のように、バイト列だけでは区別できない組み合わせは、カルチャーと改行で決めています。1.3.0 では、決めた候補でそのバイト列を正しく読めず、もう一方では読める場合は、もう一方を選びます。約 40 バイト以下の短い EUC-JP が、改行が CR-LF・改行なしのときに Shift_JIS と判定され、読み込みが `・` や私用領域の文字に黙って化けることがありました。EUC-KR と CP949、EUC-TW と CP950 の組み合わせも同じです。

### その他の変更

| 変更 | 内容 |
| :---- | :---- |
| BOM が 2 個続くファイル | 取り除く BOM は先頭の 1 個だけにしました。2 個目の U+FEFF は文字として残ります（7.x の標準の `Get-Content`・`Import-Csv` と同じ）。1.2.0 までは、`-Encoding utf8BOM` などを指定すると 2 個とも取り除いていました |
| 途中で切れたファイルの末尾 | Windows PowerShell 5.1 でも、7.x と同じく置き換えの文字（U+FFFD など）として読みます。長さが奇数の UTF-16 の最後の 1 バイトなどを、5.1 では黙って捨てていました |
| JIS（ISO-2022-JP）のまま終わっているファイルへの追記 | 先に ASCII に戻すエスケープシーケンス（`ESC ( B`）を書いてから追記します。追記した行が 2 バイト文字として読まれて消えていました |
| `-Value` の配列の `$null` | 標準の `Set-Content` / `Add-Content` と同じく飛ばします（空行を書いていました） |
| 同じファイルを 2 回指定した `Set-ProbedContent` | 1 回書いたのと同じ結果にします（2 回目が追記になっていました） |
| ほかのプログラムが書き込み中のファイル | 書き込み中のログなども、判定・変換のために読めます。読めない・大きすぎる・メモリが足りない場合は ID の付いたエラーで、`Resolve-Encoding` は次のファイルに進みます。追記は、今までどおり、ほかのプログラムが書き込み中の間はできません |
| `a.txt::$DATA` | `a.txt` として扱います。代替データストリーム（`a.txt:alt`）はエラー `AlternateDataStreamNotSupported` です |
| `Get-ProbedContent -TotalCount` | 標準と同じ別名 `-Head`・`-First` を付けました |
| `Out-ProbedFile` | `$PSDefaultParameterValues['Out-String:Width']` などの影響を受けません（標準の `Out-File` と同じ） |
| `Convert-ProbedContent` | `UTF7Encoding` のインスタンスを渡しても、UTF-7 では書きません（`utf7` と同じエラー） |
| `Import-Clixml` などを通した判定結果 | `-Encoding` に渡すと、型が違うことと渡し方（`CodePage` の値など）を示すエラーになります（今までどおり使えません） |
| メッセージ | 列名の重複のエラーにファイルのパスを示し、`EncodingChangeOnAppend` は文字エンコーディングを統一語彙名（`utf8NoBOM` など）で示します。書き込みの失敗（ディスクの空き不足など）も 5 言語のメッセージにしました |
| `Resolve-Encoding -License` | 表示の誤り（`Copyright c`）を直し、CSV の処理に含めた PowerShell のコードのライセンス表示を加えました |

## Linux / macOS の既知の制限

リリース前に、Linux（WSL の Ubuntu 24.04）と macOS 15.8 の PowerShell 7.6.9 で確かめて見つかった制限です。1.3.0 では直さず、1.4.0 で直す予定です。Windows には当てはまりません。

どれも元のファイルの中身は失いません。ただし D6 は、`-Force` を付けたときに、1 件目の変換結果を上書きして失うか、変換元をその場で変換して元のバイト列を失います。D7 は、ACL と拡張属性を失います。ただし D2・D5 では、終わらずにファイルが増え続けて、ディスクを使い切ることがあります。

| | 内容 |
| :---- | :---- |
| D1 | 1 MiB を超えるハードリンクに上書きすると、リンクが切れます（`Set-ProbedContent`・`Out-ProbedFile`・`Export-ProbedCsv`・`Convert-ProbedContent`）。書いた名前だけが新しい内容になり、ほかの名前は古い内容のままです。書く内容と元のファイルがどちらも 1 MiB 以内なら、リンクは保たれます |
| D2 | 読み込みと追記を別のハードリンク名で行うと、終わらずにファイルが増え続けます。標準の `Add-Content` も同じです |
| D3 | リンク切れのシンボリックリンクへの追記（`Add-ProbedContent`・`Out-ProbedFile -Append`・`Export-ProbedCsv -Append`）は、エラーになります（Windows では、標準の `Add-Content` と同じくリンクの先にファイルを作ります）。また、`-Encoding` を付けない `Set-ProbedContent`・`Get-ProbedContent`・`Convert-ProbedContent`・`Resolve-Encoding` も、リンク切れのリンクを「無いファイル」と扱わず、`FileNotFound` ではなく .NET の文言や `ReadFailed`（error code 0x80070002）のエラーになります |
| D4 | 共有を許さずに開かれたファイルを読み書きしたときのエラーのメッセージが、「error code 0x0000000B」（macOS では「error code 0x00000023」）になります。エラーの ID は正しく、ファイルは変わりません |
| D5 | 大文字・小文字を区別しないファイルシステム（macOS の標準の APFS など）で、大文字・小文字だけが違う名前を同じファイルと判定しません。読み込みと追記をそういう名前で行うと、D2 と同じく終わらずにファイルが増え続けることがあります。上書きは正しい結果になります（標準のコマンドは、上書きなら中身を失い、追記なら増え続けます） |
| D6 | `Convert-ProbedContent -Destination` の名前の衝突と変換元との一致の検査が、大文字・小文字の違いですり抜けます。`-Force` を付けたときだけ、1 件目の変換結果を上書きするか（`one/a.txt` と `two/A.TXT`）、変換元をその場で変換します（変換元が `sub` で `-Destination SUB`） |
| D7 | 1 MiB を超える上書き（`Set-ProbedContent`・`Out-ProbedFile`・`Export-ProbedCsv`・`Convert-ProbedContent`）で、ACL と拡張属性が消えます（一時ファイルで置き換えるため）。モードと macOS の hidden フラグは保ちます。1 MiB 以内の上書きと追記は保ちます |

このほか、次の点にも注意してください。

- **`LANG` が未設定の Linux では、日本語のファイルの判定が `-1` や `51932` などになります。** カルチャーによる絞り込みが効かないためです。`-Culture ja-JP` を指定するか、`LANG=ja_JP.UTF-8` などを設定してください。macOS では、`LANG` が未設定でもシステムの地域設定のカルチャーになるので、地域が東アジア以外の場合に同じことが起きます
- Linux / macOS では、`Export-ProbedCsv -Append` は追記先の改行（CR-LF なら CR-LF）を引き継ぎます。標準の `Export-Csv -Append` は LF を書きます

**読み込むファイルと同じファイルに、ハードリンクや大文字・小文字を変えた名前で書かないでください。**

それぞれの詳しい条件は、GitHub の README の「[Linux / macOS の既知の制限](https://github.com/motoi-tsushima/SnowStack.EncodingProbe/blob/v1.3.0/README.md#linux--macos-の既知の制限)」にあります。

## NuGet パッケージを直接使っている方へ

NuGet パッケージ（`SnowStack.EncodingProbe`）も 1.3.0 にしました。**公開 API（型・メソッド・プロパティ、引数と戻り値の型）は変えていません。** 次の点で、1.2.0 までと結果が変わります。

| 変更 | 内容 |
| :---- | :---- |
| BOM 無しの UTF-16 / UTF-32 | 英数字だけのテキストを、`us-ascii` ではなく UTF-16 / UTF-32 と判定します |
| `LineBreak` | UTF-16 / UTF-32 と判定したときは、その単位（2 / 4 バイト）で数えます。公開の `EncodingDetector.DetectLineBreak()` は、文字エンコーディングを知らずに呼ばれるメソッドなので、従来どおりバイト単位で数えます |
| 短い EUC-JP など | バイト列だけでは区別できない組み合わせで、厳密に読めない候補を選びません |
| `EncodingDetectorOptions.Culture` | 受け付けるかどうかを名前の形（BCP 47）で決めます。**`ArgumentException` になる名前の集合が変わります**（net48 と net10.0 でそろいます）。`C`・`root` は受け付けません |
| `PlatformInformation.Locale` | `AnsiCodePage` / `OemCodePage` を、Windows ではシステムのロケールの値（`GetACP` / `GetOEMCP`）にしました。Windows 以外は今までどおり現在のカルチャーの値です |
| `PlatformInformation.Os.Description` | 前後の空白を除きました（.NET Framework の値の末尾に空白が付いていました） |
| `EncodingProbe.License` | MIT License の本文の抜け（`THE USE OR`）を直し、著作権の表記を `Copyright (c) 2026 Motoi Tsushima` にそろえました |

CSV の処理に含めた PowerShell のコードは、PowerShell モジュールにだけ含まれ、NuGet パッケージには含まれません。

クラスライブラリの解説は、以下の記事にあります。

[SnowStack.EncodingProbe NuGet Package 解説](/encodingprobe_guide/)

## 更新するときの注意

1.2.0 から 1.3.0 に更新する前に、次の点を確かめてください。

1. **文字が `?` になるのを承知で書いていたスクリプトは、エラーで止まります。** 表現できない文字を先に置き換えるか、UTF-8 で書いてください
2. **英数字だけのファイルに日本語を追記すると、UTF-8 で書かれます。** そのファイルを Shift_JIS などとして読む別のプログラムがある場合は、`-Encoding` で明示してください
3. **`-Encoding ansi` / `oem` の値が変わることがあります**（Windows で「形式」をシステムのロケールと違う値にしている場合）。`Get-EncodingProbePlatformInfo` の `Locale` で値を確かめられます
4. **`Get-ChildItem` から渡したファイルは、ワイルドカードとして展開されなくなります**（名前に `[ ]` があるファイル）
5. **Linux / macOS で 1 MiB を超えるファイルに上書きすると、ハードリンクが切れ、ACL と拡張属性が消えます**
6. `-Culture C` / `root` などは、エラーになります
7. `$PSDefaultParameterValues['*:Encoding'] = 'utf8'` は、書き込みのコマンドでは PowerShell の規則で無視され（警告だけ）、`-Encoding` を省略したものとして書きます。既定値には `utf8NoBOM` か `utf8BOM` を使ってください

## 1.3.0 でも解決していないこと

| 内容 | 状況 |
| :---- | :---- |
| Linux / macOS の既知の制限（D1〜D7） | 1.4.0 で直す予定です（上の節） |
| `Get-ProbedContent -Raw` で、ファイル全体を一つの文字列にする段階のメモリ不足は、ID の無いエラー（`OutOfMemoryException`）の終了エラーになります。`-Path` に複数のファイルを渡した場合、残りのファイルは処理されません | 1.4.0 で扱う予定です。Windows PowerShell 5.1 の 32 ビット版で数百 MB のファイルを読む場合は、64 ビット版を使うか、`-Raw` を使わずに行で読んでください。復号した後の文字数が約 10.7 億を超える場合（英数字だけなら約 1 GiB）は、64 ビット版でも避けられません |
| `-DetectionLineCount` の既知の限界 | 上の「既知の限界」の節 |
| 2 バイトとも `0x80` 未満になる文字が混じる、英数字の BOM 無しの UTF-16 / UTF-32 は、`us-ascii` と判定されます | `Import-ProbedCsv` / `Export-ProbedCsv` では `-DetectionLineCount` で読めます |
| 既定の判定方式（`Combined`）は、HZ-GB-2312 と UTF-7 を判定しません（`us-ascii` になります） | HZ は `-Strategy UtfUnknownOnly` で判定できます。確実に読むには `-Encoding` で明示してください |
| `Export-ProbedCsv -Append` で、追記先の最後の値の引用符が閉じていないと、追記したレコードがその値の中に入ります | 上の `Export-ProbedCsv` の節 |
| `Convert-ProbedContent` は、BOM だけのファイルを、指定によらず 0 バイトにします（`-Bom Add` でも） | |
| 日付・数値の書式、`-UseCulture` の区切り文字が、ホストで違うことがあります | 上の `Export-ProbedCsv` の節 |
| 文字の一覧や記号・数字の多いシングルバイト系のテキストは、数百バイトあっても判定できない（`CodePage` が `-1`）ことがあります | 読み書きのコマンドの `-Encoding`（`Convert-ProbedContent` では `-SourceEncoding`）でコードページを明示してください |

## コマンド早見表

```
# 判定して CSV を読む
Import-ProbedCsv .\data.csv

# フォルダーの CSV をまとめて読む（5.1 でも動く）
Get-ChildItem *.csv | Import-ProbedCsv

# 外国語の CSV を読む（-Culture は判定、-UseCulture は区切り文字）
Import-ProbedCsv .\korean.csv -Culture ko-KR

# BOM 無しの UTF-8 で書き出す（5.1 でも同じバイト列）
Get-Process | Select-Object Name, Id | Export-ProbedCsv .\processes.csv

# Shift_JIS の CSV に、Shift_JIS のまま追記する（最後の行に改行が無くても行がつながらない）
$rows | Export-ProbedCsv .\sjis.csv -Append

# 巨大なファイルを先頭 1000 行で判定する（使う前にファイル全体を検証する）
Import-ProbedCsv .\huge.csv -DetectionLineCount 1000
Resolve-Encoding .\huge.csv -DetectionLineCount 1000

# 速さが必要なら、文字エンコーディングを明示する（判定も検証もしない）
Import-ProbedCsv .\huge.csv -Encoding shift_jis
```

## ライセンス

CSV の読み書きの処理には、PowerShell（[PowerShell/PowerShell](https://github.com/PowerShell/PowerShell)、Copyright (c) Microsoft Corporation、MIT License）のコードの一部を改変して含めています。このコードは PowerShell モジュールにだけ含まれ、NuGet パッケージには含まれません。

ライセンスの表示は、モジュールに同梱している `THIRD-PARTY-NOTICES.txt` と、`Resolve-Encoding -License` の出力にあります。ライセンス表記の書き方は、以下の記事の「ライセンス」をご覧ください。

[SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)

## お知らせ欄

### （リリース日）　Version 1.3.0 リリース

Version 1.3.0 をリリースしました。PowerShell モジュールに CSV の読み書きコマンド（`Import-ProbedCsv` / `Export-ProbedCsv`）と `-DetectionLineCount` を追加し、書き込みで文字を失わないようにしました。NuGet パッケージも、判定処理の不具合を直して 1.3.0 にしています。

## 関連資料

- [SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)
- [SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)
- [SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)
- [-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)
- [外字（私用領域）の扱い — SnowStack.EncodingProbe](/encodingprobe_private_use_area/)
- [SnowStack.EncodingProbe NuGet Package 解説](/encodingprobe_guide/)
- [SnowStack.EncodingProbe 1.3.0 の README（GitHub）](https://github.com/motoi-tsushima/SnowStack.EncodingProbe/blob/v1.3.0/README.md)
- [SnowStack.EncodingProbe 1.3.0 の CHANGELOG（GitHub）](https://github.com/motoi-tsushima/SnowStack.EncodingProbe/blob/v1.3.0/CHANGELOG.md)
