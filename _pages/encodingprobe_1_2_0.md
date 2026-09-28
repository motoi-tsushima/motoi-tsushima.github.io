---
title: "SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応"
layout: single
classes: wide
permalink: /encodingprobe_1_2_0/
author_profile: true
---
2026/09/30 document update

SnowStack.EncodingProbe のバージョン 1.2.0 で行った変更の解説です。

1.2.0 の変更は、大きく二つに分かれます。

| 対象 | 変更の内容 |
| :---- | :---- |
| PowerShell コマンドレット（SnowStack.EncodingProbe.PowerShell） | `Out-ProbedFile` と `Convert-ProbedContent` の 2 コマンドを追加しました |
| クラスライブラリ（SnowStack.EncodingProbe） | 東アジア以外の言語と、香港の Big5 を正しく判定できるようにしました |

1.1.0 のときは「NuGet パッケージのコードは変更していません」と書きましたが、**今回はクラスライブラリの判定処理そのものを改修しています。**

コマンドレットは内部でこのクラスライブラリを使っているので、判定の改善は `Resolve-Encoding` や `Get-ProbedContent` にもそのまま効きます。

この記事は、1.1.0 で導入した「統一語彙」（`utf8NoBOM` のような文字エンコーディングの名前の体系）を前提にしています。統一語彙の解説は、以下の記事で行っています。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

インストール方法は、以下の記事をご覧ください。

[SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)

（この記事に掲載した実行結果は、すべて実機で採取したものです。採取日は 2026年9月28日、環境は Windows 11 上の PowerShell 7.6.6 と Windows PowerShell 5.1.26100 です。バイト列の確認には、1.1.0 の記事で紹介した `Show-Bytes` 関数を使っています）

## 1.2.0 で追加した 2 つのコマンド

| コマンド | 役割 | 標準の対応物 |
| :---- | :---- | :---- |
| `Out-ProbedFile` | オブジェクトを整形して、ファイルに書き出す | `Out-File` |
| `Convert-ProbedContent` | 既存のファイルの文字エンコーディング・BOM・改行を変換する | （なし） |

1.1.0 で追加した `Get-ProbedContent` / `Set-ProbedContent` / `Add-ProbedContent` と合わせて、テキストファイルの読み書きに使う標準コマンドの代わりが、一通り揃ったことになります。

## Out-ProbedFile — 表や一覧を、文字エンコーディングを指定して書き出す

### 同じ Out-File が、PowerShell のバージョンで違うバイト列を書く

1.1.0 の記事は `Set-Content` の実験から始めましたが、`Out-File` でも同じことが起きます。

以下の 1 行を、Windows PowerShell 5.1 と PowerShell 7.x の両方で実行してみてください。

```
# PowerShell 標準の Out-File で、文字エンコーディングを指定せずに書き出す
'abc' | Out-File .\std.txt
```

| ホスト | できたバイト列 |
| :---- | :---- |
| Windows PowerShell 5.1 | `FF FE 61 00 62 00 63 00 0D 00 0A 00` ← **BOM 付きの UTF-16LE** |
| PowerShell 7.x | `61 62 63 0D 0A` ← **BOM 無しの UTF-8** |

Windows PowerShell 5.1 の `Out-File` は、既定で BOM 付きの UTF-16LE を書きます。リダイレクト演算子の `>` も、中身は同じ `Out-File` です。

`-Append` で追記すると、さらに困ったことになります。

```
# 既定の文字エンコーディングで書いた後、UTF8 を指定して追記する
'abc' | Out-File .\std2.txt -Append
'def' | Out-File .\std2.txt -Append -Encoding UTF8
```

| ホスト | できたバイト列 |
| :---- | :---- |
| Windows PowerShell 5.1 | `FF FE 61 00 62 00 63 00 0D 00 0A 00 64 65 66 0D 0A` |
| PowerShell 7.x | `61 62 63 0D 0A 64 65 66 0D 0A` |

Windows PowerShell 5.1 では、UTF-16LE のファイルの後ろに UTF-8 のバイト列が付け足されています。

標準の `Out-File` は、追記先のファイルの文字エンコーディングを見ません。**一つのファイルの中に、二つの文字エンコーディングが混ざってしまいました。**

1.2.0 の `Out-ProbedFile` で書くと、両方のホストで同じ結果になります。

```
# Out-ProbedFile で、文字エンコーディングを指定せずに書き出す
'abc' | Out-ProbedFile .\probed.txt
```

| ホスト | できたバイト列 |
| :---- | :---- |
| Windows PowerShell 5.1 | `61 62 63 0D 0A` |
| PowerShell 7.x | `61 62 63 0D 0A` |

### Set-ProbedContent があるのに、なぜ Out-ProbedFile が必要なのか

1.1.0 の `Set-ProbedContent` でもファイルは書けます。しかし `Set-ProbedContent` は、受け取ったオブジェクトを**そのまま文字列にする**コマンドです（標準の `Set-Content` と同じです）。

`Get-ChildItem` や `Get-Process` の結果を、画面に表示されるのと同じ表の形でファイルに残したいときは、`Out-File` を使います。

```
# 画面に出る表の形のまま、BOM 無しの UTF-8 で書き出す
Get-ChildItem | Select-Object Name, Length | Out-ProbedFile .\list.txt utf8NoBOM
```

`Out-ProbedFile` は、この `Out-File` の役割を受け持つコマンドです。

標準の `Out-File` のパラメータをすべて持っていて、そこに `-EncodingFrom` / `-LineBreak` / `-AllowEncodingChange` / `-Culture` / `-Strategy` を加えています。

`-Encoding` は `Out-File` と同じく 2 番目の位置パラメータなので、上の例のように `Out-ProbedFile .\list.txt utf8NoBOM` と書けます。

### 同じになるのは「符号化」、同じにならないのは「整形」

ここは、最初に正確に書いておきます。

`Out-ProbedFile` の処理は、二つの層に分かれています。

| 層 | 内容 | 5.1 と 7.x で同じ結果になるか |
| :---- | :---- | :---- |
| 整形の層 | オブジェクトを、表や一覧の文字列にする | **ならない**（実行中の PowerShell に任せる） |
| 符号化の層 | 文字列を、文字エンコーディング・BOM・改行を付けてバイト列にする | **なる**（このコマンドの存在理由） |

整形は、PowerShell 標準の `Out-String -Stream` を呼び出して行っています。自前の整形処理は持っていません。

そのため、表の形は標準の `Out-File` と同じく、PowerShell のバージョンによって違います。実測で確認した違いは以下のとおりです。

