---
title: "SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説"
layout: single
classes: wide
permalink: /encodingprobe_probed_content/
author_profile: true
---
2026/09/30 document update

SnowStack.EncodingProbe.PowerShell のバージョン 1.1.0 で追加した、4 つのコマンドレットの解説です。

1.1.0 は「便利なコマンドが 4 つ増えました」という話ではありません。

**同じスクリプトが、Windows PowerShell 5.1 でも PowerShell 7.x でも、同じバイト列を書く。** それが 1.1.0 の目的です。

そのため、この記事はコマンドの紹介から始めず、まず「何が困っていたのか」から入ります。

インストール方法と、1.0.x から提供している `Resolve-Encoding` の解説は、以下の記事で行っています。

[SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)

なお、`-Culture` と `-Strategy` の二つのオプションは、分量があるので別の記事に分けました。外国語のテキストファイルを扱う方は、そちらもご覧ください。

[-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)

（この記事に掲載した実行結果は、すべて実機で採取したものです。採取日は 2026年8月31日、環境は Windows 11 上の PowerShell 7.6.5 と Windows PowerShell 5.1.26100 です）

## 同じ Set-Content が、PowerShell のバージョンで違うバイト列を書く

まず、一つだけ実験をしてみてください。

以下の 1 行を、Windows PowerShell 5.1 と PowerShell 7.x の両方で実行します。

```
# PowerShell 標準の Set-Content で、UTF-8 を指定してファイルを作る
Set-Content .\std.txt -Value 'abc' -Encoding UTF8
```

できあがったファイルのバイト列は、次のようになります。

| ホスト | できたバイト列 |
| :---- | :---- |
| Windows PowerShell 5.1 | `EF BB BF 61 62 63 0D 0A` ← **BOM が付く** |
| PowerShell 7.x | `61 62 63 0D 0A` ← **BOM が付かない** |

同じコマンド、同じパラメータ、違う結果です。

BOM とは、Byte Order Mark の略で、テキストファイルの先頭に置かれる数バイトの目印のことです。UTF-8 の BOM は `EF BB BF` の 3 バイトになります。

なぜこうなるかというと、`UTF8` という名前の意味が、ホストによって違うからです。

Windows PowerShell 5.1 の `UTF8` は「BOM 付きの UTF-8」を意味し、PowerShell 7.x の `utf8` は「BOM 無しの UTF-8」を意味します。

同じ名前が、正反対の意味を持っているのです。

これは、5.1 と 7.x を両方使う現場で起きていることの縮図です。

書いた本人は同じスクリプトのつもりでも、実行されるホストによって、生成されるファイルが変わります。

1.1.0 のコマンドを使うと、両方のホストで同じ結果になります。

```
# 1.1.0 の Set-ProbedContent で、BOM 無しの UTF-8 を名前で指定する
Set-ProbedContent .\probed.txt -Value 'abc' -Encoding utf8NoBOM -LineBreak Lf
```

| ホスト | できたバイト列 |
| :---- | :---- |
| Windows PowerShell 5.1 | `61 62 63 0A` |
| PowerShell 7.x | `61 62 63 0A` |

BOM も付かず、改行も指定した LF (`0A`) のままで、両ホストが一致しています。

**BOM 無しの UTF-8 を名前で指定する。これは Windows PowerShell 5.1 の標準コマンドにはできません。**

### バイト列の確認方法

以降、この記事ではファイルの中身をバイト列で示していきます。

画面に表示された文字が正しく見えても、BOM の有無や改行コードの種類は分かりません。**文字エンコーディングの確認は、必ずバイト列で行ってください。**

以下の関数を PowerShell に貼り付けておくと、確認が楽になります。

```
# ファイルの中身を 16 進数のバイト列で表示するヘルパー関数
function Show-Bytes {
    param([string]$Path)
    if (-not (Test-Path -LiteralPath $Path)) { return '(ファイルなし)' }
    # [IO.File] は PowerShell のカレント位置を見ないため、Convert-Path で絶対パスにする
    (([IO.File]::ReadAllBytes((Convert-Path -LiteralPath $Path))) |
        ForEach-Object { $_.ToString('X2') }) -join ' '
}

# 使用例
Show-Bytes .\probed.txt
```

`[IO.File]` は .NET のクラスなので、PowerShell のカレントディレクトリを見てくれません。`Convert-Path` で絶対パスに直しているのは、そのためです。

## 1.0.2 までは、判定できてもファイルを開けなかった

1.0.x の `Resolve-Encoding` は、文字エンコーディングの判定まではできました。

しかし、その判定結果を標準コマンドへ渡す段になると詰みます。

次のイディオムが、Windows PowerShell 5.1 では成立しないからです。

```
# 判定結果のフレンドリ名を、そのまま Get-Content へ渡す書き方
Get-Content -Encoding (Resolve-Encoding $f).PSEncodingName $f
```

実際に Shift_JIS のファイルへ `Resolve-Encoding` を実行すると、次の結果になります。

| ホスト | `CodePage` | `PSEncodingName` | `UsePSName` |
| :---- | :---- | :---- | :---- |
| Windows PowerShell 5.1 | 932 | （空） | `False` |
| PowerShell 7.x | 932 | `shift_jis` | `False` |

Windows PowerShell 5.1 では、`-Encoding` が固定の列挙型（`Ascii` / `UTF8` / `Unicode` など）になっており、Shift_JIS を表す値が**存在しません**。

だから `PSEncodingName` は空を返すしかありません。

PowerShell 7.x では名前を返せますが、`UsePSName` は `False` です。標準コマンドの `-Encoding` にそのまま渡せる保証がない、という意味です。

つまり **「判定はできる。しかし判定結果でファイルを開けない」** という状態でした。

なお、`PSEncodingName` の値が実行環境によって違うのは、不具合ではなく仕様です。

.NET Framework 4.8 用のビルドは、Windows PowerShell 5.1 の `-Encoding` 列挙値に一致するときだけ値を返します。

.NET 10 用のビルドは、PowerShell 6.2 以降の登録済みフレンドリ名を返します。

