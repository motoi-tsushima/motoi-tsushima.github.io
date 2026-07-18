---
title: ".NET Framework 4.8 と .NET 10 の違い"
permalink: /dotnet-framework-48-vs-net10/
toc: true
toc_label: "目次"
toc_sticky: true
last_modified_at: 2026-07-18
---

同じ C# のコードを書いていても、それが動く基盤は「.NET Framework 4.8」と「.NET 10」で大きく異なります。ソースはよく似ていても、ランタイム・配置方法・対応 OS、そして **文字コードの既定の扱い** までが別物です。

この記事は、[.NET の歴史 ─ Framework から Core、そして統一へ](/dotnet-history/) で整理した流れの続きです。歴史編で「なぜ二つの系統に分かれ、なぜ再び一つの `.NET` に統一されたのか」を追いました。本記事はその **到達点である 4.8 と 10 を、実務上の差として具体化** します。

> 本記事のバージョン事実は **2026年7月時点** のものです。.NET のバージョンは毎年更新されるため、参照時は Microsoft の一次情報で最新を確認してください。

---

## 1. 前提 ─ 「同じ C#」でも土台が違う

まず全体像です。両者はコンパイラや言語仕様の多くを共有しますが、**実行を支えるランタイムが別系統** です。

| 観点 | .NET Framework 4.8 | .NET 10 |
|---|---|---|
| 系統 | Windows 同梱のレガシー系(4.x 系の最終) | 統一 `.NET`(旧 .NET Core 系)の現行 LTS |
| 初出/位置づけ | 2019-04(4.8.1 は 2022-08)。新機能開発は事実上終了 | 2025-11 リリース。2026年7月時点の最新 LTS |
| ランタイム | 単一の巨大ランタイム(CLR)を OS が保持 | モジュール化されたランタイム。アプリと同梱可 |
| 対応 OS | Windows 専用 | Windows / Linux / macOS |
| 既定エンコーディング | システムの ANSI コードページ(日本語環境で CP932) | 常に UTF-8(BOM なし) |

重要な前提を一つ。**.NET Framework 4.8 は「廃止」されたわけではありません。** 新機能開発は 4.8 / 4.8.1 で事実上止まっていますが、Windows に同梱され、セキュリティ更新とサービシングは継続中です。「終わったもの」ではなく「保守フェーズに入ったもの」として扱うのが正確です。一方の .NET 10 は LTS(Long Term Support:長期サポート)として2028年11月までサポートされます。

---

## 2. アーキテクチャ ─ 単一の巨大ランタイム vs モジュール化

### .NET Framework 4.8:OS に一体化した単一ランタイム

.NET Framework は、**Windows にインストールされた単一のランタイムを全アプリが共有** する設計です。アプリは「マシンに正しいバージョンの .NET Framework が入っている」ことを前提に配置されます。利点は配置物が小さいこと、欠点は **環境に強く依存すること**、そして複数バージョンの並行運用がしにくいことです。

### .NET 10:モジュール化と自己完結

.NET(Core 以降)は、必要な機能を NuGet パッケージ単位で組み合わせる **モジュール化** された構造を持ちます。これにより、配置方法の選択肢が広がりました。

- **フレームワーク依存(framework-dependent)**:実行環境に .NET ランタイムが入っている前提。配置物は小さい。
- **自己完結(self-contained)デプロイ**:ランタイムごとアプリに同梱する。対象マシンに .NET が入っていなくても動く。
- **単一ファイル発行(single-file publish)**:実行ファイルを1つにまとめて配布しやすくする。
- **Native AOT**:事前コンパイルでランタイム起動コストを削り、小さく速い実行ファイルを生成する(対応シナリオに制約あり)。

```bash
# 自己完結 + 単一ファイルで発行(例:Windows x64)
dotnet publish -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true
```

この「アプリ側がランタイムを抱えられる」という発想の転換が、**環境依存からの解放** と、後述するクロスプラットフォーム対応の土台になっています。

---

## 3. クロスプラットフォーム ─ Windows 専用 vs 3 OS 対応

- **.NET Framework 4.8** は **Windows 専用** です。CLR も BCL も Windows API を前提に作られており、Linux や macOS では動きません。
- **.NET 10** は **Windows / Linux / macOS** で動作します。同じコードベースをコンテナで Linux に載せる、macOS で開発して Linux にデプロイする、といった運用が標準です。

日本語環境の開発者にとって、この違いは「ローカルの Windows(CP932 前提)で書いたコードを、UTF-8 前提の Linux コンテナで動かす」ときの **文字コード事故** として表面化しがちです。この点は6章で扱います。

---

## 4. パフォーマンス