| 違い | Windows PowerShell 5.1 | PowerShell 7.x |
| :---- | :---- | :---- |
| 表の各行の末尾の空白 | 列幅まで空白で埋める | 埋めない |
| 切り詰めの記号 | `...` | `…` |
| 全角文字の幅の数え方 | 文字数 | 表示幅 |
| `-Width` 省略時の幅（非対話の実行時） | 119 | 120 |

表や一覧の前後に入る空行の数も違います。

文字列を渡した場合は整形の影響を受けないので、両方のホストでバイト単位まで同じ結果になります。

表の形まで揃えたい場合は、`Out-String` の代わりに、自分で文字列を組み立ててから渡してください。

### -Encoding の省略と Auto は、意味が違います

`Out-ProbedFile` では、`-Encoding` を**省略した場合**と、**`Auto` を明示した場合**とで意味が違います。

| 指定 | `-Append` なし（上書き・新規作成） | `-Append` あり |
| :---- | :---- | :---- |
| 省略 | `utf8NoBOM` | 追記先から継承する。追記先が無い、または 0 バイトなら `utf8NoBOM` |
| `Auto` | 出力先の既存ファイルから継承する。無ければ終了エラー | 追記先から継承する。無ければ終了エラー |
| 語彙名・WebName・数値 | その値 | その値（整合性検査あり） |

`Set-ProbedContent` は「省略 ＝ `Auto`」でしたが、`Out-ProbedFile` は違います。ここは注意してください。

`Out-File` の用途は、ほとんどが「新しくファイルを作る」ことです。そのたびに、これから捨てる既存ファイルの文字エンコーディングを判定して、判定に失敗したら止まる、というのでは使い物になりません。

そこで、省略時は判定を行わず、`utf8NoBOM` に決めています。

`-Append` の場合だけは、追記先から継承します。追記で文字エンコーディングが混ざる事故を防ぐためです。

`Auto` を明示した場合は、`Set-ProbedContent` と同じく「既存ファイルの性質を保って上書きする」という意味になります。出力先が無ければ、継承元が無いのでエラーです。

```
# 存在しないファイルに Auto で書こうとする
'x' | Out-ProbedFile .\none.txt -Encoding Auto
```

```
ファイル '...\none.txt' が存在しないため、文字エンコーディングを継承できません。
-Encoding で明示的に指定するか、-EncodingFrom で別のファイルから継承してください。
```

### -Append は、追記先の文字エンコーディングを保ちます

Shift_JIS のファイルを作って、`-Encoding` を付けずに追記してみます。

```
# Shift_JIS・LF でファイルを作る
'日本語' | Out-ProbedFile .\sj.txt shift_jis -LineBreak Lf

# -Encoding を付けずに追記する
'追記' | Out-ProbedFile .\sj.txt -Append
Show-Bytes .\sj.txt
```

| | バイト列 |
| :---- | :---- |
| 作成直後 | `93 FA 96 7B 8C EA 0A` |
| 追記後 | `93 FA 96 7B 8C EA 0A 92 C7 8B 4C 0A` |

追記した「追記」も Shift_JIS（`92 C7 8B 4C`）で書かれ、改行も LF（`0A`）のままです。追記先の文字エンコーディングと改行を、判定して引き継いでいます。

`-Encoding` を明示して追記する場合は、`Add-ProbedContent` と同じ**バイト列比較の整合性検査**を行います。

```
# Shift_JIS のファイルに、日本語を UTF-8 で追記しようとする
'追記' | Out-ProbedFile .\sj.txt -Append -Encoding utf8NoBOM
```

```
'...\sj.txt' に文字エンコーディング 'utf-8' で追記できません。
'utf-8' で符号化したバイト列が、このファイルの現在の文字エンコーディング 'shift_jis' で
符号化したバイト列と異なるため、追記するとファイル全体の一貫性が失われます。
文字エンコーディングを変えることが意図どおりであれば -AllowEncodingChange を指定してください。
```

ファイルは `93 FA 96 7B 8C EA 0A 92 C7 8B 4C 0A` のまま、何も書き足されていません。

検査は 1 行ずつ、書き込む前に行います。途中の行で不一致を検出した場合は、その手前の行までが書かれて止まります。書かれた行は比較が成立した行だけなので、ファイルが二つの文字エンコーディングで混ざることはありません。

バイト列比較の考え方は、1.1.0 の記事の「整合性検査はバイト列で判定します」で解説しています。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

### 読み終える前に、ファイルを消しません

標準の `Out-File` には、知っていないと必ず事故になる挙動があります。

```
# 標準の Out-File で、読んだファイルに書き戻そうとする
Get-Content .\r.txt | Out-File .\r.txt
```

実行すると、**r.txt は 0 バイトになります。** 中身はすべて消えます。

標準の `Out-File` は、処理の開始時点（上流のコマンドがファイルを読む前）に、出力先のファイルを切り詰めてしまうからです。

`Out-ProbedFile` は、ファイルを開くのを「最初の 1 行を書く直前」まで遅らせています。そのため、同じファイルを読み書きしていることを検出してから止まれます。

```
# Out-ProbedFile で、読んだファイルに書き戻そうとする
Get-ProbedContent .\r.txt | Out-ProbedFile .\r.txt
```

```
'...\r.txt' は読み取り中のため、書き込み先に指定できません。
同一のファイルを1つのパイプラインで読み書きすると、読み終える前にファイルが切り詰められます。
$text = Get-ProbedContent <path> -Raw のように、いったん変数に受けてください。
```

エラー ID は、1.1.0 の `Set-ProbedContent` と同じ `SamePathRoundTrip` です。r.txt は元のまま残ります。

### -LineBreak が決めるのは、行の後ろに付ける改行だけです

`-LineBreak` で指定するのは、**各行の後ろに付ける改行**です。

渡した文字列の中に、あらかじめ改行が含まれている場合（`"a`r`nb"` など）、その改行は置き換えません。

これは標準の `Out-File` / `Set-Content` と同じ挙動で、1.1.0 の `Set-ProbedContent` / `Add-ProbedContent` も同じです。1.2.0 で、この点をヘルプに明記しました。

ファイルの中の改行をすべて揃えたい場合は、後で解説する `Convert-ProbedContent` を使ってください。

### 標準の Out-File との違い

`Out-ProbedFile` は、標準の `Out-File` とできるだけ同じ挙動にしていますが、意図的に変えた箇所があります。