ターゲットフレームワークごとに意味が違うのは、そういう設計にしてあるからです。

フレンドリ名そのものの解説は、以下の記事で行っています。

[PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか （フレンドリ名の解説）](/why-utf8nobom-cannot-be-webname/)

1.1.0 は、標準コマンドに橋を架けるのをやめました。

**自前の読み書きコマンドを持つ**ことで、この問題を解いています。

## 1.1.0 で追加した 4 つのコマンド

追加したのは、以下の 4 つです。

| コマンド | 役割 | 標準の対応物 |
| :---- | :---- | :---- |
| `Get-ProbedContent` | 判定して読む | `Get-Content` |
| `Set-ProbedContent` | 明示して書く | `Set-Content` |
| `Add-ProbedContent` | 文字エンコーディングを保って追記する | `Add-Content` |
| `ConvertTo-DotNetEncoding` | 各種の指定を `System.Text.Encoding` に変換する | （なし） |

前の 3 つは、標準コマンドと同じ役割を、統一された文字エンコーディング名で行うコマンドです。

`ConvertTo-DotNetEncoding` だけは性格が違い、.NET のクラスライブラリを直接呼ぶ人のための変換コマンドになります。

パラメータの一覧は以下のとおりです。

| コマンド | パラメータ |
| :---- | :---- |
| `Get-ProbedContent` | `Path` , `LiteralPath` , `Encoding` , `Raw` , `TotalCount` , `Culture` , `Strategy` |
| `Set-ProbedContent` | `Path` , `LiteralPath` , `Value` , `Encoding` , `EncodingFrom` , `NoNewline` , `LineBreak` , `Force` , `Culture` , `Strategy` , `WhatIf` , `Confirm` |
| `Add-ProbedContent` | `Set-ProbedContent` と同じ ＋ `AllowEncodingChange` |
| `ConvertTo-DotNetEncoding` | `Encoding` |

`ConvertTo-DotNetEncoding` にだけ `-Culture` と `-Strategy` がありません。

このコマンドはファイルを引数に取らず、文字エンコーディングの判定処理そのものを呼び出さないため、渡す先が無いからです。

なお、1.0.x から提供している `Resolve-Encoding` と `Get-EncodingProbePlatformInfo` は、1.1.0 で**一切変更していません**。パラメータも戻り値も挙動もそのままです。

## 統一語彙 — 文字エンコーディングの名前を一本化する

4 つのコマンドに共通する土台が、この「統一語彙」です。

ここが 1.1.0 で最も重要な部分になります。

なお「統一語彙」は、この記事とモジュールのヘルプの中だけで使う言葉です。実行中の PowerShell が `-Encoding` で受け付ける名前の体系を「標準語彙」と呼び、それと区別するために私が付けました。

### 標準の名前の体系は、5.1 と 7.x で別物です

PowerShell の `-Encoding` に渡す名前の体系は、Windows PowerShell 5.1 と PowerShell 7.x で別物です。

そのうえ、PowerShell 7.x の体系そのものも不規則になっています。

| 名前 | BOM の扱い |
| :---- | :---- |
| `utf8` | **BOM 無し** |
| `unicode`（UTF-16LE） | BOM 有り |
| `bigendianunicode`（UTF-16BE） | BOM 有り |
| `utf32`（UTF-32LE） | BOM 有り |

UTF-8 だけ、裸の名前の意味が反転しています。

覚えられる体系ではありません。

### 全系統に明示形を用意しました

そこで 1.1.0 では、**全系統に明示形を用意して、裸の名前を覚えなくて済むようにしました。**

| 基底名 | BOM 有り | BOM 無し |
| :---- | :---- | :---- |
| `utf8` | `utf8BOM` | `utf8NoBOM` |
| `unicode`（UTF-16LE） | `unicodeBOM` | `unicodeNoBOM` |
| `bigendianunicode`（UTF-16BE） | `bigendianunicodeBOM` | `bigendianunicodeNoBOM` |
| `utf32`（UTF-32LE） | `utf32BOM` | `utf32NoBOM` |
| `bigendianutf32`（UTF-32BE） | `bigendianutf32BOM` | `bigendianutf32NoBOM` |

`*BOM` と `*NoBOM` の接尾辞は、このモジュールの独自拡張です。PowerShell 7.x にも存在しません。

裸の名前も互換のために受け付けますが、**記事としては明示形の使用をお勧めします。**

名前を見ただけで BOM の有無が分かるので、書いた本人も、後から読む人も迷いません。

BOM の接尾辞を付けられるのは、上の表の Unicode 系 5 系統だけです。

これに加えて、以下の指定を受け付けます。

| 種類 | 例 |
| :---- | :---- |
| その他の語彙 | `ascii` / `ansi` / `oem` / `utf7`（読み取り専用）/ `Auto` |
| WebName | `shift_jis` / `euc-jp` / `iso-2022-jp` / `big5` / `gb18030` など |
| 数値のコードページ | `932` / `65001` など |
| `System.Text.Encoding` インスタンス | `([System.Text.Encoding]::UTF8)` |
| `Resolve-Encoding` の戻り値 | `(Resolve-Encoding .\a.txt)` |

大文字と小文字は区別しません。

`shift-jis` / `sjis` / `ms_kanji` といった別名も使えますが、これは **.NET の変換表にそのまま委譲**しているだけです。モジュール側では別名の表を持っていません。

別名の表を自前で持つと、.NET が持っている表と少しずつずれていきます。持たない設計にしたのは、そのためです。

`ansi` と `oem` は、実行環境の設定値を指します。名前から環境依存であることが分かるので、そこは誤解されないと思います。

### 書き込みでは裸の utf8 を受け付けません

ここが 1.1.0 で唯一、標準コマンドより制約が厳しくなっている点です。

先ほど見たとおり、Windows PowerShell 5.1 と PowerShell 7.x で意味が衝突する名前は、`utf8` ただ一つです。

`unicode` / `utf32` / `bigendianunicode` は、どちらのホストでも BOM 有りで一致しているので、衝突しません。

