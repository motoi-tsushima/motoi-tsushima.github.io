---
title: "SnowStack.EncodingProbe.PowerShell 1.3.0 リリースのお知らせ"
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
  - CSV
  - Class library
  - クラスライブラリ
  - NuGet package
---

（リリース日）に SnowStack.EncodingProbe.PowerShell のバージョン 1.3.0 のリリースを開始したことを、ご報告します。

## 何が新しくなったか

**読み込んでいるファイルに上書きしても、中身を失わなくなりました。** 1.2.0 までは、ハードリンクを使ったり、読み込むファイルを書き込み先の後ろに並べたりすると、検出をすり抜けて、読み込み中のファイルの中身が消えることがありました。1.3.0 では、パスの文字列ではなくファイルの実体で比べて、書き込みを止めます。標準の `Out-File` などは、この場合に中身を失います。ただし Linux / macOS では、ハードリンクの別名と、大文字・小文字だけが違う名前（`t.txt` と `T.TXT`）を、同じファイルと判定しません（[既知の制限](https://snow-stack.net/encodingprobe_1_3_0/#linux--macos-の既知の制限)の D2・D5）。

以下のコマンドレットを追加しました。

`Import-ProbedCsv` : 文字エンコーディングを判定して、CSV ファイルを読み込む。

`Export-ProbedCsv` : 文字エンコーディング・BOM・改行を指定して、CSV ファイルに書き出す。

どちらも、Windows PowerShell 5.1 でも PowerShell 7.x と同じ結果になります。5.1 の標準の `Export-Csv` のように日本語が `?` になることも、5.1 の標準の `Import-Csv` のように改行が CR だけの CSV を 0 件として読むこともありません。追記するときは、追記先の文字エンコーディングと改行を引き継ぎ、最後の行に改行が無ければ補ってから追記します。

`Resolve-Encoding`・`Import-ProbedCsv`・`Export-ProbedCsv` には、先頭の N 行だけで判定する `-DetectionLineCount` を追加しました。巨大なファイルを速く判定するためのものです。`Import-ProbedCsv` / `Export-ProbedCsv` では、使う前にファイル全体を検証し、判定を誤っていれば何もせずに止まります。

**書き込みで、文字を黙って失わないようにしました。** 書き込み先の文字エンコーディングで表現できない文字（Shift_JIS のファイルに追記するハングルなど）を、これまでは `?` にして書いていましたが、1.3.0 ではエラーにして、ファイルを元のままにします。

このほか、名前に `[ ]` があるファイルの扱いや、`-Encoding ansi` の値など、既存のコマンドの改善と不具合の修正も行っています。

詳細は、以下の固定ページで解説しています。

[SnowStack.EncodingProbe 1.3.0 解説 — CSV の読み書きと、文字を失わない書き込み](https://snow-stack.net/encodingprobe_1_3_0/)

また、**NuGet パッケージの `SnowStack.EncodingProbe 1.3.0` も同時にリリースしています**。公開 API の変更はありませんが、BOM 無しの UTF-16 / UTF-32 の判定など、判定処理の不具合を 3 件直しました。NuGet パッケージを直接お使いの方にも、1.3.0 への更新をお勧めします。

## 更新するときの注意

1.3.0 には、1.2.0 までと結果が変わる変更があります。主なものは次のとおりです。

- 文字が `?` になるのを承知で書いていたスクリプトは、エラーで止まるようになります
- 英数字だけのファイル（`us-ascii` と判定されるもの）に日本語を追記すると、UTF-8 で書かれます
- Windows の `-Encoding ansi` / `oem` は、システムのロケールの値になります（「地域」の「形式」をシステムのロケールと違う値にしている場合に、値が変わります）
- `Get-ChildItem` から渡したファイルは、名前に `[ ]` があってもワイルドカードとして展開されず、そのファイルそのものを読み書きします
- Linux / macOS で 1 MiB を超えるファイルに上書きすると、ハードリンクが切れ、ACL と拡張属性が消えます。Linux / macOS には、ほかにも既知の制限があります

一覧と詳しい説明は、解説ページの「[1.2.0 からの挙動の変更](https://snow-stack.net/encodingprobe_1_3_0/#120-からの挙動の変更)」「[更新するときの注意](https://snow-stack.net/encodingprobe_1_3_0/#更新するときの注意)」「[Linux / macOS の既知の制限](https://snow-stack.net/encodingprobe_1_3_0/#linux--macos-の既知の制限)」をご覧ください。

## 入手方法

PowerShell ギャラリーに登録しています。

```
# 新しくインストールする（PowerShell 7.x）
Install-PSResource SnowStack.EncodingProbe.PowerShell

# インストール済みの 1.2.0 などを更新する
Update-PSResource SnowStack.EncodingProbe.PowerShell
```

モジュールを読み込んだままの PowerShell では、更新した版に切り替わりません。更新した後は、PowerShell を開き直してください。

Windows PowerShell 5.1 でのインストール方法は、以下の記事で解説しています。

[SnowStack.EncodingProbe.PowerShell を Windows PowerShell 5.1 へインストールする方法](https://snow-stack.net/encodingprobe_powershell_install_ps51/)

NuGet パッケージは、NuGet.org から入手できます。

```
dotnet package add SnowStack.EncodingProbe
```

## 機能説明書

インストール方法や使い方については、こちらの固定ページをご覧下さい。

[SnowStack.EncodingProbe.PowerShell 解説](https://snow-stack.net/encodingprobe_powershell_guide/)

[SnowStack.EncodingProbe NuGet Package 解説](https://snow-stack.net/encodingprobe_guide/)
