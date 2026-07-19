---
title: "Tools"
layout: single
classes: wide
permalink: /tools/
author_profile: true
---
文字エンコーディング・改行コード・BOM を扱うコンソールツールと、PowerShell コマンドレットを公開しています。

rmsmf-txprobe と mfsr-mfprobe は同じ仕様のコマンドの、対象ランタイムが異なる並列リリースです。動作環境に合わせてどちらかを選んでください。

## Console tools

### mfsr and mfprobe（.NET 10 / Native AOT 版）

rmsmf・txprobe を .NET 10 へ移植した、現行世代のコマンドです。

rmsmf の移植版が mfsr 、txprobe の移植版が mfprobe です。コマンドの仕様や使い方は、rmsmf・txprobe と変わっていません。

配布している実行ファイルは、Native AOT で発行した物で、OS のネイティブコードなので、Windows では .NET の Runtime は必要無いです。

.NET Runtime がインストールされた環境であれば、macOS・Linux でも動作します。

インストール方法は、こちらに纏めています。

[mfsr-mfprobe インストール解説](/mfsr_install_guide/)

### rmsmf and txprobe（.NET Framework 4.8 版）

.NET Framework 4.8 環境向けに、以前から公開しているコマンドです。

rmsmf は、複数のファイル内の複数の文字列を一括置換するコマンドラインツールです。文字エンコーディングの変換や、改行コードの変換に、BOM（Byte Order Mark）の制御も可能です。

txprobe は、複数のファイルの文字エンコーディング・BOM・改行コードを確認するコマンドツールです。

.NET Framework 4.8 が標準で入っている Windows 環境で、追加ランタイム無しにそのまま使いたい場合は、こちらを選んでください。

インストール方法は、こちらに纏めています。

[rmsmf-txprobe インストール解説](/rmsmf_install_guide/)

### rmsmf txprobe mfsr mfprobe 使い方ガイド

rmsmf txprobe mfsr mfprobe 、四つのコマンドの使い方を、以下の記事で説明しています（コマンド仕様は共通です）。

[rmsmf-txprobe & mfsr-mfprobe 使い方の分かりやすい解説](/tool_rmsmf_guide/)

## PowerShell コマンドレット

### Resolve-Encoding と EncodingProbe.PowerShell

SnowStack.EncodingProbe.PowerShell コマンドパッケージのインストール方法と、その中の Resolve-Encoding コマンドレット使い方を解説しています。

[SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)