そこで、書き込み系のコマンドでは裸の `utf8` を受け付けないことにしました。

```
# 書き込みで裸の utf8 を指定すると、エラーになる
Set-ProbedContent .\z.txt -Value 'x' -Encoding utf8
```

実際に表示されるメッセージは以下のとおりです。両ホストで同一の文面になります。

```
書き込みでは 'utf8' が指定できません。UTF-8 は PowerShell 5.1 と 7.x で
BOM の解釈が異なるため、'utf8NoBOM' または 'utf8BOM' のいずれかを明示してください。
```

強調しておきたいのは、**この失敗が起きるタイミング**です。

```
# エラーの後、ファイルが作られていないことを確認する
Test-Path .\z.txt

# 実行結果
False
```

パラメータ束縛の段階で弾いているので、**ファイルを開く前に失敗します。**

書きかけの破損したファイルが残りません。

一方、読み取り側の `Get-ProbedContent` では、裸の `utf8` を許容しています。

読み取りでは BOM の有無が問題にならず、BOM があれば読み飛ばすだけだからです。

この非対称は意図的なものです。

なお、WebName の経路で解決された `"utf-8"` も、書き込みでは同じ扱いになります。この経路では BOM の方針が決まらないためです。

### BOM 無しの UTF-16 / UTF-32 はお勧めしません

`unicodeNoBOM` や `utf32NoBOM` といった語彙は用意していますが、実務でこれを選ぶ理由はほとんどありません。

BOM の無い UTF-16 は、他のツールから確実に判別できないからです。

これらの語彙を用意したのは、`Resolve-Encoding` が BOM 無しの UTF-16 と UTF-32 を検出するためです。

検出できる状態はすべて語彙で指定できなければ、読んで書き戻す往復が破綻します。そのために用意しているものだと考えてください。

**新しくファイルを作るときに、BOM 無しの UTF-16 / UTF-32 を選ぶことはお勧めしません。**

ただし、警告は出しません。`-EncodingFrom` で往復するたびに警告が鳴っては、かえって邪魔になるからです。

## Get-ProbedContent — 判定して読む

文字エンコーディングを自動判定して、テキストファイルを読み込むコマンドです。

```
# 文字エンコーディングを指定せずに読む（自動判定）
Get-ProbedContent .\sjis.txt

# 文字エンコーディングを明示して読む（判定を行わない）
Get-ProbedContent .\sjis.txt -Encoding shift_jis

# ファイル全体を 1 個の文字列として受け取る
$text = Get-ProbedContent .\sjis.txt -Raw

# 先頭 10 行だけ読む
Get-ProbedContent .\big.txt -TotalCount 10
```

`-Encoding` を省略すると対象ファイルを判定します。判定に失敗した場合は、そのファイルについて非終了エラーになります。

`-Encoding` を明示した場合は判定を行いません。判定の誤りを避けたいときの手段として使えます。

パラメータは以下のとおりです。

| パラメータ | 型 | 説明 |
| :---- | :---- | :---- |
| `-Path` | `string[]` | ワイルドカード対応。パイプライン入力に対応 |
| `-LiteralPath` | `string[]` | ワイルドカード非対応 |
| `-Encoding` | 多形 | 省略時は `Auto`（対象ファイルを判定） |
| `-Raw` | switch | 行分割せず、ファイル全体を 1 個の文字列として返す |
| `-TotalCount` | `long` | 先頭 N 行のみ読み込む |
| `-Culture` | `string` | 判定に用いるカルチャー名 |
| `-Strategy` | `string` | 判定方式。`Combined`（既定）/ `NativeOnly` / `UtfUnknownOnly` |

`-Path` がパイプライン入力に対応しているので、以下の書き方が成立します。

```
# フォルダー内の .txt をまとめて読む
Get-ChildItem *.txt | Get-ProbedContent
```

複数のファイルを指定した場合は、標準の `Get-Content` と同様に内容が連結されます。

それぞれのファイルの文字エンコーディングが違っていても構いません。文字列に変換された時点で、元の文字エンコーディングは意味を失うからです。

BOM は、指定した語彙にかかわらず常に読み飛ばします。`-Encoding utf8BOM` で BOM 無しのファイルを読んでも、エラーにはなりません。

### -Raw は情報保全のために必要です

`-Raw` は、単に便利なだけのオプションではありません。

**読んだ内容を加工して書き戻す用途では、`-Raw` が唯一の無損失な読み取り手段になります。**

行分割してしまうと、行末が CRLF だったか LF だったか、末尾に改行があったかどうかが失われます。

失われた情報は、後から復元できません。

### 採用しなかったパラメータ

標準の `Get-Content` にあるパラメータのうち、以下は採用していません。

| パラメータ | 採用しなかった理由 |
| :---- | :---- |
| `-Delimiter` | `-Raw` で読んでから `-split` すれば同じことができ、正規表現が使える分そちらが強力です |
| `-Tail` | 末尾からの逆方向読み取りと文字エンコーディング判定の組み合わせは、実装が重くなります。全行読んでから末尾 N 件を返すなら <code>&#124; Select-Object -Last N</code> と変わりません |
| `-Wait` / `-Stream` / `-ReadCount` | 標準コマンドとの完全互換を目指していないためです |
| FileSystem 以外のプロバイダー | 同上です |

標準コマンドと完全に同じものを作るつもりはありません。

**このコマンド群の目的は、文字エンコーディングの扱いを揃えることであって、`Get-Content` を置き換えることではありません。**

大きなファイルを扱うときのメモリの挙動は、標準の `Get-Content` と揃えてあります。`-Raw` を指定した場合を除き、行単位で出力しています。

## Set-ProbedContent — 明示して書く

文字エンコーディング・BOM・改行コードを明示して書き込むコマンドです。

パラメータは以下のとおりです。

