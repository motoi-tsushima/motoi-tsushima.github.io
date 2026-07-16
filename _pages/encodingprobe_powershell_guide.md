---
title: "SnowStack.EncodingProbe.PowerShell 解説"
layout: single
classes: wide
permalink: /encodingprobe_powershell_guide/
author_profile: true
---
2026/07/14 document update

SnowStack.EncodingProbe.PowerShell のインストール方法と、使い方を解説します。

## インストール方法

SnowStack.EncodingProbe.PowerShell は、PowerShell ギャラリーに登録して配布しているので、PowerShell標準の Install-PSResource コマンドでギャラリーからダウンロード・インストールできます。（ユーザーが PowerShell ギャラリーを直接開く必要は無いです）

[powershellgallery.com](https://www.powershellgallery.com/)

[PowerShell ギャラリーの概要](https://learn.microsoft.com/ja-jp/powershell/gallery/getting-started?view=powershellget-3.x)

ローカルPCの PowerShell ターミナルを開き、以下のコマンドを入力するとダウンロード・インストールできます。

```
Install-PSResource SnowStack.EncodingProbe.PowerShell
```

~~（現在はプレリリース版なので -Prerelease オプションが必要です。正式版になればこのオプションは不要です）~~

コマンドを実行すると以下の確認メッセージが表示されます。

```
Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its Trusted value by running the
Set-PSResourceRepository cmdlet. Are you sure you want to install the PSResource from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): 
```

初めて利用する PowerShell Gallery のリポジトリなので、警告が出ます。

インストールするには、ここで [Y] か [A] を入力して、信頼していただく必要があります。



[Y] か [A] を入力して [Enter]キーを押すと、SnowStack.EncodingProbe.PowerShell がユーザーの PowerShell 環境にインストールされます。

### アンインストール方法

一度、インストールしたモジュールを削除するには、以下の Uninstall-PSResource コマンドレットを使用してアンインストールしてください。

```
Uninstall-PSResource SnowStack.EncodingProbe.PowerShell
```



## SnowStack.EncodingProbe.PowerShell の使い方

現在、SnowStack.EncodingProbe.PowerShell パッケージの中には、Resolve-Encoding と Get-EncodingProbePlatformInfo という二つのコマンドレットが含まれています。

### Resolve-Encoding

Resolve-Encoding は、パラメータで指定したテキストファイルの文字エンコーディングの推測を行うコマンドレット（Cmdlet）です。

基本的な操作は簡単で、Resolve-Encoding の後にテキストファイル名を指定するだけで、そのテキストファイルの文字エンコーディング・BOMの有無・改行コードの種類の推測結果を表示します。

```
Resolve-Encoding filename.txt
```

結果はオブジェクト・パイプラインに対して出力されるので、変数に代入したり他のコマンドレットのパイプで繋いだりできます。

ファイル名の指定は、絶対パスと相対パスには対応していますが、ワイルドカードには未対応です。ファイルは一つずつしか指定できません。

Resolve-Encoding は PowerShell スクリプトの中で使用できることを優先して開発したので、複数ファイルを一括で推測したい場合は、既に提供している mfprobe をご利用ください。

[rmsmf-txprobe & mfsr-mfprobe 使い方の分かりやすい解説](/tool_rmsmf_guide/)

#### Resolve-Encoding の出力値

Resolve-Encoding の出力するプロパティ変数は以下の種類があります。

| プロパティ名    | 値の内容                                                     |
| --------------- | ------------------------------------------------------------ |
| CodePage        | 文字エンコーディングのコードページ                           |
| EncodingWebName | .NET C# の中で使用する文字エンコーディング名称               |
| PSEncodingName  | PowerShell コマンドレットの -Encoding オプション等に指定する文字エンコーディングのフレンドリ名（PS5.1とPS6.2以降では値が異なります） |
| UsePSName       | PSEncodingName に有効なフレンドリ名が入っている場合は True に、Web Name や空白が入っている場合は False となります。 |
| Bom             | BOMの有無。True = BOM有り、False = BOM無し。                 |
| LineBreak       | 改行コードの種類。Windows形式 = CrLf 、UNIX形式 = Lf         |
| Culture         | コマンドが認識したカルチャー情報 (国情報)                    |

#### Resolve-Encoding のオプション

以下のオプションが指定できます。

##### -Version

SnowStack.EncodingProbe.PowerShell パッケージのバージョンを表示します。

##### -License

SnowStack.EncodingProbe.PowerShell のライセンス情報を表示します。

##### -Strategy

Resolve-Encoding は、テキストファイルをバイナリで読み込みバイナリパターンを解析することで、文字エンコーディングの推測を行っています。

-Strategy は、文字エンコーディングの推論方式をユーザーが選択できるオプションです。

文字エンコーディングの推測処理は、独自実装した処理と、UTF.Unknown というサードパーティー製品の二つを使い分けて実装しています。

UTF.Unknown は欧米などのシングルバイト文字エンコーディングを推測する場合は、高い信頼性を期待できますが、マルチバイトの東アジアの旧文字エンコーディングの推測は、やや信頼性で劣ります。日本語では Shift_JIS の半角カナ文字の判定で間違う確率が高くなります。

そこで、東アジアの文字エンコーディングの推測処理だけ、独自実装の処理を使用し、それ以外のシングルバイト文字エンコーディングの推測は、UTF.Unknown を使用して、国際化対応を実現しています。

-Strategy オプションは、この文字エンコーディングの推測処理を、ユーザーが選択するオプションです。

標準では、まず独自実装で文字エンコーディングの推測を行い、不明の場合は UTF.Unknown を使用します。(この点は mfprobe・mfsr も同様の処理を行っています)

-Strategy オプションで推測手順を変更できます。

-Strategy オプションの値には、以下の表のように「単語の値」と、簡単な「数値の値」が利用できます。

**-Strategy オプションの値**

| 単語のオプション値 | 数値のオプション値 | 動作内容                                                     |
| ------------------ | ------------------ | ------------------------------------------------------------ |
| default            | 0                  | 最初に独自実装処理で推測し、不明の場合は UTF.Unknown により推測する。オプション指定をしないと、この default になる。 |
| utfunknown         | 3                  | UTF.Unknown だけで推測する。独自実装処理は使用しない。       |
| native             | 1                  | 独自実装処理だけで推測する。UTF.Unknown は使用しない。       |

##### -Culture

東アジアの旧文字エンコーディングは、互いに似ているため、区別をするのが難しくなります。

また、東アジア諸国のユーザーが互いに、隣国の旧文字エンコーディングを参照する確率は、ほとんどゼロに等しいものと思われます。

よって、OSのカルチャー(国情報)によって、旧文字エンコーディングの推論処理を国に合わせて変更しています。

普通に自分の国のテキストファイルを使用している限りこのオプションは必要ないですが、もし他の東アジア諸国の旧テキストファイルを調査する場合は、-Culture オプションでその国のカルチャーを指定することで、その国用の文字エンコーディング推論処理を使用することができます。

日本語 Windows で日本語を使用しているだけなら、このオプションを使用する必要は無いです。

また、東アジア圏の人々は、隣国の言語を扱わない限り、自動でOSのカルチャーに合わせるので、カルチャーを切り替える必要は無いです。

東アジア圏以外の人々は、このオプションを使用する必要がありません。

つまり、ほとんどの人々にとって、-Culture オプションは使う必要の無いオプションです。

### Get-EncodingProbePlatformInfo

Get-EncodingProbePlatformInfo は、現在 SnowStack.EncodingProbe.PowerShell が認識しているプラットフォームの種類を報告するコマンドレットです。

#### 三つのレコード

主な使用方法は、以下の三通りになります。

```
(Get-EncodingProbePlatformInfo).OS
```

```
(Get-EncodingProbePlatformInfo).PowerShellHost
```

```
(Get-EncodingProbePlatformInfo).Runtime
```

大きく三つのレコードでプラットフォームの情報を報告します。

| レコード名     | 内容                                                         |
| -------------- | ------------------------------------------------------------ |
| OS             | コマンドレットが起動しているOSの種類を報告します。           |
| PowerShellHost | コマンドレットが起動しているPowerShellのバージョンを報告します。 |
| Runtime        | コマンドレットが起動している.NET基盤のバージョンを報告します。 |

それぞれのレコードの内容は以下のようになっています。

#### OS のプロパティ

| プロパティ名 | 内容                                              |
| ------------ | ------------------------------------------------- |
| IsWindows    | Windows上で起動している場合は True になります。   |
| IsMacOs      | macOS上で起動している場合は True になります。     |
| IsLinux      | Linux系OS上で起動している場合は True になります。 |
| Description  | OSの種類やバージョン情報を文字列で表記します。    |

#### PowerShellHost のプロパティ

| プロパティ名                    | 内容                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| PSVersion                       | PowerShellのバージョン情報                                   |
| PSEdition                       | PowerShellのエディション。<br />PS5.1なら 'Desktop' , PS6.2以降なら 'Core' と表記します。 |
| SupportsNumericCodePageArgument | PowerShellが Get-Contentなどの-Encodingオプションで、コードページ指定をサポートしている場合に True になります。 |
| SupportsAnsiEncodingName        | PowerShellが Get-Contentなどの-Encodingオプションで、WebName指定をサポートしている場合に True になります。 |

#### Runtime のプロパティ

| プロパティ名                          | 内容                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| FrameworkDescription                  | 起動している .NET のバージョン情報を文字列で表記します。     |
| IsDotNetFramework                     | 旧 .NET Framework の場合に True となります。                 |
| IsCodePagesEncodingProviderRegistered | 起動しているPowerShellに CodePagesEncodingProvider が登録済みであれば True となります。これは Shift_JIS のエンコーディング名を使用するとき必須となる機能です。<br />(.NET Frameworkの場合は無条件にTrueになります) |

#### 利用場面

この機能は、Resolve-Encoding の機能を実現するために、内部的に取得している情報を開示する機能です。

ユーザーが Get-EncodingProbePlatformInfo を使用する必要は、ほとんど無いと思われます。

しかし、もし .ps1 スクリプト開発において、上記のプラットフォームにより条件分岐する必要があれば、利用できます。

### コマンドレットの簡単な使用例

Resolve-Encoding 等のコマンドレットは、主にPowerShellスクリプトの中で利用するために開発したものです。

独立したコマンドとしての利便性なら、既に提供している mfprobe , mfsr の方が優れていますので、コマンド単体で利用したいのなら、 mfprobe , mfsr をお勧めします。

Resolve-Encoding を単体で使用すると以下の使い方になります。（text1.txt はテキストファイルの名前です）

```
PS C:\_test> Resolve-Encoding text1.txt

CodePage        : 65001
EncodingWebName : utf-8
PSEncodingName  : utf8NoBOM
UsePSName       : True
Bom             : False
LineBreak       : CrLf
Culture         : ja-JP
```

Resolve-Encoding は、EncodingInformation レコードを返します。

その中の PSEncodingName は、文字エンコーディングのPowerShell用フレンドリ名を返します。

フレンドリ名は、Get-Content など標準コマンドレットの -Encoding オプションで文字エンコーディングを指定するために使用できます。

Get-Content と Resolve-Encoding を以下の様に組み合わせて利用することが可能です。

```
 Get-Content -Encoding (Resolve-Encoding text1.txt).PSEncodingName text1.txt
```

このように組み合わせると、テキストファイル(text1.txt)の文字エンコーディングがわからない場合でも、テキストファイルの内容を表示してくれます。

同様の命令を変数を使用して記述すると以下の様に書けます。

```
 $textfile = "text1.txt"
 $psname = (Resolve-Encoding $textfile).PSEncodingName
 Get-Content -Encoding $psname $textfile
```

.ps1 スクリプトの中で使用するならば、このような使い方になるでしょう。

Resolve-Encoding はワイルドカードに対応していません。

以下の様にワイルドカードには PowerShell のループ機能等を使用してユーザーが自由に対応してください。

```
$param = "*.txt"
$fnarr = (Get-ChildItem $param).Name
foreach($textfile in $fnarr){
	$textfile
	$psname = (Resolve-Encoding $textfile).PSEncodingName
	$psname
	Get-Content -Encoding $psname $textfile
}
```

Resolve-Encoding はスクリプトの中で他のコマンドレットと組み合わせて使用することを想定して開発しているため、意図的にワイルドカードには対応していません。ワイルドカードに対応するとコレクションを返さなければならなくなり、コマンドレットとして使い難くなるからです。

使い方が単純になるように、意図的にワイルドカード未対応としています。将来も対応しません。

### PSEncodingName のフレンドリ名について

既に簡単に解説しましたが、PSEncodingName の返す文字エンコーディングのフレンドリ名は、PowerShell5.1と PowerShell6.2以降とでは、異なる値を返します。

PowerShell5.1 と PowerShell6.2以降では、扱える文字エンコーディングの種類の範囲が異なるからです。

PowerShell5.1では、BOMの無いUTF-8は使用できません。Unicodeに関してはBOMの無いテキストファイルを使用することを想定していません。

また、Shift_JIS もフレンドリ名としては用意されていないので、PowerShell5.1では -Encoding オプションで使用できません。Default や Oem という値を使用して Shift_JIS を使用することは可能ですが、Default や Oem はどちらもShift_JISを示す値では無く、プラットフォームの設定値を返しているだけです。Default や Oem で Shift_JISが使用できるのは日本語環境だけで、他の言語環境では Default や Oem には海外の文字エンコーディングが設定されます。

PowerShell6.2以降では幅広い文字エンコーディングに対応しています。

よって、PSEncodingName の値を使用する場合は、PowerShell5.1 と PowerShell6.2以降では動作が異なることを意識してください。

フレンドリ名については、以下の記事で解説しています。

[PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか （フレンドリ名の解説）](/why-utf8nobom-cannot-be-webname/)

## 使用しているライブラリ

SnowStack.EncodingProbe.PowerShell は、その機能のほとんどを NuGet パッケージの SnowStack.EncodingProbe によって実現しています。

SnowStack.EncodingProbe NuGet パッケージについては、以下のページで解説しています。

[SnowStack.EncodingProbe NuGet Package 解説](https://snow-stack.net/encodingprobe_guide/)

## ライセンス

ライセンスは **MITライセンス** です。

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

UTF.Unknown
Copyright (c) 2018 Nikolay Pultsin
Licensed under MIT License
https://github.com/CharsetDetector/UTF-unknown
```

## お知らせ関連

### 2026年7月14日　正式版リリース

正式版の Version 1.0.0 をリリースしました。

文字エンコーディングのフレンドリ名など、詳細な解説ドキュメントは、これから作成します。

ドキュメント類は、今しばらくお待ちください。

必要最小限の解説は、この記事で行っています。

