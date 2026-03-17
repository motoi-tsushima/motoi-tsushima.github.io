---
title: "PowerShellの文字エンコーディング問題に終止符を打つ：BOM・改行コードを自在に扱う方法"
categories:
  - 文字エンコーディング
tags:
  - Character Encoding
  - 改行コード
  - BOM
  - mfprobe
  - Windows ツール
---

PowerShell 上での文字エンコーディングの変換方法を解説します。  
これから文字エンコーディング、BOM、改行コードの3点について解説します。

以前から解説していますように、PowerShell 及び cmd では、現状のテキストファイルの文字エンコーディング・BOMの有無・改行コードの種類を確認する機能は提供されていません。  
PowerShellのスクリプトを作成すればある程度確認することはできますが、必要になる度にスクリプトを作成するのは、実用的ではありません。

この記事では、PowerShell 上での文字エンコーディング変換、BOMの変換、改行コードの変換方法の具体例を示しつつ、その問題を解説したいと思います。

コマンド操作例の中に登場する `$PWD` とは、PowerShell のカレントディレクトリを示す変数です。  
コマンド操作例の中には、.NETクラスのメソッドを呼び出す物も含まれています。  
PowerShellのカレントディレクトリと、.NETクラスのカレントディレクトリは異なるディレクトリである場合が多いので、コマンド中でカレントディレクトリを指定する場合は `$PWD` を使用するとどちらも同じディレクトリを指定できるので安全に操作できます。

コマンド操作例は、Windows PowerShell 5.1 でも PowerShell 7.x でも使い方に違いはありません。

## 文字エンコーディングの確認

PowerShell と cmd では、既存のテキストファイルの文字エンコーディングを確認する機能は提供されていません。  
しかし、PowerShell スクリプトにより、テキストファイルの先頭を4バイト読み込み、BOMのバイナリパターンを読むことは可能です。

```
$bytes = [System.IO.File]::ReadAllBytes("$PWD\text1.txt")
$bytes[0..3] | ForEach-Object { "{0:X2}" -f $_ }

# 実行結果(UTF-8 BOMあり)
EF
BB
BF
E3
```

この方法だと、BOMのある Unicode テキストならば文字エンコーディングを判別することが可能ですが、UTF-8などBOMの無い Unicode テキストや SHIFT\_JIS テキストの場合は、文字エンコーディングを判別できません。  
Windows環境では、SHIFT\_JIS、 UTF-8(BOM無し)、UTF-8(BOM有り)、UTF-16LE(BOM有り) が共存していますので、これでは実用になりません。

次は、文字エンコーディングの変換方法の実例です。

### 通常のコマンドレットを使用する

テキストファイルを入出力するコマンドレットには標準で `-Encoding` オプションが存在します。  
 `-Encoding` オプションにより、対象テキストファイルの文字エンコーディングを指定することができるので、複数のコマンドレットを組み合わせることで、文字エンコーディングの変換を行うことができます。  
以下は、SHIFT\_JIS から UTF-8 へ変換する実例です。  
Get-Content で SHIFT\_JIS テキストを読み込み、オブジェクトパイプラインで Set-Content に送り、Set-Content で UTF-8 の文字エンコーディングを指定して、新規テキストファイルに書き込みます。  
UTF-8のBOM無しと、UTF-8のBOM有りの二例を示します。

```
# Shift_JIS -> UTF-8 (BOM 無し)
Get-Content -Encoding SJIS "$PWD\text1.txt" | Set-Content -Encoding UTF8 "$PWD\text1set.txt"

# Shift_JIS -> UTF-8 (BOM あり)
Get-Content -Encoding SJIS "$PWD\text1.txt" | Set-Content -Encoding UTF8BOM "$PWD\text1setb.txt"
```

このやり方は、あらかじめ読み取りテキストファイルの文字エンコーディングが分かっていないと使用できません。

また、対象テキストファイルに上書きすることは不可能です。  
必ず別のテキストファイルに出力することになります。

そのため、ワイルドカードを使用して複数ファイルを一括で処理することもできません。  
もし、複数ファイルを一括で処理する必要があるのなら、スクリプトを作成してループ処理を作成する必要があります。

