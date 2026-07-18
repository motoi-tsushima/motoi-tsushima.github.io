---
title: "Windows PowerShell 5.1 と PowerShell 6.2 以降の違い"
permalink: /windows-powershell-51-vs-pwsh-62/
toc: true
toc_label: "目次"
toc_sticky: true
last_modified_at: 2026-07-18
---

同じ「PowerShell」という名前でも、`powershell.exe` と `pwsh.exe` は別物です。片方は Windows に同梱された **Windows PowerShell 5.1**、もう片方は個別にインストールする **PowerShell**(6.0 以降)で、動く基盤も既定の文字コードも言語機能も違います。しかも両者は同じマシンに**併存**できるため、「どちらで動かしているか」を意識しないままスクリプトの挙動が変わって戸惑う――というのは、日本語環境ではとくに起きがちです。

この記事では、Windows PowerShell 5.1 と PowerShell 6.2 以降の違いを、実行基盤・可用性・モジュール互換性・エンコーディング・言語機能の観点で整理します。

> 本記事は「.NET と PowerShell ─ 歴史と環境差」シリーズの第4回です。5.1 と 6 の分岐が「なぜ」生まれたのかは、[PowerShell の歴史](/powershell-history/)と[.NET の歴史](/dotnet-history/)で扱っています。本記事はその「では、具体的に何が違うのか」を担当します。

---

## `powershell.exe` と `pwsh` ── まず名前と実体を分ける