| 項目 | 標準の `Out-File` | `Out-ProbedFile` |
| :---- | :---- | :---- |
| `-Encoding` の既定 | 5.1 は BOM 付きの UTF-16LE、7.x は BOM 無しの UTF-8 | 上書き・新規作成は `utf8NoBOM`、`-Append` は追記先から継承 |
| `-Append` の整合性 | 追記先を見ない | バイト列比較で検査する |
| ファイルを切り詰める時点 | 処理の開始時点 | 最初の 1 行を書く直前 |
| `$null` を 1 個渡した場合 | BOM だけのファイルになる | 0 バイトのファイルになる |
| 別名 `-Path` / `-LP` | PowerShell 7.x だけ | Windows PowerShell 5.1 でも使える |
| `-LiteralPath` のパイプライン入力 | プロパティ名で受け取る | 受け取らない |

`$null` の行は、BOM の規則を「1 行でも書いたときにだけ BOM を書く」に統一した結果です。

```
# $null を 1 個だけ渡して、BOM 付きの UTF-8 を指定する
$null | Out-File .\n1.txt -Encoding utf8BOM        # → EF BB BF（BOM だけ）
$null | Out-ProbedFile .\n2.txt -Encoding utf8BOM  # → 0 バイト
```

`-LiteralPath` をパイプラインから受け取らないのは、`Get-ChildItem | Out-File` と書いたときに、`Get-ChildItem` の出力の `PSPath` が出力先として意図せず束縛される紛らわしさを避けるためです。`Out-ProbedFile` がパイプラインから受け取るのは、書き出すオブジェクト（`-InputObject`）だけです。

### パラメータ一覧

| パラメータ | 型 | 説明 |
| :---- | :---- | :---- |
| `-FilePath` | `string` | 出力先（位置 0、別名 `-Path`）。ワイルドカードは 1 件に解決されること |
| `-LiteralPath` | `string` | 出力先。ワイルドカード非対応（別名 `-PSPath` / `-LP`） |
| `-InputObject` | `PSObject` | 書き出すオブジェクト。パイプライン入力に対応 |
| `-Encoding` | 多形 | 統一語彙ほか（位置 1）。省略時の規則は上記 |
| `-EncodingFrom` | `string` | 参照ファイルから継承する |
| `-LineBreak` | `Auto` / `CrLf` / `Lf` / `Cr` | 各行の後ろに付ける改行 |
| `-Append` | switch | 追記する |
| `-AllowEncodingChange` | switch | `-Append` の整合性検査を行わない |
| `-NoClobber` | switch | 出力先が既に存在する場合はエラーにする（別名 `-NoOverwrite`） |
| `-Force` | switch | 読み取り専用のファイルにも書き込む |
| `-Width` | `int` | 整形の幅。省略時は `Out-String` の既定 |
| `-NoNewline` | switch | 改行を付けずに連結して書く |
| `-Culture` | `string` | 判定に用いるカルチャー名 |
| `-Strategy` | `string` | 判定方式 |
| `-WhatIf` / `-Confirm` | — | `SupportsShouldProcess` に対応 |

`-NoClobber` / `-Force` / `-Append` の組み合わせの挙動は、標準の `Out-File` を実測して、同じ結果になるようにしています。

`-Culture` と `-Strategy` が効くのは、`-EncodingFrom` の参照ファイル、`Auto` での出力先、`-Append` での追記先を判定するときです。

### Out-ProbedFile のエラーは、すべて終了エラーです

`Out-ProbedFile` の出力先は、常に 1 ファイルだけです。

出力先に関する失敗は「1 ファイルも処理できない」ことになるので、失敗はすべて終了エラーにしています。標準の `Out-File` も同じです。

`Set-ProbedContent` では非終了エラーだった「読み取り専用で `-Force` なし」なども、`Out-ProbedFile` では終了エラーになります。基準は同じで、出力先が 1 件か複数かの違いです。

## Convert-ProbedContent — 既存のファイルを変換する

既存のテキストファイルの**文字エンコーディング・BOM・改行を変換して書き直す**コマンドです。

1.1.0 までも、`Get-ProbedContent -Raw` で読んで `Set-ProbedContent` で書けば変換はできました。しかし、ファイルの数だけ手順を書く必要があり、失敗したときの後始末も利用者の責任でした。

`Convert-ProbedContent` は、これを 1 コマンドで、安全に行います。

```
# Shift_JIS などのファイルを、BOM 無しの UTF-8・改行 LF に変換する（その場で上書き）
Convert-ProbedContent *.txt -Encoding utf8NoBOM -LineBreak Lf

# 元の文字エンコーディングは知らなくてよい。BOM だけ外す
Convert-ProbedContent *.txt -Bom Remove

# フォルダーの下をまとめて変換する
Get-ChildItem -Recurse -Filter *.cs | Convert-ProbedContent -Encoding utf8BOM

# 元のファイルはそのまま残し、別のフォルダーへ書き出す
Convert-ProbedContent *.txt -Encoding utf8NoBOM -Destination .\out
```

変換元の文字エンコーディングは、自動で判定します。

対象の探し方（再帰や絞り込み）は、`Get-ChildItem` に任せています。`-Recurse` / `-Include` / `-Exclude` のようなパラメータは持っていません。

文字列の置換など、内容の加工も行いません。加工が必要な場合は、[rmsmf-txprobe & mfsr-mfprobe](/tool_rmsmf_guide/) の mfsr などを使ってください。なお、`Convert-ProbedContent` と mfsr は独立したコマンドで、仕様は揃えていません。

### 実行例

改行が CR-LF と LF の混在した Shift_JIS のファイルを、BOM 無しの UTF-8・改行 LF に変換します。

```
Convert-ProbedContent .\sjis.txt -Encoding utf8NoBOM -LineBreak Lf -PassThru
```

```
Path            : C:\work\sjis.txt
Destination     : C:\work\sjis.txt
SourceEncoding  : shift_jis
Encoding        : utf8NoBOM
SourceLineBreak : LfAndCrLf
LineBreak       : Lf
Changed         : True
```

| | バイト列 |
| :---- | :---- |
| 変換前 | `93 FA 96 7B 8C EA 0D 0A 41 0A` |
| 変換後 | `E6 97 A5 E6 9C AC E8 AA 9E 0A 41 0A` |

「日本語」が UTF-8 の 9 バイトになり、CR-LF（`0D 0A`）が LF（`0A`）に揃いました。

`-PassThru` を付けると、変換したファイル 1 件につき 1 個の結果を出力します。付けなければ何も出力しません。

| プロパティ | 内容 |
| :---- | :---- |
| `Path` | 変換元の絶対パス |
| `Destination` | 書き込み先の絶対パス（その場での変換は `Path` と同じ） |
| `SourceEncoding` | 変換元の文字エンコーディング（統一語彙名） |
| `Encoding` | 変換先の文字エンコーディング（統一語彙名） |
| `SourceLineBreak` | 変換元の改行（混在なら `LfAndCrLf` など） |
| `LineBreak` | 変換先の改行 |
| `Changed` | 変換結果が変換元と異なるか |

