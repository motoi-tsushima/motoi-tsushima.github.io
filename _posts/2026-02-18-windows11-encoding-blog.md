---
title: "Windows 11 環境でのテキストファイル新規作成時のデフォルトエンコーディング検証の総まとめ"
categories:
  - 文字エンコーディング
tags:
  - Character Encoding
  - Shift_JIS
  - UTF-8
  - UTF-16
  - BOM
---

## はじめに

これまで、以下の二つの記事で「Windows環境におけるテキストファイルの文字エンコーディングとBOMの扱い」について、検証確認してきました。

[Windows 11 標準アプリ 文字エンコーディング対応状況まとめ](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/02/14/encoding-report.html)

[Windowsコンソールの文字エンコーディング対応状況を検証する](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/02/16/windows-console-encoding-report.html)

前の記事は、メモ帳など主要アプリの対応状況を、
後の記事は、コンソールの対応状況を検証した結果報告です。

今回、これに加えて Visual Studio と SSMS(SQL Server Management Studio) についても検証しました。

この記事では、過去の検証結果と、Visual Studio と SSMS(SQL Server Management Studio) の「テキストファイルの新規作成」を行ったときの文字エンコーディングとBOMの有無の状態を、まとめました。

主に、それぞれのアプリやツールのデフォルト設定で新規作成した場合のレポートです。

ここから、Microsoftの意図と、Windows環境のテキスト問題が明確になってきます。

まず、検証結果のまとめを、ご覧下さい。

---

## 検証結果一覧

| ツール / 操作 | デフォルトエンコーディング | BOM |
|:--|:--|:--:|
| メモ帳 | UTF-8 | なし |
| cmd (`echo >`) | Shift_JIS | — |
| Windows PowerShell 5.1 (`>`) | UTF-16LE | あり |
| Windows PowerShell 5.1 (`Set-Content`) | Shift_JIS | — |
| PowerShell 7.x (`>`) | UTF-8 | なし |
| PowerShell 7.x (`Set-Content`) | UTF-8 | なし |
| VS Code | UTF-8 | なし |
| Excel CSV保存 | Shift_JIS（UTF-8 BOM付きも選択可） | ※後述 |
| Visual Studio 2026（C# / VB.NET / C++） | UTF-8 | あり |
| Visual Studio 2026（json / js / ts） | UTF-8 | なし |
| Visual Studio 2026（xml） | UTF-8 | あり |
| SSMS（TEXT / SQL / js） | UTF-8 | なし |
| SSMS（XML） | UTF-8 | あり |

---

## 各ツールの詳細

### メモ帳

- **デフォルト:** UTF-8（BOM無し）
- 保存ダイアログから他のエンコーディングも選択可能。以前のバージョンではUTF-8 BOM付きやANSI（Shift_JIS）がデフォルトだったが、現在のWindows 11ではBOM無しUTF-8に変更されている。

### コマンドプロンプト（cmd）

```cmd
echo 日本語です > text.txt
```

- **デフォルト:** Shift_JIS
- システムロケールのコードページ（日本語環境では CP932 = Shift_JIS）に従う。

### Windows PowerShell 5.1

```powershell
# リダイレクト演算子
"日本語です" > text.txt        # → UTF-16LE（BOM付き）

# Set-Content
"日本語です" | Set-Content text.txt   # → Shift_JIS
```

- リダイレクト演算子（`>`）と `Set-Content` で**エンコーディングが異なる**点に注意。
- `Set-Content` は `-Encoding` パラメータで明示的に指定できる。

### PowerShell 7.x

```powershell
# リダイレクト演算子
"日本語です" > text.txt        # → UTF-8（BOM無し）

# Set-Content
"日本語です" | Set-Content text.txt   # → UTF-8（BOM無し）
```

- PowerShell 7.x ではどちらの方法でも**UTF-8（BOM無し）に統一**された。
- PowerShell 5.1 から移行する際は、エンコーディングの挙動変化に注意が必要。

### VS Code

