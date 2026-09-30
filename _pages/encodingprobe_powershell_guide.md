---
title: "SnowStack.EncodingProbe.PowerShell 解説"
layout: single
classes: wide
permalink: /encodingprobe_powershell_guide/
author_profile: true
---
2026/09/30 document update

SnowStack.EncodingProbe.PowerShell のインストール方法と、使い方を解説します。

2026年9月1日に Version 1.1.0 をリリースし、テキストファイルの読み書きを行う 4 つのコマンドレットを追加しました。

2026年9月30日に Version 1.2.0 をリリースし、ファイルへの出力と変換を行う 2 つのコマンドレットを追加しました。あわせて、東アジア以外の言語と、香港の Big5 の判定を改善しています。

追加分の解説は分量があるので、以下の別記事に分けています。この記事では、インストール方法と 1.0.x から提供しているコマンドレットを扱います。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

[SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)

## インストール方法

SnowStack.EncodingProbe.PowerShell は、PowerShell ギャラリーに登録して配布しているので、PowerShell標準の Install-PSResource コマンドでギャラリーからダウンロード・インストールできます。（ユーザーが PowerShell ギャラリーを直接開く必要は無いです）

以下の手順は PowerShell 7.x を対象としています。Windows PowerShell 5.1 には Install-PSResource コマンドが用意されていないため、この手順のままではインストールできません。Windows PowerShell 5.1 をお使いの方は、後述の「Windows PowerShell 5.1 へのインストール」をご覧ください。

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

### Windows PowerShell 5.1 へのインストール

ここまでの手順は、PowerShell 7.x を前提としています。

Windows PowerShell 5.1 には Install-PSResource コマンドレットが同梱されていないため、上記のコマンドをそのまま実行してもエラーになります。

Windows PowerShell 5.1 でインストールするには、先に Microsoft.PowerShell.PSResourceGet モジュールを導入する前作業が必要になります。

その手順は分量があるので、以下の補足記事に分けて解説しています。Windows PowerShell 5.1 をお使いの方は、こちらをご覧ください。

[SnowStack.EncodingProbe.PowerShell を Windows PowerShell 5.1 へインストールする方法](/encodingprobe_powershell_install_ps51/)

### アンインストール方法

一度、インストールしたモジュールを削除するには、以下の Uninstall-PSResource コマンドレットを使用してアンインストールしてください。

```
Uninstall-PSResource SnowStack.EncodingProbe.PowerShell
```



## SnowStack.EncodingProbe.PowerShell の使い方

現在、SnowStack.EncodingProbe.PowerShell パッケージの中には、八つのコマンドレットが含まれています。

| コマンドレット | 役割 | 追加バージョン |
| :---- | :---- | :---- |
| `Resolve-Encoding` | テキストファイルの文字エンコーディングを推測する | 1.0.0 |
| `Get-EncodingProbePlatformInfo` | 実行中のプラットフォーム情報を報告する | 1.0.0 |
| `Get-ProbedContent` | 文字エンコーディングを判定して読む | 1.1.0 |
| `Set-ProbedContent` | 文字エンコーディングを明示して書く | 1.1.0 |
| `Add-ProbedContent` | 文字エンコーディングを保って追記する | 1.1.0 |
| `ConvertTo-DotNetEncoding` | 各種の指定を `System.Text.Encoding` に変換する | 1.1.0 |
| `Out-ProbedFile` | オブジェクトを整形して、文字エンコーディングを指定してファイルに書き出す | 1.2.0 |
| `Convert-ProbedContent` | 既存のファイルの文字エンコーディング・BOM・改行を変換する | 1.2.0 |

この記事では、上の二つを解説します。

1.1.0 で追加した四つのコマンドレットは、統一された文字エンコーディング名の体系を前提としており、まとめて解説しないと意味が伝わりません。よって、以下の別記事で解説しています。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

1.2.0 で追加した二つのコマンドレットも、同じ文字エンコーディング名の体系を使います。こちらは以下の別記事で解説しています。

[SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)