`SourceEncoding` の名前は、そのまま `-Encoding` に渡せる形になっています。変換を元に戻したいときは、この値を `-Encoding` に指定してください。

### 文字を失う変換は、行いません

`Convert-ProbedContent` で一番大事にしたのは、この点です。

**変換によって文字が失われることは、ありません。** 失われる場合は、そのファイルを変換しません。

一つ目は、変換先の文字エンコーディングで表現できない文字がある場合です。

```
# 日本語と韓国語が混ざった UTF-8 のファイルを、Shift_JIS に変換しようとする
Convert-ProbedContent .\k.txt -Encoding shift_jis
```

```
'...\k.txt' の 2 行 5 桁目の文字 U+D55C '한' は文字エンコーディング 'shift_jis' で
表現できないため、変換しませんでした。ファイルは変更していません。
```

Shift_JIS にはハングルがありません。.NET の既定の動作では、このような文字は黙って `?` に置き換えられます。`Convert-ProbedContent` は置き換えずに止まり、**最初に表現できなかった文字と、その行・桁**を示します。

二つ目は、変換元に、その文字エンコーディングとして不正なバイト列がある場合です。

```
# Shift_JIS のファイルを、UTF-8 だと指定して変換しようとする
Convert-ProbedContent .\s.txt -SourceEncoding utf8 -Encoding utf8BOM
```

```
'...\s.txt' には文字エンコーディング 'utf8NoBOM' として不正なバイト列がある
（バイト位置 2: 93）ため、変換しませんでした。ファイルは変更していません。
文字エンコーディングの判定が誤っている場合は -SourceEncoding で指定してください。
```

不正なバイト列を黙って `U+FFFD`（置換文字）にして変換すると、元のファイルの情報が失われます。これも、変換せずに止まります。

どちらの場合も非終了エラーで、そのファイルは無傷のまま残ります。他のファイルの処理は続きます。

表現できない文字を置き換えて変換するスイッチは、**意図的に用意していません。** 置き換えてよい場合は、利用者が事前に文字列を加工してください。「変換で文字が失われない」ことを、このコマンドの保証にしたかったからです。

### -Bom で、BOM だけを変えられます

`-Bom` には `Add` と `Remove` を指定できます。

```
# BOM を付ける
Convert-ProbedContent .\a.txt -Bom Add

# BOM を外す
Convert-ProbedContent .\a.txt -Bom Remove
```

`-Encoding` を省略すれば、元の文字エンコーディングのまま、BOM だけが変わります。元の文字エンコーディングが何だったかを、利用者が知っている必要はありません。

BOM を持てるのは、Unicode 系の 5 系統（UTF-8 / UTF-16LE / UTF-16BE / UTF-32LE / UTF-32BE）だけです。それ以外の文字エンコーディングへの指定は、以下のように扱います。

| 指定 | 変換先が Unicode 系 | 変換先が Unicode 系以外（Shift_JIS など） |
| :---- | :---- | :---- |
| `-Bom Add` | BOM を付ける | エラー（そのファイルは変換しない） |
| `-Bom Remove` | BOM を外す | 何もしない |

`Remove` を「何もしない」にしたのは、`Get-ChildItem -Recurse | Convert-ProbedContent -Bom Remove` のように、種類の混ざったファイル群へまとめて掛けたときに、エラーが大量に出ないようにするためです。外す BOM が無いのだから、要求は満たされています。

### -Bom を付ければ、裸の utf8 が使えます

1.1.0 の書き込み系のコマンドでは、裸の `utf8` を受け付けませんでした。PowerShell 5.1 と 7.x で、BOM の有無の解釈が逆になるからです。

`Convert-ProbedContent` でも、`-Bom` を付けなければ同じくエラーになります。

```
Convert-ProbedContent .\s.txt -Encoding utf8
```

```
書き込みでは 'utf8' を指定できません。UTF-8 は PowerShell 5.1 と 7.x で BOM の解釈が
異なるため、'utf8NoBOM' または 'utf8BOM' のいずれかを明示してください。
```

しかし、`-Bom` を付けた場合は受け付けます。

```
# 「UTF-8 にして、BOM を付ける」
Convert-ProbedContent .\s.txt -Encoding utf8 -Bom Add
```

裸の `utf8` を拒否していたのは、BOM が曖昧になるのを防ぐためでした。`-Bom` があれば BOM は曖昧ではないので、`utf8` を「系統の名前」として扱えます。`utf-8` のような WebName も同様です。

`utf8NoBOM` と `-Bom Add` のように、指定どうしが食い違った場合はエラーになります。

### -LineBreak は、ファイルの中の改行をすべて揃えます

ここは、書き込み系のコマンドとは意味が違います。

| コマンド | `-LineBreak` の意味 |
| :---- | :---- |
| `Set-ProbedContent` / `Add-ProbedContent` / `Out-ProbedFile` | 各要素（各行）の後ろに付ける改行を決める。文字列の中の改行は置き換えない |
| `Convert-ProbedContent` | ファイルの中の**すべての改行**（CR-LF / LF / CR）を指定の改行に揃える |

ファイルの中の改行を揃えることが、このコマンドの役割だからです。

最後の行の末尾に改行があるかないかは、元のまま保ちます。改行を足したり削ったりはしません。

`-LineBreak` を省略した場合は、混在した改行も含めて、元のまま保ちます。

### 途中で失敗しても、元のファイルは壊れません

その場で変換する場合は、**同じフォルダーに一時ファイルを書いてから、元のファイルと置き換えます。**

変換の途中で失敗しても、元のファイルは壊れません。失敗したときは、一時ファイルも削除します。

また、**変換結果が元のバイト列とまったく同じなら、書き直しません。**

```
# 同じ変換をもう一度実行する
Convert-ProbedContent .\sjis.txt -Encoding utf8NoBOM -LineBreak Lf -PassThru |
    Select-Object Changed
```

```
Changed
-------
  False
```

このとき、ファイルの更新日時も変わりません。すでに変換済みのファイルが混ざったフォルダーに何度実行しても、変換が必要なファイルだけが更新されます。バックアップや同期ツールが、変換していないファイルまで「更新された」と見なすことがありません。

読み取り専用のファイルは、`-Force` を付けたときだけ変換します。変換した後、読み取り専用の属性は元に戻します。

`-WhatIf` / `-Confirm` は、ファイルごとに効きます。

### -Destination で、別のフォルダーへ書き出す