混乱の多くは名前に由来します。本シリーズの[用語の取り決め](/dotnet-history/#用語)に沿って、まず整理しておきます。

| 実行体 | 製品名 | バージョン | 位置づけ |
|---|---|---|---|
| `powershell.exe` | **Windows PowerShell** | 1.0 〜 **5.1** | Windows 同梱・Windows 専用・保守フェーズ |
| `pwsh`(`pwsh.exe`) | **PowerShell** | **6.0 以降** | 個別インストール・クロスプラットフォーム・現行 |

- 「Windows PowerShell」は 1.0〜5.1 を指す固有名です。5.1 が最終版で、**新機能の追加は終わっています**(ただし廃止ではなく、Windows に同梱されたまま保守は継続中です)。
- 「PowerShell」は 6.0 以降(旧 "PowerShell Core")を指します。6.x 期は "PowerShell Core" と呼ばれましたが、**7.0 以降は "Core" が外れて単に "PowerShell"** に統一されました。本記事の「6.2 以降」という基準は、この 6.x〜7.x 系を指します。

両者は別実行体なので、同じ Windows マシンに並べて入れておけます。既存の 5.1 資産を壊さずに、新しい `pwsh` を検証できるということです。

---

## 実行基盤 ── .NET Framework か、.NET か

5.1 と 6+ の違いの**大半は、実は土台の違いの"結果"**です。

- **Windows PowerShell 5.1** … **.NET Framework** 上で動作します。したがって Windows 専用で、.NET Framework の制約(ANSI コードページ前提など)をそのまま引き継ぎます。
- **PowerShell 6.0 以降** … **.NET**(旧 .NET Core)上で動作します。クロスプラットフォームで、UTF-8 前提という設計思想を引き継ぎます。

現行の **PowerShell 7.6 は .NET 10 上**で動作します(2026-03 GA、2026-07 時点の最新パッチは 7.6.3。7.6 は現行 LTS)。PowerShell のメジャー/マイナーは、下敷きにする .NET の世代と歩調を合わせて進みます(7.4=.NET 8、7.6=.NET 10 など)。

ランタイムそのものの違い――単一の巨大ランタイム vs モジュール化、Windows 専用 vs クロスプラットフォーム、`Encoding.Default` の意味の違いなど――は、[.NET Framework 4.8 と .NET 10 の違い](/dotnet-framework-48-vs-net10/)で詳しく扱っています。本記事のエンコーディング差は、そこで説明したランタイム差が PowerShell の表層に現れたもの、と捉えると見通しが良くなります。

---

## クロスプラットフォームと可用性

| | Windows PowerShell 5.1 | PowerShell 6.2 以降 |
|---|---|---|
| 対応 OS | Windows のみ | Windows / Linux / macOS |
| 入手方法 | **OS 同梱**(既定で入っている) | **個別インストール**(winget / MSI / パッケージマネージャ等) |
| 更新 | Windows Update に紐づく | 独立してリリース(年次 + パッチ) |

5.1 は「Windows に最初から入っている」ことが強みでもあり、制約でもあります。バージョンが OS に固定されるため、5.1 より新しい機能は 5.1 側には来ません。一方 PowerShell 6+ は OS から独立してリリースされるので、Windows・Linux・macOS で**同じバージョンの `pwsh`** を揃えられます。運用スクリプトを複数 OS で共通化したい場合、この独立性が効いてきます。

---

## モジュール互換性

5.1 → 6+ で最も現実的にひっかかるのが、モジュールが動くかどうかです。

- **.NET Framework 依存のモジュールは、そのままでは PowerShell 6+ で動かない**ことがあります。とくに Windows 固有の COM / WMI / Win32 API に深く依存するものや、Framework 専用アセンブリを参照するものが該当します。
- PowerShell 7 には **Windows PowerShell 互換機能**があり、`Import-Module -UseWindowsPowerShell`(および暗黙のフォールバック)で、5.1 側のモジュールを別プロセスの Windows PowerShell 上で読み込み、リモーティング経由で呼び出せます。ただしプロセス境界を跨ぐため、オブジェクトの型が一部デシリアライズされた状態になるなどの制約があります。
- 純粋なスクリプトモジュール(`.psm1` / `.psd1`)や、両対応をうたうモジュールは、多くがそのまま動きます。

移行時は「動かないモジュールがあれば互換機能で橋渡しし、可能なら 7 系ネイティブ対応版に置き換える」という段取りになります。

---

## エンコーディングの違い(通し糸②)

ここが本シリーズの通し糸――「なぜ環境ごとに文字コードの挙動が違うのか」――が最もはっきり表面化する箇所です。設計思想が **ANSI コードページ前提(Framework/5.1)から UTF-8 前提(.NET/7)へ転換した**結果が、日々のコマンドレットの既定値に出ています。

### 既定の出力エンコーディングが違う

**Windows PowerShell 5.1 は、コマンドレットごとに既定エンコーディングがバラバラ**です。これが 5.1 時代の混乱の温床でした。

| 操作 | 5.1 の既定 |
|---|---|
| `Out-File` / リダイレクト `>` | **UTF-16LE(BOM 付き)** |
| `Set-Content` / `Add-Content` | **ANSI コードページ**(日本語環境なら CP932) |
| `Export-Csv` / `Export-Clixml` 等 | ANSI コードページ 等 |

同じ「ファイルに書く」でも、`>` で出すか `Set-Content` で出すかで中身のバイト列が変わる、という状態です。日本語環境では、片方は UTF-16LE、片方は CP932 になり、後工程で文字化けの原因になります。

**PowerShell 6 以降は、既定エンコーディングが `utf8NoBOM`(BOM なしの UTF-8)にほぼ統一**されました。コマンドレットをまたいで既定が揃うため、「どのコマンドレットで書いたか」を気にせずに UTF-8(BOM なし)で書き出せます。

| 操作 | PowerShell 6+ の既定 |
|---|---|
| ファイル出力系コマンドレット全般 | **UTF-8(BOM なし)= `utf8NoBOM`** |

### `-Encoding` の値の意味も違う

既定だけでなく、`-Encoding` に渡す値の意味も 5.1 と 6+ で違います。ここが最もハマりやすいポイントです。

- **Windows PowerShell 5.1**:`-Encoding utf8` は **BOM 付き UTF-8** を意味します。5.1 には「BOM なし UTF-8」を明示する値がありません(`utf8` を指定すると必ず BOM が付く)。
- **PowerShell 6 以降**:`utf8NoBOM`(BOM なし)と `utf8BOM`(BOM 付き)が別々に用意され、既定の `utf8` は **BOM なし**を指します。加えて 7.4 以降では `ansi` も指定できるようになりました。

```powershell
# Windows PowerShell 5.1
"あ" | Out-File -Encoding utf8 a.txt    # → BOM 付き UTF-8(先頭に EF BB BF)

# PowerShell 7.x
"あ" | Out-File -Encoding utf8 a.txt    # → BOM なし UTF-8(= utf8NoBOM)
"あ" | Out-File -Encoding utf8BOM b.txt # → BOM 付きを明示したいとき
```

つまり **同じ `-Encoding utf8` という指定が、5.1 と 7 系で別の結果(BOM の有無)になる**わけです。スクリプトを 5.1 と 7 系の両方で回すなら、`utf8NoBOM` / `utf8BOM` を明示するのが安全です。

> なぜ `utf8NoBOM` を `WebName` などの別名で代用できないのか――CodePage / WebName / フレンドリ名の区別と、PowerShell 側の `-Encoding` 値がどう対応するのかは、既存記事「[PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか](/PLACEHOLDER-utf8nobom-webname/)」で詳しく検証しています。<!-- TODO: 既存記事の実スラッグに差し替え -->

なお、日本語環境で Shift_JIS(CP932)を扱う場合、.NET(=PowerShell 6+ の土台)では標準で CP932 が使えず、`System.Text.Encoding.CodePages` パッケージと `CodePagesEncodingProvider` の登録が必要になる、という別の落とし穴もあります。これは PowerShell 単体というより .NET 側の事情なので、[.NET Framework 4.8 と .NET 10 の違い](/dotnet-framework-48-vs-net10/)を参照してください。

こうした「環境ごとに文字コードの既定・意味が変わる」問題を検出・判別するために作ったのが、本シリーズの締めで紹介する SnowStack.EncodingProbe です。詳しくは[EncodingProbe から見た .NET / PowerShell の文字コード環境差](/encoding-across-dotnet-powershell/)へ。

---

## 言語機能の追加(7 系)

エンコーディング以外にも、PowerShell 7 系では言語そのものが拡張されています。5.1 では書けなかった構文が使えるため、スクリプトの書き味が変わります。主なものを挙げます。

- **三項演算子**:`$result = $cond ? "真" : "偽"`(7.0〜)
- **null 合体演算子**:`$x = $a ?? "既定値"`、代入版 `$a ??= "既定値"`(7.0〜)
- **パイプラインチェーン演算子**:`cmd1 && cmd2`(成功時に次を実行)、`cmd1 || cmd2`(失敗時に次を実行)(7.0〜)
- **null 条件アクセス**:`${var}?.Prop`、`${arr}?[0]`(7.1〜)
- **並列処理**:`ForEach-Object -Parallel { ... } -ThrottleLimit N`(7.0〜)
- 改善されたエラー表示と `Get-Error`(7.0〜)

これらは Windows PowerShell 5.1 には**入りません**(5.1 は新機能の追加が終わっているため)。5.1 と 7 系の両方で動かす必要があるスクリプトでは、これらの新構文を避けるか、実行バージョンで分岐する必要があります。

---

## まとめ ── 「別物が併存している」前提で扱う

Windows PowerShell 5.1 と PowerShell 6.2 以降は、名前こそ似ていますが、**土台(.NET Framework / .NET)から違う別物**です。その違いが、可用性・モジュール互換性・エンコーディング既定・言語機能として表面化します。とくにエンコーディングは、既定値も `-Encoding utf8` の意味も両者で異なるため、両対応スクリプトでは BOM の有無を明示するのが安全策です。

次回(本シリーズ最終回)は、ここまでの4本を「文字コード」の一点で束ね直し、環境差を検出・判別する SnowStack.EncodingProbe に接続します。 → [EncodingProbe から見た .NET / PowerShell の文字コード環境差](/encoding-across-dotnet-powershell/)

---

### 参考

- Microsoft Learn: PowerShell Support Lifecycle / about_Character_Encoding
- PowerShell Team DevBlog: Announcing PowerShell 7.6 (LTS) GA(2026-03-18)
- 関連:[.NET Framework 4.8 と .NET 10 の違い](/dotnet-framework-48-vs-net10/) / [PowerShell の歴史](/powershell-history/)

<sub>※ バージョン事実は 2026-07 時点。PowerShell 7.6 が現行 LTS(最新パッチ 7.6.3)、下敷きは .NET 10 LTS。実際の執筆・更新時に最新を再確認してください。</sub>
