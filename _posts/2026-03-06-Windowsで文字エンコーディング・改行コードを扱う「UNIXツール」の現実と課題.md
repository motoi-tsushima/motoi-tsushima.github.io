---
title: "Windowsで文字エンコーディング・改行コードを扱う「UNIXツール」の現実と課題"
categories:
  - 文字エンコーディング
tags:
  - Character Encoding
  - 改行コード
  - BOM
  - file
  - iconv
  - nkf
  - dos2unix
  - unix2dos
  - WSL
  - Git Bash
  - Windows ツール
---

前回の記事で、触れた「Windows 固有の文字エンコーディング問題」には、以下の問題があります。

「古い文字エンコーディングが少なくとも Shift\_JIS, UTF-8,UTF-16LEの三つがあり確実に識別する手段は提供されていない」  
「UTF-8を使用する場合、BOM付きとBOM無しの両方が共存しており、その識別はユーザーの自己責任になっている」  
「WSLや Git などでUNIX環境とテキストファイルの相互交換する場合、CR・LF と LF の改行コード仕様の差異に手動で対処しなければならない」  
「これらの問題を解決するのに良いツールとして `file , iconv , nkf` というコマンド製品がUNIX環境で提供されている」  
「PowerShell や cmd では、文字エンコーディング・BOM・改行コードを確認する標準機能は提供されていない」

以上が、前回の記事で解説した内容の概要です。

この記事では、Windows 上で使用できる文字エンコーディングを確認したり、変換したりできる各種のツールについて、ここで解説したいと思います。  
もちろん、BOMや改行コードの確認・変換も含めて解説します。

前回の記事　:　[Windows 固有の文字エンコーディング問題](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/02/06/Windows-%E5%9B%BA%E6%9C%89%E3%81%AE%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E5%95%8F%E9%A1%8C.html)

## 問題の概要

タイトルでは「Windows で使える...」と書いていますが、現実のところ文字エンコーディングとBOMと改行コードの確認と変換を行う標準的ツールは、UNIX環境で開発提供されており、それらをWindowsで使用するには、Windows環境にUNIXのシェルを導入しなければ使用する事ができません。  
タイトルの「...その問題について」とは、UNIXシェルでしか標準的ツールが使用できないことを示しています。  
PowerShell でも文字エンコーディングの変換とBOMの変換は可能ですが、改行コードの確認・変換や既存のテキストファイルの、文字エンコーディングとBOMの確認はできません。  
前回の記事「Windows 固有の文字エンコーディング問題」でも解説したように、Windows環境におけるテキストファイルの主な問題は、「BOM付きUTF-8」と「BOM無しUTF-8」と「Shift\_JIS」のテキストファイルが混在していて、BOMだけではそれらを識別できない点にあります。  
そしてそれらを識別できるツールがUNIX環境でしか提供されていないのが、問題なのです。

この記事では、Windows環境で使用できる文字エンコーディング・BOM・改行コードの確認・変換ができる標準的ツール製品の紹介とその問題点を解説します。

PowerShellを説明すると長くなるので、別の記事で見送りたいと思います。

## UNIX環境の主要コマンドツール

テキストファイルが使用している文字エンコーディングとBOMと改行コードの確認を行う標準的コマンドツールは以下の物になります。

| ツール | 主な用途 | 日本国内での普及 | 海外での普及 |
| :---- | :---- | :---- | :---- |
| iconv | エンコーディング変換 | ◎ | ◎（標準） |
| file | エンコーディング判別 | ◎ | ◎（標準） |
| dos2unix / unix2dos | 改行コード変換 | ○ | ◎（標準的） |
| nkf | 変換・判別・改行・BOM | ◎（日本のみ） | △（ほぼ使われない） |

nkf（Network Kanji Filter）は日本語処理のために日本で開発されたツールで、海外ではほとんど使われません。海外では iconv \+ dos2unix の組み合わせが標準です。