| パラメータ | 型 | 説明 |
| :---- | :---- | :---- |
| `-Path` | `string[]` | ワイルドカード対応。パイプライン入力に対応 |
| `-LiteralPath` | `string[]` | ワイルドカード非対応 |
| `-Value` | `object[]` | 書き込む内容。パイプライン入力に対応 |
| `-Encoding` | 多形 | 省略時は `Auto` |
| `-EncodingFrom` | `string` | 参照ファイルから継承する |
| `-NoNewline` | switch | 要素間と末尾に改行を出力しない |
| `-LineBreak` | `Auto` / `CrLf` / `Lf` / `Cr` | 改行コードの指定 |
| `-Force` | switch | 読み取り専用ファイルへも書き込む |
| `-Culture` | `string` | 判定に用いるカルチャー名 |
| `-Strategy` | `string` | 判定方式 |
| `-WhatIf` / `-Confirm` | — | `SupportsShouldProcess` に対応 |

`-LineBreak` に `Cr`（旧 Macintosh 形式）があるのは、`Resolve-Encoding` が `Cr` を返しうるからです。検出できる状態は、すべて指定できなければなりません。

標準の `Out-File` にある `-NoClobber` は採用していません。標準の `Set-Content` にも存在しないパラメータだからです。

### Shift_JIS を名前で書く

Windows PowerShell 5.1 の標準コマンドには、Shift_JIS を指す `-Encoding` の値がありません。

`Default` や `Oem` は「日本語環境ならたまたま CP932」に過ぎず、環境が変われば別の文字エンコーディングになります。

1.1.0 では、名前で指定できます。

```
# Shift_JIS で、改行を LF にして書き込む
Set-ProbedContent .\sjis.txt -Value '日本語' -Encoding shift_jis -LineBreak Lf
Show-Bytes .\sjis.txt
```

| ホスト | バイト列 |
| :---- | :---- |
| Windows PowerShell 5.1 | `93 FA 96 7B 8C EA 0A` |
| PowerShell 7.x | `93 FA 96 7B 8C EA 0A` |

両ホストで一致します。

`93 FA` が「日」、`96 7B` が「本」、`8C EA` が「語」、末尾の `0A` が LF です。

### 無損失のラウンドトリップ

実務でいちばん効くのが、この使い方だと思います。

「読んで、加工して、元の文字エンコーディング・BOM・改行のまま書き戻す」という操作が、3 行で書けます。

```
# 元のファイルを丸ごと読み、文字列を置換して、同じ性質のまま書き戻す
$text = Get-ProbedContent .\sjis.txt -Raw
$text.Replace('日本語', '日本國語') |
    Set-ProbedContent .\sjis.txt -EncodingFrom .\sjis.txt -NoNewline
```

| | バイト列 |
| :---- | :---- |
| 加工前 | `93 FA 96 7B 8C EA 0A` |
| 加工後 | `93 FA 96 7B 9A A0 8C EA 0A` |

Shift_JIS のまま、改行 `0A`（LF）のまま、BOM 無しのまま、「國」を表す `9A A0` の 2 バイトだけが増えています。

これも両ホストで同じ結果になります。

`-EncodingFrom` は、参照したファイルから **文字エンコーディング・BOM の有無・改行コードの 3 点をすべて継承**します。

ここで一つ注意があります。

同じファイルを `-Raw` 無しで読み書きすると、読み終える前にファイルが切り詰められて、内容が消えます。

モジュールはこれを検出して、非終了エラーにします。

```
# 同じファイルの読み書きは、エラーになる（ファイルは無傷のまま残る）
Get-ProbedContent .\sjis.txt | Set-ProbedContent .\sjis.txt
```

エラー ID は `SamePathRoundTrip` です。上の 3 行の書き方を案内するメッセージが出ます。

`-Raw` は出力する前にファイルを閉じるので、この検査の対象外です。だから上の例のように、読んで加工して書き戻せます。

### -Encoding と -EncodingFrom は同時に指定できません

どちらも文字エンコーディングと BOM を決めようとするので、競合します。

同時に指定すると終了エラーになります。

ただし、`-EncodingFrom` と `-LineBreak` の組み合わせは有効です。

```
# 文字エンコーディングと BOM は a.txt から継承し、改行だけ LF に変える
Set-ProbedContent .\b.txt -Value $text -EncodingFrom .\a.txt -LineBreak Lf
```

**個別の明示指定は、継承より優先されます。** 粒度の細かい方が勝つ、という決め方です。

### -Encoding の入力形式で、改行の決まり方が変わります

ここは、ハマりやすい箇所です。

`-Encoding` に `Resolve-Encoding` の戻り値（`EncodingInformation` 型）を渡した場合だけ、**改行コードも継承されます。**

```
# EncodingInformation を渡すと、改行コードまで引き継がれる
$enc = Resolve-Encoding .\a.txt
Get-ProbedContent .\a.txt -Encoding $enc | Set-ProbedContent .\b.txt -Encoding $enc
```

`EncodingInformation` は `LineBreak` プロパティを持っているからです。

一方、`utf8NoBOM` のような語彙の名前を渡した場合は、改行の情報がどこにもありません。

そのため `-LineBreak` を省略すると、OS の既定に落ちます。

`-LineBreak` を省略したときの決まり方を整理すると、以下のようになります。

| 優先 | 条件 | 結果 |
| :---- | :---- | :---- |
| 1 | 参照情報がある（`-EncodingFrom` / `-Encoding Auto` / `-Encoding <EncodingInformation>`） | 参照元の改行を継承する（OS を問わない） |
| 2 | 参照情報が無い | OS の既定 |

OS の既定は、Windows なら CRLF、Linux と macOS なら LF です。これは `Environment.NewLine` と一致します。

PowerShell 7.x の標準の `Set-Content` も同じ挙動なので、標準コマンドとの一貫性も保たれています。

なお、参照元の改行が CRLF と LF の混在だった場合は、**CR-LF を含むなら CR-LF、含まないなら LF** と決めています。OS に依存しない、決定的な規則にしました。

### -Encoding Auto の意味

`-Encoding` を省略すると `Auto` になります。

`Set-ProbedContent` の `Auto` は、**書き込み先の既存ファイルから継承する**という意味です。

