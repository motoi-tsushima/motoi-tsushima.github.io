---
title: "SnowStack.EncodingProbe NuGet Package 解説"
layout: single
classes: wide
permalink: /encodingprobe_guide/
author_profile: true
---
2026/06/21 document update

## EncodingProbe とは

SnowStack.EncodingProbe とは、文字エンコーディングの不明なテキストファイルをバイナリ解析して、その文字エンコーディングを推測するクラスライブラリです。

MITライセンスの下で NuGet パッケージとして公開しており、「ライセンス情報の公開」を条件に誰でもソフトウェア開発に利用できるクラスライブラリ・モジュールです。

既に公開済みの mfprobe・mfsr コマンドの中で使用している文字エンコーディング推測モジュールを、NuGet パッケージとして切り出した物です。

## リポジトリ

ソースコードは mfprobe・mfsr 同様にオープンソースで提供しており、以下の GitHub リポジトリからダウンロード可能です。

[SnowStack.EncodingProbe](https://github.com/motoi-tsushima/SnowStack.EncodingProbe)

```
git clone https://github.com/motoi-tsushima/SnowStack.EncodingProbe
```

このリポジトリには、EncodingProbe クラスライブラリの他に、EncodingProbe.PowerShell コマンドレットのプロジェクトとソースコードも含まれています。

SnowStack.EncodingProbe が クラスライブラリで、SnowStack.EncodingProbe.PowerShell がコマンドレットです。

## リリース状況

2026年6月6日に最初の preview1 をリリースし、若干のテスト・デバッグを進めて、2026年6月19日に preview2 をリリースしました。

現在はまだ正式リリースは行っておらず、プレリリース段階です。

まだプレリリース段階なので、NuGetパッケージの外部仕様は変更する可能性があります。プレリリースの現状ではNuGetパッケージを使用した開発はお勧めできません。正式版のリリースまでお待ちください。

## NuGetパッケージのインストール方法

NuGetパッケージのインストール方法は、三通りあります。

[Visual Studio（GUI）を使う方法](https://learn.microsoft.com/ja-jp/nuget/quickstart/install-and-use-a-package-in-visual-studio)

[パッケージ マネージャー コンソールを使う方法](https://learn.microsoft.com/ja-jp/nuget/consume-packages/install-use-packages-powershell)

[dotnet CLI を使う方法](https://learn.microsoft.com/ja-jp/nuget/install-nuget-client-tools?tabs=windows)

dotnet CLI だけ解説します。

あらかじめ、開発プロジェクト(.csproj) を作成した上で、そのコマンドラインでプロジェクトフォルダーへ移動し、以下のコマンドを入力します。

```
dotnet package add SnowStack.EncodingProbe
```

これで、クラスライブラリの SnowStack.EncodingProbe が使用できます。

注意点として .NET 10 SDK 以降では dotnet package add、.NET 9 SDK 以前では dotnet add package を使用します。[dotnet package add](https://learn.microsoft.com/ja-jp/dotnet/core/tools/dotnet-package-add)

※ Visual Studio から NuGet パッケージを検索する場合は、「プレリリースを含む」チェックボックスをオンにして検索インストールしてください。

## EncodingProbe NuGetパッケージの使い方

### クラス構成

SnowStack.EncodingProbe を使用するには、以下の四つのクラスが関係します（うちユーザーが直接使うのは3つ）。

| クラス名                | 機能                                                         |
| ----------------------- | ------------------------------------------------------------ |
| EncodingInfomation      | 推測した文字エンコーディングの情報を格納するレコード。       |
| EncodingDetector        | 文字エンコーディングのバイナリ解析を行うメインクラス。       |
| EncodingProbe           | ユーザーが直接使用するクラスライブラリ。<br />EncodingDetector も UTF.Unknown も、ここから呼び出しています。 |
| EncodingDetectorOptions | 文字エンコーディングの推測方針を格納し、EncodingProbe に伝えるオプションパラメータ用のクラス。 |

NuGetパッケージをユーザーが使用するときに、利用するクラスは EncodingProbe とオプションパラメータ格納用の EncodingDetectorOptions になります。

EncodingProbe の文字エンコーディングの解析結果は、EncodingInfomation レコードクラスに格納されて、ユーザーに返されます。

文字エンコーディングの解析処理を行うメイン処理は EncodingDetector とサードパーティ製品の UTF.Unknown が行いますが、これらは必要に応じて EncodingProbe が呼び出しますので、ユーザーが直接使用する必要は無いです。

つまり、ユーザーが直接使用するのは、EncodingProbe とパラメータ格納用の EncodingDetectorOptions と、結果を格納している EncodingInfomation だけです。

#### UTF.Unknown について

EncodingProbe は、独自実装の EncodingDetector と、サードパーティ製品の UTF.Unknown （MITライセンス）を言語に応じて使い分けることで、文字エンコーディングの解析を行っています。

後述する EncodingDetectorOptions をデフォルトモードで使用する場合、最初に EncodingDetector による解析が行われ、解析結果が不明になった場合は、UTF.Unknown により文字エンコーディングの解析を行います。

独自実装の EncodingDetector は、ASCII コード・JISコード・Unicode・旧日本語文字コード・旧韓国語文字コード・旧繁体字中国語文字コード・旧簡体字中国語文字コードの解析を行い、それで解析結果が不明の場合は UTF.Unknown で解析します。UTF.Unknown は世界多数の言語に対応していますので、世界中の文字エンコーディングの解析が可能です。

UTF.Unknown は欧米など旧シングルバイト文字コードの解析に優れており、シングルバイト文字エンコーディングの解析は信頼できるのですが、日本語を始めとした旧マルチバイト文字エンコーディングの解析では、若干解析精度が落ちます。

また、Unicode の UTF-16 と UTF-32 は「先頭にBOMを付ける事を推奨」していますが、あくまで「推奨」であって「必須」ではないため、BOMの無い UTF-16 と UTF-32 も規格上は有り得ます。

UTF.Unknown は、BOMの無い UTF-16 と UTF-32 に対応していません。

そのため、この弱点をカバーするため、日本語やUnicodeの解析は、EncodingDetector で行っています。

日本語（及び東アジア漢字文化圏）では、EncodingDetector の解析だけで済むはずです。

### 提供されるメソッドとプロパティ

EncodingProbe は、static クラスなのでインスタンス化する必要はありません。

#### 基本メソッド Detect

文字エンコーディングの解析を行うメソッドは Detect メソッドで、解析結果として返り値に EncodingInfomation のレコードクラスを返します。

Detect メソッドには以下の三つのオーバーロードがあります。

三つのオーバーロードの違いは、解析対象ファイルの渡し方です。

(1) 解析対象をバイト並びで渡す。

```
EncodingInfomation Detect(byte[] buffer, EncodingDetectorOptions options = null)
```

解析対象ファイルを呼び出すアプリ側でバイナリモードでオープンして読み込み、その内容をバイト配列でパラメータに渡します。

アプリ側でファイルの内容を参照する必要があるとき便利な関数仕様です。

(2) 解析対象をファイルストリームで渡す。

```
EncodingInfomation Detect(Stream stream, EncodingDetectorOptions options = null)
```

解析対象ファイルを呼び出すアプリ側でファイルストリームを開いてから、そのストリームをパラメータに渡します。

テキストやバイナリといったストリームのモードをアプリ側で制御したい場合に便利な関数仕様です。

(3) 解析対象ファイルのパス名を渡す。

```
EncodingInfomation Detect(string filePath, EncodingDetectorOptions options = null)
```

解析対象ファイルのファイル名やフルパス名をパラメータに渡す関数仕様です。

一番簡単に利用できる関数仕様です。アプリ側で対象ファイルを制御できないのが欠点です。

##### EncodingDetectorOptions パラメータ

全てのオーバーロードの EncodingDetectorOptions パラメータは省略可能です。省略するとデフォルト設定で動作します。

通常の使用では、EncodingDetectorOptions パラメータは省略しても問題無いです。

EncodingDetectorOptions パラメータでは、文字エンコーディングの解析方法を選択できます。

EncodingDetectorOptions の中のパラメータ・プロパティは以下の物があります。

| プロパティ | パラメータの機能                                             |
| ---------- | ------------------------------------------------------------ |
| Strategy   | 解析手順を選択する。<br />DetectionStrategy 列挙型で指定され、デフォルトではCombinedに設定される。 |
| Culture    | 解析の基準となる言語(国)を指定する。デフォルトではOSのカルチャーに設定される。 |

DetectionStrategy 列挙型では、文字エンコーディングの解析方針を選択できるようにしています。

| DetectionStrategy のメンバ | 意味                                                         |
| -------------------------- | ------------------------------------------------------------ |
| NativeOnly                 | 文字エンコーディングの解析を、独自実装の EncodingDetector だけで行う。UTF.Unknown の補完を使用しない。 |
| UtfUnknownOnly             | 文字エンコーディングの解析を、サードパーティ製品の UTF.Unknown だけで行う。独自実装は使用しない。 |
| Combined                   | 文字エンコーディングの解析を、まず独自実装の EncodingDetector で行った上で、解析結果が不明の場合に、サードパーティ製品の UTF.Unknown で補完的に解析を実行する。<br />デフォルト設定ではこれが選択される。 |

参考までに、DetectionStrategy パラメータを省略した場合、日本語環境では { Strategy = Combined , Culture = ja-JP } となります。

###### なぜ EncodingDetectorOptions が必要なのか

文字エンコーディングの推測は、テキストファイルをバイナリモードで開き、そのコード番号をバイナリ解析することで、該当する文字エンコーディングを推測します。

アルゴリズムだけで文字エンコーディングの完全な解析は不可能です。

特に、日本語を含む東アジア漢字文化圏の Shift_JIS , EUC-JP , CP949 , BIG5 , GBK などの旧マルチバイト文字コードの構造は物によっては似通った構造をしていて、バイナリ解析だけで文字エンコーディングを識別するのは非常に困難です。

しかし、日本を含む東アジア諸国は既に Unicode をメインとするテキスト環境に移行しており、旧マルチバイト文字コードを使用するのは、古いデータや環境を使用する場合に限られます。

旧マルチバイト文字コードを東アジアの隣国で使用することは、ほとんど有り得ません。

よって、旧マルチバイト文字コードの解析は、その国の言語で使用されている旧マルチバイト文字コードの解析だけできれば事足ります。

日本語OS上で、台湾のBIG5を使用することは無いですし、韓国語OS上で日本のShift_JISを使用することもほぼありません。

よって、EncodingDetector の中でOSのカルチャー(国情報)を取得し、その国に合わせた解析処理を走らせることで、文字エンコーディングの解析精度を高めています。

デフォルト設定では自動のカルチャー設定に任せて良いはずですが、例外的に外国の旧マルチバイト文字コードを解析する必要があったときに、EncodingDetectorOptions.Culture に対象国のカルチャーを設定すると、その国の旧マルチバイト文字コードの解析処理で解析できます。

例えば、在日韓国人が日本語OS上で韓国語の可能性の高いテキストファイルの文字エンコーディングの推測を行いたいときは、EncodingDetectorOptions.Culture に "ko" を指定すると、EUC-KR や CP949 の解析処理が走ります。

台湾人が日本語OS上で繁体字中国語の推測を行いたいときは、EncodingDetectorOptions.Culture に "zh-TW" を指定すると、EUC-TW や BIG5 などの台湾文字エンコーディングの解析処理が走ります。

中国人なら、"zh-CN" です。

それぞれの母国語OSで実行するならば、EncodingDetectorOptions を省略して問題ありません。

また、UTF.Unknown だけを使用するなら、EncodingDetectorOptions を省略して問題ありません。

### 返り値 EncodingInfomation 

Detect メソッドにより文字エンコーディングを解析した結果は、EncodingInfomation レコードクラスで返されます。その内容には以下のプロパティが含まれます。

| 型            | プロパティ     | 値の種類                            | 内容解説                                                     |
| ------------- | -------------- | ----------------------------------- | ------------------------------------------------------------ |
| int           | CodePage       | コードページ                        | 解析した文字エンコーディングのコードページ                   |
| string        | EncodingName   | エンコーディング名                  | 文字エンコーディング名(Web Name)                             |
| string        | PSEncodingName | エンコーディング名                  | PowerShell用 フレンドリ名<br />（コマンドレットの -Encoding オプションに使用します） |
| bool          | Bom            | BOMの有無                           | True=BOM有り、False=BOM無し。                                |
| LineBreakType | LineBreak      | 改行コード種類（Windows型、UNIX型） | CrLf , Lf , Cr の、どれかを示す。<br />混合状態も表します。  |
| bool          | IsWindowsOs    | OSがWindowsである                   | Windowsの場合はTrue、他はFalse                               |
| bool          | IsMacOs        | OSがMacである                       | macOSの場合はTrue、他はFalse                                 |
| bool          | IsLinuxOs      | OSがLinuxである                     | Linuxの場合はTrue、他はFalse                                 |
| string        | Culture        | 国情報                              | 例として、日本=ja-JP, 韓国=ko, 台湾=zh-TW, 中国=zh-CN        |

### .NET(Core) と .NET Framework 4.8 対応

EncodingProbe NuGet パッケージ は .NET (Core) 10 と、.NET Framework 4.8 の両方に対応しています。

.NET10 用のビルドモジュールと、.NET Framework 4.8 用のビルドモジュールの両方を用意して提供しているので、どちらでも利用可能です。

### 使用法サンプル

提供するメイン機能は Detect メソッドだけであり、その引数は先に解説したように「ファイルパス名」「ファイルストリーム」「バイト配列」の三種類です。

最も簡単な使い方は、.NETコンソールアプリの場合、以下の様になります。

```
static void Main(string[] args)
{
    Console.WriteLine("Detection FileName = " + args[0]);

    var encinfo = EncodingProbe.Detect(args[0]);

    Console.WriteLine($"{args[0]} {encinfo}");
}
```

コマンドパラメータに解析対象テキストファイルのパス名を指定してコマンドを起動すると解析結果が表示されます。

Detection FileName = text1.txt
text1.txt EncodingInfomation { CodePage = 1200, EncodingName = utf-16, PSEncodingName = unicode, Bom = False, LineBreak = CrLf, IsWindowsOs = True, IsMacOs = False, IsLinuxOs = False, Culture = ja-JP }

表示されるプロパティは、EncodingInfomation のメンバです。

※ EncodingInfomation は、Information のスペルを間違っているため、後日修正します。現状では EncodingInfomation で実装しています。

## お知らせ欄

繰り返しになりますが、現在 EncodingProbe NuGet パッケージはプレリリース版であり、まだ正式版のリリースは行っておりません。

正式版リリースまでに外部仕様も変更する可能性があります。

よって、現状ではテスト利用目的以外で、EncodingProbe NuGet パッケージを利用することはお勧めできません。

正式版リリースのときは、ここでアナウンスします。

しばらくお待ちください。

