---
title: "SnowStack.EncodingProbe.PowerShell 解説"
layout: single
classes: wide
permalink: /encodingprobe_powershell_guide/
author_profile: true
---
SnowStack.EncodingProbe.PowerShell のインストール方法と、使い方を解説します。

## インストール方法

SnowStack.EncodingProbe.PowerShell は、PowerShell ギャラリーに登録して配布しているので、PowerShell標準の Install-PSResource コマンドでギャラリーからダウンロード・インストールできます。（ユーザーが PowerShell ギャラリーを直接開く必要は無いです）

[powershellgallery.com](https://www.powershellgallery.com/)

[PowerShell ギャラリーの概要](https://learn.microsoft.com/ja-jp/powershell/gallery/getting-started?view=powershellget-3.x)

ローカルPCの PowerShell ターミナルを開き、以下のコマンドを入力するとダウンロード・インストールできます。

```
Install-PSResource SnowStack.EncodingProbe.PowerShell -Prerelease
```

（現在はプレリリース版なので -Prerelease オプションが必要です。正式版になればこのオプションは不要です）

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

現在、SnowStack.EncodingProbe.PowerShell パッケージの中には、Resolve-Encoding というコマンドレットが一つしか含まれていません。

### Resolve-Encoding の使い方

Resolve-Encoding は、パラメータで指定したテキストファイルの文字エンコーディングの推測を行うコマンドレット（Cmdlet）です。

基本的な操作は簡単で、Resolve-Encoding の後にテキストファイル名を指定するだけで、そのテキストファイルの文字エンコーディング・BOMの有無・改行コードの種類の推測結果を表示します。

```
Resolve-Encoding filename.txt
```

結果はオブジェクト・パイプラインに対して出力されるので、変数に代入したり他のコマンドレットのパイプで繋いだりできます。

ファイル名の指定は、絶対パスと相対パス指定には対応していますが、ワイルドカードには未対応です。ファイルは一つずつしか指定できません。

Resolve-Encoding は PowerShell スクリプトの中で使用できることを優先して開発したので、複数ファイルを一括で推測したい場合は、既に提供している mfprobe をご利用ください。

#### Resolve-Encoding の出力値

Resolve-Encoding の出力するプロパティ変数は以下の種類があります。

| プロパティ名   | 値の内容                                                     |
| -------------- | ------------------------------------------------------------ |
| CodePage       | 文字エンコーディングのコードページ                           |
| EncodingName   | .NET C# の中で使用する文字エンコーディング名称               |
| PSEncodingName | PowerShell コマンドレットの -Encoding オプション等に指定する文字エンコーディングのフレンド名 |
| Bom            | BOMの有無。True = BOM有り、False = BOM無し。                 |
| LineBreak      | 改行コードの種類。Windows形式 = CrLf 、UNIX形式 = Lf         |
| IsWindowsOs    | 実行している OS が Windows のとき True になる                |
| IsMacOs        | 実行している OS が Mac のとき True になる                    |
| IsLinuxOs      | 実行している OS が Linux のとき True になる                  |
| Culture        | コマンドが認識したカルチャー情報 (国情報)                    |

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

## 使用しているライブラリ

SnowStack.EncodingProbe.PowerShell は、その機能のほとんどを NuGet パッケージの SnowStack.EncodingProbe によって実現しています。

SnowStack.EncodingProbe NuGet パッケージについては、以下のページで解説しています。

[SnowStack.EncodingProbe NuGet Package 解説](https://snow-stack.net/encodingprobe_guide/)

## お知らせ関連

2026年6月7日現在、SnowStack.EncodingProbe.PowerShell はプレビュー版で提供しています。

もう少し、テストを重ねてから正式版のリリースを行うことになります。

機能的にも、後々機能追加や変更を行う可能性があります。

また、この SnowStack.EncodingProbe.PowerShell は、mfprobe・mfsrの文字エンコーディングの推測処理部分を NuGetパッケージに分離独立させたモジュールを組み込んだコマンドレットとなります。

その NuGetパッケージ も SnowStack.EncodingProbe という名称で、NuGet.org からプレリリースしました。

将来的には、この SnowStack.EncodingProbe を mfprobe・mfsr から参照して、同じクラスライブラリを参照する予定です。

 SnowStack.EncodingProbe も MITライセンスで提供しているので、誰でもライブラリとして利用できます。

UTF.Unknown を組み込んでいるのも、SnowStack.EncodingProbe なので、このクラスライブラリだけで世界の文字エンコーディングに対応できます。

こちらのお知らせは、もう少しテストしてから、しばらく後に行う予定です。

### 2026年6月19日　追記

SnowStack.EncodingProbe.PowerShell の Preview2 版をリリースしました。

SnowStack.EncodingProbe NuGetパッケージの Preview2 版もリリースしました。

マニュアルなどは、後で書く予定です。

正式版リリースは、Preview2 で、もう少しテストしてからにします。

  しばらくお待ちください。

### 2026年6月23日　追記

SnowStack.EncodingProbe.PowerShell の Preview3 版をリリースしました。

SnowStack.EncodingProbe NuGetパッケージの Preview3 版もリリースしました。

EncodingInformation のスペルミスなど、いくつかの不具合を解消しました。

### 2026年7月2日　追記

SnowStack.EncodingProbe.PowerShell の Preview4 版をリリースしました。

SnowStack.EncodingProbe NuGetパッケージの Preview4 版もリリースしました。

カルチャー情報の更新を行わない方式に修正しました。

韓国語・繁体字中国語（台湾）・簡体字中国語の判定処理を厳格化しました。

EUC-KR と CP949 の両方に該当する場合は、CP949と判定します。

EUC-TW と CP950(Big5) の両方に該当する場合は、CP950(Big5) と判定します。

EUC-CN と CP936(GBK) の両方に該当する場合は、CP936(GBK) と判定します。

ちなみに、日本語の EUC-JP と Shift_JIS の両方に該当する場合は、改行コードがWindows型ならば Shift_JIS と UNIX型ならば EUC-JP と判定します。改行コードが無い場合は OS が Windowsの場合は Shift_JIS と Linux|Mac の場合は EUC-JP と判定します。

バイト並び解析の詳細方針は、正式リリースのときに説明します。

