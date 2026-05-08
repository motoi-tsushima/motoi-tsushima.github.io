---
title: "Avalonia UI のライセンス規定を解説します"
layout: single
classes: wide
permalink: /avaloniaui-license-guide/
author_profile: true
---

## Avalonia UI 概要

この記事のライセンス規定の内容は、2026年5月時点の情報を元に書かれています。ライセンス規定は変更されることがありますので、ご注意ください。（最終更新日：2026年5月9日）

Avalonia UI は、.NET のクロスプラットフォームGUIフレームワークです。

.NETのフレームワークは、ほとんどがMicrosoft標準の物ですが、Avalonia はエストニアの **AvaloniaUI OÜ**から提供されているサードパーティ製品で、Microsoftとの資本関係はありません。（以降、Avalonia社 と呼称します）

Avalonia の基本的な解説は以下の記事に書きましたので、ここでは解説しません。

[Avalonia UI アーキテクチャパターン](https://snow-stack.net/dotnet10-GUI-architecture-list/#avalonia-ui-%E3%82%A2%E3%83%BC%E3%82%AD%E3%83%86%E3%82%AF%E3%83%81%E3%83%A3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3)

Avalonia のライセンス規定は、わかりにくいので、この記事で詳しく解説しておきたいと思います。

### Avaloniaライセンス規定は2026年4月に変更された

Avaloniaはオープンソース製品として10年のリリース実績を持ちますが、最近ライセンス規定の変更を行いました。

2026年3月〜4月にかけて、Schedule 1系の旧 Accelerate ライセンスから、Schedule 8〜12 系の新ライセンス体系に再編され、2026年4月7日に、最新の Avalonia 12 がリリースされました。

特に重要な変更点として、無料と有料の範囲が変更されたので、最新のライセンス規定を良く理解しておく必要があります。今回の変更で、それまで無料で使用できていたユーザーの一部が有料枠に含まれる可能性があるので、無料枠と有料枠の違いを明確にしておく必要があります。

Avalonia の公式サイトは以下のリンクになります。

[**Avalonia 公式サイト**](https://avaloniaui.net/)

今後、ライセンス規定の変更が行われるかどうかわかりませんが、ライセンス規定の内容から考えてかなり堅実な方針になったので、今後変更の可能性は少ないと個人的には考えています。

## 1. Avaloniaライセンスの解説

### Avaloniaライセンスは本体と開発支援ツールで異なる

Avalonia のライセンスを理解する上で、注意しなければならない**要点**があります。

Avalonia は単独で動くソフトウェアではなく、アプリの開発者が部品として利用するコンポーネントフレームワークです。Avaloniaのライセンスは、**コンポーネント本体**と、**コンポーネントを使用した開発を支援するツール群**とで、それぞれ**異なるライセンス**を定めています。

### 1-1. Avalonia本体は無料

最新の Avalonia 12 以前から現在でも一貫して、**Avaloniaコンポーネント本体**は、**オープンソース**の**MITライセンス**で提供されており**無料**で使用できます。

MITライセンスとは、プログラムを利用・再配布する場合、著作権表示とライセンス条文を、ソフトウェアの全ての複製、または重要な部分に含めることを義務づけるライセンスです。また、本ソフトウェアは無保証であり、利用者の自己責任で使用することに合意することが求められます。

**商用利用は制限されません**が、**著作権表示**と**利用者の自己責任**での利用が義務づけられます。

[Avalonia License(MIT)](https://avaloniaui.net/legal/13)

### 1-2. 開発支援ツールは有料版と無料版がある

Avalonia には、コンポーネント本体とは別に**開発支援ツール群**が提供されています。Avaloniaライセンス規定でわかりにくいのは、この開発支援ツール群のライセンスです。

Avalonia 社が提供する有料の開発ツールおよびコンポーネント群は「Avalonia Accelerate」と総称されます。

開発支援ツール群は以下の四つに分類できます。

#### (1) IDE拡張

Avaloniaによる.NET開発は、Visual Studio による開発と、VS Code による開発の二種類の開発方式がMicrosoftだけでなくAvalonia社からも提供されており、それぞれのAvalonia開発の開発ツール（IDE）拡張機能が提供されています。

それぞれの正式な名称は、**Avalonia for Visual Studio** (Visual Studio 拡張) と **Avalonia for Code** (VS Code 拡張) と呼びます。

#### (2) DevTools

Avalonia DevToolsは、Avalonia UIアプリケーションのデバッグ、UI検査、プロパティ編集をリアルタイムに行える強力な診断ツールです。デバッグ実行中にF12キーで呼び出し、ビジュアルツリーの確認、レイアウト調整、イベント監視などをブラウザのデベロッパーツールの感覚で実行可能です。

#### (3) Parcel

Avalonia Parcelは、Avalonia UIで作成されたアプリをWindows、macOS、Linux向けに署名・パッケージ化・配布するための専用ツールです。CLI（コマンドライン）とGUIの両方で動作し、複雑なクロスプラットフォームのインストーラー作成を効率化します。

#### (4) Components

Avalonia Components（Pro 以上の有料プランで利用できる premium UI コンポーネント群）とは、オープンソースのクロスプラットフォームUIフレームワーク「Avalonia UI」を開発・運営するAvalonia社が提供する、有料の拡張コンポーネントとプロフェッショナルツールセットです。

（公式サイト上では「Pro components」「premium UI components」という表現が使われています）

無償版（MITライセンス）のコアフレームワークを補完するコンポーネントで、企業レベルの高度な開発を支援するために設計されています。

具体的な対象は:

- **MediaPlayer**(動画・音声プレイヤー)
- **On Screen Keyboard**(仮想キーボード)
- **Markdown Viewer**(Markdown レンダラー)
- **TreeDataGrid**(高機能データグリッド)
- **RichTextEditor**(近日公開)

となります。

### 1-3. 開発ライセンス契約の種類

ライセンス規定は、Avalonia法的文書センター([avaloniaui.net/legal](https://avaloniaui.net/legal/))に掲載されています。

Avalonia のライセンスは合計 9 つの Schedule（附則） に分かれており、一見複雑に見えますが、3 つの軸の組み合わせで決まっています。

•	軸 1 - 機能レベル: Free → Community → Plus → Pro → Enterprise の順で、利用できる機能が拡大する
•	軸 2 - 購入主体: Individual(個人が自費購入)か Organisation(法人・組織が購入)か
•	軸 3 - 利用形態: 通常契約か Trial(試用)か

「軸3」については、他のソフトウェア販売によく見られるように、通常購入契約か「期間限定の体験版契約」としてのTrial版契約の違いです。製品として利用する場合は、「軸1」と「軸2」の組み合わせだけで考えて問題ありません。

Plus と Pro には、それぞれ Individual 版と Organisation 版が存在します。また、Pro には加えて Trial 版が存在します。

具体的には以下の種類になります。

> Free , Community ,
>
> Plus Individual , Plus Organisation ,
>
> Pro Individual , Pro Organisation , Pro Trial
>
> Enterprise 

### 1-4. 開発支援製品の種類

ライセンス契約の種類と別に、もう一つ、製品の種類を理解しておく必要があります。

製品の種類は Essentials と Complete の二つです。

#### (1) Essentials

原則として商用利用が不可ですが、無料で使用できるライセンス製品です。

但し、例外規定([Schedule 12](https://avaloniaui.net/legal/12))として **Avalonia for Code** (VS Code 拡張) だけは、商用利用が可能で完全無料となります。

**Avalonia for Visual Studio** (Visual Studio 拡張)  と DevTools・Parcel は、商用利用が不可です。

基本的な XAML 編集機能を提供しています。視覚的デザイン機能は提供されず、テキストでXAML編集しながら開発を行うことになります。

#### (2) Complete

有料で、商用利用が可能なライセンス製品です。例外規定はありません。

Essentials には無い「視覚的に UI をデザインできる数々の機能」や「モバイルアプリのデバッグ機能」、「パフォーマンス分析機能」など、より高度な機能が提供されています。

### 1-5. 機能レベルの解説

機能レベルによって、機能と有料・無料と価格が変わってきます。

価格と機能数の順に並べると以下の順になります。

Free → Community → Plus → Pro → Enterprise

#### (1) Free

Avaloniaからオープンソースで無償提供されているソフトウェアとツールを、**無料で使用できる**契約と機能レベルです。

利用できるツールは**制限**されますが、**商用利用も許されます**。

Free 契約を定義すると、Avalonia 本体(MIT) + Avalonia for Code Essentials （VS Code 拡張のこと）の組み合わせのみで開発する行為が、Free 契約に該当します。

開発者用のAvaloniaユーザーアカウントを作成することなく、Avaloniaをダウンロードして開発した場合、このFree 契約をしていることになります。

使用している開発支援ツール群のライセンス製品は、**Essentials** になります。

使用できる開発支援ツール製品は、**Avalonia for Code (VS Code 拡張)** の Essentials 版になります。
Essentials 版ではありますが、Avalonia for Code (VS Code 拡張) を使用して開発したアプリは、例外的に商用利用が可能となります。

#### (2) Community

Avaloniaから提供されているソフトウェアとツールを、**無料で使用できる**契約と機能レベルです。

基本的に**非商用に限って**利用が**許**されます。**商用利用は認められません**。

Community 契約を定義すると、Avalonia 本体(MIT) + Visual Studio 拡張・DevTools・Parcel の Essentials も使うのに必要な無償ライセンスと言えます。

開発者用のAvaloniaユーザーアカウントを無償で作成した場合、この Community 版の契約をしていることになります。

Avalonia の **開発支援ツール**(Visual Studio 拡張・DevTools・Parcel)は使用状況テレメトリを Avalonia 社へ送信しており、Community ではテレメトリの無効化ができない仕様です。

テレメトリはPlusやProなど有料版では無効化できます。

使用している開発支援ツール群のライセンス製品は、**Essentials** になります。

使用できる開発支援ツール製品は、Avalonia for Code (VS Code 拡張) ・Avalonia for Visual Studio (Visual Studio 拡張) ・DevTools・Parcel  の各Essentials版です。

[Schedule 8: Avalonia Community Licence](https://avaloniaui.net/legal/8)

#### (3) Plus

Avaloniaから提供されているソフトウェアとツールを、**有料で使用できる**契約と機能レベルです。

**商用利用も許されます**。

開発者用のAvaloniaユーザーアカウントを作成して、Plus版の登録をすると、この Plus 版の契約を選択できます。Plus 契約登録後に使用状況テレメトリは無効化されます。

有料契約で、価格は月17ドルのサブスクリプション契約となります。

使用している開発支援ツール群のライセンス製品は、Complete になります。

使用できる開発支援ツール製品は、Avalonia for Code (VS Code 拡張) ・Avalonia for Visual Studio (Visual Studio 拡張) ・DevTools・Parcel  の各Complete版です。

[Schedule 9A: Avalonia Plus Licence (Individual)](https://avaloniaui.net/legal/9A)

[Schedule 9B: Avalonia Plus Licence (Organisation)](https://avaloniaui.net/legal/9B)

#### (4) Pro

Avaloniaから提供されているソフトウェアとツールを、**有料で使用できる**契約と機能レベルです。

**商用利用も許されます**。

開発者用のAvaloniaユーザーアカウントを作成して、Pro版の登録をすると、この Pro 版の契約を選択できます。Pro 契約登録後に使用状況テレメトリは無効化されます。

有料契約で、価格は月49ドルのサブスクリプション契約となります。

使用している開発支援ツール群のライセンス製品は、Complete になり、プレミアム UI コンポーネントが利用できるようになります。

使用できる開発支援ツール製品は、Avalonia for Code (VS Code 拡張) ・Avalonia for Visual Studio (Visual Studio 拡張) ・DevTools・Parcel  の各Complete版と、以下の**UI コンポーネント**です。

- MediaPlayer(動画・音声プレイヤー)

- TreeDataGrid(Pro 版、高機能データグリッド)

- Markdown Viewer

- On Screen Keyboard(仮想キーボード)

- RichTextEditor(近日公開)

**UI コンポーネントは、この Pro 版でしか利用できません**。

また、**DevTools MCP Server**・**Parcel MCP Server** を用いたAIエージェントも利用できます。

**Priority GitHub Responses** により、Avalonia の OSS リポジトリで Issue や Discussion を立てた際、Pro 以上のユーザーには優先的に対応されます。

[Schedule 10A: Avalonia Pro Licence (Individual)](https://avaloniaui.net/legal/10A)

[Schedule 10B: Avalonia Pro Licence (Organisation)](https://avaloniaui.net/legal/10B)

#### (5) Enterprise

Avaloniaから提供されているソフトウェアとツールを、**企業組織がビジネスで使用できる**契約と機能レベルです。

ビジネス用なので当然の事ながら、**商用利用も許されます**。

有料契約ですが、価格とサービスの詳細はAvalonia社との相談で決まります。

Enterpriseが提供するサービスは、Proが提供するサービスに加えて、以下のサービスが付加されます。

•    **1. ソースコードアクセス:** Avalonia 本体は MIT で公開されているが、Pro UI コンポーネントのソースは通常は非公開。Enterprise のみソースコードを閲覧・内部改変可能。デバッグ・トラブルシューティング目的に限定。

•    **2. SLA 付き直接エンジニアリングサポート:** 1 営業日以内応答の SLA(2024 年実績は平均 11 時間で初回応答、16 時間で解決)。Avalonia フレームワークを構築している本人たちが直接対応。

•    **3. プライベート issue tracker:** 公開 GitHub ではなく、専用のプライベートチケットシステムで質問・報告。秘匿情報も扱える。

•    **4. Slack Connect 連携:** 自社 Slack ワークスペースから Avalonia エンジニアと直接コミュニケーション。チケット起票よりも即応的。

Enterprise 契約を必要とする組織像は、

「本番障害時に Avalonia 内部を理解するエンジニアと直接話したい」

「規制業界(金融、医療、政府系)でソースコード保有や監査要件がある」

「10 年以上動かす予定で、自前メンテナンス可能性を担保したい」

といった、Avalonia を自社の中核技術として戦略的に採用しており、開発元と深い関係を持ちたい法人向けの契約形態です。

### 1-6. 購入主体の解説

Individual(個人が自費購入)か Organisation(法人・組織が購入)かを機能レベルごとに区別してライセンス契約を分割しています。

Community は Individual のみですが、Plus と Pro には、それぞれ Individual と Organisation の二種類のライセンスがあります。また、Pro には Trial 版も存在します。

Free に Individual と Organisation の区別はありません。

Enterprise は法人組織専用の契約のため、Individual / Organisation の区別表示は設けられていません。

#### (1) Individual (個人が自費購入)

自然人(natural person)である個人が自分自身のために自費購入する。法人、企業、パートナーシップ、団体は購入できない。

#### (2) Organisation (法人・組織が購入)

法人・組織が従業員や契約者に提供する目的で購入する。Named User(指名ユーザー)単位で課金。

### 1-7. ライセンス契約一覧

Avalonia の法的文書センター([avaloniaui.net/legal](https://avaloniaui.net/legal/))に掲載されている Schedule の一覧を、理解しやすい順序で整理します。

| **Schedule**                                     | **プラン**      | **購入主体**    | **料金**              | **特徴**                                                     |
| ------------------------------------------------ | --------------- | --------------- | --------------------- | ------------------------------------------------------------ |
| [(MIT)](https://avaloniaui.net/legal/13)         | Avalonia(本体)  | 全員            | 無料                  | MIT ライセンス、フレームワーク本体、商用 OK                  |
| [Schedule 8](https://avaloniaui.net/legal/8)     | Community       | Individual のみ | 無料                  | 非商用限定、開発支援ツールの Essentials、テレメトリ必須      |
| [Schedule 9A](https://avaloniaui.net/legal/9A)   | Plus            | Individual      | 月17ドル, 年135ドル   | 自費購入、開発支援ツール Complete、商用 OK                   |
| [Schedule 9B](https://avaloniaui.net/legal/9B)   | Plus            | Organisation    | 年345ドル(seat課金)   | 法人購入、Named  User 単位                                   |
| [Schedule 10A](https://avaloniaui.net/legal/10A) | Pro             | Individual      | 月49ドル, 年405ドル   | Plus Individual +  Pro UI コンポーネント                     |
| [Schedule 10B](https://avaloniaui.net/legal/10B) | Pro             | Organisation    | 年1,050ドル(seat課金) | Plus Organisation  + Pro UI コンポーネント                   |
| [Schedule 10C](https://avaloniaui.net/legal/10C) | Pro Trial       | (試用)          | 無料                  | Pro 機能の試用ライセンス、評価目的のみ                       |
| [Schedule 11](https://avaloniaui.net/legal/11)   | Enterprise      | Organisation    | 年8,259ドル(seat課金) | Pro 全機能 + ソースコードアクセス + SLA 付き直接サポート + Slack Connect 連携 + プライベート issue tracker |
| [Schedule 12](https://avaloniaui.net/legal/12)   | Code Essentials | 全員            | 無料                  | VS Code 拡張、商用 OK、アカウント不要、特殊扱い              |

### 1-8. 機能レベル（契約）別の機能対応表

料金ページの比較表に基づく機能レベル（契約）階層別の機能対応関係を一覧します。

列が契約機能レベル、行が提供される具体的な機能製品です。

先に解説したように **Essentials**版は原則**無料**・商用利用**不可**（例外規定あり）の製品で、**Complete**版は**有料**・商用利用**可能**の製品です。

[Avalonia Pricing](https://avaloniaui.net/pricing) : 料金ページ

| **機能**                        | **Free** | **Community** | **Plus** | **Pro**  | **Enterprise** |
| ------------------------------- | -------- | ------------- | -------- | -------- | -------------- |
| Avalonia 本体(MIT)              | ✓        | ✓             | ✓        | ✓        | ✓              |
| 商用利用                        | ✓        | 不可          | ✓        | ✓        | ✓              |
| VS 拡張                         | —        | Essentials    | Complete | Complete | Complete       |
| DevTools                        | —        | Essentials    | Complete | Complete | Complete       |
| Parcel                          | —        | Essentials    | Complete | Complete | Complete       |
| Pro UI コンポーネント           | —        | —             | —        | ✓        | ✓              |
| Priority GitHub  Responses      | —        | —             | —        | ✓        | ✓              |
| ソースコードアクセス(Pro  含む) | —        | —             | —        | —        | ✓              |
| SLA 付きサポート                | —        | —             | —        | —        | ✓              |

## 2. 商用利用の定義

商用利用や非商用利用の定義は、Avalonia公式サイトの以下のページに記載されています。

[Schedule 8: Avalonia Community Licence](https://avaloniaui.net/legal/8)

以下は一部、引用したものと、その翻訳です。

> "Commercial Use" means any use of the Software to develop commercial products (products distributed or made available for a fee, or used as part of the Customer's business activity) or to provide paid services. Creation of audiovisual content, where the use of the Software is only demonstrated by the Customer for educational or other non-commercial purposes, is not considered Commercial Use, even if the content itself is made available for a fee or profit.
>
> "Non-Commercial Use" means any use that does not fall under the definition of Commercial Use.
>
> 「商用利用」とは、商用製品（有料で配布または提供される製品、または顧客の事業活動の一部として使用される製品）の開発、または有料サービスの提供を目的としたソフトウェアの利用を意味します。お客様が教育目的またはその他の非営利目的でソフトウェアの使用を実演するだけの視聴覚コンテンツの作成は、たとえコンテンツ自体が有料または営利目的で提供される場合でも、商用利用とはみなされません。
>
> 「非商用利用」とは、商用利用の定義に該当しないあらゆる利用を意味します。

#### 非商用・商用の具体例

#### 非商用として Community で合法な活動

•    Avalonia の学習・自己教育

•    個人的な趣味プロジェクト

•    商業的利益を得ていない OSS 貢献

•    チュートリアル、ブログ記事、講座などのコンテンツ作成

•    業務時間外の個人的な技術検証

#### 商用として Plus 以上が必要な活動

•    有償ソフトウェアの開発(販売前の開発段階を含む)

•    所属先の業務で Avalonia を使用する開発

•    受託・フリーランス案件での Avalonia 開発

•    業務時間内に Avalonia の検証・PoC を行うこと

•    広告収益等で直接収益化されるサービスの開発

## 3. 製品ページ

| **製品**                                                     | **URL**                               |
| ------------------------------------------------------------ | ------------------------------------- |
| Avalonia 12 概要                                             | https://avaloniaui.net/avalonia       |
| Avalonia for Code                                            | https://avaloniaui.net/vscode         |
| Avalonia for  Visual Studio                                  | https://avaloniaui.net/visual-studio  |
| Avalonia for VS  12.0 リリースノート(Complete機能の明記あり) | https://avaloniaui.net/whats-new/12-0 |
| DevTools                                                     | https://avaloniaui.net/devtools       |
| Parcel                                                       | https://avaloniaui.net/parcel         |

### 3-1. Pro コンポーネント個別ページ

| **コンポーネント**  | **URL**                                   |
| ------------------- | ----------------------------------------- |
| Media Player        | https://avaloniaui.net/media-player       |
| On Screen  Keyboard | https://avaloniaui.net/on-screen-keyboard |
| Markdown Viewer     | https://avaloniaui.net/markdown-renderer  |
| Tree Data Grid      | https://avaloniaui.net/tree-data-grid     |

### 3-2. ポータル・コミュニティ

| **項目**                 | **URL**                                                      |
| ------------------------ | ------------------------------------------------------------ |
| Avalonia ポータル        | https://portal.avaloniaui.net                                |
| Community ライセンス取得 | [portal.../tier=community](https://portal.avaloniaui.net/avalonia/get-license?tier=community) |
| ドキュメント             | https://docs.avaloniaui.net/                                 |
| GitHub                   | https://github.com/AvaloniaUI/Avalonia                       |

## 4. ライセンスの選択肢

最後に、簡単にライセンスの選択肢をガイドします。

（日本円価格は、為替レート 1ドル＝155円 換算の金額です）

### 4-1. 無償配布なら

フリーソフトの配布など収益の発生しない非営利目的でのAvaloniaの利用ならば、Free と Community が選択できます。

使用できる製品は Essentials版に限られます。

### 4-2. 個人開発（商用）なら

商用目的でソフトウェアを配布するのなら、ソフトウェア製品販売でも受託開発でも社内システム開発でも有料契約のPlusかPro契約を交わす必要があります。

個人でソフトウェア製品販売するならば、Plus か Pro の Individual 契約を選択することになります。

[Schedule 9A: Avalonia Plus Licence (Individual)](https://avaloniaui.net/legal/9A)

価格は、

Plus Individual なら、月17ドル（2,635円）, 年135ドル（20,925円）のサブスクリプション

Pro Individual なら、月49ドル（7,595円）, 年405ドル（62,775円）のサブスクリプションです。

（Annual 契約なら 2 ヶ月分お得になります）

Pro の購入前には 30 日無料の Trial 版で試用できます([Schedule 10C](https://avaloniaui.net/legal/10C))。

もし、便利な開発支援ツールを使用することなく、Avalonia本体と**Avalonia for Code (VS Code 拡張)** の **Essentials** 版だけでアプリを開発し、販売するのならば Free契約（無料）でも商用利用が許されます。

この場合は、個人だけではなく、法人組織でも商用利用が許されます。

[Schedule 12: Avalonia for Code (Essentials)](https://avaloniaui.net/legal/12)

### 4-3. 法人開発（商用）配布なら

営利目的法人がソフトウェアを開発しビジネスに利用するのならば、PlusかProかEnterprise契約を交わす必要があります。

法人でソフトウェア製品販売するならば、Plus か Pro の Organisation 契約かEnterprise契約を選択することになります。

[Schedule 9B: Avalonia Plus Licence (Organisation)](https://avaloniaui.net/legal/9B)

[Schedule 10B: Avalonia Pro Licence (Organisation)](https://avaloniaui.net/legal/10B)

[Schedule 11: Avalonia Enterprise Licence](https://avaloniaui.net/legal/11)

価格は 1 シート(1 ユーザー)あたり 以下のようになります。実際の総額は契約 Named User 数を掛けた金額になります、

Plus Organisation なら、年 345ドル/seat（53,475円/seat）のサブスクリプション

Pro  Organisation なら、年 1,050ドル/seat（162,750円/seat）のサブスクリプション

Enterprise なら、年 8,259ドル/seat（1,280,145円/seat）のサブスクリプションとなります。

[Avalonia Pricing](https://avaloniaui.net/pricing)

例として、5 人の開発者で Pro Organisation を契約する場合、年 1,050 ドル × 5 = **5,250 ドル(約 81 万円)** となります。

### 4-4. 受託開発（事業用）なら

営利企業の事業用（社内業務用を含む）ソフトウェアの委託先企業による受託開発の場合は、発注者がPlusかProかEnterprise契約を交わす必要があります。

Plus・Pro・Enterprise のどの機能レベルが必要なのかは、その開発規模と開発内容によって異なります。

価格については、「法人開発（商用）配布なら」の解説と同様です。

## 5. WPF か Avalonia か

Avalonia UI は、WPF並のグラフィカルな表現力を持ち、使い方も WPF によく似て、WPF の XAML 開発のスキルが生かせるクロスプラットフォーム開発コンポーネントです。

さほどグラフィック機能を必要としなければ Webアプリで良いと思われますが、高度なグラフィック機能を必要とする場合、WPF・MAUI・C++&WinUI3・Avalonia が選択肢に入ってくると思います。

MAUIはスマホに最適化しているので表現力に限りがあります。高度な表現とUIを必要とする場合 WPFはまだ必要性が高いと考えられます。

WPF を選択すべきか、Avalonia UI を選択すべきかの判断基準は、以下の点に注目すると良いと思います。

- ライセンス料の負担（WPFは無料、Avaloniaは有料）
- 対応OS（WPFは Windowsのみ、Avaloniaは Windows・Linux・macOS・iOS・Android・WebAssembly）
- ネイティブコードの必然性（WPFはNativeAOT未対応、AvaloniaはNativeAOT対応）
- 使用するコンポーネントの仕様（WPF対応製品を使用するか、Avaloniaコンポーネントを使用するか）
- 日本語の文書やサポート体制

米国などは、Mac と Linux の普及率が上昇していますから、クロスプラットフォーム対応は重要度が高いですが、日本では今でもWindowsの天下ですから、業務システムのクロスプラットフォーム対応の必然性は低いと思われます。

しかし、業務システム端末にタブレット端末やスマホを使用する事を検討しているのなら、.NET MAUIと共に Avalonia は検討対象に入ります。

コンポーネントにWPFに依存したサードパーティ製品を使用したい場合は、WPFを選択することになります。（[InputManPlus for WPF](https://developer.mescius.jp/inputmanplus-wpf) など）

WPFは無料ですが、Avaloniaは有料（例えば 50 人規模の開発組織で Pro Organisation だと年 5 万 2,500 ドル＝約 800 万円になります）なので、コスト面を考えたときは、WPF一択になるかも知れません。

比較的、コスト負担の軽い個人開発者には、Avaloniaの方が魅力的になる可能性もあります。

AI翻訳や機械翻訳全盛の現代において、日本語の言葉の壁の高さは、それほど大きくないかもしれませんが、これは個人差があるでしょう。

低レイヤーの描画機構には、WPF は DirectX9を、Avalonia は Skia を使用しており[将来は Impeller の対応](https://avaloniaui.net/blog/avalonia-partners-with-google-s-flutter-t-eam-to-bring-impeller-rendering-to-net)も計画されています。（DirectX の最新版は DirectX12です）

日本においては、ほとんど無名に近い Avalonia ですが、積極的なアップデートが行われているGUIコンポーネントであり、クロスプラットフォーム開発コンポーネントとしては将来有望な製品でもあります。

総合的に考えると、WPF か Avalonia かの選択は、悩ましい選択になると思います。

## 6. 最後に

Avalonia のライセンス規定は、非常にわかりにくく、生成AIに質問しても簡単には理解できませんでした。

私のプロンプトが悪いのかも知れませんが、予備知識の無い人が簡単にAIに質問して、一発で適切な回答を得られる性質の情報ではないと思います。

この記事は、AIに質問して簡単に回答を得られない記事の一例でもあります。

この記事を、Avalonia ライセンス規定の理解の一助として、ご利用ください。