手軽にできるものではありません。

### .NETクラスを使用する

PowerShell は .NETクラスのメソッドを直接使用することができます。  
ここでは、.NETクラスのメソッドを使用したやり方の実例を示します。

```
# Shift_JIS → UTF-8 (BOM なし)
$sjis    = [System.Text.Encoding]::GetEncoding("shift_jis")
$utf8nobom = New-Object System.Text.UTF8Encoding($false)
$content = [System.IO.File]::ReadAllText("$PWD\text3.txt", $sjis)
[System.IO.File]::WriteAllText("$PWD\output3.txt", $content, $utf8nobom)

# UTF-8 → Shift_JIS
$sjis = [System.Text.Encoding]::GetEncoding("shift_jis")
$utf8 = [System.Text.Encoding]::UTF8
$content = [System.IO.File]::ReadAllText("$PWD\text2.txt", $utf8)
[System.IO.File]::WriteAllText("$PWD\output2.txt", $content, $sjis)

# Shift_JIS → UTF-8 (BOM 有り)
$sjis    = [System.Text.Encoding]::GetEncoding("shift_jis")
$utf8bom = New-Object System.Text.UTF8Encoding($true)  # BOM あり
$content = [System.IO.File]::ReadAllText("$PWD\text3.txt", $sjis)
[System.IO.File]::WriteAllText("$PWD\output3b.txt", $content, $utf8bom)
```

スクリプトを作成する場合は、このやり方が参考になる場合もありますが、通常の使用では先のコマンドレットを使用したやり方の方が、手軽で良いでしょう。

## BOMの確認方法

文字エンコーディングの確認機能は提供されていないので、BOMの確認機能も存在しません。  
しかし、PowerShellはスクリプトから .NETクラスを直接使用できるので、BOMを確認するスクリプトを作成することはできます。

```
# BOM の確認
function Test-BOM {
    param([string]$Path)
    $bytes = [System.IO.File]::ReadAllBytes($Path)
    if ($bytes[0] -eq 0xEF -and $bytes[1] -eq 0xBB -and $bytes[2] -eq 0xBF) {
        return "UTF-8 with BOM"
    } elseif ($bytes[0] -eq 0xFF -and $bytes[1] -eq 0xFE) {
        return "UTF-16 LE with BOM"
    } elseif ($bytes[0] -eq 0xFE -and $bytes[1] -eq 0xFF) {
        return "UTF-16 BE with BOM"
    } else {
        return "BOM なし"
    }
}
Test-BOM "$PWD\text2.txt"
```

ご覧の通り、PowerShellスクリプトでBOMの確認は可能ですが、手軽とはいえません。

## BOMを付ける

対象テキストの文字エンコーディングが分かっているのなら、BOMを付けるのは、コマンドレットで比較的簡単にできます。

```
Get-Content  -Encoding UTF8 "$PWD\text1.txt" | Set-Content -Encoding UTF8BOM "$PWD\text1set.txt"
```

以下は .NETクラスを直接使用した実例です。

```
# BOM を付ける（UTF-8 BOM なし → BOM あり）
$utf8nobom = New-Object System.Text.UTF8Encoding($false)
$utf8bom   = New-Object System.Text.UTF8Encoding($true)
$content   = [System.IO.File]::ReadAllText("$PWD\text2.txt", $utf8nobom)
[System.IO.File]::WriteAllText("$PWD\text2out.txt", $content, $utf8bom)

# UTF-16 LE（BOM あり）で保存
$utf16le = [System.Text.Encoding]::Unicode   # = UTF-16 LE with BOM
$content = [System.IO.File]::ReadAllText("$PWD\text1.txt", [System.Text.Encoding]::UTF8)
[System.IO.File]::WriteAllText("$PWD\text1out16.txt", $content, $utf16le)
```

## BOMを削除する

BOMを付けるのも、BOMを削除するのも、コマンドレットの使い方は同じです。

```
Get-Content  -Encoding UTF8BOM "$PWD\text1.txt" | Set-Content -Encoding UTF8 "$PWD\text1set.txt"
```

