---
title: "Tools"
layout: single
permalink: /tools/
author_profile: true
---
## Console tools

### rmsmf and txprobe

rmsmf は、複数のファイル内の複数の文字列を一括置換するコマンドラインツールです。文字エンコーディングの変換や、改行コードの変換に、BOM（Byte Order Mark）の制御も可能です。

txprobe は、複数のファイルの文字エンコーディング・BOM・改行コードを確認するコマンドツールです。

インストール方法:

以下の Download から rmsmf.zip をダウンロードして解凍し、展開された 全てのファイルを 環境変数 Path の通ったフォルダーにコピーすれば、コンソールアプリとして使用できます。
単純な 実行ファイルだけで成り立つツールなので、インストーラーは用意していません。

使用方法は、それぞれ /h オプションで表示されます。

**Download** [https://github.com/motoi-tsushima/rmsmf/releases/tag/v1.0.4.2](https://github.com/motoi-tsushima/rmsmf/releases/tag/v1.0.4.2)

**Repository** [https://github.com/motoi-tsushima/rmsmf](https://github.com/motoi-tsushima/rmsmf)


※ rmsmf は以前から公開していましたが、今回 rmsmf の機能強化とリファクタリングをして、確認ツールの txprobe を追加して、正式版として再リリースしました。

### mfsr and mfprobe

.NET Framework 4.8 版の rmsmf と txprobe を、.NET 10 へ移植したモノです。

rmsmf の移植版が mfsr 、txprobe の移植版が mfprobe です。

コマンドの仕様や使い方は、変わっていません。

配布している実行ファイルは、Native AOT で発行した物で、OS のネイティブコードなので、実行に .NET の RunTime は必要無いです。

インストール方法:

以下の Download から mfsr.win.zip をダウンロードして解凍し、展開された .exe ファイルを 環境変数 Path の通ったフォルダーにコピーすれば、コンソールアプリとして使用できます。
単純な 実行ファイルだけで成り立つツールなので、インストーラーは用意していません。

**Download** [https://github.com/motoi-tsushima/mfsr/releases/tag/v1.0.2.5](https://github.com/motoi-tsushima/mfsr/releases/tag/v1.0.2.5)

**Repository** [https://github.com/motoi-tsushima/mfsr](https://github.com/motoi-tsushima/mfsr)

### rmsmf txprobe mfsr mfprobe 使い方ガイド

rmsmf txprobe mfsr mfprobe 、四つのコマンドの使い方を、以下の記事で説明しています。

[rmsmf-txprobe & mfsr-mfprobe 使い方の分かりやすい解説](/tool_rmsmf_guide/)

