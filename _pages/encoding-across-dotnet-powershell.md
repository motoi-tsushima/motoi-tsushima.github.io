---
title: "EncodingProbe から見た .NET / PowerShell の文字コード環境差"
permalink: /encoding-across-dotnet-powershell/
excerpt: ".NET Framework と .NET、Windows PowerShell 5.1 と PowerShell 7 ── 4本の記事を「文字コード」の一点で束ね直し、SnowStack.EncodingProbe が解く問題へつなげます。"
toc: true
toc_label: "目次"
toc_sticky: true
last_modified_at: 2026-07-18
---

このシリーズでは、ここまで4本にわたって .NET と PowerShell の歴史と環境差をたどってきました。最後の1本は、そのすべてを **「文字コード(エンコーディング)」** という一点で束ね直します。

- [.NET の歴史 ─ Framework から Core、そして統一へ](/dotnet-history/)
- [.NET Framework 4.8 と .NET 10 の違い](/dotnet-framework-48-vs-net10/)
- [PowerShell の歴史 ─ cmd から Windows PowerShell、そして PowerShell 7 へ](/powershell-history/)
- [Windows PowerShell 5.1 と PowerShell 6.2 以降の違い](/windows-powershell-51-vs-pwsh-62/)

一見ばらばらに見えるこれらの差異は、実は1本の背骨でつながっています。**Windows PowerShell 5.1 は .NET Framework の上で動き、PowerShell 6 以降は .NET(旧 .NET Core)の上で動く。** だから 5.1 と 7 の文字コードの挙動差は、突き詰めれば Framework と .NET のランタイム差の「結果」なのです。本記事はその結果を整理し、`SnowStack.EncodingProbe` がどの問題を解くのかまでを一続きで示します。

## なぜ環境ごとに挙動が変わるのか

差異の根っこにあるのは、たった一つの設計思想の転換です。

**旧世代(.NET Framework / Windows PowerShell 5.1)は「ANSI コードページ前提」で作られています。** ここでいう ANSI コードページとは、システム既定のレガシーコードページのことで、日本語環境では CP932(Windows の Shift_JIS 実装、Windows-31J)を指します。文字列とバイト列を相互変換するときの既定が、OS のロケールに依存していたわけです。

**新世代(.NET / PowerShell 7系)は「UTF-8(BOM なし)前提」に切り替わりました。** 既定はロケールに依存せず、どのプラットフォームでも UTF-8。クロスプラットフォーム化(Windows / Linux / macOS)を果たした .NET にとって、Windows のコードページを既定にし続ける理由はもうありません。

この転換点こそが、シリーズを貫くすべての差異の源です。歴史的経緯は[.NET の歴史](/dotnet-history/)と[PowerShell の歴史](/powershell-history/)を、具体的な差分は[記事2](/dotnet-framework-48-vs-net10/)・[記事4](/windows-powershell-51-vs-pwsh-62/)を参照してください。

## 具体的なハマりどころ一覧

シリーズ全体で扱った文字コード関連の差を、環境ごとに一覧にまとめます(2026年7月時点)。

| 項目 | .NET Framework / Windows PowerShell 5.1 | .NET / PowerShell 7系 | 詳細 |
|---|---|---|---|
| `Encoding.Default` の意味 | システムの ANSI コードページ(日本語環境で **CP932**) | 常に **UTF-8(BOM なし)** | [記事2](/dotnet-framework-48-vs-net10/) |
| Shift_JIS 等のコードページ利用 | 標準で利用可 | `System.Text.Encoding.CodePages` パッケージと `CodePagesEncodingProvider` の登録が必要 | [記事2](/dotnet-framework-48-vs-net10/) |
| PowerShell の `-Encoding utf8` | **BOM 付き** UTF-8 | **BOM なし** UTF-8(別途 `utf8NoBOM` / `utf8BOM` を持つ) | [記事4](/windows-powershell-51-vs-pwsh-62/) |
| PowerShell の既定出力エンコーディング | コマンドレットごとにまちまち(`Out-File` / `>` は UTF-16LE、他は ANSI など) | **BOM なし UTF-8 に統一** | [記事4](/windows-powershell-51-vs-pwsh-62/) |

`.NET Core` 以降でコードページプロバイダーを登録する典型は次のとおりです。Framework では書く必要のなかった一行です。

```csharp
// .NET(5 以降)で Shift_JIS(CP932)を扱う前に一度だけ実行
System.Text.Encoding.RegisterProvider(
    System.Text.CodePagesEncodingProvider.Instance);

var sjis = System.Text.Encoding.GetEncoding("shift_jis"); // これで取得できるようになる
```

PowerShell 側で `-Encoding utf8` が環境によって BOM の有無を変えてしまう問題、そして「WebName で代用できない」理由については、既存記事で実機検証込みで掘り下げています。