.NETクラスを使用したやり方は、BOMのバイナリパターンを直接指定して削除し、BOMより後ろの全バイナリを、バイナリファイルとして書き込むやり方になります。

```
# BOM を除去（UTF-8 BOM あり → BOM なし）
$bytes = [System.IO.File]::ReadAllBytes("$PWD\text1.txt")
if ($bytes[0] -eq 0xEF -and $bytes[1] -eq 0xBB -and $bytes[2] -eq 0xBF) {
    $bytes = $bytes[3..($bytes.Length - 1)]
}
[System.IO.File]::WriteAllBytes("$PWD\text1out.txt", $bytes)
```

ご覧の通り、.NETクラスではバイナリ操作が必要になり、手軽とはいえません。

## 改行コードの確認と変換

PowerShell と cmd は、改行コードについては、一貫して CR-LF を使用します。  
よって、改行コードを確認したり変換する機能はどこにも存在しません。  
.NETクラスを直接使用しスクリプトを作成して確認するしかありません。

```
# 改行コードの確認
$bytes = [System.IO.File]::ReadAllBytes("$PWD\text1.txt")
$hasCR = $bytes -contains 0x0D
$hasLF = $bytes -contains 0x0A

if ($hasCR -and $hasLF) { "CRLF (Windows)" 
} elseif ($hasCR)          { "CR のみ (Classic Mac)" 
} elseif ($hasLF)          { "LF のみ (Unix/Linux)" 
} else                     { "改行コードなし" }
```

改行コードの変換も、同様に.NETクラスを直接使用したスクリプトで行う必要があります。

```
# CRLF → LF
$enc     = [System.Text.Encoding]::UTF8
$content = [System.IO.File]::ReadAllText("$PWD\text1.txt", $enc)
$content = $content -replace "`r`n", "`n"
# WriteAllText は内部で環境依存の改行を追加しないよう、WriteAllBytes を使う
$outBytes = $enc.GetBytes($content)
[System.IO.File]::WriteAllBytes("$PWD\text1lf.txt", $outBytes)
```

```
# LF → CRLF
$enc     = [System.Text.Encoding]::UTF8
$content = [System.IO.File]::ReadAllText("$PWD\text1.txt", $enc)
# 既に CRLF の箇所を二重変換しないよう、まず LF のみを対象にする
$content = $content -replace "(?<!\r)`n", "`r`n"
$outBytes = $enc.GetBytes($content)
[System.IO.File]::WriteAllBytes("$PWD\text1crlf.txt", $outBytes)
```

## PowerShell標準機能の問題

PowerShell と cmd の環境では、SHIFT\_JIS・UTF-8のBOM無しとBOM有り・UTF-16LEのBOM有りの四種類の文字エンコーディングテキストが、共存しています。  
UTF-8のBOM無しとBOM有りが両方存在しているため、BOMによる文字エンコーディングの識別が意味を持ちません。  
よって、ユーザーはBOMに頼らない文字エンコーディングの識別機能を必要とするのですが、そのような機能は標準では提供されておりません。

文字エンコーディングとBOMの変換機能は、標準コマンドレットの `-Encoding` オプションによって提供されています。  
しかし、テキストファイルの読み取りと、書き込みのコマンドが別々に提供されており、両者をパイプで繋いで利用する仕様なので、ワイルドカードなどによる複数一括処理ができません。  
また、対象ファイルに上書きすることもできないので、一度別のテキストファイルに出力して、元ファイルと入れ替える手間が掛かります。

改行コードについては、一貫して CR-LF を使用しているため、変換機能は存在しません。  
PowerShellスクリプトでバイナリレベルの操作編集を行う必要があります。

よって、PowerShellの標準機能だけでは、UNIX環境とWindows環境の間でのテキストファイルの交換に対応するのは、難しいのが実情です。

Windowsの文字エンコーディング問題の解説は、過去に書いておりますので、詳しく知りたい方は以下の記事をご覧下さい。

[Windows 固有の文字エンコーディング問題](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/02/06/Windows-%E5%9B%BA%E6%9C%89%E3%81%AE%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E5%95%8F%E9%A1%8C.html)

[Windows 11 標準アプリ 文字エンコーディング対応状況まとめ](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/02/14/encoding-report.html)

[Windowsコンソールの文字エンコーディング対応状況を検証する](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/02/16/windows-console-encoding-report.html)

[Windows 11 環境でのテキストファイル新規作成時のデフォルトエンコーディング検証の総まとめ](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/02/18/windows11-encoding-blog.html)

## 一般的な問題の対策

PowerShellの標準機能だけでは、対策として不十分なので、一般的にはUNIXツールをWindows環境に導入して、対策します。  
UNIXツールについては、以下の記事で詳しく解説しています。

[Windowsで文字エンコーディング・改行コードを扱う「UNIXツール」の現実と課題](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/03/06/Windows%E3%81%A7%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0-%E6%94%B9%E8%A1%8C%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E6%89%B1%E3%81%86-UNIX%E3%83%84%E3%83%BC%E3%83%AB-%E3%81%AE%E7%8F%BE%E5%AE%9F%E3%81%A8%E8%AA%B2%E9%A1%8C.html)

## mfsr と mfprobe による対策

私は、このような標準機能の限界を克服するため、文字エンコーディング・BOM・改行コードを一括で検知・変換できる mfsr と mfprobe という対のツールを提供しています。

そのマニュアルは以下になります。

[rmsmf-txprobe & mfsr-mfprobe 使い方の分かりやすい解説](https://snow-stack.net/tool_rmsmf_guide/)

ここで、mfsr と mfprobe を使用した、文字エンコーディングの変換・BOMの追加と削除・改行コードの確認と変換の方法を、解説したいと思います。  
mfsr と mfprobe は、Window PowerShell 5.1 でも PowerShell 7.x でも cmd でも、同じように使用できます。  
PowerShell のオブジェクトパイプラインに依存しない仕様になっているので、PowerShell以外のシェルでも使用できます。

### mfprobe による確認（文字エンコーディングとBOMと改行コード）

mfprobe は対象テキストファイルの文字エンコーディングとBOMの有無と改行コードの種類を探索して一度に表示します。  
ファイル名にはワイルドカードが使用でき、複数テキストファイルの確認が一度にできます。  
また、その結果をテキストファイルへ出力できます。  
オブジェクトパイプラインを経由しないので、cmd や Git Bash などでも使用可能です。

```
# 通常の使い方
mfprobe text1.txt