書き込み先が存在しない場合は、継承元が無いので非終了エラーになります。

この既定が適合するのは「上書きするときに、既存ファイルの性質を壊さない」という限定的な用途だけです。

パイプラインで繋いだ場合、読み取り元の性質は引き継がれません。そこは `-EncodingFrom` を使ってください。

### -NoNewline との関係

`-LineBreak` と `-NoNewline` は、別のことを制御しています。

- `-LineBreak` … 改行を出力するとき、**どの文字を使うか**
- `-NoNewline` … **末尾（および要素間）に改行を付けるか**

矛盾はしませんが、同時に指定すると `-LineBreak` の指定が単に使われないだけになります。

意図と違う結果になる可能性があるので、同時指定のときは Warning を出します。エラーにはしません。

`-NoNewline` は、`-Raw` で読んだ内容をそのまま書き戻す往復に不可欠です。

## Add-ProbedContent — 文字エンコーディングを保って追記する

追記のコマンドです。パラメータは `Set-ProbedContent` と同じで、`-AllowEncodingChange` が一つ増えます。

追記は、上書きと違って**既存のバイト列の続きに書く**操作です。

途中で文字エンコーディングが変わると、ファイルの途中から化けます。

### 整合性検査はバイト列で判定します

そこで `Add-ProbedContent` は、追記の可否を検査します。

判定しているのは、文字エンコーディングの**名前の一致ではありません**。

**実際に書き出されるバイト列を比較しています。**

規則は一つだけです。

```
指定された文字エンコーディング X で符号化したバイト列
  == 既存ファイルの文字エンコーディング Y で符号化したバイト列
```

成立すれば、X で追記した結果は Y で追記したのとバイト単位で同一になります。ファイルは Y のまま一貫します。

成立しなければエラーです。

この規則から導かれる結果は、直感的にはややこしく見えるかも知れません。

| 既存ファイル | 追記の指定 | 追記する内容 | 判定 |
| :---- | :---- | :---- | :---- |
| US-ASCII | UTF-8 | 日本語を含む | 拒否 |
| UTF-8 | US-ASCII | ASCII のみ | 許可 |
| UTF-8 | Shift_JIS | ASCII のみ | 許可 |
| UTF-8 | Shift_JIS | 日本語を含む | 拒否 |
| Shift_JIS | UTF-8 | 日本語を含む | 拒否 |
| UTF-8 | UTF-16LE | 何でも | 拒否 |

「UTF-8 のファイルに Shift_JIS で追記できる」という行が意外に見えると思います。

しかし、追記する内容が ASCII の範囲だけなら、UTF-8 で符号化しても Shift_JIS で符号化しても、出てくるバイト列は同じです。ファイルは壊れません。

逆に、既存ファイルが US-ASCII だからといって UTF-8 の日本語を黙って追記すれば、**ファイル全体の文字エンコーディングが UTF-8 に変わります。** それは `Add-` の役割を超えています。

実際に動かすと、以下のようになります。

```
# UTF-8 (BOM 無し) のファイルに、ASCII だけを ascii で追記する → 許可される
Add-ProbedContent .\log.txt -Value 'ABC' -Encoding ascii -LineBreak Lf
Show-Bytes .\log.txt

# 実行結果（元の内容は「日本語」+ LF）
E6 97 A5 E6 9C AC E8 AA 9E 0A 41 42 43 0A

# 日本語を Shift_JIS で追記する → バイト列が変わるので拒否される
Add-ProbedContent .\log.txt -Value 'あ' -Encoding shift_jis
```

後者のエラー ID は `EncodingChangeOnAppend` です。

### -Force では回避できません

意図的に文字エンコーディングを変えたい場合は、`-AllowEncodingChange` を指定します。

```
# ASCII のログファイルを UTF-8 に格上げしながら追記する
Add-ProbedContent .\log.txt -Value $line -Encoding utf8NoBOM -AllowEncodingChange
```

**`-Force` では回避できません。**

`-Force` は「読み取り専用ファイルへ書き込む」という別の意味を、既に持っているからです。

この二つを相乗りさせると、「読み取り専用属性を外したいだけなのに、文字エンコーディングの検査まで無効になっていた」という事故が起きます。

分けたのは意識的な設計判断です。

### 追記では BOM の指定が無視されます

もう一点、`Add-ProbedContent` では **BOM の指定が常に無視されます。**

新規にファイルを作る場合も同様です。

ファイルの途中に BOM を書き込むことは、いかなる場合も正しくないからです。

したがって、`Add-ProbedContent` において `-Encoding utf8BOM` と `-Encoding utf8NoBOM` は、まったく同じ挙動になります。

ここでも警告は出しません。`-EncodingFrom` で BOM 付きのファイルから継承したとき、正当な使い方なのに毎回鳴ってしまうためです。

改行コードについては、不一致でも許可します。

CRLF のファイルに LF を追記すると混在改行になりますが、読めなくなるわけではありません。

`-LineBreak Lf` の明示指定は「今後は LF に統一したい」という正当な意図でありえるので、エラーにはしませんでした。

## ConvertTo-DotNetEncoding — .NET のクラスライブラリへ渡す

`System.Text.Encoding` インスタンスを得るための、専用のコマンドです。

**.NET のクラスライブラリを直接利用する場合にのみ使います。**

`[IO.File]::ReadAllLines` や `WriteAllText`、`StreamWriter`、`XmlWriter`、サードパーティ製のライブラリなどが対象です。

```
# 判定結果を .NET の Encoding インスタンスに変換して、ReadAllLines へ渡す
$enc     = Resolve-Encoding .\file.txt
$encobj  = ConvertTo-DotNetEncoding $enc
$content = [IO.File]::ReadAllLines('C:\work\file.txt', $encobj)

# パイプラインでも書ける
$encobj = Resolve-Encoding .\file.txt | ConvertTo-DotNetEncoding

# 語彙名やコードページからも作れる
$encobj = ConvertTo-DotNetEncoding utf8NoBOM
$encobj = ConvertTo-DotNetEncoding 932
```

