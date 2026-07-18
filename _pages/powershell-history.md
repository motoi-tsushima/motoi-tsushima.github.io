---
title: "PowerShell の歴史 ─ cmd から Windows PowerShell、そして PowerShell 7 へ"
permalink: /powershell-history/
toc: true
toc_label: "目次"
toc_sticky: true
last_modified_at: 2026-07-18
---

`powershell.exe` と `pwsh`、あるいは「Windows PowerShell」と「PowerShell」。よく似た名前が二つ並んでいて、どちらが新しいのか、なぜ二つあるのか、迷ったことはないでしょうか。この分岐は行きあたりばったりに生まれたものではなく、**土台となる .NET の分岐がそのまま現れたもの**です。

本記事は、姉妹記事「[.NET の歴史 ─ Framework から Core、そして統一へ]({{ "/dotnet-history/" | relative_url }})」の続きとして読むことを想定しています。というのも、両者の歴史は次の一点で密接に結びついているからです。

> **Windows PowerShell(1.0〜5.1)は .NET Framework 上で動き、PowerShell(6.0 以降)は .NET Core / .NET 上で動く。**

つまり、**PowerShell 5.1 → 6.0 の分岐は、.NET Framework → .NET Core の分岐そのもの**です。「なぜ 6.0 で名前まで変わり、クロスプラットフォームになったのか」という疑問の答えは、ほとんど .NET 側の歴史に書いてあります。ランタイムの世代が変われば、その上に載る PowerShell の世代も変わる──この対応関係を頭に置くと、以下の歴史はぐっと見通しよく読めます。

なお本記事では「文字コードがなぜ環境で違うのか」という話には深入りしません。それは実行基盤(.NET Framework か .NET か)の違いから生じる結果であり、具体的な差異は続く「[Windows PowerShell 5.1 と PowerShell 6.2 以降の違い]({{ "/windows-powershell-51-vs-pwsh-62/" | relative_url }})」で扱います。

## 出発点:cmd とバッチの限界

Windows には長らく `cmd.exe` とバッチファイル(`.bat` / `.cmd`)という自動化手段がありました。手軽ではあるものの、その根本は**テキストの受け渡し**です。コマンドの出力は「ただの文字列」であり、そこから必要な値を取り出すには `for /f` で行を切り、区切り文字で分割し……と、文字列処理を積み重ねる必要がありました。出力書式がわずかに変われば壊れる、脆いスクリプトになりがちです。

UNIX 系のシェルも思想は同じく「テキストのパイプライン」ですが、成熟したツール群と相まって強力でした。一方 Windows 側には、システム管理を体系的に自動化できる本格的なシェルが不足していました。この空白を埋めるために生まれたのが PowerShell です。

## Windows PowerShell 1.0(2006):Monad 構想とオブジェクトパイプライン

2006 年 11 月、**Windows PowerShell 1.0** が登場します。開発時のコードネームは *Monad* で、設計者 Jeffrey Snover による構想は「テキストではなく**オブジェクト**を流すシェル」でした。

これが PowerShell 最大の発明です。コマンド(コマンドレット)の出力は文字列ではなく **.NET オブジェクト**であり、パイプ `|` で次のコマンドレットへ渡されるのもオブジェクトそのものです。たとえばプロセス一覧をメモリ使用量で並べ替えて上位を取る、といった処理を、文字列の切り貼りなしに、プロパティ名を指定するだけで書けます。

この設計が可能だったのは、Windows PowerShell が **.NET Framework の上に構築された**からです。コマンドレットが返すのは `System.Diagnostics.Process` のような実体を持ったオブジェクトであり、BCL(基底クラスライブラリ)の型がそのまま使えます。ここで PowerShell と .NET Framework の運命は結ばれました。

## 2.0 から 5.1 へ:Windows と共に成熟

1.0 の後、Windows PowerShell は Windows / Windows Server に同梱されながら着実に機能を増やしていきます。主な節目を挙げます。