1.2.0 では、`Resolve-Encoding` のパラメータと戻り値は変更していません。ただし、内部で使っている判定処理を改善したので、東アジアの環境で欧米の言語のファイルを判定した場合などに、判定結果が 1.1.0 と変わります。詳しくは上の 1.2.0 の記事をご覧ください。

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
| UsePSName       | PSEncodingName に有効なフレンドリ名が入っている場合は True に、Web Name や空白が入っている場合は False となります。False のときは、CodePage で文字エンコーディングを指定してください。 |
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

1.2.0 からは、独自実装が東アジアの旧マルチバイト文字エンコーディングと推測した場合にも、UTF.Unknown の推測結果と突き合わせるようにしました。UTF.Unknown が十分な確かさで欧米などのシングルバイト文字エンコーディングと推測した場合は、そちらを採用します。これにより、日本語環境でドイツ語のファイルを Shift_JIS と誤って推測する問題を解消しています。

-Strategy オプションで推測手順を変更できます。

-Strategy オプションの値には、以下の表のように「単語の値」と、簡単な「数値の値」が利用できます。

**-Strategy オプションの値**

| 正式名 | 単語のオプション値 | 数値のオプション値 | 動作内容                                                     |
| :---- | :---- | :---- | :---- |
| Combined | default            | 0                  | 最初に独自実装処理で推測し、不明の場合は UTF.Unknown により推測する。オプション指定をしないと、この Combined になる。 |
| UtfUnknownOnly | utfunknown         | 3                  | UTF.Unknown だけで推測する。独自実装処理は使用しない。       |
| NativeOnly | native             | 1                  | 独自実装処理だけで推測する。UTF.Unknown は使用しない。       |

一番左の列が正式名で、NuGet パッケージ側の `DetectionStrategy` 列挙型のメンバ名と一致しています。どの書き方でも動作は同じで、大文字と小文字も区別しません。

1.1.0 で追加した `Get-ProbedContent` などのコマンドレットも、同じ値を受け付けます。新しく書くスクリプトでは、意味が名前から読み取れる正式名の使用をお勧めします。

-Strategy を実際に切り替えると結果がどう変わるかは、以下の記事で実測値を示して解説しています。

[-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)

##### -Culture

東アジアの旧文字エンコーディングは、互いに似ているため、区別をするのが難しくなります。

また、東アジア諸国のユーザーが互いに、隣国の旧文字エンコーディングを参照する確率は、ほとんどゼロに等しいものと思われます。

よって、OSのカルチャー(国情報)によって、旧文字エンコーディングの推論処理を国に合わせて変更しています。

普通に自分の国のテキストファイルを使用している限りこのオプションは必要ないですが、もし他の東アジア諸国の旧テキストファイルを調査する場合は、-Culture オプションでその国のカルチャーを指定することで、その国用の文字エンコーディング推論処理を使用することができます。

日本語 Windows で日本語を使用しているだけなら、このオプションを使用する必要は無いです。

また、東アジア圏の人々は、隣国の言語を扱わない限り、自動でOSのカルチャーに合わせるので、カルチャーを切り替える必要は無いです。

東アジア圏以外の人々は、このオプションを使用する必要がありません。

つまり、ほとんどの人々にとって、-Culture オプションは使う必要の無いオプションです。

1.2.0 からは、香港・マカオのカルチャー（`zh-HK` / `zh-MO`）と広東語（`yue`）を、台湾とは分けて扱うようになりました。また、`zh-Hant-HK` のような用字を含むカルチャー名も、正しく解釈します。

もし使う必要が出てきた場合は、以下の記事で具体例を示して解説しています。日本語環境で韓国語のファイルを読むと、どう文字化けするかの実測値を載せています。

[-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)

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

但し、この書き方は **Windows PowerShell 5.1 では成立しない場合があります。** 上の例は PSEncodingName に utf8NoBOM が入っている場合ですが、Shift_JIS のファイルでは PowerShell 5.1 の PSEncodingName が空になるからです。

その理由は、この後の「PSEncodingName のフレンドリ名について」で解説します。