- **デフォルト:** UTF-8（BOM無し）
- ステータスバーからエンコーディングを変更可能。設定で `files.encoding` を変更すればデフォルトも変えられる。

### Excel CSV保存

- **通常保存:** Shift_JIS
- **「UTF-8（カンマ区切り）」で保存:** UTF-8（BOM付き）
- UTF-8（BOM無し）で保存する選択肢は**用意されていない**。

### Visual Studio 2026

| ファイル種別 | エンコーディング | BOM |
|:--|:--|:--:|
| C#（.cs） | UTF-8 | あり |
| VB.NET（.vb） | UTF-8 | あり |
| C++（.cpp / .h） | UTF-8 | あり |
| json | UTF-8 | なし |
| xml | UTF-8 | あり |
| JavaScript（.js） | UTF-8 | なし |
| TypeScript（.ts） | UTF-8 | なし |

> **注意:** VB.NET の `Program.vb` だけが、なぜか UTF-8（BOM無し）で生成される。テンプレートの不整合またはバグの可能性がある。

### SSMS（SQL Server Management Studio）

| ファイル種別 | エンコーディング | BOM |
|:--|:--|:--:|
| TEXT（.txt） | UTF-8 | なし |
| SQL（.sql） | UTF-8 | なし |
| XML（.xml） | UTF-8 | あり |
| JavaScript（.js） | UTF-8 | なし |

---

## 傾向の整理

検証結果から、以下の傾向が読み取れます。

### モダンなツール → UTF-8（BOM無し）

メモ帳、PowerShell 7.x、VS Code など、比較的新しいツールはUTF-8（BOM無し）をデフォルトとしています。Web標準やLinux環境との親和性を重視した流れです。

### レガシー系 → Shift_JIS / UTF-16LE

コマンドプロンプトや Windows PowerShell 5.1 は、従来のWindowsの文字コード体系を引き継いでいます。既存のバッチファイルやスクリプトとの互換性が理由と考えられます。

### ソースコード（C#, VB.NET, C++）→ UTF-8 BOM付き

Visual Studio はコンパイラがソースファイルのエンコーディングを確実に認識できるよう、BOMを付与しています。

### Web系ファイル（json, js, ts）→ UTF-8（BOM無し）

Web の世界ではBOM無しが標準のため、Visual Studio・SSMS ともにWeb系ファイルはBOM無しで生成されます。

### XML → UTF-8 BOM付き

Visual Studio、SSMS のいずれにおいても XML は BOM付きです。XML宣言の `encoding` 属性との整合性を意識した実装と考えられます。

---

## 検証結果まとめ

Windows 11 環境でも、ツールやファイル形式によってデフォルトのエンコーディングはバラバラです。特に以下のポイントは押さえておきたいところです。

1. **Windows PowerShell 5.1 のリダイレクト（`>`）は UTF-16LE（BOM付き）** — 他のツールで開くと文字化けの原因になりやすい
2. **PowerShell 5.1 では `>` と `Set-Content` でエンコーディングが異なる** — 同じPowerShellでも出力方法で結果が変わる
3. **Excel CSV は UTF-8（BOM無し）で保存できない** — 他システム連携時に注意
4. **Visual Studio のソースコードは BOM付き、Web系ファイルは BOM無し** — 混在するプロジェクトでは意識が必要

チームでの開発では `.editorconfig` や各ツールの設定でエンコーディングを統一しておくと、トラブルを未然に防げます。

---

## 所感

以前書いた [Windows 固有の文字エンコーディング問題](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/02/06/Windows-%E5%9B%BA%E6%9C%89%E3%81%AE%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E5%95%8F%E9%A1%8C.html) という記事の中で、私は「MicrosoftがUNIX標準に合わせるのは、時間の問題」と言いました。

今回、Windows環境の主要ソフトウェア全ての文字エンコーディングとBOMの有無を確認して、得られた認識は、「Microsoftは既にBOM無しUTF-8のテキストに軸足を移し始めている」ということです。

改行コードは、全てのツールとアプリにおいて CR-LF を堅持しています。