- **2.0(2009)**:リモート処理(PowerShell Remoting)、バックグラウンドジョブ、モジュール、そして統合スクリプト環境である **ISE(Integrated Scripting Environment)** を導入。この版で「本格的な運用自動化ツール」としての骨格が固まります。
- **3.0 / 4.0(2012 / 2013)**:ワークフロー、そして構成管理の仕組みである **DSC(Desired State Configuration)** が加わります。
- **5.0 / 5.1(2016〜)**:`class` キーワードによるクラス定義、対話体験を改善する PSReadLine、パッケージ管理機能などが揃います。

一貫していたのは、**Windows PowerShell は Windows に付属するコンポーネントである**という位置づけです。OS と一体で提供されるため、どの Windows にも同じシェルがある反面、そのバージョンは OS の世代に縛られます。

## Windows PowerShell 5.1:Framework 系の「最終版」

**Windows PowerShell 5.1** は、この系譜の**最終版**です。以降、Windows PowerShell に新機能が追加されることはありません。ただし誤解のないように付け加えると、これは「廃止」ではありません。5.1 は現在も Windows / Windows Server に同梱され、**保守フェーズ**として動き続けています。長年の資産(スクリプト・モジュール・運用手順)がこの上で回っている現場は依然として数多く、Microsoft もそれを前提に扱っています。

なぜ 5.1 で止まったのか。答えは実行基盤にあります。Windows PowerShell は .NET Framework に強く依存しており、その .NET Framework 自体が 4.8 で新機能開発を事実上終えてレガシー保守フェーズに入りました(この経緯は[.NET の歴史]({{ "/dotnet-history/" | relative_url }})を参照)。**土台が動きを止めれば、その上のシェルも動きを止める。** 前進の舞台は、新しい基盤の上に移されることになります。

## PowerShell Core 6.0(2018):クロスプラットフォームへの跳躍

2018 年 1 月、**PowerShell Core 6.0** が登場します。名前に付いた *Core* が示すとおり、これは **.NET Core 上に再構築された PowerShell** です。変化は劇的でした。

- **クロスプラットフォーム**:Windows だけでなく Linux / macOS でも動作。
- **オープンソース**:GitHub で開発され、**MIT ライセンス**で公開。
- **新しい実行体 `pwsh`**:従来の `powershell.exe` とは別のバイナリとして提供され、同じマシン上で **Windows PowerShell 5.1 と併存**できます。

ここで冒頭の「二つの PowerShell」が生まれます。`powershell.exe`(Windows PowerShell)は Windows 同梱の 5.1 系、`pwsh`(PowerShell)は個別にインストールするクロスプラットフォーム系。両者は上書き関係ではなく、共存する別系統です。

そして繰り返しになりますが、**この 5.1 → 6.0 の跳躍は、.NET Framework → .NET Core の跳躍と完全に対応**しています。Windows 専用の巨大ランタイムから、モジュール化されクロスプラットフォームな新ランタイムへ──土台の思想転換が、そのままシェルに反映されたのです。

## 6.2 から 7.0 へ:"Core" が外れる

Core 系は **6.1**、そして「6.2 以降」の基準点となる **6.2(2019 年 3 月)**へと進みます。この間、Windows PowerShell 5.1 との互換性向上が重ねられ、多くのモジュールが Core 上でも動くよう整えられていきました。

転機は **2020 年 3 月の PowerShell 7.0** です。ここで名称から *Core* が外れ、単に **「PowerShell」** に統一されました。番号も 6 から 7 へ上がっています。これは単なるバージョンアップ以上の意味を持ちます。「Windows PowerShell を置き換えうる、標準的な PowerShell はこちらである」という Microsoft のメッセージです。7.0 は当時の最新ランタイムである .NET Core 3.1 上に構築されました。

> **用語の整理**:6.x 期の呼称が「PowerShell Core」、7.0 以降は単に「PowerShell」です。本シリーズでも 7.0 以降を「Core」とは呼びません。1.0〜5.1 を指すときだけ「Windows PowerShell」と明記します。

7 系では言語機能も拡充されました。三項演算子や null 合体演算子、パイプラインチェーン演算子、並列処理などが加わっていますが、これらの具体的な差は次記事で扱います。

## 7.x の年次リリースと LTS

