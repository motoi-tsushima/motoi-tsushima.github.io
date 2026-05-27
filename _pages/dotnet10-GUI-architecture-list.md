---
title: ".NET 10 の画面有りアーキテクチャの解説とサンプルコード"
layout: single
classes: wide
permalink: /dotnet10-GUI-architecture-list/
author_profile: true
---

（この記事は、.NET10 リリース後の2026年4月時点の事実と見解を述べています。状況は年々変わっていくので、その点は配慮して読んでください）

Claude Code や GitHub Copilot Agent などのAIコーディングエージェントにより、定型的処理ならばほとんど自動でエージェントがコーディングをしてくれる時代になりました。

しかし、コーディングエージェントにコーディングの大半を任せる場合、作成するソフトウェアの設計と仕様について、的確に指示しなければ期待した通りのコーディングをしてもらえません。

適切な指示をAIエージェントに出す為には、「対象となるソフトウェアの仕様を適切に文書化しておく」ことと、「そのソフトウェアを、どのようなアーキテクチャで開発するのか」設計方針を適切にAIに伝えることが必須となります。

つまり人間には、実現可能な仕様書を書く能力と、アーキテクチャの知識が必要になります。

アーキテクチャの知識の無い状態で、仕様書だけでコーディングエージェントに開発を依頼すると、見当違いの実装を始めてしまい、いつまで経ってもソフトウェアが完成しない袋小路にハマってしまいます。特に Blazor の構成は様々なパターンが存在するので、適切なアーキテクチャの指定無しにはコーディングエージェントも期待に応えることができません。

ソフトウェア・アーキテクチャは、いくつかのレイヤーに分けて考えることが可能です。

ここでは .NET10アプリケーション開発における、ユーザーインターフェース・API・フレームワーク構成アーキテクチャの種類について、サンプルプログラムを提示して解説します。
つまりソフトウェアの「部分アーキテクチャ」です。

.NETアーキテクチャの中で、GUI画面を持つ物だけを一覧し解説します。
バックグラウンドで稼働する物と、クラウド固有の.NETアーキテクチャは、この記事では扱いません。

クリーン・アーキテクチャやマイクロサービス・アーキテクチャなどシステムの「全体アーキテクチャ」は、この記事では扱っていません。

## サンプルプログラムについて

私の GitHub リポジトリから提供しているサンプルプログラムのプロジェクト（ソリューション）は、全て同じ仕様の課題管理ツール（IssueBoard）にしています。

課題の一覧・詳細・編集・新規作成・削除の画面からなる典型的なCRUDプログラムです。
ソフトウェアのユーザーテストなどで発見した課題を、チームのメンバー間で管理共有する簡単なツールです。

単純なサンプルプログラムなので、ユーザー認証などの機能は作っていません。起動するといきなり課題一覧が表示されます。
安全な社内ネットワークの中で少人数で使用する簡易グループウェアです。

あくまでサンプルプログラムなので実用性は期待できないとお考えください。

### データベースとテーブル

多数のサンプルプログラム IssueBoard は、全て同じデータベースとテーブルを参照・更新します。

そのデータベースとテーブルは以下のリポジトリで、DDLとテストデータのDMLを提供しています。

