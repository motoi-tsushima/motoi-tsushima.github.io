---
title: "SnowStack.EncodingProbe NuGet Package 解説"
layout: single
classes: wide
permalink: /encodingprobe_guide/
author_profile: true
---
2026/09/30 document update

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

2026年6月6日に最初の preview1 をリリースし、何度か改修版のプレリリースを行った後、
2026年7月14日に正式版の Version 1.0.0 をリリースしました。

その後、2026年9月1日に Version 1.1.0 を、2026年9月30日に Version 1.2.0 をリリースしています。

### Version 1.2.0

**1.2.0 では、この NuGet パッケージの判定処理を改修しました。** 1.0.x・1.1.0 を使用中の方は、1.2.0 への更新をお勧めします。

主な変更は以下のとおりです。

| 変更 | 内容 |
| :---- | :---- |
| 東アジア以外の言語への対応 | 日本語などの東アジアのカルチャーで、欧米のシングルバイトのテキスト（windows-1252 のドイツ語など）を、Shift_JIS などと誤判定しなくなりました |
| UTF-8 の判定の厳密化 | 規格（RFC 3629）どおりの整形式のバイト列だけを UTF-8 と判定します |
| 香港の Big5 への対応 | 香港・マカオ・広東語のカルチャーを台湾と分けて扱い、`zh-Hant-HK` のようなカルチャー名も解釈します |
| 繁体字と簡体字の取り違えの修正 | 台湾・香港のカルチャーで簡体字を Big5 と、大陸のカルチャーで繁体字を GB18030 と誤判定しなくなりました |
| 不具合の修正 | ルーマニア語などのテキストで `NullReferenceException` が発生していたのを直しました（1.1.0 から存在した不具合） |
| 不具合の修正 | `Detect(Stream)` で、UTF.Unknown による判定が機能していなかったのを直しました |

**公開 API は変更していません。** クラス構成も、メソッドも、1.1.0 のままです。判定結果が変わるのは、上の表に挙げたケースです。

変更の詳しい内容は、以下の記事で解説しています。

[SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)

### Version 1.1.0

**1.1.0 では、この NuGet パッケージのコードを変更していません。** 公開 API も、クラス構成も、1.0.x のままです。

1.1.0 の機能追加は、すべてコマンドレット側（SnowStack.EncodingProbe.PowerShell）で行っています。

それでもこの NuGet パッケージのバージョン番号を 1.1.0 へ上げたのは、同じ配布物に含まれる DLL のバージョンが食い違わないようにするためです。

コマンドレット側で何を追加したかは、以下の記事で解説しています。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

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

~~※ Visual Studio から NuGet パッケージを検索する場合は、「プレリリースを含む」チェックボックスをオンにして検索インストールしてください。~~

※ 正式版 1.0.0 の検索には「プレリリースを含む」チェックボックスをオンにする必要はありません。

## EncodingProbe NuGetパッケージの使い方

### クラス構成

SnowStack.EncodingProbe を使用するには、以下の4つのクラスが関係します（うちユーザーが直接使うのは3つ）。

| クラス名                | 機能                                                         |
| ----------------------- | ------------------------------------------------------------ |
| EncodingInformation      | 推測した文字エンコーディングの情報を格納するレコード。       |
| EncodingDetector        | 文字エンコーディングのバイナリ解析を行うメインクラス。       |
| EncodingProbe           | ユーザーが直接使用するクラスライブラリ。<br />EncodingDetector も UTF.Unknown も、ここから呼び出しています。 |
| EncodingDetectorOptions | 文字エンコーディングの推測方針を格納し、EncodingProbe に伝えるオプションパラメータ用のクラス。 |

NuGetパッケージをユーザーが使用するときに、利用するクラスは EncodingProbe とオプションパラメータ格納用の EncodingDetectorOptions になります。

EncodingProbe の文字エンコーディングの解析結果は、EncodingInformation レコードクラスに格納されて、ユーザーに返されます。

文字エンコーディングの解析処理を行うメイン処理は EncodingDetector とサードパーティ製品の UTF.Unknown が行いますが、これらは必要に応じて EncodingProbe が呼び出しますので、ユーザーが直接使用する必要は無いです。

つまり、ユーザーが直接使用するのは、EncodingProbe とパラメータ格納用の EncodingDetectorOptions と、結果を格納している EncodingInformation だけです。

#### UTF.Unknown について