.NET(Core 以降)は、世代を追うごとに **GC・JIT・ライブラリ実装** の最適化を積み重ねてきました。.NET Framework 4.8 は保守フェーズにあり、こうした継続的な性能改善の対象ではありません。おおまかな傾向として、次のような領域が効いています。

- **GC の進化**:リージョンベース GC、DATAS(Dynamically Adapting To Application Sizes)などによるメモリ効率の改善。
- **JIT の進化**:動的 PGO(Profile-Guided Optimization)の既定化、階層コンパイル、より賢いインライン化。
- **ライブラリの再実装**:`Span<T>` / `Memory<T>` を活用した割り当て削減、ホットパスの最適化。

> 具体的な数値(スループット改善率・メモリ削減率)は版ごとに異なります。本文で数字を出す際は、Microsoft の「Performance Improvements in .NET(各版)」ブログを一次情報として、対象バージョンの計測値を確認・引用してください。ここで固定値を書き込むと古びます。

実務上の要点は「**新しいアプリほど、コードを変えずにランタイム更新だけで速くなる余地がある**」という点です。これは 4.8 に留まる限り得られない利点です。

---

## 5. API の差 ─ .NET 側で非対応・置換されたもの

統一 .NET は Framework の全 API をそのまま引き継いだわけではありません。**Windows 密結合の技術や、設計を刷新した領域** は、非対応化・置換されています。移行時に必ず当たる代表例を挙げます。

| .NET Framework の技術 | .NET 10 での扱い | 代替・備考 |
|---|---|---|
| ASP.NET Web Forms | 非対応 | Blazor / ASP.NET Core MVC / Razor Pages へ |
| WCF(サーバー側) | 非対応 | gRPC、ASP.NET Core Web API、CoreWCF(コミュニティ) |
| .NET Remoting | 非対応 | gRPC / HTTP ベースの通信へ |
| `AppDomain`(複数ドメイン) | 実質非対応 | `AssemblyLoadContext` へ |
| Windows Workflow Foundation | 非対応 | 代替フレームワーク/再設計 |
| WPF / WinForms | 対応(**Windows 専用**) | クロスプラットフォームにはならない点に注意 |

ポイントは、**「非対応=移行できない」ではなく「設計を置き換える必要がある」** ということです。WebForms のようにパラダイムごと変わるものは書き換え、WPF / WinForms のように動くが Windows 専用のままのものは「動くが可搬にはならない」と理解して計画します。

---

## 6. エンコーディングの違い(通し糸①)

ここが、本シリーズを貫く「文字コード」というテーマの最初の実例です。**4.8 と 10 で、既定の文字コードの扱いそのものが変わっています。**

### `Encoding.Default` の意味が違う

`Encoding.Default` は同名のプロパティですが、**返すエンコーディングが系統によって別物** です。

- **.NET Framework 4.8**:`Encoding.Default` は **システムの ANSI コードページ**。日本語環境では **CP932(Windows の Shift_JIS 実装、Windows-31J)**。
- **.NET 10**:`Encoding.Default` は **常に UTF-8(BOM なし)**。OS のロケールに依存しません。

```csharp
using System.Text;

// .NET Framework 4.8(日本語環境)
Console.WriteLine(Encoding.Default.CodePage);       // 932
Console.WriteLine(Encoding.Default.EncodingName);   // 日本語 (シフト JIS)

// .NET 10(どの OS・ロケールでも)
Console.WriteLine(Encoding.Default.CodePage);       // 65001
Console.WriteLine(Encoding.Default.EncodingName);   // Unicode (UTF-8)
```

同じ `Encoding.Default` を「システム既定」のつもりで書いたコードが、Framework では CP932、.NET では UTF-8 として動く。**これが Windows→Linux 移行時の文字化けの典型的な震源** です。

### CP932 / Shift_JIS を使うには登録が要る

もう一つの重要な差です。**.NET(Core 以降)では、CP932 などのレガシーコードページが標準では使えません。**

- **.NET Framework 4.8**:`Encoding.GetEncoding("shift_jis")` や `Encoding.GetEncoding(932)` が **標準で使えました**。
- **.NET 10**:同じ呼び出しは、そのままでは `NotSupportedException` になります。使うには **`System.Text.Encoding.CodePages` パッケージ** を参照し、**`CodePagesEncodingProvider` を登録** する必要があります。

```csharp
using System.Text;

// .NET(Core 以降)で CP932 等を使う前準備
Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);

// これで初めて取得できる
var sjis = Encoding.GetEncoding("shift_jis");  // CP932
byte[] bytes = sjis.GetBytes("日本語");
```

`csproj` 側の参照:

```xml
<ItemGroup>
  <PackageReference Include="System.Text.Encoding.CodePages" Version="..." />
</ItemGroup>
```