世界的な意味での標準コマンドは、iconv , file , dos2unix , unix2dos です。  
nkf は日本で開発された日本語文字エンコーディングに特化したコマンドで、日本語以外では使用できません。その代わり日本語文字エンコーディングに関しては最も信頼できるツールと言えるでしょう。

世界的標準コマンドの iconv , file , dos2unix , unix2dos は、bash や zsh などのUNIXシェル環境でしか使用できません。  
PowerShell や cmd などの Windowsシェル環境では使用できません。

`nkf`コマンドもオリジナルはUNIXシェル環境（bashやzsh）向けに開発されていますが、`nkf`に限りWindows環境に移植されたクローンが存在し、一応 Windowsシェル環境でも使用することができます。  
しかし、nkf のオリジナルコマンドが UNIXシェル環境で開発されているため、その仕様は UNIXシェル環境の仕様に最適化しています。当然のことです。  
よって、nkf を PowerShell や cmd などの Windows環境で使用すると、使えない機能がいくつかあり、現実的には nkf の真価を発揮できないのが現実です。  
これは nkf コマンドが悪いわけではなく、「Windowsに最適化したコマンドが存在しない」事が問題なのです。

次に、詳細を説明します。

## Windows環境にUNIXシェルを導入する

Windows環境でUNIXシェルを使用する方法は、いくつか存在します。

### WSL（Windows Subsystem for Linux）

最も正攻法と言える使い方は、WSLを使用する方法です。  
WSLとは、Windowsが内部的に持っている Hyper-V の機構を使用して、LinuxカーネルをWindows上でエミュレーション実行して、Linuxの機能を丸ごと使用できる機能です。  
プログラマーが行うLinux上での開発作業のほとんどを、このWSLで実行可能です。  
WSL では Ubuntu、Debian、Kaliなど、いくつかのLInuxディストリビューションがアプリとして用意されており、Microsoft Store などから導入できるようになっています。

具体的な機能と使い方は、Microsoft 公式サイトで確認する事をお勧めします。