1.1.0 では、この問題を解消するために Get-ProbedContent コマンドレットを追加しました。標準コマンドに橋を架けるのではなく、判定と読み込みを一つのコマンドで行います。

```
# 1.1.0 の書き方。PSEncodingName を経由しないので、PowerShell 5.1 でも動く
Get-ProbedContent text1.txt
```

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

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

なお、1.1.0 で追加した Get-ProbedContent の方は、ワイルドカードとパイプライン入力に対応しています。読み込んだ結果は文字列なので、コレクションを返しても使い難くならないからです。

```
# Get-ProbedContent はワイルドカードとパイプライン入力に対応している
Get-ProbedContent *.txt
Get-ChildItem *.txt | Get-ProbedContent
```

### PSEncodingName のフレンドリ名について

既に簡単に解説しましたが、PSEncodingName の返す文字エンコーディングのフレンドリ名は、PowerShell5.1と PowerShell6.2以降とでは、異なる値を返します。

PowerShell5.1 と PowerShell6.2以降では、扱える文字エンコーディングの種類の範囲が異なるからです。

PowerShell5.1では、BOMの無いUTF-8は使用できません。Unicodeに関してはBOMの無いテキストファイルを使用することを想定していません。

また、Shift_JIS もフレンドリ名としては用意されていないので、PowerShell5.1では -Encoding オプションで使用できません。Default や Oem という値を使用して Shift_JIS を使用することは可能ですが、Default や Oem はどちらもShift_JISを示す値では無く、プラットフォームの設定値を返しているだけです。Default や Oem で Shift_JISが使用できるのは日本語環境だけで、他の言語環境では Default や Oem には海外の文字エンコーディングが設定されます。

PowerShell6.2以降では幅広い文字エンコーディングに対応しています。

よって、PSEncodingName の値を使用する場合は、PowerShell5.1 と PowerShell6.2以降では動作が異なることを意識してください。

フレンドリ名については、以下の記事で解説しています。

[PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか （フレンドリ名の解説）](/why-utf8nobom-cannot-be-webname/)

#### 1.1.0 では、標準コマンドへ橋を架けるのをやめました

ここまで解説したとおり、PSEncodingName の値を標準コマンドへ渡す書き方は、Windows PowerShell 5.1 で成立しない場合があります。

そもそも、Windows PowerShell 5.1 の -Encoding は固定の列挙型で、Shift_JIS を表す値が存在しません。ここに橋を架けるのは無理があります。

そこで Version 1.1.0 では、**自前の読み書きコマンドを持つ**方向へ舵を切りました。

Get-ProbedContent / Set-ProbedContent / Add-ProbedContent は、PowerShell の標準語彙を使いません。このモジュールが定義した統一語彙を使うので、Windows PowerShell 5.1 でも PowerShell 7.x でも同じ名前で同じ結果になります。

なお、Resolve-Encoding の PSEncodingName と UsePSName の仕様は 1.1.0 でも変更していません。標準コマンドと組み合わせる既存のスクリプトは、そのまま動きます。

1.2.0 では、PowerShell 7.x で windows-1252 や iso-8859-1 のようにフレンドリ名の無い文字エンコーディングを判定したとき、PSEncodingName に `I do not know.` という文字列が入っていた不具合を直しました（1.0.0 から存在した不具合です）。1.2.0 では、この場合の PSEncodingName は空（null）になります。Windows PowerShell 5.1 では、もともと null でした。

UsePSName が False のときは、PSEncodingName ではなく CodePage を使ってください。

```
# 判定結果のコードページで読む
$info = Resolve-Encoding .\it1252.txt
Get-ProbedContent .\it1252.txt -Encoding $info.CodePage
```

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

## 使用しているライブラリ

SnowStack.EncodingProbe.PowerShell は、その機能のほとんどを NuGet パッケージの SnowStack.EncodingProbe によって実現しています。

SnowStack.EncodingProbe NuGet パッケージについては、以下のページで解説しています。