Framework 時代は意識せず使えていた CP932 が、.NET では **明示的なパッケージ参照とプロバイダー登録を経なければ触れない**。日本語を扱う開発者にとっては、地味ですが確実に踏むポイントです。

### なぜツールが要るのか

このように、**同じ C# コードでも「どの系統で動くか」で文字コードの既定が変わり、CP932 のような日本語レガシーエンコーディングは追加の準備を要求する** ── これが、`SnowStack.EncodingProbe` のような **文字コードの検出・判別ツール** が必要になる背景の一つです。手元のバイト列がどのエンコーディングなのかを取り違えると、`Encoding.Default` の解釈違いと相まって被害が広がります。

環境ごとの文字コード挙動の差を横断的に束ね直し、EncodingProbe がどの問題を解くのかは、シリーズの締めで扱います。 → [EncodingProbe から見た .NET / PowerShell の文字コード環境差](/encoding-across-dotnet-powershell/)

---

## 7. プロジェクト形式とツール

配置や依存管理の仕組みも更新されています。移行実務で最初に触るのはここです。

### csproj の形式

- **.NET Framework 4.8**:従来の **冗長な csproj**。ファイルを一つずつ列挙し、多くのメタデータを明示する形式が一般的でした。
- **.NET 10**:**SDK スタイル csproj**。既定でフォルダ配下のソースを含め、記述が大幅に簡潔になります。

```xml
<!-- SDK スタイル csproj(.NET 10) -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

### NuGet 参照:`packages.config` vs PackageReference

- **`packages.config`**:旧来方式。パッケージ一覧を専用ファイルで管理し、依存の推移解決が弱い。.NET Framework の古いプロジェクトに残っています。
- **PackageReference**:csproj に直接依存を書く現行方式。推移的依存の解決やバージョン統合に優れ、SDK スタイルの前提です。

.NET Framework でも PackageReference は使えるため、**移行の第一歩として先に `packages.config` → PackageReference へ寄せておく** と、その後の SDK スタイル化・.NET 化がスムーズになります。

---

## 8. 移行の指針 ─ 何を持っていけて、何を書き換えるか

大づかみの判断軸です。

**ほぼそのまま持っていけるもの**

- C# の言語資産、ビジネスロジック、`Span<T>` 等を使わない一般的な BCL 利用
- PackageReference 化済みの依存(対応パッケージであれば)
- WPF / WinForms(ただし **Windows 専用のまま**。可搬にはならない)

**書き換え・再設計が要るもの**

- ASP.NET Web Forms → Blazor / ASP.NET Core への作り替え
- WCF サーバー / .NET Remoting → gRPC・Web API・CoreWCF 等へ
- `AppDomain` 前提の設計 → `AssemblyLoadContext`
- **文字コード前提のコード**(6章):`Encoding.Default` の解釈、CP932 利用箇所の洗い出しとプロバイダー登録

**進め方の目安**

1. まず対象を **.NET Framework のまま SDK スタイル + PackageReference 化** する。
2. .NET Upgrade Assistant 等で非対応 API を棚卸しする。
3. **文字コード依存箇所を先に特定** しておく(移行後にいちばん静かに壊れる領域)。
4. 置き換えが必要な技術を再設計し、テスト環境(Windows Sandbox / コンテナ等)で挙動を確認する。

---

## 9. まとめと次への橋渡し

.NET Framework 4.8 と .NET 10 は、**同じ C# を書きながら、動く基盤・配置・対応 OS・そして文字コードの既定が異なる** 別系統です。4.8 は保守フェーズの安定版、.NET 10 は継続的に進化する現行 LTS ── どちらを選ぶかは「安定した Windows 資産の維持」か「クロスプラットフォームと性能・機能の前進」かで分かれます。

そして本記事で見た **`Encoding.Default` の意味の違い** と **CP932 利用時のプロバイダー登録** は、シリーズを貫く「文字コード環境差」というテーマの入口です。同じ問題は PowerShell 側(Windows PowerShell 5.1 と PowerShell 7 の既定エンコーディングの差)にも現れ、最終回でまとめて [EncodingProbe](/encoding-across-dotnet-powershell/) に接続します。

---

### 関連記事

- 前提の歴史編:[.NET の歴史 ─ Framework から Core、そして統一へ](/dotnet-history/)
- シリーズの締め:[EncodingProbe から見た .NET / PowerShell の文字コード環境差](/encoding-across-dotnet-powershell/)

### 参考(一次情報)

- Microsoft .NET Blog「Announcing .NET 10」および各版の「Performance Improvements in .NET」
- .NET support policy(サポート期限・LTS/STS の定義)
- Microsoft Learn:`Encoding.Default` / `CodePagesEncodingProvider` / .NET Framework から .NET への移行
