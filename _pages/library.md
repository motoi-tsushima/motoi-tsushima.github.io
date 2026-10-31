---
title: "Library"
layout: single
classes: wide
permalink: /library/
author_profile: true
---
.NET 開発で利用できるクラスライブラリを NuGet パッケージとして公開しています。

## NuGet package

### SnowStack.EncodingProbe

文字エンコーディングの不明なテキストファイルをバイナリ解析して、その文字エンコーディングを推測するクラスライブラリです。[Tools](/tools/) の mfsr・mfprobe や、PowerShell の Resolve-Encoding（EncodingProbe.PowerShell）も、内部でこのライブラリを使用しています。

.NET(Core) 10 と .NET Framework 4.8 の両方に対応しています。

[SnowStack.EncodingProbe NuGet Package 解説](/encodingprobe_guide/)

1.2.0 では判定処理を改修し、東アジアの環境で欧米の言語のテキストを誤判定する問題と、香港の Big5 の扱いを改善しました。公開 API は変更していません。変更の詳しい内容は、以下の記事で解説しています。

[SnowStack.EncodingProbe 1.2.0 解説 — ファイル出力・変換コマンドと世界の言語への対応](/encodingprobe_1_2_0/)

1.3.0 では、BOM 無しの UTF-16 / UTF-32 の判定など、判定処理の不具合を直しました。公開 API は変更していませんが、カルチャー名の検証などで結果が変わる点があります。変更の詳しい内容は、以下の記事で解説しています。

[SnowStack.EncodingProbe 1.3.0 解説 — CSV の読み書きと、文字を失わない書き込み](/encodingprobe_1_3_0/)

#### 外字（私用領域）の扱い

Shift_JIS などの外字を、このライブラリがどう読み書きするかを解説しています。外字を含む古いデータを UTF-8 へ移行する方は、ご覧ください。

[外字（私用領域）の扱い — SnowStack.EncodingProbe](/encodingprobe_private_use_area/)

