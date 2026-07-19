---
layout: splash
title: "雪塚 (snow-stack)"
excerpt: ".NET / PowerShell 向けの文字エンコーディング関連 OSS ツール・ライブラリと、技術解説記事を公開しています。"
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header.jpg
  actions:
    - label: "Tools を見る"
      url: "/tools/"
    - label: "GitHub"
      url: "https://github.com/motoi-tsushima"
feature_row:
  - image_path: /assets/images/tools-placeholder.svg
    alt: "EncodingProbe"
    title: "EncodingProbe"
    excerpt: "文字エンコーディングを推測する .NET クラスライブラリと PowerShell コマンドレット。"
    url: "/encodingprobe_guide/"
    btn_label: "詳しく見る"
    btn_class: "btn--primary"
  - image_path: /assets/images/tools-placeholder.svg
    alt: "mfsr"
    title: "mfsr"
    excerpt: ".NET 10 / Native AOT 版の文字列一括置換コマンド。文字コード・改行コード・BOM を変換します。"
    url: "/mfsr_install_guide/"
    btn_label: "詳しく見る"
    btn_class: "btn--primary"
  - image_path: /assets/images/tools-placeholder.svg
    alt: "mfprobe"
    title: "mfprobe"
    excerpt: ".NET 10 / Native AOT 版の文字エンコーディング・BOM・改行コード確認コマンド。"
    url: "/tool_rmsmf_guide/"
    btn_label: "詳しく見る"
    btn_class: "btn--primary"
---

{% include feature_row %}

## Guides

.NET・PowerShell・文字エンコーディングまわりの解説記事です。

- [.NET 10 の画面有りアーキテクチャの解説とサンプルコード](/dotnet10-GUI-architecture-list/)
- [PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか （フレンドリ名の解説）](/why-utf8nobom-cannot-be-webname/)
- [.NET の歴史 ─ Framework から Core、そして統一へ](/dotnet-history/)
- [.NET Framework 4.8 と .NET 10 の違い](/dotnet-framework-48-vs-net10/)
- [PowerShell の歴史 ─ cmd から Windows PowerShell、そして PowerShell 7 へ](/powershell-history/)
- [Windows PowerShell 5.1 と PowerShell 6.2 以降の違い](/windows-powershell-51-vs-pwsh-62/)
- [EncodingProbe から見た .NET / PowerShell の文字コード環境差](/encoding-across-dotnet-powershell/)
- [Avalonia UI のライセンス規定を解説します](/avaloniaui-license-guide/)

[Guides 一覧を見る &raquo;](/guides/){: .btn .btn--light-outline}

## 新着のお知らせ

<ul>
{% for post in site.posts limit:3 %}
  <li>{{ post.date | date: "%Y/%m/%d" }}&nbsp;<a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>

[お知らせ一覧を見る &raquo;](/blog/){: .btn .btn--light-outline}