[Windows Subsystem for Linux のドキュメント](https://learn.microsoft.com/ja-jp/windows/wsl/)

仮に WSL の Ubuntu を導入して、bash を使用した場合、そのストレージ領域は 仮想ディスク領域(VHD)が割り当てられますが、WSL・UbuntuからWindowsのストレージ領域にアクセスすることも可能です。  
具体的には、`/mnt/c/` ディレクトリが、Windowsでの `C:\\` フォルダを意味します。  
bash上で `cd /mnt/c` と実行すれば、Windowsの `C:\\` フォルダの内容を閲覧・操作できます。  
この仕組みにより、WSL・Ubuntuから全ての Windowsのストレージ領域にアクセスできます。  
この bash から iconv , file , dos2unix , unix2dos コマンドを使用して、 Windowsのテキストファイルの文字エンコーディングやBOMや改行コードの確認・変換を行うことができます。

### MSYS2

Windows環境でUNIXライクなシェルを提供する環境です。  
Bashやcoreutils（ls, grep, sed, awkなど）を提供します。  
Arch Linuxのpacmanパッケージマネージャーを採用しており、ツールのインストール・管理が容易にできます。  
MinGW-w64ツールチェーン（GCC, Clangなど）を簡単に導入可能です。  
MSYS2は、OS自体をUNIX化するものではありません。提供しているのはあくまでUNIXの『ツール群』と『シェル環境』であり、一部のPOSIX APIをエミュレーションしてWindows APIの呼び出しに変換します。  
MinGWでビルドしたバイナリはWindows APIを直接呼び出すネイティブアプリになります。よって、MSYS2上からWindows用に作られたコマンドを呼び出すことも可能ですが、UNIXシェル環境に最適化していなければ正常に動作しません。  
dir や find などは動作します。VS Code が Windowsにインストールされていれば、code と打ち込むと VS Code が起動します。

以下のサイトから配布されています。

[https://www.msys2.org/](https://www.msys2.org/)

詳しい使い方は、ここでは省略します。

### Git Bash

Git Bashは単独の製品ではなく、**Git for Windows**に含まれるコンポーネントです。配布元は以下の2つです。  
Git公式サイト: [https://git-scm.com/install/windows](https://git-scm.com/install/windows)  
Git for Windowsプロジェクト: [https://gitforwindows.org/](https://gitforwindows.org/)  
どちらからダウンロードしても同じインストーラーが取得できます。

Git for Windowsは、Git本体に加えて、コマンドラインで使うためのBASHエミュレーション（Git Bash）と、GUIツール（Git GUI）を提供します。

Git for Windowsは軽量なネイティブツールセットとして、GitのフルセットをWindows上で提供するものです。  
**内部的にはMSYS2のランタイムを使って**、Bash、ls、grep、sedなどのUNIXコマンドをWindows上で動かしています。  
つまりGit Bashは**「MSYS2ベースのBash環境 ＋ Git」**というパッケージです。

**もし、WSL を使用しないのなら、Windows上でUNIXシェルを使用するのに、最も相応しいのはこの Git Bash でしょう。**

[https://git-scm.com/download/win](https://git-scm.com/download/win) にアクセスすると、自動的にダウンロードが開始されます。x64版とARM64版があるので、お使いの環境に合ったものを選びます（通常はx64）。

### Cygwin

CygwinはRed Hat（現IBM）が中心となって開発してきたプロジェクトで、cygwin1.dllというPOSIX互換レイヤーを提供します。  
POSIX APIをWindows API上にエミュレーションするため、UNIX向けソースコードをほぼそのままコンパイル・実行できます。  
POSIX互換環境を構築しているので、Bash, X Window System, SSH, Python, Perl, gccなど非常に多くのパッケージが利用可能です。

ただし、Cygwin上でビルドしたバイナリはcygwin1.dllに依存するため、ネイティブWindowsアプリとしては配布しにくいです。

MSYS2では、ビルドしたアプリはWindowsアプリですが、CygwinではCygwinに依存したアプリになります。

UNIXエミュレーションの抽象化水準を比較すると、WSLが最も高く、次が Cygwin で、MSYS2 はシェルインターフェイスだけUNIXライクで中身はWindowsという抽象化水準になります。  
（WSL は完全に GNU Linux そのものです）

Cygwin の公式サイトは [**https://cygwin.com/**](https://cygwin.com/) です。ここからセットアッププログラムをダウンロードします。  
公式サイトから setup-x86\_64.exe をダウンロードして実行します。64bit版のWindowsであればこちらを使います。

詳しい使い方は、ここでは省略します。

### WindowsのUNIXシェル補足

UNIXシェル環境は、PowerShell や cmd とは使い方が異なります。  
簡単には説明できませんが、例えば ファイルパスの区切り文字は、PowerShell や cmd だとバックスラッシュですが、UNIXシェルではスラッシュです。  
PowerShell はオブジェクトパイプラインを実装していますが、UNIXシェルのパイプラインはバイトスルーです。  
ファイル一覧の取得や、検索、テキストファイルの表示など基本的なコマンドも異なります。  
ワイルドカードの展開手順も異なり、シェルスクリプトは高度に発達していて、その為の仕様も複雑です。  
環境変数・動作設定の仕様もWindowsとは大きく異なります。  
使ったことの無い人が、いきなり使えるものではありません。

UNIXシェル環境はプログラマにとっては馴染み深いものですが、主にExcelやWord、ブラウザ、企業業務システムを利用しているような一般ユーザーには、お勧めできるものではありません。

しかし、どうしてもUNIXシェル環境を使用するなら、**WSLをお勧めします。**  
これは Windows の標準機能であり、完全な GNU Linux なので信頼性で最も勝ります。  
**次にお勧めするのは、Git Bash です。**これならデフォルトで iconv , file , dos2unix , unix2dos がインストールされているので導入が簡単です。  
Git Bash は MSYS2 を内蔵しているので、MSYS2 をお勧めする意味はありません。  
また、WSL が実現した現在では Cygwin を導入する意味も無いと思います。Cygwinもお勧めしません。

## 主要コマンドツールの使い方（世界標準）

テキストファイルの文字エンコーディング・BOM・改行コードの確認と変換を行う、世界標準のコマンドは以下のコマンドです。

`iconv , file , dos2unix , unix2dos`

これら四つのコマンドは全て先に紹介した UNIXシェル環境でしか使用できません。  
PowerShell と cmd では、使用できません。

Git Bash の場合は、インストールした時点でこれらのコマンドが装備されています。  
WSLでUbuntuをインストールした場合は、apt install コマンドでそれぞれのコマンドをインストールできます。  
MSYS2やCygwinについては、説明を省略します。

コマンドの詳しい使い方は省略し、文字エンコーディング・BOM・改行コードの確認と変換のやり方だけ、簡単に説明します。  
（ちなみに iconv,dos2unix,unix2dos のオプションによるエンコーディング指定は大文字小文字を区別しません）

### 文字エンコーディングの変換

#### エンコーディングの確認

```shell
# file コマンド（標準・国内外共通）
file -i input.txt          # Linux: MIMEタイプ形式で表示
file -I input.txt          # macOS: 同上

file input.txt		# 他の情報も表示されるが、オプション無しでも確認できる。

```

#### Shift\_JIS → UTF-8

```shell
# iconv（国内外共通・推奨）
iconv -f SHIFT_JIS -t UTF-8 input.txt > output.txt

# 上書きしたい場合
iconv -f SHIFT_JIS -t UTF-8 input.txt --output=input.txt
# 上書きしたい場合（-oでも良い）
iconv -f SHIFT_JIS -t UTF-8 input.txt -oinput.txt

```

#### UTF-8 → Shift\_JIS

```shell
# iconv
iconv -f UTF-8 -t SHIFT_JIS input.txt > output.txt
```

#### 複数ファイルの一括変換（bash/zsh）

iconv は、上書きするとき –output オプションで「上書き対象ファイル名」を記入しなければならないため、複数ファイルを一括処理する場合は、シェルスクリプトの for 文でループ処理を書く必要があります。

```shell
# iconv で *.txt を一括変換
for fn in *.txt
do
  iconv -f shift_jis -t utf-8 "${fn}" --output="${fn}"
done
```

**iconv の注意点**：変換できない文字があるとエラーで止まります。-c オプションで変換不能文字をスキップできます（iconv \-f SHIFT\_JIS \-t UTF-8 \-c input.txt）。

### BOM の付加・除去

#### BOM の確認

```shell
# 先頭バイトを16進で確認
head -c 4 input.txt | xxd
# UTF-8 BOM: ef bb bf
# UTF-16LE BOM: ff fe

# file コマンド
file input.txt   # "with BOM" と表示されることがある（環境依存）

```

#### UTF-8 BOM の除去

世界標準のUNIXコマンドでは「BOMだけを付加・除去」する機能はありません。  
BOMだけを操作するならば、以下の様にBOMのバイナリを先頭に付加したり除去する操作を行うことになります。

```shell
# sed（Linux/macOS 共通）
sed -i 's/^\xEF\xBB\xBF//' input.txt        # Linux
sed -i '' 's/^\xEF\xBB\xBF//' input.txt     # macOS（-i の後に '' が必要）

# tail（安全な方法）
tail -c +4 input.txt > output.txt    # 先頭3バイトをスキップ

# Python（環境依存が少ない）
python3 -c "
import sys
data = open('input.txt','rb').read()
open('output.txt','wb').write(data.lstrip(b'\xef\xbb\xbf'))
"
```

#### UTF-8 BOM の付加

```shell
# printf で BOM を先頭に付加
printf '\xEF\xBB\xBF' | cat - input.txt > output.txt

```

#### UTF-16LE への変換（BOM付き）

iconv の \-t UTF-16LE 指定の場合は、BOM無しになりますが、  
\-t UTF-16  指定の場合は、BOM付きになります。

```shell
# iconv
iconv -f UTF-8 -t UTF-16LE input.txt > output_nobom.txt
# BOM付き UTF-16LE
printf '\xff\xfe' | cat - <(iconv -f UTF-8 -t UTF-16LE input.txt) > output.txt


# iconv で UTF-16（自動でBOM付き）
iconv -f UTF-8 -t UTF-16 input.txt > output.txt   # BOM自動付加

```

#### UTF-16LE → UTF-8（BOM除去を含む）

```shell
# iconv（BOMがあっても自動処理される場合が多い）
iconv -f UTF-16LE -t UTF-8 input.txt > output.txt

# BOMありUTF-16の場合
iconv -f UTF-16 -t UTF-8 input.txt > output.txt   # BOM見て自動判別

```

UTF-16 の場合は、UTF-16LE と UTF-16 のエンコーディング指定を使い分けることで、BOMの有無を操作できますが、UTF-8 ではできません。  
環境による違いもあると聞きますが未確認です。（自分の Ubuntu では BOM付きUTF-8 は扱えません）

### 改行コードの変換

#### 改行コードの確認

```shell
# file コマンド
file input.txt     # "CRLF line terminators" などと表示

# cat -A（Linux）/ cat -e（macOS）
cat -A input.txt   # CR が ^M として表示される

# xxd で確認
xxd input.txt | grep -E "0d 0a|0a"

```

改行コードは、file コマンドで確認するのが良いでしょう。

#### CRLF → LF（dos2unix）

```shell
# dos2unix（国内外問わず最も一般的）
dos2unix input.txt                  # 上書き
dos2unix input.txt output.txt       # 別ファイルに出力
dos2unix *.txt                      # 一括処理

# sed（dos2unix が使えない環境）
sed -i 's/\r//' input.txt           # Linux
sed -i '' 's/\r//' input.txt        # macOS

# tr
tr -d '\r' < input.txt > output.txt

```

dos2unix , unix2dos コマンドには、BOMを制御するオプションがあります。  
改行コードとBOMを一括で変換するなら、BOMの制御が可能です。

```shell
# dosテキストからUNIXテキストへ変換
dos2unix  --remove-bom input.txt                  # 上書き
dos2unix  --remove-bom input.txt output.txt       # 別ファイルに出力
dos2unix  --remove-bom *.txt                      # 一括処理

```

#### LF → CRLF（unix2dos）

```shell
# unix2dos
unix2dos input.txt
unix2dos input.txt output.txt

# sed
sed -i 's/$/\r/' input.txt          # Linux
sed -i '' 's/$/\r/' input.txt       # macOS

# awk
awk '{print $0"\r"}' input.txt > output.txt

```

改行コードとBOMを一括で変換するなら、BOMの制御が可能です。

```shell
# UNIXテキストからdosテキストへ変換
unix2dos  --add-bom input.txt
unix2dos  --add-bom input.txt output.txt

```

## nkf コマンドの使い方（日本のみ）

日本語環境では、世界標準コマンドよりも使い勝手の良い「nkf」というコマンドが提供されています。  
nkf コマンドは日本語の文字エンコーディングに特化した、文字エンコーディング・BOM・改行コードの確認と変換が可能なコマンドです。  
日本国内では nkf が標準ツールと言って良いでしょう。  
ただ、日本語文字エンコーディング専用なので海外では存在すら知られていません。  
nkf も UNIXシェル環境で提供されています。

基本的な使い方を以下に解説します。

### 文字エンコーディングの変換

#### エンコーディングの確認

```shell
nkf --guess input.txt
```

#### Shift\_JIS → UTF-8

```shell
nkf -w input.txt > output.txt        # -w = UTF-8出力
nkf -w --overwrite input.txt         # 上書き（nkf限定の便利オプション）

```

#### UTF-8 → Shift\_JIS

```shell
nkf -s input.txt > output.txt        # -s = Shift_JIS出力
nkf -s --overwrite input.txt

```

#### 複数ファイルの一括変換（bash/zsh）

```shell
# nkf --overwrite で一括上書き
nkf -w --overwrite *.txt
```

### BOM の付加・除去

#### BOM の確認

```shell
nkf --guess input.txt	# BOMがあれば (BOM) と表示する、無ければ表示しない
```

#### UTF-8 BOM の除去

```shell
nkf -w80 --overwrite input.txt
```

#### UTF-8 BOM の付加

```shell
nkf -w8 --overwrite input.txt
```

#### UTF-16LE への変換（BOM付き）

```shell
nkf -w16L --overwrite input.txt
```

#### UTF-16LE → UTF-8（BOM除去を含む）

```shell
nkf -w80 --overwrite input.txt
```

BOMを除去するときは、オプション末尾に 0 を付けます。  
末尾に 0 が無い場合はBOMを付加します。

```shell
# BOMを除去する
-w80  
-w16L0
-w16B0
# BOMを付加する
-w8
-w16L
-w16B
```

### 改行コードの変換

#### 改行コードの確認

```shell
nkf --guess input.txt	# Windows型改行コードならば (CRLF) と表示する。
```

#### CRLF → LF（dos2unixと同様）

```shell
nkf -Lu --overwrite input.txt       # -Lu = LF（Unix）改行

```

#### LF → CRLF（unix2dosと同様）

```shell
nkf -Lw --overwrite input.txt       # -Lw = CRLF（Windows）改行
```

#### nkf 関連オプションまとめ

nkf は、文字エンコーディングを自動推定する機能を持ち、対象テキストファイルの文字エンコーディングを特に指定しなくても、自動で識別してくれますが、その自動推定機能は完璧ではありません。  
そもそも「完璧な自動推定」など、どこにも存在しないので、100%自動推定機能に依存してはいけないのです。  
nkf には入力テキストファイルの文字エンコーディングを指定するオプションもあるので、対象文字エンコーディングが分かっているならば、オプションでそれを指定するべきでしょう。  
文字エンコーディングのオプションは以下のようになります。入力と出力は別々です。

| option | input/output | 機能 |
| :---- | :---- | :---- |
| \-j | output | ISO-2022-JP で出力する。 |
| \-s | output | Shift\_JIS で出力する。 |
| \-e | output | EUC-JP で出力する。 |
| \-w | output | UTF-8 で出力する。 |
| \-w8 | output | UTF-8 で出力する。 |
| \-w16L | output | UTF-16LE で出力する。 |
| \-w16B | output | UTF-16BE で出力する。 |
| \-w32L | output | UTF-32LE で出力する。 |
| \-w32B | output | UTF-32BE で出力する。 |
| \-J | input | ISO-2022-JP で入力する。 |
| \-S | input | Shift\_JIS で入力する。 |
| \-E | input | EUC-JP で入力する。 |
| \-W | input | UTF で入力する。 |
| \-W8 | input | UTF-8 で入力する。 |
| \-W16L | input | UTF-16LE で入力する。 |
| \-W16B | input | UTF-16BE で入力する。 |
| \-W32L | input | UTF-32LE で入力する。 |
| \-W32B | input | UTF-32BE で入力する。 |

大文字オプションは、入力テキストファイルの文字エンコーディングを指定し、  
小文字オプションは、出力テキストファイルの文字エンコーディングを指定します。  
文字エンコーディングの自動推定に任せるならば、入力テキストファイルの文字エンコーディングを指定する必要はありません。

文字エンコーディング・オプションの末尾に 0 を付けると『BOMを除去』することを意味します。末尾に 0 が無い場合は、BOMを付加します。（または「末尾に 0 が無い場合は、出力文字エンコーディングの標準設定に従います」）

```shell
# BOMを除去する場合のオプション
-w80
-w16L0
-w16B0
-w32L0
-w32B0
```

改行コードの変換は \-L オプションで行います。  
オプションの二文字目(u w m)で改行コードの種類を指定します。

| option | 改行コードの種類 |
| :---- | :---- |
| \-Lu | LF （UNIX規格の改行コード） |
| \-Lw | CR・LF （Windows規格の改行コード） |
| \-Lm | CR（旧Mac規格の改行コード） |

## コマンド操作まとめ

### 推奨コマンド早見表

| 操作 | 日本国内（推奨） | 海外（推奨） |
| :---- | :---- | :---- |
| エンコーディング判別 | nkf \--guess / file \-i | file \-i |
| SJIS→UTF-8 | nkf \-w \--overwrite | iconv \-f SHIFT\_JIS \-t UTF-8 |
| UTF-8→SJIS | nkf \-s \--overwrite | iconv \-f UTF-8 \-t SHIFT\_JIS |
| UTF-8 BOM除去 | nkf \-w80 \--overwrite | sed \-i 's/^\\xEF\\xBB\\xBF//' |
| UTF-8 BOM付加 | nkf \-w8 \--overwrite | printf '\\xEF\\xBB\\xBF' \\| cat \- file |
| CRLF→LF | dos2unix / nkf \-Lu | dos2unix |
| LF→CRLF | unix2dos / nkf \-Lw | unix2dos |

**日本国内と海外の最大の違い**は nkf の有無です。  
国内では nkf が「エンコード変換・BOM操作・改行変換」を1ツールで担える万能ツールとして浸透していますが、海外では存在自体が知られていないことがほとんどで、iconv \+ dos2unix の組み合わせが事実上の標準です。  
macOS では Homebrew で両方インストールできます（brew install nkf dos2unix）。

日本語UNIXシェル環境ならば、nkf 一択と言えるでしょう。

## Windows版の nkf コマンド

nkf は UNIXで提供されているコマンドですが、nkf に限り Windows版が提供されています。  
Windows版 nkf は Vector から配布されています。

[**nkf.exe nkf32.dll Windows用 ダウンロード**](https://www.vector.co.jp/soft/dl/win95/util/se295331.html)

圧縮ファイルを解凍し、exe を環境変数 PATH の通ったフォルダーにコピーすれば使用可能になります。

Windows版の nkf コマンドは Git Bash を使用するときに利用します。  
WSL の場合は、apt install nkf 等で Linux にインストールした Linux版 nkf を使用することになります。  
この点は、間違えないように注意してください。

### Windows版 nkf の問題点

Windows版 nkf は PowerShell や cmd でも使用することができます。  
しかし、その仕様には少し問題があります。

nkf は UNIXシェル（`bash , zsh` など）に合わせて開発されています。  
Windowsの PowerShell や cmd の仕様に合わせていません。  
nkf の場合は、**二つの問題点**があります。

**問題の一つ**はワイルドカード展開です。  
UNIXシェルでは、コマンドパラメータのファイル名指定で使用されるワイルドカード（`*` :アスタリスク）は、まずシェル側で展開されコマンドに対しては検索されたファイル名のリストで引き渡されます。  
例えばカレントディレクトリに以下のファイルがあるとします。

```shell
$ ls *
text1.txt  text2.txt  text3.txt
```

この状況で、以下の様にコマンド com にワイルドカードを指定するとします。（com は仮のコマンドです）

```shell
com *
```

すると、UNIXシェルは `*` を検索してファイル名のリストにしてから、コマンドに引き渡します。

```shell
com text1.txt text2.txt text3.txt
```

つまり、UNIX環境では、シェル側がワイルドカードに該当するファイル名を検索してくれるので、コマンド側ではワイルドカードを展開する必要が無いのです。  
UNIX系のコマンドにはワイルドカードを展開する機能が備わっていません。

nkf コマンドも同様で、Windows 環境で `nkf *`  とワイルドカードを指定しても該当ファイルを検索することができません。  
UNIXシェルで nkf を使用するとワイルドカード展開をしてくれますが、PowerShell と cmd ではワイルドカード展開をしてくれません。  
これはUNIXとWindowsのシェル仕様の違いによるものです。

つまり、Windows環境（PowerShell と cmd）では nkf はワイルドカードを使用することができないのです。

#### PowerShell 固有の問題

nkf を含めて UNIX系コマンドは、出力結果を標準出力に出力する仕様のコマンドが多いです。  
nkf もUNIX標準的な使い方では、以下の様に標準出力に出力します。対象ファイルに上書きしたりはしません。

```shell
# Shift_JIS から UTF-8のBOM付き で標準出力に出力する
nkf -S -w8 input.txt
```

対象ファイルに上書きするときは、--overwrite オプションを指定します。

```shell
nkf -S -w8 --overwrite input.txt
```

標準的使い方では、上書きしないで標準出力に出力するのは、UNIX文化では一つのコマンドはできるだけ単純な機能だけを提供し、複雑な処理をするときはパイプラインを使用して、別のコマンドと組み合わせて使用するのがUNIXの作法になっているからです。

上書きするときは、上書き用のオプションを指定することが多いです。

UNIXのパイプラインの仕様はバイト配列をそのまま送る仕様です。

ここで**二つ目の問題**があります。  
PowerShellではパイプラインの機構にオブジェクトパイプラインという独自の実装があります。  
PowerShellのコマンドレットでは、標準出力への出力文字列は .NET の System.String 型のコレクションにして出力されます。

PowerShell 上で、Windows版の nkf を使用して標準出力の結果を、パイプラインで別のコマンドの標準入力に繋いでも、テキストを正常に受け取れない場合があります。  
ここで注意が必要なのは、**PowerShell 7.4 以降はこのパイプラインの仕様に改善があり、nkf も正常に稼働することがあります**。  
**PowerShell 7.3 以前や Windows PowerShell では、nkf のパイプライン出力は文字化けや、意図しない文字エンコーディングでの出力という、不本意な結果になる場合があります**。  
PowerShell は進化を続けていますが、パイプラインの問題が抜本的に解決したわけではありません。  
PowerShell で nkf を使用する変換処理を行うときは、--overwrite オプションを使用するようにした方が堅実です。

## WindowsではUNIX標準コマンドは使い難い

長々と解説してきましたが、これまでの説明を見て分かるように、Windows環境では「文字エンコーディングやBOMの有無・改行コードの確認」と、「それぞれの変換」操作は、いちいちUNIXシェルを導入して操作しなければならないので、非常に面倒です。  
文字エンコーディングとBOM変換の方は、PowerShell のコマンドレットで操作できますが、現状の確認ができません。  
文字エンコーディングの自動識別機能も存在しません。

私が、mfsr-mfprobe コマンドを作成したのも、この操作の面倒臭い問題を解消したかったからです。

[rmsmf-txprobe & mfsr-mfprobe 使い方の分かりやすい解説](https://snow-stack.net/tool_rmsmf_guide/)

今回は、Windows 上に UNIXシェルを導入することにより、文字エンコーディングとBOMと改行コードの確認と変換を行う方法の解説をしました。

次回は、PowerShell による文字エンコーディングとBOMの操作方法を解説したいと思っています。  
但し、既に解説したように、PowerShell には 文字エンコーディングとBOM を確認する機能はありません。その点は先に説明しておきます。

PowerShell の解説は、いつ投稿するか分かりませんが、ゆるゆると原稿を書いていきたいと思います。

では、また。  