[SnowStack.EncodingProbe NuGet Package 解説](https://snow-stack.net/encodingprobe_guide/)

但し、1.1.0 で追加した文字エンコーディング名の統一語彙と、読み書きの処理は、コマンドレット側で実装しています。NuGet パッケージ側は 1.1.0 でコードを変更していません。

1.2.0 では、NuGet パッケージ側の判定処理を改修しました（東アジア以外の言語と、香港の Big5 への対応）。コマンドレットの判定の改善は、この改修によるものです。

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

UTF.Unknown 2.7.0 (net10.0 build) / 2.6.0 (net48 build)
Copyright (c) 2018 Nikolay Pultsin
Licensed under MPL 1.1 / GPL 2.0 or later / LGPL 2.1 or later
(This software uses UTF.Unknown under the terms of MPL 1.1)
https://github.com/CharsetDetector/UTF-unknown
https://www.mozilla.org/MPL/1.1/
```

UTF.Unknown のバージョンが .NET 10 用と .NET Framework 4.8 用で分かれているのは、意図的なものです。理由は以下の記事の「なぜこの作りにしたのか」で解説しています。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

同じ文面は、コマンドからも取得できます。

```
# ライセンス情報を表示する
Resolve-Encoding -License
```

## お知らせ関連

### 2026年9月30日　Version 1.2.0 リリース

Version 1.2.0 をリリースしました。

Out-ProbedFile / Convert-ProbedContent の二つのコマンドレットを追加しています。

Out-ProbedFile は、標準の Out-File の代わりに、Windows PowerShell 5.1 と PowerShell 7.x で同じバイト列を書き出すコマンドです。Convert-ProbedContent は、既存のファイルの文字エンコーディング・BOM・改行を、文字を失わずに変換するコマンドです。

判定処理も改善し、東アジアの環境で欧米の言語のファイルを誤判定する問題と、香港の Big5 の扱いを直しました。ルーマニア語などのファイルで例外が発生する、1.1.0 から存在した不具合も修正しています。

Set-ProbedContent / Add-ProbedContent の -Force が、書き込み後に読み取り専用の属性を元に戻すようになりました（1.1.0 の不具合の修正）。

Set-ProbedContent / Add-ProbedContent の -WhatIf が、読み取り専用のファイルなど、実行すれば失敗することを報告するようになりました。

Resolve-Encoding などが返す PSEncodingName に `I do not know.` という文字列が入る不具合（1.0.0 から存在。PowerShell 7.x のみ）を直しました。

Convert-ProbedContent の -PassThru の結果には、変換元のコードページ番号（SourceCodePage）が含まれます。Unicode 系以外の文字エンコーディングへ元に戻すときは、この番号を -Encoding に渡してください。

また、香港・マカオのカルチャーでも、Get-Help のヘルプとエラーメッセージが繁体字中国語で表示されるようになりました。

追加分の解説は、以下の記事で行っています。

[SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)

### 2026年9月1日　Version 1.1.0 リリース

Version 1.1.0 をリリースしました。

Get-ProbedContent / Set-ProbedContent / Add-ProbedContent / ConvertTo-DotNetEncoding の四つのコマンドレットを追加しています。

**Windows PowerShell 5.1 と PowerShell 7.x で、同じスクリプトが同じバイト列を書く**ことが、このバージョンの目的です。

Resolve-Encoding と Get-EncodingProbePlatformInfo は変更していないので、1.0.x 向けに書いたスクリプトはそのまま動きます。

あわせて、Get-Help のヘルプとエラーメッセージを、英語・日本語・韓国語・繁体字中国語・簡体字中国語の 5 言語に対応させました。

追加分の解説は、以下の記事で行っています。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

[-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)

### 2026年7月14日　正式版リリース

正式版の Version 1.0.0 をリリースしました。

このとき「詳細な解説ドキュメントは、これから作成します」と書いていましたが、上記の 1.1.0 の記事と、以下のフレンドリ名の記事で解説を行いました。

[PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか （フレンドリ名の解説）](/why-utf8nobom-cannot-be-webname/)