EncodingProbe は、独自実装の EncodingDetector と、サードパーティ製品の UTF.Unknown （MPL 1.1）を言語に応じて使い分けることで、文字エンコーディングの解析を行っています。

UTF.Unknown は Mozilla Universal Charset Detector の系譜にあるライブラリで、Mozilla Public License 1.1 / GPL 2 以降 / LGPL 2.1 以降 のいずれかを利用者が選択できる三重ライセンスとなっています。SnowStack.EncodingProbe では MPL 1.1 を選択し、NuGet パッケージを未改変のまま参照しています。MPL 1.1 はファイル単位の弱いコピーレフトであり、未改変のライブラリを別アセンブリとして参照する形であれば呼び出し側に義務は波及しないため、SnowStack.EncodingProbe 自体は MIT ライセンスで提供しています。

後述する EncodingDetectorOptions をデフォルトモードで使用する場合、最初に EncodingDetector による解析が行われ、解析結果が不明になった場合は、UTF.Unknown により文字エンコーディングの解析を行います。

独自実装の EncodingDetector は、ASCII コード・JISコード・Unicode・旧日本語文字コード・旧韓国語文字コード・旧繁体字中国語文字コード・旧簡体字中国語文字コードの解析を行い、それで解析結果が不明の場合は UTF.Unknown で解析します。UTF.Unknown は世界多数の言語に対応していますので、世界中の文字エンコーディングの解析が可能です。

UTF.Unknown は欧米など旧シングルバイト文字コードの解析に優れており、シングルバイト文字エンコーディングの解析は信頼できるのですが、日本語を始めとした旧マルチバイト文字エンコーディングの解析では、若干解析精度が落ちます。

また、Unicode の UTF-16 と UTF-32 は「先頭にBOMを付ける事を推奨」していますが、あくまで「推奨」であって「必須」ではないため、BOMの無い UTF-16 と UTF-32 も規格上は有り得ます。

UTF.Unknown は、BOMの無い UTF-16 と UTF-32 に対応していません。

そのため、この弱点をカバーするため、日本語やUnicodeの解析は、EncodingDetector で行っています。

日本語（及び東アジア漢字文化圏）では、EncodingDetector の解析だけで済むはずです。

##### 1.2.0 での変更 — UTF.Unknown との突き合わせ

1.1.0 までは、EncodingDetector が答えを出した時点で解析を終えていました。

ところが、EncodingDetector はバイト構造の妥当性だけを見ているので、欧米のシングルバイトのテキストが、東アジアの旧マルチバイト文字コードとして構造上成立してしまうことがあります。たとえば日本語 OS 上で windows-1252 のドイツ語を解析すると、`üß`（`FC DF`）が Shift_JIS の外字として成立し、Shift_JIS と誤判定していました。

1.2.0 では、以下の二点を変更しました。

- **東アジア以外のカルチャーでは、東アジアの旧マルチバイト文字コードの解析を行いません。** 1.1.0 でも大部分はこの動作でしたが、カルチャーと解析対象の対応表を一か所にまとめ、規則として明確にしました。ただし、BOM・ISO-2022・ASCII・Unicode（UTF-8 / UTF-16 / UTF-32）の解析は、カルチャーに関係なく行います
- **デフォルトの Combined では、EncodingDetector が東アジアの旧マルチバイト文字コードと判定した場合も、UTF.Unknown の結果と突き合わせます。** UTF.Unknown が信頼度 0.55 を超えてシングルバイト文字エンコーディングと判定した場合は、そちらを採用します

本物の日本語や中国語のテキストに対して、UTF.Unknown がシングルバイトを高い信頼度で返すことはないので、東アジアの言語の判定結果は変わりません。

同じ仕組みで、繁体字と簡体字の取り違えも直しています。UTF.Unknown が EncodingDetector と反対の系統（Big5 系と GB 系）を信頼度 0.8 以上で返した場合は、その系統で解析をやり直します。

なお、おおむね 90 バイト以下の短いシングルバイト系のテキストでは、UTF.Unknown の精度の限界により、東アジアのカルチャーで誤判定する場合が残っています。

##### UTF.Unknown のバージョン

参照している UTF.Unknown のバージョンは、ターゲットフレームワークによって分けています。

| ターゲット | UTF.Unknown |
| :---- | :---- |
| .NET 10 | 2.7.0 |
| .NET Framework 4.8 | 2.6.0 |

これは揃え忘れではなく、意図的な非対称です。