受け付ける入力は、Probed 系コマンドの `-Encoding` とまったく同じです。変換のロジックを共有しています。

入力の種別ごとにパラメータを分けてはいません。`-Name` と `-WebName` の境界は、利用者から見て意味がないからです。

### Auto は指定できません

`Auto` を渡すと終了エラーになります。

`Auto` は「ファイルからの検出」を指す語彙であり、ファイルを引数に取らないこのコマンドでは解決できないからです。

エラーメッセージは、正しい書き方へ誘導する内容になっています。

> `Auto` はファイルからの検出を指す語彙のため、`ConvertTo-DotNetEncoding` では指定できません。ファイルから解決するには `Resolve-Encoding <path> | ConvertTo-DotNetEncoding` を使用してください。

「実行環境の既定の文字エンコーディングを返す」という案も検討しましたが、採りませんでした。

Windows PowerShell 5.1 では、既定の文字エンコーディングがコマンドごとに違います。`Get-Content` は ANSI、`Set-Content` は ASCII、`Out-File` は UTF-16LE です。

さらに `$OutputEncoding` や `[Console]::OutputEncoding` という別の軸もあり、「その PowerShell の既定」を一意に書き切れません。

それに、同じスクリプトが 5.1 と 7.x で違うバイト列を出力することになり、**このモジュールの存在意義と逆行します。**

環境依存の値が必要な場合は、`ansi` か `oem` を使ってください。名前から環境依存であることが分かるので、利用者を誤解させません。

## エラーは、ファイルを開く前に失敗させる

終了エラーと非終了エラーは、以下の基準で分けています。

- **終了エラー** … その指定では 1 ファイルも処理できないもの。パラメータの組み合わせの誤りや、パラメータ束縛の段階での失敗が該当します
- **非終了エラー** … 対象ファイルごとの失敗。他のファイルの処理は続行します。`-ErrorAction Stop` を付ければ終了エラーにできます

主なものを表にすると、以下のようになります。

| 事象 | 扱い |
| :---- | :---- |
| 書き込み系での裸の `utf8` の指定 | 終了エラー（パラメータ束縛の段階） |
| 書き込み系での `utf7` の指定 | 終了エラー（パラメータ束縛の段階） |
| BOM 接尾辞を許さない語彙への接尾辞付き指定 | 終了エラー（パラメータ束縛の段階） |
| `-Encoding` と `-EncodingFrom` の同時指定 | 終了エラー |
| `-EncodingFrom` の参照先が存在しない | 終了エラー |
| 解釈できない `-Culture` / `-Strategy` の指定 | 終了エラー（ファイルを開く前） |
| `ConvertTo-DotNetEncoding` への `Auto` の指定 | 終了エラー（誘導メッセージ付き） |
| 読み取り対象のファイルが存在しない | 非終了エラー |
| 文字エンコーディングの判定に失敗した | 非終了エラー |
| 判定できても、実行環境がそのコードページを提供していない | 非終了エラー（`CodePageNotAvailable`） |
| `-Encoding Auto` で書き込み先・追記先が存在しない | 非終了エラー |
| `Add-ProbedContent` でのバイト列の不一致 | 非終了エラー（`-AllowEncodingChange` で回避可） |
| 同一パスの読み書きを 1 つのパイプラインで行った | 非終了エラー |
| 書き込み先が読み取り専用・書き込み権限が無い | 非終了エラー |
| `-LineBreak` と `-NoNewline` の同時指定 | Warning |

**書き込み系で終了エラーになるものは、すべてファイルを開く前に失敗します。**

書きかけの破損したファイルを残さないためです。ここは徹底しました。

読み取り側の「存在しない」「判定失敗」を非終了エラーにしたのは、標準の `Get-Content` の実際の挙動に合わせたためです。

なお `CodePageNotAvailable` は、判定そのものは成功したのに、実行環境の .NET がそのコードページを提供していない場合のエラーです。

ISO-2022-TW（50229）などが該当します。この場合は `-Encoding` での明示指定を案内するメッセージが出ます。

## ヘルプとメッセージは 5 言語に対応しています

判定処理が対象としている言語圏に合わせて、ヘルプとエラーメッセージを揃えてあります。

| 対応 | 言語 |
| :---- | :---- |
| `Get-Help`（MAML ヘルプ） | 英語 / 日本語 / 韓国語 / 繁体字中国語 / 簡体字中国語 |
| エラーメッセージ | 同上 |

1.1.0 では、未対応のカルチャー（`zh-HK` など）では英語になります。

香港・マカオ（`zh-HK` / `zh-MO`）には、1.2.0 で対応しました。1.2.0 では、これらのカルチャーで繁体字中国語のヘルプとエラーメッセージを表示します。

[SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)

一点、注意していただきたいことがあります。

> **`Get-Help` の言語は、`Import-Module` した時点の `CurrentUICulture` で決まります。**
> 読み込んだ後にカルチャーを変えても、切り替わりません。

他の言語のヘルプを確認したい場合は、以下の順序で実行してください。

```
# UI カルチャーを変えるのは、Import-Module の「前」
[System.Threading.Thread]::CurrentThread.CurrentUICulture =
    [System.Globalization.CultureInfo]::GetCultureInfo('ko-KR')
Import-Module SnowStack.EncodingProbe.PowerShell
(Get-Help Get-ProbedContent).Synopsis
```

日本語環境のコンソールでは、韓国語や中国語が `????` のように表示されることがあります。

これはコンソールのコードページにその文字が無いための**表示上の問題**で、ヘルプ自体は正しく切り替わっています。

文字列として確認したい場合は、ファイルへ書き出してエディターで開いてください。

```
# 韓国語のヘルプをファイルへ書き出して確認する
(Get-Help Get-ProbedContent).Synopsis | Set-ProbedContent .\help.txt -Encoding utf8BOM
```

こういう場面で、`Set-ProbedContent` が役に立ちます。

## 必ず読んでいただきたい注意事項

書かないと必ずハマる、という点をまとめました。