しかし、メモ帳・VS Code・PowerShell7.x・SSMS においては、既にBOM無しUTF-8が主流となっています。

それに対し、レガシーデータの多い Excel が 今でも Shift_JIS をメインとし、補助的に BOM付きUTF-8 を採用しているのは、理解できるとして、Visual Studio 2026 が今でも C#・C++ においても BOM付きUTF-8 を採用しているのは、やや意外でした。

開発環境は、全てBOM無しUTF-8になるのかと思いましたが、Visual Studio はExcel同様にレガシーデータが多いので、簡単にBOM無しUTF-8を採用できないのが、理解できます。

### AI でも文字化けを解消できない

　Visual Studio 2026 に統合された GitHub Copilot Agent を使用して、AI Agent にコーディングをやらせてみると、「BOM無しUTF-8のソースファイルを作成しようとして、AIがMAUIコンパイラを呼び出すと、コンパイラが Shift_JIS と誤解してしまい、MAUI画面が文字化けする」という現象が頻繁に起きます。
例えば PowerShell 7.x のコマンドレットで新規ファイルを作成すると、「BOM無しUTF-8」のテキストファイルを作成しますが、Visual Studio 環境では「BOM無しはShift_JIS」「BOM付きはUTF-8」と解釈するので、PowerShellコマンドレットで「BOM無しUTF-8」を作成すると、Shift_JISと解釈されてその後の編集で文字化けするのです。PowerShell 7.x と Visual Studio の間で、UTF-8ファイルの解釈が統一されていないことで起きる不具合です。
なお、PowerShell 7.x のリダイレクト演算子（>）や単純な `Out-File` , `Set-Content` ではBOM付きUTF-8を作成できません。`-Encoding` 指定で `Set-Content -Encoding utf8BOM` ならば作成可能です。

この現象が起きると AIエージェントは、自力では文字化けを解消できません。何度も失敗して最後は人間に助けを求めます。(私の観測範囲ではMAUIコンパイラがBOM無しUTF-8に対応していません、他にもこのような問題があると思われます)

 AIエージェント は素晴らしいコーディング能力を発揮しますが、独力では文字化け一つ解決できないのです。

世間ではあまり意識されませんが AIエージェント の内部では、コンパイラなど AI (LLM)ではない従来型のソフトウェアが呼び出されて使用されています。

 AIエージェント は、LLMと従来型アルゴリズムの組み合わせでできているので、Windows 環境のように文字エンコーディング問題が、従来型アルゴリズムの段階で解決していないと、 AIエージェント でも問題を解決できません。

文字エンコーディング問題などは、現状では人間が手動で解決しなければならない問題の一つです。
 AIエージェント が文字化けで停止することを回避するには、あらかじめ人間が文字化け問題を解決しておかなければならないのです。(ルンバも、部屋を片付けておかなければ掃除ができないのと同じです)

Visual Studio 2026 に統合された GitHub Copilot Agent が文字化けに悩み人間に助けを求める状況は、 AIエージェント の構造をよく表している現象だと思います。

### 文字化け問題は深刻化している