7.0 以降、PowerShell は **.NET の年次リリースに歩調を合わせて**進むようになりました。ここで **LTS(Long Term Support:長期サポート)** と **STS(Standard Term Support:標準サポート)** の別が効いてきます。PowerShell の各版は、その土台となる .NET の版のサポート区分を引き継ぎます。

| PowerShell | 基盤 | サポート区分 |
|---|---|---|
| 7.2(2021) | .NET 6 | LTS |
| 7.3(2022) | .NET 7 | STS |
| 7.4(2023) | .NET 8 | LTS |
| 7.5(2024) | .NET 9 | STS |

偶数番の .NET(6 / 8 / 10)が LTS、奇数番(7 / 9)が STS──.NET 側の交互リリースが、そのまま PowerShell 側の LTS/STS の並びになっています。運用で長く据え置きたい環境は、LTS 版を選ぶのが定石です。

## 現在地(2026 年 7 月時点):7.6 が現行 LTS

本記事執筆時点の最新 LTS は **PowerShell 7.6** です。**2026 年 3 月に GA(一般提供)**となり、**.NET 10(LTS)上**に構築されています。Microsoft Learn のサポートライフサイクル情報によれば、現行 LTS は **7.6.3**(最新パッチ)で、前 LTS の 7.4 系も 2026 年 11 月まではサポートが継続されます。7.6 は派手な新機能よりも、エンジン・モジュール・対話シェルまわりの信頼性向上に主眼を置いた「運用で信頼できる版」として位置づけられています。

構図をまとめると、次のようになります。

| 系統 | 実行体 | 基盤 | 立ち位置(2026-07 時点) |
|---|---|---|---|
| **Windows PowerShell**(〜5.1) | `powershell.exe` | .NET Framework | Windows 同梱・保守フェーズ |
| **PowerShell**(6.0〜) | `pwsh` | .NET Core / .NET | 現役・現行 LTS は 7.6 |

> バージョンは動きます。7.7 系はすでにプレビュー開発が進んでいます。参照時は Microsoft Learn / PowerShell Team ブログで最新を確認してください。

## .NET の歴史との対応(背骨のまとめ)

冒頭で述べた対応関係を、ここで一枚に束ね直します。PowerShell の節目は、ほぼ例外なく .NET の節目に対応しています。

| PowerShell の出来事 | 対応する .NET の出来事 |
|---|---|
| Windows PowerShell 1.0(2006) | .NET Framework の時代 |
| Windows PowerShell 5.1(最終版) | .NET Framework 4.8(新機能開発の事実上の終了) |
| **PowerShell Core 6.0(2018)** | **.NET Core への移行** |
| 7.0 で "Core" を外す | .NET 5 で名称統一("Core" を外す) |
| 7.2 / 7.4 / 7.6(LTS) | .NET 6 / 8 / 10(LTS) |

「PowerShell の歴史」は、独立した物語というより、**[.NET の歴史]({{ "/dotnet-history/" | relative_url }})の上に重ね書きされた一章**なのです。だからこそ、片方だけを見ると謎めいて見える分岐や改名が、両方を並べると自然に読み解けます。

## 次回:では 5.1 と 7 は「具体的に」何が違うのか

ここまでで、なぜ二つの PowerShell が並び立っているのか、その歴史的な背景は見えてきました。残るのは実務的な問いです──**同じマシンに `powershell.exe` と `pwsh` があるとき、両者は具体的に何が違うのか。**

実行基盤(.NET Framework か .NET か)、クロスプラットフォーム対応、モジュール互換性、そして**既定の文字コードの扱い**まで、無視できない差があります。とりわけ文字コードは、本シリーズを貫くテーマです。続く「[Windows PowerShell 5.1 と PowerShell 6.2 以降の違い]({{ "/windows-powershell-51-vs-pwsh-62/" | relative_url }})」で、この差を一つずつ確認していきます。

---

*本記事のバージョン・日付は 2026 年 7 月時点の情報です。出典は主に Microsoft Learn および PowerShell Team ブログ(DevBlog)に基づきます。最新の状況は各一次情報でご確認ください。*