# 複数ファイルの一括確認
mfprobe *.txt

# サブディレクトリ配下ファイルの一括確認
mfprobe -d *.txt

# 確認結果のファイルへの出力
mfprobe -d *.txt -o:result.txt
```

注意事項として、Git Bash などUNIXシェルで使用する場合は、ワイルドカードをダブルクォーテーションで囲んで、UNIXシェルがワイルドカード展開をしないようにする必要があります。  
この点は mfprobe と mfsr で、同様です。

```shell
# bash で使用するとき
mfprobe "*.txt"
```

### mfsr による文字エンコーディングの変換

文字エンコーディングを変換するときは、`mfprobe` ではなく `mfsr` を使用します。  
mfprobe は読み取り専用コマンドであり、更新する機能は `mfsr` に集約しています。  
テキストファイルの文字エンコーディングなどを確認するだけなら、誤って更新することを防ぐために、読み取り専用コマンドと更新コマンドを分けて開発しています。  
書き込み先文字エンコーディングは `/w:` オプションを使用して変換先の文字エンコーディングを指定します。  
オプションの頭文字は `/` (スラッシュ)でも `-` (ハイフン)でも、どちらでも良いです。  
`/w:` の `:` (コロン)はオプションのパラメータの指定子です。`:` の前後にはスペースを空けないように注意してください。  
 `:` (コロン)は、`=` (イコール)でも代用できます。
オプションの位置はコマンド名より後ろであれば順番は自由です。

```
# 通常の使い方
mfsr text1.txt /w:utf-8

# 読み取り文字エンコーディングの指定もする場合
mfsr /c:shift_jis text1.txt /w:utf-8

# 複数ファイルを一括変換
mfsr *.txt /w:utf-8