私は、2020年に [rmsmf](https://snow-stack.net/tool_rmsmf_guide/) を作ったとき、「文字エンコーディング問題など、3年もすれば解消するだろう」と思っていました。だから rmsmf のアップデートは一時停滞していました。

しかし、現在のWindows環境を検証してみると、昔より文字エンコーディングとBOMのトラブルは多くなっているように見えます。

昔の文字エンコーディング問題は、「Shift_JIS か UTF-8 か UTF-16LE」の問題が主で、解決策も「ファイル先頭にBOMを付ける」ことで済んでいました。

しかし、MicrosoftがWSLを導入して以降、新たな問題が生じています。

それは「BOM無しUTF-8」の問題です。当初MicrosoftはUTF-8を「BOM付き」で導入することで、Shift_JISとの混在による混乱を回避していました。

しかし、WSLによりLinux環境とテキストファイルを共有するようになってから、「BOM無しUTF-8」のテキストをWindows環境に受け入れなければならなくなりました。

結果として、それまでテキストファイルの先頭のBOMだけ確認していれば、複数文字エンコーディングの共存ができていたのに、「BOM無しUTF-8」の存在により「BOMによる文字エンコーディングの判別」ができなくなってしまいました。

しかも、テキストファイルの文字エンコーディングを判定する手段は、Windowsユーザーには提供されていません。

メモ帳 や VS Code は一応、「BOM無しUTF-8」を自動判定してくれます。しかし、一般ユーザーはそれが「BOM付きUTF-8」なのか「BOM無しUTF-8」なのかを区別できません。

Visual Studio の検証結果を見れば分かるように、Windows環境の C# や C++ や VB.NET・XML などのソースファイルは、今でも「BOM付きUTF-8」が基本になっています。

Excel CSV や VBA なども Shift_JIS をデフォルトとしながら「BOM付きUTF-8」が基本になっています。

一方、メモ帳 と VS Code と PowerShell 7.x は、「BOM無しUTF-8」が基本になっており、PowerShell 7.x に至っては「BOM付きUTF-8」テキストを作成する手段が存在しません。

Windows PowerShell 5.1 もOSの深部との結びつきが強く、まだ現役で標準装備されています。
Windows PowerShell 5.1 は、「BOM付きUTF-16LE」テキストが標準の上に、コマンドレットによって「Shift_JIS」テキストと混在しており、日本語文字エンコーディングが統一されていません。

つまり、Windows環境では、テキストファイルの「Shift_JIS」「BOM付きUTF-8」「BOM無しUTF-8」「BOM付きUTF-16LE」が混在していながら、それぞれを区別する手段が提供されていないのです。

「どうして、いまさら文字エンコーディング対応など、調べているのか」と言えば、WSL導入以降のWindows環境において、BOMを中心とした文字エンコーディングの混乱が拡大しているからです。

### 改行コードの問題も無視できない

Microsoftは GitHubを買収し、積極的に Windows環境に GitHub のアプリケーションを受け入れています。

その最たるものが、GitHub Copilot Agent だと思います。

GitHub は Git をベースにした Linux と OSS 文化圏のサービスです。そのシステムは全て UNIX 標準で統一されています。

Linux OS と UNIXのmacOS のテキスト改行コードは LF で統一されていますが、Windows のテキスト改行コードは、CR-LF で統一されており、GitHub へソースを Commit-Push するとき変換する必要があります。

今のWindowsテキストの改行コードは CR-LF のまま変更される様子は無いですが、これまでの文字エンコーディング対応の変化を見る限り、改行コードが CR-LF から LF へ変更される可能性を否定できません。

現状で CR-LF の CR に存在価値があるとは思えないからです。

### 残りの牙城の寿命は少ない

UNIX標準テキストが、「BOM無しUTF-8」ならば、古いテキスト「Shift_JIS」「BOM付きUTF-8」「BOM付きUTF-16LE」を支えるアプリケーションとレガシーデータの牙城は、Excel, Visual Studio, Windows PowerShell 5.1 ということになります。

Visual Studio は、過去に Shift_JIS を標準としていた時代があり、現在の「BOM付きUTF-8」標準を廃して「BOM無しUTF-8」を標準とするのも時間の問題でしょう。

PowerShell は 7.x で既に「BOM無しUTF-8」を標準としており、Windowsのシステムも Windows PowerShell 5.1への依存部分を徐々に PowerShell 7.x へ移行していて、旧 5.1 が廃止されることは約束された未来です。

 レガシーデータの牙城として唯一終わりが見えないのは、Excel CSV と VBA です。

Microsoft は Word においては、テキストインポートのときに、ユーザーに文字エンコーディングを選択させる形で、複数文字エンコーディングに対応しています。

しかし、何故か Excel では依然として文字エンコーディングの判別を、テキスト先頭の BOM に依存しています。
また、デフォルトでの CSVファイルへの保存も Shift_JIS のままです。

Excel 以外の全てのアプリとツールは、明らかに「BOM無しUTF-8」への移行に向かっているのですが、Excel だけはその兆候が見えません。

### BOMが消えるのはいつなのか

Windows環境において、そのテキストファイルの規格が「BOM無しUTF-8」へ完全に切り替わるまで、まだまだ時間がかかると思われます。

Visual Studio が、そのソースファイルを全て「BOM無しUTF-8」へ完全に切り替えるのは、あと二・三世代ぐらい後のバージョンになると思っています。少なくとも現在の 2026 ではオプション設定などで「BOM無しUTF-8」を標準とする機能がやっと追加された段階です。

Windows PowerShell 5.1 が完全に廃止され、PowerShell 7 以降に統合されるのは、10年は先の話になると思います。(関係ないけどコントロールパネルはいつ無くなるのでしょうね？)

Excel CSV と VBA については、先が全く見えません。Excel による CSV出力や、CSV の Excel による閲覧は、本当に一般ユーザーの中でよく使用される機能です。
Excel VBA も利用している人々が多すぎて、消滅することが想像し難い状況です。(モダンExcelはどこへ行った？)

Excel 自身が Shift_JIS の CSV を今でも量産して、文字エンコーディング問題を拡大しています。

将来の Windows環境が「BOM無しUTF-8」へ切り替わることは、ほぼ確実なのですが、今後10年以上は Windows環境の文字エンコーディングとBOMの問題に、付き合わなければならないと思われます。

また、先に説明した改行コードの問題も未解決であることもお忘れなく。

### まだまだ終わらない文字エンコーディング問題の後始末

これまで報告してきたように、Windows環境では2000年代に一度、Shift_JISからUTF-16LEへ移行し、その後さらにBOM付きUTF-8へ移行したことがあります。

その後、2010年代後期に一部の機能が「BOM付きUTF-8」から「BOM無しUTF-8」へ移行しています。この移行はこれまで報告してきたように、まだ「移行途中」であり、古いShift_JIS・BOM付きUTF-8とUTF-16LEが一部のアプリで併存している状況です。

結果として、Windows環境には Shift_JIS・BOM付きUTF-16LE・BOM付きUTF-8・BOM無しUTF-8 のテキストが混在する状況になっています。

BOM付きUTF-8・BOM無しUTF-8 が混在しているので、BOMによる識別もできません。

Shift_JISとBOM付きUTFだけが混在していた昔より、文字エンコーディングの識別が困難になっています。

しかも、文字エンコーディングの識別はユーザーの自己責任です。

このような文字エンコーディング周りの状況が悪化しているのを受けて、最近私は mfprobe-mfsr というツールを開発してリリースしました。

これは、2020年に開発して公開していた rmsmf という .NET Framework 4.8用の ツールを改良し .NET10 Native AOT で再構築して提供したものです。
元々は、文字列置換ツールだったのですが、Windows環境特有の文字エンコーディング問題を無視しては置換処理すらできないことから、文字エンコーディング判別変換機能とBOM確認追加削除機能と改行コードの変換機能を有しています。

Windows環境で、手軽に複数ファイルを対象に文字エンコーディングやBOMの有無や改行コードの種類を確認・変換できるツールが見当たらなかったので、自分で開発したツールです。

もし良ければ、文字エンコーディング問題への対処にご利用ください。

以下のページで公開しています。

[rmsmf-txprobe & mfsr-mfprobe 使い方の分かりやすい解説](https://snow-stack.net/tool_rmsmf_guide/)

MITライセンスで公開しています。オープンソースのフリーソフトです。



---

## 検証環境

- **OS:** Windows 11 Pro 25H2
- **PowerShell:** Windows PowerShell 5.1 / PowerShell 7.5.4
- **Visual Studio:** Visual Studio 2026
- **VS Code:** 1.109.4
- **SSMS:** SQL Server Management Studio 22.3
- **Excel:** Microsoft Office 2024

---

*最終更新: 2026年2月*