| 注意 | 内容 |
| :---- | :---- |
| 書き込みで裸の `utf8` は使えない | `utf8NoBOM` か `utf8BOM` を明示してください |
| `-Encoding` の入力形式で改行の決まり方が変わる | `EncodingInformation` を渡したときだけ改行も継承されます。語彙の名前には改行の情報が無いため、`-LineBreak` 省略時は OS の既定になります |
| BOM 無しの UTF-16 / UTF-32 は非推奨 | 語彙としては用意していますが、他のツールから確実に判別できません。往復のために用意しているものです |
| `Encoding.GetEncoding(65001)` を渡すと BOM が付く | `System.Text.Encoding` インスタンスを直接渡した場合だけ、そのインスタンスの `GetPreamble()` を尊重するためです |
| `.ps1` は UTF-8 (BOM 付き) で保存する | **Windows PowerShell 5.1 は BOM 無しの `.ps1` を ANSI として読みます。** 日本語を含む行がパースエラーになります |
| 判定できても .NET が扱えない文字エンコーディングがある | ISO-2022-TW（50229）などです。`CodePageNotAvailable` の非終了エラーで報告します |

最後から二番目の `.ps1` の話は、この記事のためにコマンドの実行結果を採取している最中に、私自身が実際に踏みました。

コンソールに貼り付けて実行する分には起きません。スクリプトファイルに保存したときだけ起きます。

## なぜこの作りにしたのか

ここからは、使い方ではなく設計の話です。読み飛ばしていただいて構いません。

### 判定は、ファイル全体を対象にしています

文字エンコーディングの判定は、ファイルの先頭だけを見ているわけではありません。

実装の途中で、先頭 1 MiB までという制限を入れていた時期があります。しかし、これは撤廃しました。

**英数字が延々と続いた後に、最後の方でだけマルチバイト文字が現れるソースファイル**を誤判定するからです。

日本語のコメントが 1 行だけ末尾にある、といったファイルは普通にあります。

処理は重くなりますが、正しさを優先しました。

### BOM は GetPreamble() に頼っていません

これは、実装していて一番の落とし穴でした。

`Encoding.GetEncoding("utf-8")` も `Encoding.UTF8` も、**BOM 付きのインスタンス**を返します。

そのため、`GetPreamble()` に素直に従うと、同じ `utf8NoBOM` の指定でも、解決の経路によって BOM が付いたり付かなかったりします。

そこで、BOM を出すかどうかは語彙の側だけで決めることにしました。書き込み処理が自分で BOM を書き出しています。

Unicode 系のインスタンスは、必ずコンストラクタで組み立てています。

```
// BOM 付きインスタンスが返るので、この書き方は使わない
Encoding.GetEncoding("utf-8")
Encoding.UTF8

// BOM の方針を明示して構築する
new UTF8Encoding(encoderShouldEmitUTF8Identifier: false)
new UnicodeEncoding(bigEndian: false, byteOrderMark: false)
```

例外は一つだけで、利用者が `System.Text.Encoding` インスタンスを直接渡した場合です。そのときは、利用者が自分で構築した意図を尊重して `GetPreamble()` に従います。

注意事項の表に書いた `Encoding.GetEncoding(65001)` の話は、この例外に該当します。

### 読み取り時の BOM の混入

`StreamReader` に `detectEncodingFromByteOrderMarks: false` を渡すと、BOM が `U+FEFF` として **1 行目の先頭に混入します。**

判定結果の文字エンコーディングを強制しつつ、BOM は確実に落とす、という処理を自前で持っています。

画面上は見えないので、気付きにくい不具合になります。テストで確実に検証するようにしました。

### CodePagesEncodingProvider の登録

.NET Core 以降では、CP932 などのコードページが標準では取得できません。`CodePagesEncodingProvider` の登録が必要になります。

これをホスト側の登録に依存させると、環境によって動いたり動かなかったりします。

そこで、`Import-Module` の時点でモジュール自身が登録するようにしました。

### 依存ライブラリのバージョンが、ターゲットで非対称です

サードパーティ製の判定ライブラリである UTF.Unknown は、ターゲットフレームワークによってバージョンを分けています。

| ターゲット | UTF.Unknown |
| :---- | :---- |
| .NET 10 | 2.7.0 |
| .NET Framework 4.8 | 2.6.0（据え置き） |

これは揃え忘れではなく、意図的な非対称です。

UTF.Unknown 2.7.0 の `netstandard2.0` 向けアセットは、`System.Memory` に依存するようになりました。

`System.Memory` は厳密名付きのアセンブリで、.NET Framework では参照アセンブリと実行時アセンブリのバージョンが食い違います。

通常は `app.config` のバインディングリダイレクトで解決します。

しかし、**バイナリモジュールを `Import-Module` する Windows PowerShell 5.1 のホストには、`app.config` を差し込めません。**

ホストは `powershell.exe.config` を読むので、こちらの都合を反映させる手段がありません。

.NET Framework 4.8 側では 2.7.0 で追加された API を使っておらず、2.6.0 と 2.7.0 で判定結果も変わりません。

リスクだけを負う変更になるので、据え置きました。

### 検証は、両ホストで突き合わせています

同一のシナリオ集を Windows PowerShell 5.1 と PowerShell 7.x の両方で実行し、結果を突き合わせています。

**「バージョンによらず同じ結果になる」がこのモジュールの存在意義そのもの**なので、そこをテストで固定しました。

テストデータのファイルは、バイト列を明示的に指定して生成しています。

既存の書き込みコードでテストファイルを作ると、バグが自己整合してしまい検出できないからです。

5 言語のヘルプについても、本文を伏せた骨格が 5 言語で完全に一致することを機械的に検証しています。

要素の構成・属性・出現順・コード例まで一致を要求するので、片方の言語にだけパラメータを足すと必ず落ちます。

翻訳を人手で保守すると必ずずれる、という問題への対処です。

## 入手方法

PowerShell ギャラリーに登録しています。

```
# PowerShell 7.x でのインストール
Install-PSResource SnowStack.EncodingProbe.PowerShell
```