`-Destination` にフォルダーを指定すると、元のファイルはそのまま残し、変換結果をそのフォルダーへ書き出します。

```
# Shift_JIS・CR-LF のファイルを、out フォルダーへ UTF-8 で書き出す
Get-ChildItem -Filter s2.txt | Convert-ProbedContent -Encoding utf8NoBOM -Destination .\out
```

| ファイル | バイト列 |
| :---- | :---- |
| s2.txt（元のまま） | `93 FA 0D 0A` |
| out\s2.txt | `E6 97 A5 0D 0A` |

`-LineBreak` を指定していないので、改行は元の CR-LF のままです。

`-Destination` には、以下の決まりがあります。

- 出力先のフォルダーは、あらかじめ作っておいてください。存在しない場合はエラーになります（フォルダーは作成しません）
- `Get-ChildItem -Recurse` から受け取った場合も、**フォルダー構造は保ちません。** すべて `-Destination` の直下に置きます
- 出力先に同じ名前のファイルがある場合はエラーです。`-Force` を付ければ上書きします
- 同じ実行の中でファイル名が重なった場合、2 件目以降はエラーになります。`-Force` を付けても上書きしません（同じ実行で書いたファイルを消さないためです）

### 判定を誤るファイルは、-SourceEncoding で指定します

変換元の文字エンコーディングは、`Resolve-Encoding` と同じ処理で判定しています。

判定を誤るファイルがある場合は、`-SourceEncoding` で変換元の文字エンコーディングを明示してください。明示した場合は判定を行いません。

```
# 変換元が EUC-JP だと分かっているファイルを、UTF-8 に変換する
Convert-ProbedContent .\old.txt -SourceEncoding euc-jp -Encoding utf8NoBOM
```

`-Culture` と `-Strategy` は、判定を行うときだけ使われます。

### パラメータ一覧

| パラメータ | 型 | 説明 |
| :---- | :---- | :---- |
| `-Path` | `string[]` | 変換元（位置 0）。ワイルドカード対応 |
| `-LiteralPath` | `string[]` | 変換元。ワイルドカード非対応。`Get-ChildItem` の出力はここに束縛される |
| `-Encoding` | 多形 | 変換先の文字エンコーディング（位置 1）。省略時は変換元のまま |
| `-EncodingFrom` | `string` | 変換先を参照ファイルから継承する |
| `-Bom` | `Add` / `Remove` | 変換先の BOM。省略時は変えない |
| `-LineBreak` | `Auto` / `CrLf` / `Lf` / `Cr` | 変換先の改行。省略時（`Auto`）は変えない |
| `-SourceEncoding` | 多形 | 変換元の文字エンコーディング。省略時は判定する |
| `-Culture` | `string` | 変換元の判定に用いるカルチャー名 |
| `-Strategy` | `string` | 変換元の判定方式 |
| `-Destination` | `string` | 出力先のフォルダー。省略時はその場で上書き |
| `-Force` | switch | 読み取り専用も変換する。`-Destination` では同名ファイルを上書きする |
| `-PassThru` | switch | 結果を出力する |
| `-WhatIf` / `-Confirm` | — | ファイルごとに効く |

`-Encoding` / `-EncodingFrom` / `-Bom` / `-LineBreak` のどれも指定しない場合は、何も変換しないのでエラーになります。

`-Encoding Auto` も受け付けません。「変換元のまま」は、`-Encoding` の省略で表します。

変換元にフォルダーが渡された場合（`Get-ChildItem -Recurse` の出力に含まれます）は、エラーにせず飛ばします。

### 注意 — ファイル全体をメモリに読みます

`Convert-ProbedContent` は、文字を失わないことを確認するために、ファイル全体をメモリに読んでから変換します。

非常に大きなファイルを変換する場合は、メモリの使用量に注意してください。

## 既存のコマンドの変更

1.1.0 のコマンドの変更は、二つだけです。どちらも、既存のスクリプトの動作を壊す変更ではありません。

### -Force の後に、読み取り専用の属性を元に戻すようにしました

**これは 1.1.0 の不具合の修正です。**

1.1.0 の `Set-ProbedContent` / `Add-ProbedContent` は、`-Force` で読み取り専用のファイルに書き込んだ後、読み取り専用の属性を外したままにしていました。

標準の `Set-Content` / `Add-Content` / `Out-File` は、いずれも書き込んだ後で属性を元に戻します。1.2.0 では、これに合わせました。

```
# 読み取り専用のファイルに、-Force で書き込む
Set-ItemProperty .\ro.txt IsReadOnly $true
Set-ProbedContent .\ro.txt -Value 'y' -Encoding utf8NoBOM -Force
(Get-Item .\ro.txt).IsReadOnly
```

```
True
```

書き込みが途中で失敗した場合も、属性は元に戻します。`Out-ProbedFile` と `Convert-ProbedContent` も、同じ処理を共用しています。

### -LineBreak の範囲を、ヘルプに明記しました

`Set-ProbedContent` / `Add-ProbedContent` の `-LineBreak` は、要素の後ろに付ける改行だけを決め、文字列の中に含まれる改行は置き換えません。

これは 1.1.0 からの挙動で、変更はしていません。ヘルプに書いていなかったので、5 言語のヘルプに明記しました。

## クラスライブラリ — 判定を、世界の言語に対応させました

ここからは、クラスライブラリ（SnowStack.EncodingProbe）の判定処理の変更です。

NuGet パッケージを直接使っている方にも、コマンドレットを使っている方にも関係します。

### 1.1.0 では、日本語環境でドイツ語を読むと文字化けしました

1.1.0 で、日本語の Windows から windows-1252（西欧の文字エンコーディング）のドイツ語のファイルを読むと、次のようになりました。

```
# 日本語環境で、windows-1252 のドイツ語のファイルを読む（1.1.0）
Get-ProbedContent .\german.txt
```

```
Gr・e aus M・chen. Die Straﾟe ist f・ Fuﾟg舅ger gesperrt.
```

1.2.0 では、正しく読めます。

```
Grüße aus München. Die Straße ist für Fußgänger gesperrt.
```

1.1.0 の判定結果は `932 / shift_jis` でした。ドイツ語のファイルを、Shift_JIS だと判定していたのです。

[-Culture と -Strategy の解説](/encodingprobe_culture_strategy/) の記事では、この問題を回避するために `-Strategy UtfUnknownOnly` を指定するよう案内していました。1.2.0 では、既定の判定方式のままで正しく判定します。

### なぜ Shift_JIS と判定していたのか

このライブラリは、二つの判定処理を組み合わせています。