UTF.Unknown 2.7.0 の netstandard2.0 向けアセットは、System.Memory に依存するようになりました。System.Memory は厳密名付きのアセンブリで、.NET Framework では参照アセンブリと実行時アセンブリのバージョンが食い違います。

通常は app.config のバインディングリダイレクトで解決しますが、バイナリモジュールを Import-Module する Windows PowerShell 5.1 のホストには、app.config を差し込めません。

.NET Framework 4.8 側では 2.7.0 で追加された API を使っておらず、2.6.0 と 2.7.0 で解析結果も変わりません。リスクだけを負う変更になるので、据え置きました。

### 提供されるメソッドとプロパティ

EncodingProbe は、static クラスなのでインスタンス化する必要はありません。

#### 基本メソッド Detect

文字エンコーディングの解析を行うメソッドは Detect メソッドで、解析結果として返り値に EncodingInformation のレコードクラスを返します。

Detect メソッドには以下の三つのオーバーロードがあります。

三つのオーバーロードの違いは、解析対象ファイルの渡し方です。

(1) 解析対象をバイト並びで渡す。

```
EncodingInformation Detect(byte[] buffer, EncodingDetectorOptions options = null)
```

解析対象ファイルを呼び出すアプリ側でバイナリモードでオープンして読み込み、その内容をバイト配列でパラメータに渡します。

アプリ側でファイルの内容を参照する必要があるとき便利な関数仕様です。

(2) 解析対象をファイルストリームで渡す。

```
EncodingInformation Detect(Stream stream, EncodingDetectorOptions options = null)
```

解析対象ファイルを呼び出すアプリ側でファイルストリームを開いてから、そのストリームをパラメータに渡します。

テキストやバイナリといったストリームのモードをアプリ側で制御したい場合に便利な関数仕様です。

※ 1.1.0 までの Detect(Stream) には、独自実装の解析でストリームを読み切った後のストリームを UTF.Unknown に渡してしまい、UTF.Unknown による解析が機能しない不具合がありました。1.2.0 で修正し、ストリームを一度だけ読んで両方の解析に渡すようにしています。ストリームを渡して使用している方は、1.2.0 へ更新してください。

(3) 解析対象ファイルのパス名を渡す。