[DATABASE.Samples.Issues](https://github.com/motoi-tsushima/DATABASE.Samples.Issues)

以下のコマンドでダウンロード可能です。

```
git clone https://github.com/motoi-tsushima/DATABASE.Samples.Issues
```

サンプルプログラムを起動する場合は、ローカルPCに SQL Server をインストールした上で、このDDLをダウンロードして、その SQL Server の SSMS 等でDDLを実行してデータベースとテーブルを作成し、続けてDMLを実行しテスト用データを導入します。

インストールする SQL Server は無料版（Express版, Developer版）で問題ありません。

[Microsoft SQL Server のダウンロード](https://www.microsoft.com/ja-jp/sql-server/sql-server-downloads)

SQL Server は最新版である必要も無いので、既に旧バージョンをご利用の方は、そのまま利用できると思います。保証はできませんが。

### APIについて

クライアントサーバー構成アーキテクチャのサンプルプログラムの場合、必ず以下の二つのどちらかのAPIサービスを使用しています。

**Api.Rest.IssueBoard**	: ASP.NET Core Web API 形式の REST API サービス

**Api.Grpc.IssueBoard**	: ASP.NET Core gRPC Service 形式の gRPC API サービス

サンプルプログラムごとに、これらのどちらかのサービスプロジェクトを格納していますが、同一名のプロジェクトは、同じ内容のプログラムです。サンプルプログラムとして扱い易いように、このようなプロジェクトの構成にしています。

クライアントとサーバー間のプロトコルは REST か gRPC のどちらかです。

例外的に Blazor Server は、REST / gRPC による明示的なAPI呼び出しではなく、SignalR による常時接続のため、クライアントサーバー構成とは言えないでしょう。

APIサービスは、全て EntityFrameworkCore を使用してローカルの SQL Server に接続します。

SQL Server への接続文字列は、APIプロジェクト配下の `appsettings.json` の中の `ConnectionStrings : DefaultConnection` で定義しています。接続先の変更が必要な方は、ご自分で変更してください。

## .NET サーバー側 アーキテクチャパターン

### ASP.NET Core Web API

標準的な ASP.NET Core Web API のサービス。Restful API でもあります。

APIサービスなので、これ自身は画面を持ちません。しかし、これから解説する様々な画面有りアーキテクチャがサーバーサイド処理でこのAPI処理を使用します。

#### サンプルコード

これから解説するWebアプリケーション以外の多くのサンプルコードのサーバーサイドで使用しています。

プロジェクト名は `Api.Rest.IssueBoard` で統一しています。内容も同じです。
主にソリューション名に `.Rest.` が含まれているサンプルプログラムで使用しています。

### ASP.NET Core gRPC Service

gRPCサービス。

ASP.NET Core Web API の機構に統合されています。（プロジェクト・テンプレートは独立した「gRPCサービス」として用意されています）

APIサービスなので、これ自身は画面を持ちません。しかし、これから解説する様々な画面有りアーキテクチャがサーバーサイド処理でこのAPI処理を使用します。

#### サンプルコード

これから解説するWebアプリケーション以外の多くのサンプルコードのサーバーサイドで使用しています。

プロジェクト名は `Api.Grpc.IssueBoard` で統一しています。内容も同じです。
主にソリューション名に `.Grpc.` が含まれているサンプルプログラムで使用しています。

### Minimal API Service

Minimal API は、ASP.NET Core Web API（Rest API）の一種です。

通常の ASP.NET Web API がコントローラ（ControllerBase の継承クラス）によってイベントを処理するのに対して、Minimal API は Program.cs 内で直接イベント処理を記述します。イベントから実行に至る内部処理のオーバーヘッドも短く、簡潔にコーディングできて実行も早いのが特徴です。

NativeAOTにも対応しており、Minimal API を NativeAOT で構築するとコンパクトで高速の APIモジュールを構築できます。ただし、現状では EntityFrameworkCore が NativeAOT に対応していないので、NativeAOTによる構築は実用段階に達していません。.NET12 がリリースされるころには、EntityFrameworkCore が NativeAOT に対応することが期待できるので、Minimal API の将来に期待するのが現実的です。

#### サンプルコード: WPF and Minimal API (Rest)

サンプルコードは、WPFとRest Minimal APIのクライアントサーバーのサンプルコードです。 

##### リポジトリ:
[WPF and Minimal API (Rest)](https://github.com/motoi-tsushima/Net10.Wpf.MinimalAPI.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Wpf.MinimalAPI.IssueBoard
```

## .NET Web アプリケーション アーキテクチャパターン

### ASP.NET Web Forms

Web Forms は旧.NET Framework の時代の最初の ASP.NET Webアプリケーションです。

現在は廃止されており、最新の .NET(Core) へ移行する手段は提供されていません。

Web Forms で構築されたレガシーシステムは、MVC, Razor Pages, Blazor などの新しいフレームワークで再構築する必要があります。

旧.NET Framework が廃止されたら、共に廃止され、サポートも受けられなくなるでしょう。

サンプルコードは作成していません。

### ASP.NET Core MVC（Traditional MVC）

MVC は Web Forms の後に提供された .NET の Webアプリケーションフレームワークです。

.NET Framework のころから提供されており、最新の .NET(Core) においても現役です。
アップデートを何度も繰り返して進化を継続しており、古いサイトから新しいサイトに至るまで広く採用されています。今後もアップデートが続くことが期待できます。

Traditional MVC は、イベント処理をコントローラーで行う標準的なMVCフレームワークです。

ページ画面（*.cshtml）とイベント処理（Controller継承クラス）が分離しており処理ロジック中心にアプリケーションを構築するのに向いています。

ECショップのように単一画面に複雑な処理を組み込む場合に適しています。

逆にページ画面と処理ロジックが一対一に対応した方が良い単純な CRUD などには向いていません。
その場合は Razor Pages が向いているでしょう。
（CRUDが作れないわけでは無いので、誤解の無いようお願いします）

#### サンプルコード:

標準的 MVC により、IssueBoard を実装しています。DBアクセスもMVCプロジェクトの中で行っています。

##### リポジトリ:

[ASP.NET Core Mvc](https://github.com/motoi-tsushima/Net10.Mvc.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Mvc.IssueBoard
```

### ASP.NET Core MVC （MVC with API Backend）

MVCはサーバーサイドのコントローラーでイベント処理を行うので、通常はDBアクセス処理もコントローラーと同一モジュールの中で行いますが、DBアクセスの部分を別のAPIサービスに任せる構成にもできます。

MVC with API Backend アーキテクチャは、画面処理だけMVC側で行い、DB処理などは外部の Rest API を呼び出して処理するアーキテクチャです。

MVCをクライアントに見立てたクライアントサーバー型と考えると分かりやすいと思います。

スマホのアプリなどと Rest API を共有するとき便利な構成です。

#### サンプルコード:

MVC をクライアントにしてRest API と通信することで、IssueBoard を実装しています。DBアクセスはRest API プロジェクトの中で行っています。

##### リポジトリ:

[ASP.NET Core Mvc with API](https://github.com/motoi-tsushima/Net10.Mvc.Rest.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Mvc.Rest.IssueBoard
```

### ASP.NET Core MVC （MVC with gRPC Backend）

上記の MVC with API Backend のAPIを、Restではなく gRPC Service に置き換えたものです。
MVC with gRPC Backend アーキテクチャは、画面処理だけMVC側で行い、DB処理などは外部の gRPC API を呼び出して処理するアーキテクチャです。

MVCをクライアントに見立てたクライアントサーバー型と考えると分かりやすいと思います。

スマホのアプリなどと gRPC API を共有するとき便利な構成です。

#### サンプルコード:

MVC をクライアントにして gRPC API と通信することで、IssueBoard を実装しています。DBアクセスは gRPC API プロジェクトの中で行っています。

##### リポジトリ:

[ASP.NET Core Mvc with gRPC](https://github.com/motoi-tsushima/Net10.Mvc.Grpc.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Mvc.Grpc.IssueBoard
```

### ASP.NET Core Razor Pages（Page-based Application）

ページ画面とイベント処理を一対一に対応させたWebアプリケーションのフレームワークです。

ページ画面は *.cshtml に、イベント処理ロジックは *.cshtml.cs に、それぞれ記述することにより、ロジックと画面を一つのモジュールにまとめることができます。

CRUD 形式のアプリケーション開発に向いています。

逆に複雑な処理ロジックを共通化したい場合には、別途共通モジュールを分離しなければならない為、不便になります。その場合は MVC の方が向いているでしょう。

#### サンプルコード:

標準的 Razor Pages により、IssueBoard を実装しています。DBアクセスも Razor Pages プロジェクトの中で行っています。

##### リポジトリ:

[ASP.NET Core Razor Pages](https://github.com/motoi-tsushima/Net10.Razor.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Razor.IssueBoard
```

### ASP.NET Core Razor Pages（Razor Pages with API Backend）

MVC with API Backend の Razor Pages 版です。

画面処理だけ Razor Pages側で行い、DB処理などは外部の Rest API を呼び出して処理するアーキテクチャです。

Razor Pages を丸ごとクライアントに見立てたクライアントサーバー型と考えると分かりやすいと思います。

スマホのアプリなどと Rest API を共有するとき便利な構成です。

#### サンプルコード:

Razor Pages をクライアントにして Rest API と通信することにより、IssueBoard を実装しています。DBアクセスは Rest API プロジェクトの中で行っています。

##### リポジトリ:

[ASP.NET Core Razor Pages with API](https://github.com/motoi-tsushima/Net10.Razor.Rest.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Razor.Rest.IssueBoard
```

### ASP.NET Core Razor Pages（Razor Pages with gRPC Backend）

MVC with gRPC Backend の Razor Pages 版であり、Razor Pages with API Backend の Rest を gRPC に置き換えたものです。

画面処理だけ Razor Pages側で行い、DB処理などは外部の gRPC API を呼び出して処理するアーキテクチャです。

スマホのアプリなどと gRPC API を共有するとき便利な構成です。

#### サンプルコード:

Razor Pages をクライアントにして gRPC API と通信することにより、IssueBoard を実装しています。DBアクセスは gRPC API プロジェクトの中で行っています。

##### リポジトリ:

[ASP.NET Core Razor Pages with gRPC](https://github.com/motoi-tsushima/Net10.Razor.Grpc.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Razor.Grpc.IssueBoard
```

## .NET デスクトップクライアント アーキテクチャパターン

### Windows Forms

Web Forms と同様に、旧.NET Framework で最初に採用されたデスクトップクライアントを作成するためのフレームワークです。

通常は SDI (Single Document Interface) で開発しますが、旧デスクトップで使用されていた MDI (Multiple Document Interface) が使用できるのが、Windows Forms 固有の特徴と言えます。

MDI 以外ならば、WPF でも Windows Forms と同様のアーキテクチャが構成できます。

バックグラウンド描画処理が、GDIに直接描画する古い処理系であるため、近年の4K高解像度スクリーンに、Windows Forms で作成した画面を表示すると、ウインドウや文字が小さく表示されてしまう欠点があります。

Web Forms と違い、Windows Forms は最新の .NET(Core) でもサポートされています。
しかし、既にアップグレードの対象ではなく、サポートだけが提供されている状況です。

新規開発に採用すべきではないアーキテクチャです。個人的に Windows Forms の採用は推奨しません。

#### サンプルコード:

Windows Forms クライアントから、ASP.NET Core Web API の Rest API を用いて、三階層クライアントサーバーを実装しています。

##### リポジトリ:

[Windows Forms and Rest API](https://github.com/motoi-tsushima/Net10.WinForms.Rest.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.WinForms.Rest.IssueBoard
```

### WPF (Windows Presentation Foundation)

Windows Forms の次の世代のGUIフレームワークです。

端末のスクリーンが小さなモバイルPCやタブレットから、大型の4Kスクリーンに至るまで多様化したことに対応して、画面解像度が大きなスクリーンから小さなスクリーンまで、アプリ側で自動的にスクリーンサイズに合わせて表示できるフレームワークです。

.NET Framework のころから提供されておりWPFも既にレガシーなフレームワークで、既に保守モードになっています。何年も新しい機能は提供されていません。

低レイヤーの描画機構は、古い DirectX 9 を使用しており、最新の DirectX 12 に対応したアップデートは行われていません。DX9 と DX12 には後方互換性があります（DX12 のランタイムが DX9 のAPI呼び出しを処理できる）から、DX12でも描画は可能です。
それでも現在の.NET標準の描画機構のなかでは最も高性能な描画フレームワークとなります。

.NET Framework のころの設計なので、.NET(Core) になっても Windows 環境でしか使用できません。
将来的なアップデートも現状では期待できません。

同系統の描画フレームワークとして、C++から使用する WinUI3 と、.NET MAUI があり、今後のデスクトップGUIの発展は、そちらに集中するのではないかと思われます。

また、業務システムのGUI開発は、技術トレンドの推移から考えて、デスクトップアプリから Web技術を使用した Blazor に移ることが予想されます。
新規開発のGUIフレームワーク選定としては、Blazor を検討対象に入れる事をお勧めします。

### WPF : Standalone Desktop Application

WPFで作成した画面アプリ単独で起動する .NETアーキテクチャです。

最も原始的な二層クライアントサーバーを構築するときは、Standalone Desktop Application を作成することになります。
つまり、WPFスタンドアロンのアプリを作成し、その中から直接DBアクセスを行いクライアントサーバー処理を実装します。

#### サンプルコード:

サンプルコードは、WPFスタンドアロン・アプリの中で EntityFrameworkCore を直接使用し、DBアクセスすることで、IssueBoard の機能を実装しています。

##### リポジトリ:

[WPF Standalone](https://github.com/motoi-tsushima/Net10.Wpf.Standalone.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Wpf.Standalone.IssueBoard
```

### WPF : Client-Server with Web API

WPFクライアントから、Rest API を呼び出し三階層クライアントサーバーを実装する .NETアーキテクチャです。

Rest API 側は必ずしも .NET で実装する必要はありませんが、もし.NET で実装するならば、ASP.NET Core Web API を使用することになります。
Web API は、コントローラーを使用する通常の Web API 以外にも Minimal API を使用する選択肢もあります。

#### サンプルコード:

WPFクライアントから、ASP.NET Core Web API の Rest API を用いて、三階層クライアントサーバーを実装しています。

Minimal API のサンプルコードは、先の WPF and Minimal API (Rest) で紹介しています。

##### リポジトリ:

[WPF and Rest API](https://github.com/motoi-tsushima/Net10.Wpf.Rest.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Wpf.Rest.IssueBoard
```

### WPF : Client-Server with gRPC

WPFクライアントから、gRPC API を呼び出し三階層クライアントサーバーを実装する .NETアーキテクチャです。

gRPC API 側は必ずしも .NET で実装する必要はありませんが、もし.NET で実装するならば、ASP.NET Core gRPC Service を使用することになります。Visual Studioプロジェクトテンプレートでは `gRPCサービス` という呼称になっているはずです。 

#### サンプルコード:

WPFクライアントから、ASP.NET Core gRPC Service の gRPC API を用いて、三階層クライアントサーバーを実装しています。

##### リポジトリ:

[WPF and gRPC API](https://github.com/motoi-tsushima/Net10.Wpf.Grpc.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Wpf.Grpc.IssueBoard
```

### WPF with BlazorWebView

後の Blazor Hybrid の章で解説します。

### .NET MAUI (Multi-platform App UI)

.NET(Core) 6 から提供された、同一コードで Android、iOS、macOS、Windows に対応できるクロスプラットフォームGUIフレームワークです。

初めからクロスプラットフォームに対応しているため、機能的にも対応OSの最大公約数的な機能になっており、WPFに比べて描画機能は少なくなります。

低レイヤーな描画機能は、各OSの描画機能を直接使用し独自のレンダリング機能は持っていません。
Windowsの場合は、内部で WinUI3 を呼び出すことになります。

WPF とよく似た XAML でGUI画面を記述するので、WPF開発者には習得が容易になる仕様になっています。

スマホのアプリも開発できる仕様のため、デスクトップアプリの複雑な画面を作成するには、やや機能不足を感じます。WPFが残っている理由も、WindowsのGUIを最大限生かすのなら、WPFの方が相応しいからかも知れません。

### .NET MAUI : Client-Server with Web API

MAUIクライアント と Rest API のサーバーで、クライアントサーバーを構成したアーキテクチャです。互いは Rest プロトコルで通信します。

#### サンプルコード:

Rest API に ASP.NET Core Web API を使用しています。

##### リポジトリ:

[MAUI and Rest API](https://github.com/motoi-tsushima/Net10.Maui.Rest.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Maui.Rest.IssueBoard
```

### .NET MAUI : Client-Server with gRPC

MAUIクライアント と gRPC API のサーバーで、クライアントサーバーを構成したアーキテクチャです。互いは gRPC プロトコルで通信します。

#### サンプルコード:

gRPC API に、ASP.NET Core gRPC Service を使用しています。

##### リポジトリ:

[MAUI and gRPC API](https://github.com/motoi-tsushima/Net10.Maui.Grpc.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Maui.Grpc.IssueBoard
```

### MAUI Blazor Hybrid

後の Blazor Hybrid の章で解説します。

## .NET Blazor Server アーキテクチャパターン

Blazor Server は、SPA形式の Webアプリケーションで、通信プロトコルにステートフルの SignalR を使用します。画面のレンダリングを含む画面処理を全てサーバー側で行うシンクライアントに似た実装をするアーキテクチャです。

ステートフルなので、一度接続すると同じサーバーを占有します。ダウンロードモジュールは少ないので起動が速い長所がありますが、その反面サーバー側の負荷が高くなる欠点があります。

Blazor では全般に、画面デザインは Razorコンポーネント というBlazor共通の規格を使用して作成します。

### Standard Blazor Server

標準的な Blazor Server アーキテクチャは、画面レンダリングを行うサーバープロジェクトと同一プロジェクトで、DBアクセスを含むサーバーサイド処理を全て行ってしまいます。

クライアントのロジックも同じプロジェクトに内包されているので、実質的に「サーバー側で動作するスタンドアロン・アーキテクチャ」です。サーバースタンドアロンに対し、ブラウザでリモートデスクトップ接続するイメージです。

当然、サーバーの負荷は非常に高くなります。

#### サンプルコード:

Server の中で EntityFrameworkCore を使用し、直接DBアクセスしています。
クライアントレンダリングも Server で処理します。

##### リポジトリ:

[Blazor Server Standard](https://github.com/motoi-tsushima/Net10.BlazorServer.Std.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.BlazorServer.Std.IssueBoard
```

### Blazor Server with API Backend

標準的 Blazor Server では、自身でDBアクセスなどサーバーサイド処理を行いますが、Blazor Server with API Backend アーキテクチャでは、Rest API にサーバーサイド処理を任せます。

スマホのアプリなどと、サーバーサイド処理を共有したい時に、このアーキテクチャを採用します。

画面処理を行うサーバーと、DBアクセス等サーバーサイド処理を行うサーバーの二つのサーバーが起動して処理を行います。

画面レンダリングがサーバーサイドで、ブラウザとステートフル接続する点において、Standardと変わりません。サーバーが二つある分、もっとサーバーの負担が大きくなります。

#### サンプルコード:

Rest API に、ASP.NET Core Web API を使用して、IssueBoard を実装しています。

##### リポジトリ:

[Blazor Server and Rest API](https://github.com/motoi-tsushima/Net10.BlazorServer.Rest.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.BlazorServer.Rest.IssueBoard
```

### Blazor Server with gRPC Backend

構成は、Blazor Server with API Backend アーキテクチャと同じで、Rest API の代わりに gRPC API を使用するのが、Blazor Server with gRPC Backend アーキテクチャです。

特徴も長所も欠点も、Blazor Server with API Backend と同じです。

#### サンプルコード:

gRPC API に、ASP.NET Core gRPC Service を使用して、IssueBoard を実装しています。

##### リポジトリ:

[Blazor Server and gRPC API](https://github.com/motoi-tsushima/Net10.BlazorServer.Grpc.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.BlazorServer.Grpc.IssueBoard
```

## .NET Blazor WebAssembly アーキテクチャパターン

Blazor WebAssembly アーキテクチャは、Blazor Server と異なり、画面レンダリングなどクライアント側の処理を、ブラウザ側で行います。

ブラウザで対象ページにアクセスすると、 WebAssembly で構築された .NET Runtime をダウンロードし、続いて Blazorクライアントモジュールをダウンロードして、先にダウンロードしていた .NET Runtime 上で Blazorクライアントモジュールを起動します。もちろん Blazorクライアントモジュールは .NETマネージドコードです。

つまり、ブラウザの WebAssembly で動作する .NET リッチクライアントです。「スマートクライアント」という呼び方もありますね。

フロントエンド（PC,スマホなど）にはブラウザだけ用意すれば動作するので、Webアプリケーションと同様に手軽に導入できます。

また、レンダリングなどクライアント処理は、フロントエンド側で行うので、サーバーの負荷は軽くなります。

通信もWebアプリケーションと同様にステートレスなので、一度接続したサーバーを占有しなくても良いです。

しかし、.NET Runtime と.NETクライアントモジュールをダウンロードしてから起動するので、起動するまで時間がかかります。

### Standalone

Blazor WebAssembly の機構を、単体で起動するアーキテクチャで、Electron に似たアーキテクチャです。

WebAssembly 上で起動するので、WPFのスタンドアロンアプリのように、DBアクセスなどをスタンドアロンで行うことができません。DBアクセスを行う場合は、別途APIを用意しなければならないので、スタンドアロンではなくなります。

Electron はローカルファイルシステムやNode.jsバックエンドにアクセスできるのに対し、Blazor WASM Standalone はブラウザのサンドボックス内で動作する点で大きく異なります。

よってスタンドアロンでは、DBアクセスなどブラウザ外部と繋がることができません。

DBアクセスできないので、サンプルコードは作成しませんでした。

### ASP.NET Core Hosted<br />(Hosted Blazor WebAssembly)

Hosted は標準的な Blazor WebAssembly のアーキテクチャです。

ブラウザで対象ページにアクセスすると、ホスト（サーバー）から画面モジュールをダウンロードして、実行します。その画面ダウンロードに使用するサーバーがDBアクセスなどのサーバーサイド処理も行う構成のアーキテクチャです。

よってサーバーは一つで済みます。

#### サンプルコード:

Blazor のホスト（サーバー）の中で EntityFrameworkCore を使用しDBアクセスを実行しています。

通信プロトコルとして、Rest を使用することも、gRPCを使用することも可能です。

両方のサンプルコードを用意しています。

それぞれのソリューションが単独起動できます。互いに依存しません。

##### リポジトリ:

[Blazor Web Assembly Hosted Rest](https://github.com/motoi-tsushima/Net10.BWASM.Hosted.Rest.IB)

[Blazor Web Assembly Hosted gRPC](https://github.com/motoi-tsushima/Net10.BWASM.Hosted.Grpc.IB)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.BWASM.Hosted.Rest.IB

git clone https://github.com/motoi-tsushima/Net10.BWASM.Hosted.Grpc.IB
```

### Standalone with Separate API<br />(Decoupled Architecture)

Decoupled アーキテクチャは、クライアントに Hosted Blazor WebAssembly を採用し、DBアクセスなどサーバーサイド処理を、Rest API や gRPC API 側に任せるアーキテクチャです。

ブラウザに対してクライアントモジュールをダウンロードする為の Hostedサーバーと、DBアクセス等サーバー処理を行うAPIサーバーの二つのサーバーが必要です。ただし、クライアントモジュールをダウンロードしたその後は、Hostedサーバーには負荷がかからないので、サーバーサイドの負荷はWPFなどリッチクライアント・アプリケーションと変わりません。

#### サンプルコード:

APIサーバーに Rest API を使用するサンプルと、gRPCサービスを使用するサンプルの、二つのサンプルコードを用意しました。

それぞれのソリューションが単独起動できます。互いに依存しません。

##### リポジトリ:

[Blazor Web Assembly Decoupled Rest API](https://github.com/motoi-tsushima/Net10.BWASM.Decoupled.Rest.IB)

[Blazor Web Assembly Decoupled gRPC API](https://github.com/motoi-tsushima/Net10.BWASM.Decoupled.Grpc.IB)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.BWASM.Decoupled.Rest.IB

git clone https://github.com/motoi-tsushima/Net10.BWASM.Decoupled.Grpc.IB
```

## .NET Blazor Auto アーキテクチャパターン

Blazor Auto は、Blazor Server と Blazor WebAssembly の長所を合わせて、両者の欠点を解消したアーキテクチャです。

Blazor Server は画面処理もサーバー側で行うので、サーバー負荷が高くなってしまいます。
Blazor WebAssembly はクライアントモジュールをダウンロードしてから、リッチクライアントとして機能するので、サーバー側の負荷が軽くなりますが、ダウンロードが終わるまで起動できないので、初回起動に時間がかかりすぎます。

Blazor Auto は、初回起動だけ Blazor Server の機構でサーバー側でレンダリング等全ての処理を行い、その間に同時並行でクライアントモジュールをダウンロードします。
ダウンロードが完了したら、クライアントモジュールを起動して、以降は Blazor WebAssembly の機構で稼働します。

よって、Blazor Auto は Blazor Server と同様に素早く起動し、ダウンロード後は Blazor WebAssembly と同様の軽いサーバー負荷で稼働できるアーキテクチャです。

開発方法やアーキテクチャは、Blazor WebAssembly と、ほとんど同じと考えて良いです。

### ASP.NET Core Hosted<br />(Hosted Blazor Auto)

Blazor WebAssembly の Hosted と同じです。
Hosted は標準的な Blazor Auto のアーキテクチャです。

ブラウザで対象ページにアクセスすると、ホスト（サーバー）から画面モジュールをダウンロードして、実行します。その画面ダウンロードに使用するサーバーがDBアクセスなどのサーバーサイド処理も行う構成のアーキテクチャです。

よってサーバーは一つで済みます。

#### サンプルコード:

Blazor のホスト（サーバー）の中で EntityFrameworkCore を使用しDBアクセスを実行しています。

通信プロトコルとして、Rest を使用することも、gRPCを使用することも可能です。

両方のサンプルコードを用意しています。

それぞれのソリューションが単独起動できます。互いに依存しません。

##### リポジトリ:

[Blazor Auto Hosted Rest](https://github.com/motoi-tsushima/Net10.BWASM.A.Hosted.Rest.IB)

[Blazor Auto Hosted gRPC](https://github.com/motoi-tsushima/Net10.BWASM.A.Hosted.Grpc.IB)


##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.BWASM.A.Hosted.Rest.IB

git clone https://github.com/motoi-tsushima/Net10.BWASM.A.Hosted.Grpc.IB
```

### Standalone with Separate API<br />(Decoupled Architecture)

Decoupled アーキテクチャは、クライアントに Hosted Blazor Auto を採用し、DBアクセスなどサーバーサイド処理を、Rest API や gRPC API 側に任せるアーキテクチャです。

ブラウザに対してクライアントモジュールをダウンロードする為の Hostedサーバーと、DBアクセス等サーバー処理を行うAPIサーバーの二つのサーバーが必要です。ただし、クライアントモジュールをダウンロードしたその後は、Hostedサーバーには負荷がかからないので、サーバーサイドの負荷はWPFなどリッチクライアント・アプリケーションと変わりません。

#### サンプルコード:

APIサーバーに Rest API を使用するサンプルと、gRPCサービスを使用するサンプルの、二つのサンプルコードを用意しました。

それぞれのソリューションが単独起動できます。互いに依存しません。

##### リポジトリ:

[Blazor Auto Decoupled Rest API](https://github.com/motoi-tsushima/Net10.BWASM.A.Decoupled.Rest.IB)

[Blazor Auto Decoupled gRPC API](https://github.com/motoi-tsushima/Net10.BWASM.A.Decoupled.Grpc.IB)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.BWASM.A.Decoupled.Rest.IB

git clone https://github.com/motoi-tsushima/Net10.BWASM.A.Decoupled.Grpc.IB
```


## .NET Blazor Hybrid アーキテクチャパターン

Blazor Hybrid とは、WPF, Windows Forms, .NET MAUI の三種類のデスクトップ・フレームワークが、クライアント画面の中に、Blazor の Razorコンポーネントを組み込んでアプリケーションを作成することができるフレームワークです。
Razorコンポーネントは、Blazorで使用しているものと同じ物が共有できます。

### WPF BlazorWebView Hosting

WPF画面 に Blazor Hybrid を組み込むには、通常のWPFプロジェクトを開き、Microsoft.AspNetCore.Components.WebView.Wpf という NuGet パッケージをプロジェクトにインストールします。

画面のXAMLの中で BlazorWebView タグを定義することで、WPF画面の中にRazorコンポーネントを展開します。

#### サンプルコード:

このサンプルコードは、APIプロジェクトとRazorコンポーネントのプロジェクトを、WPF, Windows Forms, MAUI の三つのクライアントで共有する形で、三つのサンプルプロジェクトを共有しています。

WPF, Windows Forms, MAUI のそれぞれのクライアント画面が、全て同じ Razorコンポーネントを共有しており、それぞれ同じ IssueBoard の機能を実装しています。

いずれの画面も、一つの基本画面の全体にRazorコンポーネントを展開しています。
よって見た目は、ほとんど同じに見えます。

##### リポジトリ:

[Blazor Hybrid and Rest API (WinForms,WPF,MAUI)](https://github.com/motoi-tsushima/Net10.BHybrid.WPF.Rest.IB)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.BHybrid.WPF.Rest.IB
```

### Windows Forms BlazorWebView Hosting

Windows Forms 画面 に Blazor Hybrid を組み込むには、通常の Windows Forms プロジェクトを開き、Microsoft.AspNetCore.Components.WebView.WindowsForms という NuGet パッケージをプロジェクトにインストールします。

MainForm のコンストラクタの中で Controls.Add により、BlazorWebView のインスタンスを登録することで、Windows Forms の画面にRazorコンポーネントを展開します。

サンプルコードは、WPFのサンプルの Net10.BHybrid.WPF.Rest.IB 内に内包しています。

### .NET MAUI Blazor Hybrid

MAUI の場合は、WPF や Windows Forms と異なり、専用のプロジェクトテンプレートが用意されています。

MAUI Blazor Hybrid テンプレートで作成した MAUI のメイン画面の中に、Razorコンポーネントを展開します。

専用のプロジェクトテンプレートには Microsoft.AspNetCore.Components.WebView.Maui が組み込まれています。

サンプルコードは、WPFのサンプルの Net10.BHybrid.WPF.Rest.IB 内に内包しています。

## Avalonia UI アーキテクチャパターン

Avalonia UI とは、WPF によく似たXAMLベースの.NET向けクロスプラットフォーム GUI フレームワークです。

オープンソースコミュニティにより開発されており、エストニアの Avalonia社 から提供されています。

[Avalonia 公式サイト](https://avaloniaui.net/)



Windows, macOS, iOS, Android, Linux, WebAssembly のクロスプラットフォーム対応で、NativeAOTにも対応しています。

描画エンジンには、Google 開発の Skia を使用しており、これによりクロスプラットフォーム対応と高速描画を実現しています。



WPFが保守中心のフェーズに入っているのに対し、Avaloniaは活発な機能追加とアップデートが続けられています。

2026年4月7日に Avalonia 12 がリリースされ大幅に描画性能が向上しました。[（複雑なレイアウトでFPSが最大1,867%向上）](https://avaloniaui.net/avalonia)

FlutterチームとAvaloniaが協力して、Flutterの次世代GPUレンダラー「Impeller」を.NETに持ち込む取り組みが進んでいます。[(Avalonia Partnering with Google's Flutter Team to Bring Impeller Rendering to .NET)](https://avaloniaui.net/blog/avalonia-partners-with-google-s-flutter-t-eam-to-bring-impeller-rendering-to-net)

MAUIアプリをLinuxとWebAssemblyでも動かせるようにするプロジェクトが進行中です。これは公式MAUIではサポートされていないLinuxとWebAssemblyを、Avaloniaのレンダラーをバックエンドとして差し替えることで実現するもので、[MAUIエコシステムのエンジニアからの助言とフィードバックを受けながら](https://avaloniaui.net/blog/net-maui-is-coming-to-linux-and-the-browser-powered-by-avalonia)進められています。

Avaloniaのコアフレームワーク自体はMITライセンスで永久に無料のまま維持され、商用アプリケーションの開発・配布も無料で可能です。

一方、プロフェッショナル向けの開発支援ツール（Visual Studio拡張の高機能版、DevTools、Parcel等）と高度なUIコンポーネントは有償サブスクリプションで提供されています。なお、Visual Studio Code拡張は機能制限なしで完全に無料です。

2026年4月のAvalonia 12リリースと同時に、従来「Accelerate」ブランドで提供されていた有償プランは廃止され、単一の「Avalonia」ブランドの下に以下のティア構成へ再編されました。

- Avalonia（無料・MIT、フレームワーク本体）

- Community（無料、非商用の個人・教育・OSS用途向け）

- Plus（個人向け月額$17、商用可、上級開発ツール一式）

- Pro（個人向け月額$49、Plus に加えてプレミアムUIコンポーネント・チャート等）

組織向けには別途 Organization Plus / Pro / Enterprise が用意されています。

参考資料 → [Avalonia UI のライセンス規定を解説します](/avaloniaui-license-guide/)

OSSプロジェクトでありながら、有償ティアによる収益で資金源が確立されており、既に十年以上提供されている実績があるため、将来にわたって安定的に開発が継続されることが期待できます。

### Avalonia UI : Client-Server with Web API

Avalonia UI クライアントから、Rest API を呼び出し三階層クライアントサーバーを実装する .NETアーキテクチャです。

WPF版と同じ解説になりますが、Rest API 側は必ずしも .NET で実装する必要はありませんが、もし.NET で実装するならば、ASP.NET Core Web API を使用することになります。

Web API は、コントローラーを使用する通常の Web API 以外にも Minimal API を使用する選択肢もあります。

#### サンプルコード:

WPFサンプルコードの Client-Server with Web API と同じ構成を、Avalonia UI で作成しました。

API側で SQL Server へアクセスしているため、Windowsでしか稼働しません。クライアント側の Avalonia UI はクロスプラットフォーム対応なので Linux, macOS 等でも動くはずですが、作者の都合で未確認です。（特にmacOSでSQL Serverが動きません）

##### リポジトリ:

[AvaloniaUI Rest API](https://github.com/motoi-tsushima/Net10.Avalonia.Rest.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Avalonia.Rest.IssueBoard
```



### Avalonia UI : Client-Server with gRPC

Avalonia UI クライアントから、gRPC API を呼び出し三階層クライアントサーバーを実装する .NETアーキテクチャです。

WPF版と同じ解説になりますが、gRPC API 側は必ずしも .NET で実装する必要はありませんが、もし.NET で実装するならば、ASP.NET Core gRPC Service を使用することになります。

#### サンプルコード:

WPFサンプルコードの Client-Server with Web gRPC と同じ構成を、Avalonia UI で作成しました。

API側で SQL Server へアクセスしているため、Windowsでしか稼働しません。クライアント側の Avalonia UI はクロスプラットフォーム対応なので Linux, macOS 等でも動くはずですが、作者の都合で未確認です。（特にmacOSでSQL Serverが動きません）

##### リポジトリ:

[AvaloniaUI Grpc API](https://github.com/motoi-tsushima/Net10.Avalonia.Grpc.IssueBoard)

##### ダウンロード:

```
git clone https://github.com/motoi-tsushima/Net10.Avalonia.Grpc.IssueBoard
```