# : は = でも代用可能
mfsr *.txt /w=utf-8
```

### BOMの変換方法

BOMについても、確認は `mfprobe` を、変換更新は `mfsr` を使用します。

#### BOMの確認

mfprobe は、ファイル名と文字エンコーディング名と改行コードの種類とBOMの有無を一度に表示するので、それらの確認は全て以下の様に使用して確認します。

```
mfprobe text1.txt

mfprobe *.txt
```

#### BOMの追加

BOMの更新には、`/b:` オプションを使用します。  
`/b:true` で「BOMを追加する」、`/b:false` で「BOMを削除する」と動作更新します。  
`/b:` オプションが無ければ現状維持になります。

```
# text1.txt にBOMを追加する
mfsr text1.txt /b:true

# 複数ファイルに対して一括でBOMを追加する
mfsr *.txt /b:true
```

#### BOMの削除

BOMの削除も「BOMの追加」の説明の通りに使用します。

```
# text1.txt のBOMを削除する
mfsr text1.txt /b:false

# 複数ファイル一括でBOMを削除する
mfsr *.txt /b:false
```

BOMの操作は、Unicode に対してのみ有効な操作で、Shift\_JIS などBOMが存在しない文字エンコーディングの場合は、`/b:` オプションが無視されます。

### 改行コードの変換方法

改行コードについても、確認は `mfprobe` を、変換更新は `mfsr` を使用します。

#### 改行コードの確認

mfprobe は、ファイル名と文字エンコーディング名と改行コードの種類とBOMの有無を一度に表示するので、それらの確認は全て以下の様に使用して確認します。

```
# 改行コードなど確認する
mfprobe text1.txt 

# サブディレクトリ配下も含めて複数ファイルを一括で確認する
mfprobe *.txt /d
```

#### 改行コードの変換

改行コードの変換更新には、`/nl:` オプションを使用します。  
`/nl:crlf` で「Windows型改行コード」、`/nl:lf` で「UNIX型改行コード」に変換し更新します。  
`/nl:` オプションが無ければ、改行コードを変更しません。

```
# UNIX改行形式に変換する
mfsr text1.txt /nl:lf

mfsr text1.txt /nl:unix

# Windows改行形式に変換する(複数一括変換)
mfsr *.txt /nl:crlf

mfsr *.txt /nl:win
```

「Windows型改行コード」は、`/nl:crlf` 以外にも `/nl:win` や `/nl:w` も使用できます。  
「UNIX型改行コード」は、`/nl:lf` 以外にも `/nl:unix` や `/nl:u` も使用できます。

ちなみに `/nl` は `New Line` の略です。

## 文字エンコーディング問題は、まだまだ終わらない

古い文字エンコーディング問題は、Windowsだけの問題ではなく、日本社会全般に横たわる問題です。  
特に歴史的経緯でWindowsに多くの問題が現れているだけであり、Windowsだけの問題ではないです。  
この辺の話は、以下の記事で解説しました。

[「文字化け」はなぜ終わらない？ 日本のITを縛る文字エンコーディングの深い闇](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/01/12/%E6%96%87%E5%AD%97%E5%8C%96%E3%81%91-%E3%81%AF%E3%81%AA%E3%81%9C%E7%B5%82%E3%82%8F%E3%82%89%E3%81%AA%E3%81%84-%E6%97%A5%E6%9C%AC%E3%81%AEIT%E3%82%92%E7%B8%9B%E3%82%8B%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E3%81%AE%E6%B7%B1%E3%81%84%E9%97%87.html)

古い文字エンコーディング問題が一番大きな問題として現れているのは、Windowsよりも自治体情報システムの「外字」の問題でしょう。  
他にも医療系システムや金融機関のレガシーシステムなども問題です。  
いずれも、古いデータがペタバイト級のデータ量に及んでおり、コンバートもままならない状況と聞きます。

長期的には、UNIX標準に全てのテキスト規格が集約されていくことは、ほぼ確実の状勢なので、Windows環境の膨大なテキストのレガシーデータ問題は、これからも課題として存続し続けるでしょう。  
今後は、新たに改行コードが問題として現れてきそうな気がしています。  