| 判定処理 | 得意なもの |
| :---- | :---- |
| 独自実装の判定 | Unicode と、東アジアの旧マルチバイト（Shift_JIS / EUC / GBK / Big5 など） |
| UTF.Unknown（サードパーティ製） | 欧米などのシングルバイト |

既定の判定方式（`Combined`）では、まず独自実装で判定し、判定できなかったときだけ UTF.Unknown に任せます。

独自実装の判定は、「その文字エンコーディングのバイト構造として成立しているか」を見ています。

ところが、ドイツ語の `üß` は windows-1252 で `FC DF` の 2 バイトになり、これが Shift_JIS の外字領域の 2 バイト文字として、構造上は成立してしまいます。

独自実装が先に「Shift_JIS です」と答えを出すので、UTF.Unknown の出番がありませんでした。

韓国語の環境なら CP949、簡体字中国語の環境なら GB18030、繁体字中国語の環境なら Big5 と、同じことがそれぞれの言語の環境で起きていました。

### 1.2.0 の対策 — UTF.Unknown と突き合わせる

1.2.0 では、以下の対策を行いました。

**一つ目は、東アジア以外のカルチャーでは、東アジアの旧マルチバイトの判定を行わないことです。**

ドイツ語やフランス語の環境で、Shift_JIS や Big5 の判定を行う必要はありません。判定しなければ、誤判定も起きません。

実は 1.1.0 でも、大部分はこの動作になっていました。1.2.0 では、カルチャーごとの判定の対象を一か所の対応表にまとめ、規則として明確にしました。

ただし、Unicode（UTF-8 / UTF-16 / UTF-32）と BOM・ISO-2022・ASCII の判定は、カルチャーに関係なく行います。UTF.Unknown は BOM 無しの UTF-16 / UTF-32 を判定できないので、ここを外すわけにはいかないからです。

**二つ目は、独自実装の答えを、UTF.Unknown と突き合わせることです。**

東アジアのカルチャーでは、東アジアの旧マルチバイトの判定を止めるわけにはいきません。

そこで、独自実装が東アジアの旧マルチバイトを答えたときは、UTF.Unknown にも判定させます。UTF.Unknown が十分な確かさ（信頼度 0.55 超）で「シングルバイトの文字エンコーディングだ」と判定した場合は、そちらを採用します。

本物の日本語や中国語のテキストに対して、UTF.Unknown がシングルバイトを高い信頼度で返すことはないので、東アジアの言語の判定結果は変わりません。

この突き合わせは、既定の `Combined` だけで行います。`-Strategy NativeOnly`（独自実装だけで判定する）を指定した場合は行わないので、日本語環境のドイツ語は、1.2.0 でも Shift_JIS と判定されます。

**三つ目は、UTF-8 の判定を厳密にしたことです。**

1.1.0 の UTF-8 の判定は、多バイト文字の途中で ASCII に戻るバイト列を見逃していました。

たとえば windows-1252 のフランス語 `Français` は `46 72 61 6E E7 61 69 73` というバイト列で、`E7` は UTF-8 なら 3 バイト文字の先頭です。しかし後ろに続くのは `61 69`（ASCII の `ai`）なので、UTF-8 としては成立しません。1.1.0 は、これを UTF-8 と判定していました。

1.2.0 では、UTF-8 の規格（RFC 3629）どおりに検証し、途中で途切れた多バイト文字や、冗長な符号化などを、UTF-8 ではないと判定します。

### 判定結果の比較

ライブラリのテストデータを、1.1.0 と 1.2.0 で判定した結果です（判定方式は既定の `Combined`）。

**日本語カルチャー（ja-JP）での結果**

| ファイル | 1.1.0 | 1.2.0 |
| :---- | :---- | :---- |
| ドイツ語 windows-1252 | `932 / shift_jis` | `28591 / iso-8859-1` |
| スペイン語 windows-1252 | `932 / shift_jis` | `28591 / iso-8859-1` |
| タイ語 cp874 | `932 / shift_jis` | `874 / tis-620` |
| ルーマニア語 ISO-8859-16 | **例外で停止** | 判定失敗のエラー（後述） |

windows-1252 のファイルが `iso-8859-1` と判定されるのは UTF.Unknown の答えで、ドイツ語やスペイン語の文字の範囲では、どちらで読んでも同じ文字列になります。

**スペイン語 windows-1252 を、東アジアの各カルチャーで判定した結果**

| カルチャー | 1.1.0 | 1.2.0 |
| :---- | :---- | :---- |
| ja-JP | `932 / shift_jis` | `28591 / iso-8859-1` |
| zh-CN | `54936 / gb18030` | `28591 / iso-8859-1` |
| zh-TW | `950 / big5` | `28591 / iso-8859-1` |
| zh-HK | `950 / big5` | `28591 / iso-8859-1` |

### 確認した言語

1.2.0 では、テストデータに以下の言語を追加しました。

| 言語 | 文字エンコーディング |
| :---- | :---- |
| ドイツ語 | windows-1252 / ISO-8859-1 / UTF-8 |
| フランス語 | windows-1252 / ISO-8859-15 / UTF-8 |
| スペイン語 | windows-1252 / UTF-8 |
| ポーランド語 | windows-1250 / ISO-8859-2 / UTF-8 |
| ロシア語 | windows-1251 / KOI8-R / UTF-8 |
| タイ語 | cp874 / UTF-8 |
| エストニア語 | windows-1257 / ISO-8859-15 / UTF-8 |
| ウクライナ語 | windows-1251 / KOI8-U / UTF-8 |
| ルーマニア語 | ISO-8859-16 / UTF-8 |
| アイスランド語 | windows-1252（短い文のみ） |

これらを 12 のカルチャーで判定して、結果がカルチャーによって変わらないこと、復号した本文が元の本文と一致することを、自動テストで確認しています。

ただし、ウクライナ語・ルーマニア語と、アイスランド語などの短い文のデータは、後述する「判定できないもの」の現在の挙動を固定するためのデータです。正しく判定できることを確認しているわけではありません。

さらに、テストデータに無い 17 言語（31 種類の文字エンコーディング）の文で、東アジアの 5 つのカルチャーから判定する測定を行いました。通常の長さ（約 170 バイト以上）の文では、東アジアの文字エンコーディングと誤判定されたまま残るものは 0 件でした。

### 短い文では、まだ誤判定することがあります

正直に書いておきます。

おおむね **90 バイト以下の短いシングルバイト系のテキスト**を、東アジアのカルチャーで判定すると、誤った文字エンコーディングになる、または東アジアの文字エンコーディングのまま残る場合があります。

原因は、短い文に対する UTF.Unknown の判定の精度です。UTF.Unknown 自身が誤った答えを返したり、答えの信頼度が低すぎたりするので、このライブラリの側では解決できません。

