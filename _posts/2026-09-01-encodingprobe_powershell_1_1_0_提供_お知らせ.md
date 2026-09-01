---
title: "SnowStack.EncodingProbe.PowerShell 1.1.0 リリースのお知らせ"
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

2026年9月1日に SnowStack.EncodingProbe.PowerShell のバージョン 1.1.0 のリリースを開始したことを、ご報告します。

これまで提供していた バージョン 1.0.2 では、`Resolve-Encoding`による文字エンコーディングの検出だけしかできませんでした。

今回リリースしたバージョン 1.1.0 では、Windows PowerShell 5.1 と PowerShell 7.x.x の双方において、文字エンコーディングを自動検出した後に、閲覧・編集・追記ができるようになりました。

具体的には以下のコマンドレットを追加しました。

`Get-ProbedContent` : テキストファイルの文字エンコーディングを検出して表示する。

`Set-ProbedContent` : 指定テキストファイル（隣接ファイルなど）の文字エンコーディングを検出して出力する。

`Add-ProbedContent` : 指定テキストファイル（追記対象など）の文字エンコーディングを検出して追記する。

`ConvertTo-DotNetEncoding` : 文字エンコーディングを指定して、`System.Text.Encoding` オブジェクトを作成する。

詳細は、以下の固定ページで解説しています。

[SnowStack.EncodingProbe.PowerShell 1.1.0 新コマンド解説](https://snow-stack.net/encodingprobe_probed_content/)



また、NuGet パッケージの EncodingProbe 1.1.0 も同時にリリースしています。こちらは機能の変更は無く、参照している `UTF.Unknown`のUpgradeに対応したことにより、リビルドして再リリースしました。

## 機能説明書

NuGet パッケージとPowerShellコマンドレットの機能説明は既に6月のプレリリース時点で、固定ページで公開しております。

インストール方法や使い方については、こちらの固定ページをご覧下さい。

[SnowStack.EncodingProbe NuGet Package 解説](https://snow-stack.net/encodingprobe_guide/)

[SnowStack.EncodingProbe.PowerShell 解説](https://snow-stack.net/encodingprobe_powershell_guide/)