```
EncodingInformation Detect(string filePath, EncodingDetectorOptions options = null)
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
| Combined                   | 文字エンコーディングの解析を、まず独自実装の EncodingDetector で行った上で、解析結果が不明の場合に、サードパーティ製品の UTF.Unknown で補完的に解析を実行する。<br />1.2.0 からは、EncodingDetector が東アジアの旧マルチバイト文字コードと判定した場合も、UTF.Unknown の結果と突き合わせる。<br />デフォルト設定ではこれが選択される。 |

参考までに、EncodingDetectorOptions パラメータを省略した場合、日本語環境では { Strategy = Combined , Culture = ja-JP } となります。

###### なぜ EncodingDetectorOptions が必要なのか

文字エンコーディングの推測は、テキストファイルをバイナリモードで開き、そのコード番号をバイナリ解析することで、該当する文字エンコーディングを推測します。

アルゴリズムだけで文字エンコーディングの完全な解析は不可能です。

特に、日本語を含む東アジア漢字文化圏の Shift_JIS , EUC-JP , CP949 , BIG5 , GBK などの旧マルチバイト文字コードの構造は物によっては互いに似通っていて、バイナリ解析だけで文字エンコーディングを識別するのは非常に困難です。

しかし、日本を含む東アジア諸国は既に Unicode をメインとするテキスト環境に移行しており、旧マルチバイト文字コードを使用するのは、古いデータや環境を使用する場合に限られます。

旧マルチバイト文字コードを東アジアの隣国で使用することは、ほとんど有り得ません。

よって、旧マルチバイト文字コードの解析は、その国の言語で使用されている旧マルチバイト文字コードの解析だけできれば事足ります。

日本語OS上で、台湾のBIG5を使用することは無いですし、韓国語OS上で日本のShift_JISを使用することもほぼありません。

よって、EncodingDetector の中でOSのカルチャー(国情報)を取得し、その国に合わせた解析処理を走らせることで、文字エンコーディングの解析精度を高めています。

デフォルト設定では自動のカルチャー設定に任せて良いはずですが、例外的に外国の旧マルチバイト文字コードを解析する必要があったときに、EncodingDetectorOptions.Culture に対象国のカルチャーを設定すると、その国の旧マルチバイト文字コードの解析処理で解析できます。

例えば、在日韓国人が日本語OS上で韓国語の可能性の高いテキストファイルの文字エンコーディングの推測を行いたいときは、EncodingDetectorOptions.Culture に "ko" を指定すると、EUC-KR や CP949 の解析処理が走ります。

台湾人が日本語OS上で繁体字中国語の推測を行いたいときは、EncodingDetectorOptions.Culture に "zh-TW" を指定すると、EUC-TW や BIG5 などの台湾文字エンコーディングの解析処理が走ります。

中国人なら、"zh-CN" です。

香港なら "zh-HK" です。1.2.0 からは、香港・マカオ（"zh-HK" / "zh-MO"）と広東語（"yue"）を台湾と分けて扱い、台湾の文字エンコーディングである EUC-TW を候補から外します。

ただし、台湾の Big5 と香港の Big5（HKSCS）はバイト構造から区別できないので、判定結果はどちらも `950 / big5` になります。HKSCS 固有の文字は、Unicode の私用領域（U+E000〜U+F8FF）の文字として読み込まれ、同じ Big5 で書き戻せば元のバイト列に戻ります。

私用領域の文字（日本語の外字を含みます）の扱いは、以下の記事で解説しています。EUC-JP の外字を扱う場合は、`EncodingWebName` ではなく `CodePage` でエンコーディングを作る必要があるので、ご注意ください。

[外字（私用領域）の扱い — SnowStack.EncodingProbe](/encodingprobe_private_use_area/)

1.2.0 からは、カルチャー名を「言語・用字・地域」に分解して解釈するので、.NET 10 が使う "zh-Hant-HK" や "zh-Hans-CN" のような形のカルチャー名も指定できます。

それぞれの母国語OSで実行するならば、EncodingDetectorOptions を省略して問題ありません。

また、UTF.Unknown だけを使用するなら、EncodingDetectorOptions を省略して問題ありません。

### 返り値 EncodingInformation 

Detect メソッドにより文字エンコーディングを解析した結果は、EncodingInformation レコードクラスで返されます。その内容には以下のプロパティが含まれます。

| 型            | プロパティ      | 値の種類                            | 内容解説                                                     |
| ------------- | --------------- | ----------------------------------- | ------------------------------------------------------------ |
| int           | CodePage        | コードページ                        | 解析した文字エンコーディングのコードページ                   |
| string        | EncodingWebName | エンコーディング名                  | 文字エンコーディング名(Web Name)                             |
| string        | PSEncodingName  | エンコーディング名                  | PowerShell用 フレンドリ名<br />（コマンドレットの -Encoding オプションに使用します） |
| bool          | UsePSName       | フレンドリ名が有効                  | PSEncodingName に有効なフレンドリ名が入っている場合は True に、Web Name や空白が入っている場合は False となります。 |
| bool          | Bom             | BOMの有無                           | True=BOM有り、False=BOM無し。                                |
| LineBreakType | LineBreak       | 改行コード種類（Windows型、UNIX型） | CrLf , Lf , Cr の、どれかを示す。<br />混合状態も表します。  |
| string        | Culture         | 国情報                              | 例として、日本=ja-JP, 韓国=ko, 台湾=zh-TW, 中国=zh-CN        |

PSEncodingName の値は、PowerShell 6.2 以降であれば、UsePSName の値に関係無く Get-Content 等の -Encoding で使用できます。PowerShell 6.2 以降は WebName も -Encoding が受け付けるからです。

但し、`UsePSName = False` は「標準コマンドの -Encoding にそのまま渡せる保証はない」という意味の値です。Windows PowerShell 5.1 では実際に渡せない場合があるので、その点は後述します。

preview5 では `UsePSName = False` の場合に CodePage を返していましたが、この仕様は廃止し、`EncodingWebName` を返すようにしました。

Windows PowerShell 5.1 ではBOM無しのUTF-8やUTF-16などUnicode文字エンコーディングが全て使用できないので、その場合は PSEncodingName に空白を返します。

PowerShell 6.2 以降の場合は、BOM無しでも PSEncodingName に該当するフレンドリ名を返します。 

UTF.Unknown は、.NET が提供していない文字エンコーディング（ルーマニア語の ISO-8859-16 など）を判定結果として返すことがあります。その場合、EncodingInformation の CodePage は -1 となり、EncodingWebName に UTF.Unknown が返した名前（`iso-8859-16` など）が入ります。1.1.0 まではこの場合に `NullReferenceException` が発生していましたが、1.2.0 で修正しました。

### EncodingDetector 

通常の利用では、EncodingDetector を開発者が直接使用する必要はありません。

EncodingProbeクラスのDetectメソッドを使用するだけで充分のはずです。 

よって解説を省略させて頂きます。

### .NET(Core) と .NET Framework 4.8 対応

EncodingProbe NuGet パッケージは .NET(Core)10 と、.NET Framework 4.8 の両方に対応しています。

.NET10用のビルドモジュールと、.NET Framework 4.8 用のビルドモジュールの両方を用意して提供しているので、どちらでも利用可能です。

注意点として、PowerShell 6.2 以上と Windows PowerShell 5.1 では、標準の文字エンコーディングのフレンドリ名は異なる値が定義されています。

よって、PSEncodingName のフレンドリ名は、PowerShell 6.2以上と Windows PowerShell 5.1 で異なるフレンドリ名を返します。

Windows PowerShell 5.1 では、BOM無しのUnicode文字エンコーディングを扱わない仕様になっているので、PS5.1ではBOM無しのUnicodeの場合、PSEncodingName に空白を返します。Shift_JISもフレンドリ名が用意されていないので同様に空白を返します。

PowerShell 6.2 以上では正常なフレンドリ名を返します。

この辺の詳しい解説は、以下の記事で行っています。

[PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか （フレンドリ名の解説）](/why-utf8nobom-cannot-be-webname/)

なお、この「Windows PowerShell 5.1 では判定結果を標準コマンドへ渡せない」という問題は、コマンドレット側の Version 1.1.0 で解決しました。

標準コマンドに橋を架けるのをやめ、自前の読み書きコマンドを持つ方向へ舵を切っています。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

### 使用法サンプル

提供するメイン機能は Detectメソッドだけであり、その引数は先に解説したように「ファイルパス名」「ファイルストリーム」「バイト配列」の三種類です。

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
text1.txt EncodingInformation { CodePage = 1200, EncodingWebName = utf-16, PSEncodingName = unicode, Bom = False, LineBreak = CrLf, Culture = ja-JP }

表示されるプロパティは、EncodingInformation のメンバです。

## ライセンス

ライセンスは **MITライセンス** です。

アプリ開発や商用などに利用する場合は、以下のライセンス表記を、ユーザーが閲覧可能な場所に表記してください。

```
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