- 関連記事:**「PowerShell の `-Encoding utf8NoBOM` は、なぜ WebName で代用できないのか」**
  <!-- TODO: 既存記事の permalink をここに設定 -->

## SnowStack.EncodingProbe が解く問題

上のハマりどころは、どれも **「今どのエンコーディングで、BOM は付いているのか」を確実に把握できていないこと** に起因します。既定任せにした瞬間、Framework と .NET で、あるいは 5.1 と 7系で、結果が食い違うからです。`SnowStack.EncodingProbe` は、この不確かさを2つの役割で取り除きます。

**1. 検出(判別)** ── バイト列やファイルから、BOM の有無と内容のヒューリスティックによって、実際のエンコーディングを推定します。BOM 付き UTF-8 / UTF-16 / UTF-32 の判定に加え、日本語(CP932・EUC-JP・ISO-2022-JP など)を含むレガシーコードページも判別対象です。上表の「BOM の有無」「既定出力の違い」に、書き出す前・読み込む前に確証を与えます。

**2. 名前の正規化** ── 検出したエンコーディングを、文脈ごとに異なる呼び名へ橋渡しします。同じ文字コードでも、.NET の WebName、PowerShell の `-Encoding` に渡す名前、コードページ番号はそれぞれ別物です。まさに utf8NoBOM 記事で扱った「WebName をそのまま `-Encoding` に使えない」問題に、正しい対応名を返すことで応えます。

そして重要なのが **環境をまたぐ一貫性** です。`SnowStack.EncodingProbe` は `net48` と `net10.0` の両方をマルチターゲットしています。`net48` は Framework / Windows PowerShell 5.1 の世界、`net10.0` は .NET / PowerShell 7系の世界に対応します。**同じ判別ロジックを、Framework 側でも .NET 側でも、5.1 でも 7系でも使える** ── これが、シリーズで見てきた「環境ごとに既定が違う」問題への直接の答えです。

導入と使い方の詳細は、パッケージの README(取説)を参照してください。

## 実務の指針

環境差を踏まえた、日々の作業での勘所をまとめます。

**新規は UTF-8(BOM なし)を既定に。** .NET / PowerShell 7系の思想に合わせるのが素直です。ただし読み手に Windows PowerShell 5.1 が含まれる場合は BOM の有無に注意します。たとえば PowerShell のモジュール定義ファイル(`.psd1` / `.psm1`)は、5.1 互換のために **BOM 付き UTF-8** が求められる場面があります。既定に流されず、用途ごとに BOM 有無を意識してください。

**Framework / 5.1 環境で CP932 を扱うなら、`Encoding.Default` に頼らず明示する。** ロケール依存の既定は、他環境に持ち込んだ途端に破綻します。

**.NET で Shift_JIS などを扱うなら、コードページプロバイダーの登録を忘れない。** 登録なしに `GetEncoding("shift_jis")` を呼ぶと例外になります([記事2](/dotnet-framework-48-vs-net10/))。

**PowerShell の出力は、環境の既定を当てにせず `-Encoding` を明示する。** 5.1 と 7系で既定が違う以上、明示だけが移植性を担保します([記事4](/windows-powershell-51-vs-pwsh-62/))。

**由来の不明なファイルを読むときは、まず判別する。** 中身の想像で読み始めると文字化けを踏みます。ここが `SnowStack.EncodingProbe` の出番です。

## まとめと関連リンク

.NET Framework から .NET への統一、Windows PowerShell から PowerShell 7 への刷新 ── 4本で見てきた変化は、文字コードの視点で並べ直すと「ANSI コードページ前提から UTF-8 前提へ」という一つの流れに収れんします。環境差そのものは消えませんが、**明示すること・判別すること** で、確実に御せます。

### シリーズの記事

- [記事1:.NET の歴史 ─ Framework から Core、そして統一へ](/dotnet-history/)
- [記事2:.NET Framework 4.8 と .NET 10 の違い](/dotnet-framework-48-vs-net10/)
- [記事3:PowerShell の歴史 ─ cmd から Windows PowerShell、そして PowerShell 7 へ](/powershell-history/)
- [記事4:Windows PowerShell 5.1 と PowerShell 6.2 以降の違い](/windows-powershell-51-vs-pwsh-62/)

### OSS(検出・判別ツール)

- **NuGet**:[SnowStack.EncodingProbe](https://www.nuget.org/packages/SnowStack.EncodingProbe/)
- **PowerShell Gallery**:[SnowStack.EncodingProbe.PowerShell](https://www.powershellgallery.com/packages/SnowStack.EncodingProbe.PowerShell/)

<!-- 参考:バージョン事実は 2026年7月時点。.NET 10(2025-11 LTS)、PowerShell 7.6(2026-03 GA、.NET 10 上、現行 LTS。最新パッチ 7.6.3)。更新時に再確認すること。 -->