短いファイルで判定を誤る場合は、`-Encoding`（`Convert-ProbedContent` では `-SourceEncoding`）で文字エンコーディングを明示してください。

また、以下の言語は、通常の長さの文でも判定できません。

| 言語 | 理由 |
| :---- | :---- |
| ウクライナ語（windows-1251 / KOI8-U） | UTF.Unknown の信頼度が低く、判定不能になる |
| ルーマニア語（ISO-8859-16） | UTF.Unknown は判定するが、.NET がこの文字エンコーディングを提供していない |

### ルーマニア語で例外が出る不具合を直しました

**これは、リリース済みの 1.1.0 から存在していた不具合です。**

UTF.Unknown は、.NET が提供していない文字エンコーディング（ルーマニア語の ISO-8859-16 など）を答えることがあります。1.1.0 はこの場合に `NullReferenceException` を起こし、`Resolve-Encoding` や `Get-ProbedContent` の利用者まで、そのまま例外が届いていました。

```
# ルーマニア語 ISO-8859-16 のファイルを読む（1.1.0）
Get-ProbedContent .\romanian.txt
```

```
Object reference not set to an instance of an object.
```

1.2.0 では、判定失敗の非終了エラーとして報告し、`-Encoding` での明示指定を案内します。

```
'...\romanian.txt' の文字エンコーディングを判定できませんでした。
-Encoding で明示的に指定してください。
```

クラスライブラリの `Detect` メソッドは、この場合に例外を投げず、`CodePage` が `-1` で、`EncodingWebName` に UTF.Unknown が答えた名前（`iso-8859-16`）を入れた結果を返します。

同じ扱いになるのは、ほかに `iso-8859-10` / `viscii` / `euc-tw` などです。

## クラスライブラリ — 香港の Big5 に対応しました

1.1.0 の記事で「香港（繁体字広東語）は、後のバージョンで対応する予定です」と書いていた件です。

### 台湾の Big5 と香港の Big5 は、区別できません

最初に、できないことを書いておきます。

香港では、台湾の Big5 に香港独自の文字（HKSCS）を追加した文字エンコーディングが使われています。

HKSCS は Big5 の上位互換で、使うバイトの範囲が Big5 を完全に含んでいます。そのため、**台湾の Big5 と香港の Big5 は、バイト列から区別できません。**

香港の文書であっても、判定結果は台湾と同じ `950 / big5` です。

HKSCS の固有の文字を含まない文書なら、どちらとして読んでも同じ文字列になるので、実害はありません。

### .NET には、HKSCS のデコーダーがありません

もう一つ、.NET 側の制約があります。

.NET には HKSCS を読み書きする機能がありません。`Encoding.GetEncoding("big5-hkscs")` を呼んでも、台湾の Big5（コードページ 950）が返ってきます。

そのため、HKSCS の固有の文字は、Unicode の**私用領域**（U+E000〜U+F8FF）の符号位置として読み込まれます。

私用領域は、Unicode が文字の意味を決めていない領域です。日本語の Shift_JIS の外字も、同じく私用領域として読み込まれます。

### 私用領域の文字は、そのまま書き戻せます

私用領域の符号位置は、同じ文字エンコーディングで書き戻せば、元のバイト列に戻ります。

HKSCS の固有の文字を含む Big5 のファイルを、UTF-8 に変換して、また Big5 に戻してみます。

```
# 香港カルチャーで判定する
Resolve-Encoding .\hk.txt -Culture zh-HK
```

```
CodePage        : 950
EncodingWebName : big5
PSEncodingName  : big5
UsePSName       : False
Bom             : False
LineBreak       : CrLf
Culture         : zh-HK
```

```
# 読み込んで、私用領域の文字を探す
$t = Get-ProbedContent .\hk.txt -Culture zh-HK -Raw
[regex]::Matches($t, '[\uE000-\uF8FF]') | ForEach-Object { 'U+{0:X4}' -f [int][char]$_.Value }
```

```
U+F325
U+F570
U+E01F
U+E286
```

このファイルには、HKSCS の固有の文字が 4 つ含まれていて、それぞれ私用領域の符号位置として読み込まれています。

```
# UTF-8 に変換して、また Big5 に戻す
Convert-ProbedContent .\hk.txt -Culture zh-HK -Encoding utf8NoBOM
Convert-ProbedContent .\hk.txt -Encoding big5
```

Big5 に戻したファイルは、元のファイルと**バイト単位で一致しました。**

UTF-8 に変換したファイルでは、私用領域の符号位置がそのまま UTF-8 で書かれています。対応するフォントが無い環境では、その文字は正しく表示されません。

このライブラリは、私用領域の符号位置を HKSCS の文字へ写し替えることはしません。同じ私用領域の符号位置が、香港では HKSCS の文字、台湾では別の拡張文字、別の会社では独自の造字を意味することがあり、どの文字なのかはバイト列からは分からないからです。

写し替えが必要な場合は、読み込んだ文字列に対して、利用者が対応表を使って置き換えてください。このライブラリが私用領域の符号位置をそのまま返すので、その処理が成り立ちます。

日本語の外字も含めた私用領域の扱いと、置き換えの具体例は、以下の記事で解説しています。

[外字（私用領域）の扱い — SnowStack.EncodingProbe](/encodingprobe_private_use_area/)

### 香港のために直した点

区別できないのに、何を直したのかというと、主に以下の点です。

**カルチャー名の解釈を直しました。**

1.1.0 は、カルチャー名を完全一致で調べていました。そのため、.NET 10 が使う `zh-Hant-HK` のような形のカルチャー名では、判定できませんでした。

1.2.0 では、カルチャー名を「言語・用字・地域」に分解して解釈します。`zh-Hant-HK` / `zh-Hans-CN` / `zh_HK`（アンダースコア区切り）なども、正しく解釈します。広東語（`yue`）は、香港の繁体字として扱います。

**繁体字と簡体字の取り違えを直しました。**

1.1.0 では、台湾・香港のカルチャーで簡体字（GBK）の文書を読むと Big5 と、大陸のカルチャーで繁体字（Big5）の文書を読むと GB18030 と、誤って判定していました。

1.2.0 では、UTF.Unknown が反対側の系統（Big5 と GB 系）を高い信頼度（0.8 以上）で答えたときは、その系統で判定をやり直します。

| ファイル | カルチャー | 1.1.0 | 1.2.0 |
| :---- | :---- | :---- | :---- |
| 簡体字 GBK | zh-TW | `950 / big5` | `936 / gbk` |
| 簡体字 GBK | zh-HK | `950 / big5` | `936 / gbk` |
| 繁体字 Big5 | zh-CN | `54936 / gb18030` | `950 / big5` |
| HKSCS 入りの Big5 | zh-Hant-HK | 判定不能 | `950 / big5` |