Windows PowerShell 5.1 には、`Install-PSResource` コマンドレットが同梱されていません。

そのため、先に `Microsoft.PowerShell.PSResourceGet` モジュールを導入する前作業が必要になります。

その手順は、以下の記事で解説しています。

[SnowStack.EncodingProbe.PowerShell を Windows PowerShell 5.1 へインストールする方法](/encodingprobe_powershell_install_ps51/)

ソースコードは、以下の GitHub リポジトリで公開しています。

[SnowStack.EncodingProbe](https://github.com/motoi-tsushima/SnowStack.EncodingProbe)

## ライセンス

ライセンスは **MIT ライセンス** です。

ただし、依存している UTF.Unknown は MIT ではありません。ここは正確に記載してください。

| 対象 | ライセンス |
| :---- | :---- |
| SnowStack.EncodingProbe 本体 | MIT |
| UTF.Unknown（依存ライブラリ） | **MPL 1.1**（GPL 2.0 以降 / LGPL 2.1 以降 とのトリプルライセンス） |

MPL 1.1 は Mozilla Public License 1.1 の略で、ファイル単位の弱いコピーレフトです。

未改変のライブラリを別アセンブリとして参照する形であれば、呼び出し側に義務は波及しません。そのため SnowStack.EncodingProbe 自体は MIT ライセンスで提供できています。

アプリ開発や商用などに利用する場合は、以下のライセンス表記を、ユーザーが閲覧可能な場所に表記してください。

```
SnowStack.EncodingProbe.PowerShell is licensed under MIT License.
Copyright c 2026 motoi.tsushima
https://github.com/motoi-tsushima/SnowStack.EncodingProbe
https://snow-stack.net/encodingprobe_powershell_guide/

This software includes the following third-party components:

SnowStack.EncodingProbe
Copyright c 2026 motoi.tsushima
Licensed under MIT License
https://github.com/motoi-tsushima/SnowStack.EncodingProbe
https://snow-stack.net/encodingprobe_guide/

UTF.Unknown 2.7.0 (net10.0 build) / 2.6.0 (net48 build)
Copyright (c) 2018 Nikolay Pultsin
Licensed under MPL 1.1 / GPL 2.0 or later / LGPL 2.1 or later
(This software uses UTF.Unknown under the terms of MPL 1.1)
https://github.com/CharsetDetector/UTF-unknown
https://www.mozilla.org/MPL/1.1/
```

同じ文面は、コマンドからも取得できます。

```
# ライセンス情報を表示する
Resolve-Encoding -License
```

## 1.1.0 でも解決していないこと

正直に書いておきます。

ISO-2022 系の判定には、まだ問題が残っています。

| 現象 | 状態 |
| :---- | :---- |
| ISO-2022-TW が ISO-2022-CN と誤判定される | 未対応 |
| SO/SI 形式の 1 バイトカナを検出できない | 未対応 |
| 判定できても .NET が扱えないコードページがある | 1.1.0 で非終了エラーとして報告する対応を実施 |

いずれも判定エンジン側の課題です。1.2.0 以降で扱う予定でしたが、1.2.0 では対応していません（1.2.0 では、世界の言語と香港の Big5 への対応を優先しました）。

該当する文字エンコーディングを確実に扱いたい場合は、`-Encoding` で明示的に指定してください。判定を行わないので、この問題を回避できます。

## コマンド早見表

最後に、この記事で解説した内容を 1 枚にまとめます。

| やりたいこと | 書き方 |
| :---- | :---- |
| 文字エンコーディングを判定して読む | `Get-ProbedContent .\a.txt` |
| ファイル全体を 1 個の文字列で読む | `Get-ProbedContent .\a.txt -Raw` |
| BOM 無しの UTF-8 で書く | `Set-ProbedContent .\a.txt -Value $v -Encoding utf8NoBOM` |
| Shift_JIS で書く | `Set-ProbedContent .\a.txt -Value $v -Encoding shift_jis` |
| 改行を LF にする | `-LineBreak Lf` を付ける |
| 元のファイルと同じ性質で書き戻す | `-EncodingFrom .\元ファイル -NoNewline` |
| 文字エンコーディングを保って追記する | `Add-ProbedContent .\a.txt -Value $v` |
| 追記しつつ文字エンコーディングを変える | `-AllowEncodingChange` を付ける |
| .NET の Encoding インスタンスを得る | `ConvertTo-DotNetEncoding utf8NoBOM` |
| 外国語のファイルを読む | `-Culture ko-KR` などを付ける |
| 欧米のテキストを日本語環境で読む | `-Strategy UtfUnknownOnly` を付ける |

以上、SnowStack.EncodingProbe.PowerShell 1.1.0 で追加した 4 つのコマンドの解説でした。

最後の 2 行、`-Culture` と `-Strategy` については、別の記事で詳しく解説しています。

[-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)

Windows PowerShell 5.1 と PowerShell 7.x を両方使う環境で、テキストファイルの文字エンコーディングに悩まされている方は、試してみてください。

## お知らせ欄

### 2026年9月1日　Version 1.1.0 リリース

Version 1.1.0 をリリースしました。

`Get-ProbedContent` / `Set-ProbedContent` / `Add-ProbedContent` / `ConvertTo-DotNetEncoding` の 4 コマンドを追加しています。

既存の `Resolve-Encoding` と `Get-EncodingProbePlatformInfo` は変更していないので、1.0.x 向けに書いたスクリプトはそのまま動きます。

## 関連資料

- [SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)
- [SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)
- [SnowStack.EncodingProbe NuGet Package 解説](/encodingprobe_guide/)
- [-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)
- [PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか （フレンドリ名の解説）](/why-utf8nobom-cannot-be-webname/)
- [SnowStack.EncodingProbe.PowerShell を Windows PowerShell 5.1 へインストールする方法](/encodingprobe_powershell_install_ps51/)
- [SnowStack.EncodingProbe（GitHub リポジトリ）](https://github.com/motoi-tsushima/SnowStack.EncodingProbe)
