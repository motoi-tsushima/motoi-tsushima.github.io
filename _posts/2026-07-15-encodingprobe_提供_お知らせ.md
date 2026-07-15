---
title: "SnowStack.EncodingProbe 1.0.0 正式版リリースのお知らせ"
categories:
  - お知らせ
tags:
  - Character Encoding
  - 文字エンコーディング
  - Character Encoding Detection
  - 文字エンコーディング検出
  - PowerShell
  - Cmdlet
  - コマンドレット
  - Class library
  - クラスライブラリ
  - NuGet package
---

2026年7月14日に SnowStack.EncodingProbe の正式版バージョン 1.0.0 のリリースを開始したことを、ご報告します。

2026年6月6日から固定ページでプレリリース版の告知は行っていましたが、オープンソース製品として完成水準に到達し正式版として提供開始したので、ブログで改めて告知します。

## SnowStack.EncodingProbe とは

今回提供開始した SnowStack.EncodingProbe とは、

PowerShell環境や.NETとWindowsアプリの開発において、Shift_JIS, UTF-8, UTF-16LEなどの文字エンコーディングの混在問題の解決のために開発した NuGetパッケージとPowerShellコマンドレットです。

以前から提供している mfprobe, mfsr というコマンドツールに内蔵されていた文字エンコーディング検出処理を、クラスライブラリとして切り出し、NuGet パッケージとして一般提供したものです。

また、その NuGet パッケージを使用して PowerShell コマンドとして文字エンコーディング検出を行う事が可能なコマンドレットも同梱しています。

このコマンドレットを使用するとPowerShellスクリプト（*.ps1）の中でテキストファイルの文字エンコーディング検出が可能になります。

前者のクラスライブラリは、パッケージ名 SnowStack.EncodingProbe として NuGet.org より提供しています。

後者のコマンドレットは、パッケージ名 SnowStack.EncodingProbe.PowerShell として PowerShell Gallery より提供しています。

両者は、同一のソリューションに纏めており、一つの GitHub リポジトリでオープンソース・ソフトウェアとして提供しています。

以下のコマンドでソースのダウンロードが可能です。

```
git clone https://github.com/motoi-tsushima/SnowStack.EncodingProbe
```

ライセンスは MITライセンスです。

## 機能説明書

NuGet パッケージとPowerShellコマンドレットの機能説明は既に6月のプレリリース時点で、固定ページで公開しております。

インストール方法や使い方については、こちらの固定ページをご覧下さい。

[SnowStack.EncodingProbe NuGet Package 解説](https://snow-stack.net/encodingprobe_guide/)

[SnowStack.EncodingProbe.PowerShell 解説](https://snow-stack.net/encodingprobe_powershell_guide/)

## 今後の予定

バージョン 1.0.0 の開発をしていく過程で、PowerShellの Get-Content コマンドなどの -Encoding オプションに使用される文字エンコーディング・フレンドリ名が、PowerShellのバージョンによって大きく異なることがわかりました。

また、Resolve-Encoding をユーザーが使いこなすには、フレンドリ名の知識が必要になると思います。

まず、文字エンコーディング・フレンドリ名がどのようなものなのか、解説記事を提供する必要があります。

また、.NETをよく知らないユーザーのために、.NET Framework と .NET(Core) の違いや、Windows PowerShell5.1 と PowerShell6.2以降(最新は7.6.x) の違いを把握してもらう必要があります。

今後は、まずこれらの必須知識の解説記事をブログに書いて公開するつもりです。（これらは単純な公開知識なので、AIを積極活用するつもりです）

また、Windows PowerShell5.1 は旧式PowerShellでありながら、Windows標準搭載のShellでもあるため、多くのユーザーにとって標準の環境でもあります。

この旧型PowerShellは、UTF-16LE を標準文字エンコーディングとしており、BOM無しUnicodeを扱えません。

しかし、現在のWindows環境ではGitやWSLが使用可能で、必然的にBOM無しのUTF-8テキストファイルが混在します。

Windows PowerShell5.1 上でも、BOM無しのUTF-8テキストファイル等、幅広い文字エンコーディングを扱う必要があります。

今後、SnowStack.EncodingProbe.PowerShell では、次のバージョン 1.1.0 で、これらの問題に対処する機能を開発する予定です。

ご報告まで。



SnowStack は、GitHubで公開しているソフトウェアの補助的な解説用のWebサイトです。

このブログは、どちらかと言えば「固定ページ」中心に運営しているので、ブログ部分は補助的な解説記事ぐらいしか書いていません。更新ペースが遅いのもそのような運用方針だからです。

ご了承ください。