## お知らせ欄

### 2026年9月30日　Version 1.2.0 リリース

Version 1.2.0 をリリースしました。

1.1.0 とは違い、**今回はこの NuGet パッケージの判定処理を改修しています。** 公開 API は変更していません。

東アジアの環境で欧米の言語のテキストを誤判定する問題、繁体字と簡体字の取り違え、香港のカルチャーの扱いを直しました。ルーマニア語などで例外が発生する不具合と、Detect(Stream) の不具合も修正しています。

1.0.x・1.1.0 をご利用中の方は、1.2.0 への更新をお勧めします。

[SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)

### 2026年9月1日　Version 1.1.0 リリース

Version 1.1.0 をリリースしました。

繰り返しになりますが、**この NuGet パッケージのコードは 1.1.0 で変更していません。** バージョン番号だけを、コマンドレット側と揃えています。

1.0.2 をご利用中の方が、更新する必要はありません。

機能追加は、すべてコマンドレット側で行っています。テキストファイルの読み書きを行う 4 つのコマンドレットを追加しました。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)

[-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)

### 2026年7月14日　正式版リリース

EncodingProbe NuGet パッケージの正式版をリリースしました。

Version 1.0.0 のリリース日は、2026年7月14日になります。

このとき「詳細ドキュメント類は、しばらくお待ちください」と書いていましたが、フレンドリ名の解説と、コマンドレット 1.1.0 の解説で、必要な内容は書き終えたと考えています。

クラスライブラリとして使う場合の説明は、この記事で足りるはずです。

## 関連資料

- [SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)
- [外字（私用領域）の扱い — SnowStack.EncodingProbe](/encodingprobe_private_use_area/)
- [SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)
- [SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](/encodingprobe_probed_content/)
- [-Culture と -Strategy の解説 — 外国語のテキストファイルを読む](/encodingprobe_culture_strategy/)
- [PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか （フレンドリ名の解説）](/why-utf8nobom-cannot-be-webname/)
- [SnowStack.EncodingProbe（GitHub リポジトリ）](https://github.com/motoi-tsushima/SnowStack.EncodingProbe)