**その他の修正**

- 香港・マカオのカルチャーでは、台湾の文字エンコーディングである EUC-TW を候補から外しました
- Big5 の 2 バイト目の範囲を、規格どおり厳密にしました
- コンカニ語（`kok`）のカルチャーを、韓国語と判定していたのを直しました（`ko` で始まる名前を前方一致で調べていたためです）

日本語・韓国語・繁体字中国語・簡体字中国語の環境で、それぞれの言語のテキストを判定した結果は、1.1.0 から変わっていません。

### 既知の限界

大陸のカルチャー（zh-CN）で、HKSCS の固有の文字を含む Big5 を読むと、`54936 / gb18030` と判定されます。

HKSCS の固有の文字が入った文書では、UTF.Unknown が系統を判定できなくなるので、繁簡の取り違えの修正が働かないためです。

大陸の環境で香港の文書を読む場合は、`-Culture zh-HK` を指定するか、`-Encoding big5` で明示してください。

### ヘルプとメッセージも、香港・マカオに対応しました

1.1.0 では、香港・マカオのカルチャーの `Get-Help` は英語でした。

1.2.0 では、香港（`zh-HK`）・マカオ（`zh-MO`）のカルチャーで、繁体字中国語のヘルプとエラーメッセージを表示します。広東語（`yue`）のカルチャーのエラーメッセージも、繁体字中国語になります。

文面は台湾向けと同じ内容です。Microsoft は Windows の香港向けの言語パックの提供をやめ、台湾向けを案内しているので、香港の Windows で実際に表示される繁体字の文面に合わせました。

以下の組み合わせでは、まだヘルプが英語になります。

- 広東語（`yue`）のカルチャー
- PowerShell 7.x で、`zh-Hant-TW` / `zh-Hant-HK` / `zh-Hant-MO` / `zh_HK` のカルチャー名を使った場合

## NuGet パッケージを直接使っている方へ

1.2.0 では、公開 API は変更していません。クラスもメソッドも、1.1.0 のままです。

判定結果が変わるのは、この記事で解説したケース（東アジアの環境での欧米の言語、繁簡の取り違え、香港のカルチャー名など）です。

もう一点、ストリームを渡す `Detect(Stream)` の不具合を直しました。

1.1.0 の `Detect(Stream)` は、独自実装の判定でストリームを読み切った後、そのストリームを UTF.Unknown にも渡していました。そのため、UTF.Unknown は空のストリームを見ていて、UTF.Unknown による判定が機能していませんでした。

1.2.0 では、ストリームを一度だけ読んでバイト配列にし、両方の判定に渡します。

`Detect(byte[])` と `Detect(string filePath)` には、この問題はありません。PowerShell のコマンドレットは、この二つしか使っていないので、影響を受けていません。

1.1.0 のときとは違い、**今回はクラスライブラリの判定処理を改修しているので、1.2.0 への更新をお勧めします。**

## 1.2.0 でも解決していないこと

1.1.0 の記事で「1.2.0 以降で扱う予定」と書いた ISO-2022 系の判定の問題は、1.2.0 では扱っていません。

| 現象 | 状態 |
| :---- | :---- |
| ISO-2022-TW が ISO-2022-CN と誤判定される | 未対応 |
| SO/SI 形式の 1 バイトカナを検出できない | 未対応 |
| 短いシングルバイト系のテキストを、東アジアのカルチャーで誤判定する | UTF.Unknown の限界のため未対応 |
| 大陸のカルチャーで、HKSCS 入りの Big5 を GB18030 と判定する | 未対応 |

該当する文字エンコーディングを確実に扱いたい場合は、`-Encoding` で明示的に指定してください。判定を行わないので、これらの問題を回避できます。

## コマンド早見表

1.2.0 で追加した機能を、1 枚にまとめます。

| やりたいこと | 書き方 |
| :---- | :---- |
| 表や一覧を、BOM 無しの UTF-8 で書き出す | <code>... &#124; Out-ProbedFile .\a.txt</code> |
| 表や一覧を、Shift_JIS で書き出す | <code>... &#124; Out-ProbedFile .\a.txt shift_jis</code> |
| 文字エンコーディングを保って追記する | <code>... &#124; Out-ProbedFile .\a.txt -Append</code> |
| ファイルを UTF-8 に変換する | `Convert-ProbedContent .\a.txt -Encoding utf8NoBOM` |
| 改行を LF に揃える | `Convert-ProbedContent .\a.txt -LineBreak Lf` |
| BOM だけ外す | `Convert-ProbedContent .\a.txt -Bom Remove` |
| フォルダーの下をまとめて変換する | <code>Get-ChildItem -Recurse -Filter *.txt &#124; Convert-ProbedContent -Encoding utf8NoBOM</code> |
| 元のファイルを残して変換する | `-Destination .\out` を付ける |
| 変換前に対象を確認する | `-WhatIf` を付ける |
| 変換の結果を確認する | `-PassThru` を付ける |
| 変換元の判定を誤るファイルを変換する | `-SourceEncoding euc-jp` などを付ける |
| 香港の Big5 を読む | `Get-ProbedContent .\a.txt -Culture zh-HK` |

以上、SnowStack.EncodingProbe 1.2.0 の解説でした。

## お知らせ欄

### 2026年9月30日　Version 1.2.0 リリース

Version 1.2.0 をリリースしました。

PowerShell モジュールに `Out-ProbedFile` / `Convert-ProbedContent` の 2 コマンドを追加し、クラスライブラリの判定処理を改修しています。

既存のコマンドのパラメータと、クラスライブラリの公開 API は変更していないので、1.1.0 向けに書いたスクリプトやプログラムはそのまま動きます。ただし、この記事で解説したケースでは、判定結果が 1.1.0 と変わります。

## 関連資料

- [SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)
- [SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)
- [SnowStack.EncodingProbe NuGet Package 解説](/encodingprobe_guide/)
- [外字（私用領域）の扱い — SnowStack.EncodingProbe](/encodingprobe_private_use_area/)
- [-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)
- [PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか （フレンドリ名の解説）](/why-utf8nobom-cannot-be-webname/)
- [SnowStack.EncodingProbe.PowerShell を Windows PowerShell 5.1 へインストールする方法](/encodingprobe_powershell_install_ps51/)
- [SnowStack.EncodingProbe（GitHub リポジトリ）](https://github.com/motoi-tsushima/SnowStack.EncodingProbe)
