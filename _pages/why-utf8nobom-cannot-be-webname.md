---
title: "PowerShell の -Encoding utf8NoBOM は、なぜ WebName で代用できないのか （フレンドリ名の解説）"
layout: single
classes: wide
permalink: /why-utf8nobom-cannot-be-webname/
author_profile: true
---

## はじめに —— 同じ UTF-8 に、なぜ 3 つの名前があるのか

PowerShell でファイルを UTF-8 で書き出そうとしたとき、次の 3 つの書き方が目に入ります。

```powershell
[System.Text.Encoding]::GetEncoding(65001)      # 65001
[System.Text.Encoding]::GetEncoding("utf-8")    # utf-8
Set-Content -Path a.txt -Value 'あ' -Encoding utf8   # utf8
```

`65001`、`utf-8`、`utf8`。どれも同じ UTF-8 を指しているように見えます。実際、最初の 2 つはまったく同じ `Encoding` オブジェクトを返します。

ところが、3 つ目だけは事情が違います。PowerShell の `-Encoding` には `utf8` のほかに `utf8BOM` と `utf8NoBOM` という値があり、この 2 つは **CodePage も WebName もまったく同じ**なのに、書き出されるファイルの中身が変わります。

なぜでしょうか。`utf8NoBOM` は、`utf-8` という WebName では表現できない何かを持っているのです。

本記事では、.NET と PowerShell で文字エンコーディングを指定するときに登場する 3 つの名前——**CodePage**、**WebName**、そして **フレンドリ名**——を整理し、とくに **WebName とフレンドリ名の違い**を明らかにします。

結論を先に一文で言ってしまいます。

> **WebName は「文字集合の名前」であり、フレンドリ名は「書き出し方の名前」です。**

この違いさえ掴めば、`utf8BOM` と `utf8NoBOM` が別々に用意されている理由も、`-Encoding shift_jis` が PowerShell 5.1 で動かない理由も、すべて一本の線でつながります。

### 「フレンドリ名」という言葉について

先に用語の断りを入れておきます。

**「フレンドリ名」は .NET の公式用語ではありません。** `System.Text.Encoding` クラスに `FriendlyName` というプロパティは存在しません。PowerShell の `-Encoding` パラメーターに指定する `utf8` や `unicode` といった短いキーワードを、慣習的にそう呼んでいるにすぎません。

さらに紛らわしいことに、.NET には `Encoding.EncodingName` という「人間向けの表示名」を返すプロパティがあり、こちらを「フレンドリ名」と呼ぶ流儀もあります。両者はまったくの別物です。

そこで本記事では、次のように定義して話を進めます。

> **本記事における「フレンドリ名」とは、PowerShell の `-Encoding` パラメーターが受け付ける短縮キーワードのことを指します。**

### 本記事のスコープ

本記事は「名前」の話に集中します。以下は扱いません。

- 文字エンコーディングそのものの解説（Shift-JIS とは何か、UTF-8 の仕組みは、など）
- .NET Framework 4.8 と .NET 10 の違い
- PowerShell 5.1 と PowerShell 6.2 以降の全般的な違い（既定の出力エンコーディングなど）

ただし、**フレンドリ名の意味そのものがバージョンで変わった箇所**については、避けて通れないので本記事内で扱います。

### 検証環境

- Windows 11（日本語版）
- Windows PowerShell 5.1
- PowerShell 7.6.3

---

## CodePage —— 数値による識別子

`System.Text.Encoding.CodePage` プロパティは、そのエンコーディングを表す **コードページ番号**（`int`）を返します。Windows が長年使ってきた識別体系で、数値であるがゆえに曖昧さがありません。

```powershell
[System.Text.Encoding]::UTF8.CodePage        # 65001
[System.Text.Encoding]::Unicode.CodePage     # 1200
[System.Text.Encoding]::GetEncoding(932)     # 番号を指定して取得できる
```

代表的なコードページ番号を挙げておきます。

| CodePage | 内容 |
|---|---|
| 65001 | UTF-8 |
| 1200 | UTF-16 リトルエンディアン |
| 1201 | UTF-16 ビッグエンディアン |
| 12000 | UTF-32 リトルエンディアン |
| 12001 | UTF-32 ビッグエンディアン |
| 20127 | US-ASCII |
| 932 | Shift-JIS（Windows 拡張版、いわゆる CP932） |
| 51932 | EUC-JP |
| 50220 | ISO-2022-JP |
| 1252 | 欧文 ANSI（Windows-1252） |

CodePage は、いわば **商品に貼られた JAN コード**のようなものです。人間が見て意味は分かりませんが、機械にとっては最も扱いやすい識別子です。整数なので比較も高速で、設定ファイルのキーや判定ロジックの内部表現に向いています。

一方で、限界もあります。

- Windows 由来の体系であり、世界標準の名前と 1 対 1 に対応しているわけではありません。
- そして本記事にとって決定的なことですが、**BOM の有無を表現できません。** BOM 付き UTF-8 も BOM なし UTF-8 も、CodePage はどちらも 65001 です。

---

## WebName —— IANA charset 名

`System.Text.Encoding.WebName` プロパティは、**IANA に登録された charset 名**（`string`）を返します。

```powershell
[System.Text.Encoding]::UTF8.WebName                  # utf-8
[System.Text.Encoding]::GetEncoding(932).WebName      # shift_jis
[System.Text.Encoding]::GetEncoding("shift_jis")      # 名前を指定して取得できる
```

これは、HTTP ヘッダーの `Content-Type: text/html; charset=utf-8`、HTML の `<meta charset="utf-8">`、XML 宣言の `encoding="utf-8"` などで使われる名前です。つまり **アプリケーションの「外」の世界とやり取りするための、公式な氏名**にあたります。

CodePage を JAN コードに例えるなら、WebName は **商品の正式名称**です。誰に対しても通用する、標準化された呼び名です。

| CodePage | WebName |
|---|---|
| 65001 | `utf-8` |
| 1200 | `utf-16` |
| 1201 | `unicodeFFFE` ← 要注意 |
| 12000 | `utf-32` |
| 12001 | `utf-32BE` |
| 20127 | `us-ascii` |
| 932 | `shift_jis` |
| 51932 | `euc-jp` |
| 50220 | `iso-2022-jp` |
| 1252 | `windows-1252` |

`Encoding.GetEncoding(string)` は、WebName そのものだけでなく **別名（alias）も受け付けます**。たとえば UTF-8 は `utf-8` でも `utf8` でも `unicode-1-1-utf-8` でも取得できます。

ここで一つ、有名な罠を紹介しておきます。**UTF-16 ビッグエンディアンの WebName は `unicodeFFFE`** です。IANA の標準名である `utf-16be` ではありません。Microsoft 独自の命名が WebName として居座っているという、なかなか厄介な例外です（`utf-16be` は別名として受け付けられます）。

そして WebName にも、CodePage と同じ限界があります。**BOM の有無を表現できません。**

### 紛らわしい兄弟プロパティ

`Encoding` クラスには、名前を返すプロパティが他にもあります。ここが混乱の温床になります。

| プロパティ | CP932 での値 | 用途 |
|---|---|---|
| `WebName` | `shift_jis` | IANA 名。HTTP / HTML / XML 用 |
| `BodyName` | `iso-2022-jp` | メール本文用の名前 |
| `HeaderName` | `iso-2022-jp` | メールヘッダー用の名前 |
| `EncodingName` | `日本語 (シフト JIS)` | **人間向けの説明文。ロケール依存** |

とくに注意していただきたいのが `EncodingName` です。これは「人が読むための表示名」であり、日本語環境では日本語で返ってきます。英語環境で実行すれば `Japanese (Shift-JIS)` になります。

先ほど述べたとおり、これを指して「フレンドリ名」と呼ぶ人もいます。しかし **PowerShell の `-Encoding` に指定する名前とはまったく無関係**です。当然ですが、次のコードは失敗します。

```powershell
[System.Text.Encoding]::GetEncoding("日本語 (シフト JIS)")   # 失敗します
```

`EncodingName` は「表示専用」と覚えておけば十分です。

一覧で確認したい場合は、次のコマンドが便利です。

```powershell
[System.Text.Encoding]::GetEncodings() |
    Select-Object CodePage, Name, DisplayName |
    Sort-Object CodePage

# Name        は WebName に相当します
# DisplayName は EncodingName に相当します
```

---

## フレンドリ名 —— PowerShell の -Encoding が受け付ける名前

さて、本題です。

`Out-File`、`Set-Content`、`Get-Content`、`Export-Csv` といった cmdlet が持つ `-Encoding` パラメーターに指定する、`utf8` や `unicode` といった短いキーワード。これが本記事でいう **フレンドリ名**です。

```powershell
Set-Content -Path a.txt -Value 'あ' -Encoding utf8BOM
```

ここで決定的に重要なことを申し上げます。

> **フレンドリ名は「文字集合の識別子」ではありません。「書き出し方の選択肢」です。**

`utf8BOM` と `utf8NoBOM` を思い出してください。この 2 つは、CodePage も WebName も完全に同一です。

| フレンドリ名 | CodePage | WebName | BOM |
|---|---|---|---|
| `utf8BOM` | 65001 | `utf-8` | **あり** |
| `utf8NoBOM` | 65001 | `utf-8` | **なし** |

違うのは **BOM（プリアンブル）を書き出すかどうか**だけです。そして CodePage も WebName も BOM の有無を表現できない以上、**この区別はフレンドリ名という層でしか表現できない**のです。

コーヒーに例えるなら、こういうことです。CodePage と WebName は「豆の種類」を指す名前です。同じ豆を使っていれば、名前は同じになります。しかしフレンドリ名は「ホットで」「アイスで」まで含んだ**注文の仕方**を指しています。豆が同じでも、出てくるものは違います。

実際に確認してみましょう。

```powershell
$bom   = New-Object System.Text.UTF8Encoding $true   # BOM あり
$noBom = New-Object System.Text.UTF8Encoding $false  # BOM なし

$bom.CodePage -eq $noBom.CodePage      # True  同じコードページ
$bom.WebName  -eq $noBom.WebName       # True  同じ WebName

$bom.GetPreamble().Length              # 3     BOM の 3 バイト
$noBom.GetPreamble().Length            # 0     何も出力しない
```

CodePage も WebName も一致しているのに、`GetPreamble()` の結果だけが違います。BOM は**エンコーディングの「同一性」の外側にある情報**なのです。

実ファイルでも見てみましょう。

```powershell
Set-Content -Path .\bom.txt   -Value 'あ' -Encoding utf8BOM
Set-Content -Path .\nobom.txt -Value 'あ' -Encoding utf8NoBOM

Format-Hex .\bom.txt     # EF BB BF E3 81 82 ...
Format-Hex .\nobom.txt   # E3 81 82 ...
```

先頭 3 バイトの `EF BB BF` が BOM です。この 3 バイトを付けるか付けないかを指示できるのは、フレンドリ名だけです。

### PowerShell 5.1 —— 12 個の列挙値しかない世界

Windows PowerShell 5.1 の `-Encoding` は、**あらかじめ決められた値しか受け付けません**。

| フレンドリ名 | 内容 |
|---|---|
| `Ascii` | ASCII（7 ビット） |
| `BigEndianUnicode` | UTF-16 ビッグエンディアン |
| `BigEndianUTF32` | UTF-32 ビッグエンディアン |
| `Byte` | バイト列として扱う |
| `Default` | **システムの ANSI コードページ**（日本語 Windows では CP932） |
| `Oem` | システムの OEM コードページ |
| `String` | `Unicode` と同じ |
| `Unicode` | UTF-16 リトルエンディアン |
| `Unknown` | `Unicode` と同じ |
| `UTF7` | UTF-7 |
| `UTF32` | UTF-32 リトルエンディアン |
| `UTF8` | **UTF-8（BOM 付き）** |

この 12 個がすべてです。ここに `932` も `shift_jis` もありません。試してみましょう。

```
PS 5.1> Get-Content -Encoding 932 text1.txt
Get-Content : パラメーター 'Encoding' をバインドできません。列挙値が無効なため、値 "932" を型 "Microsoft.PowerShell.Commands.FileSys
temCmdletProviderEncoding" に変換できません。次のいずれかの列挙値を指定し、再試行してください。有効な列挙値: "Unknown,String,Unicode
,Byte,BigEndianUnicode,UTF8,UTF7,UTF32,Ascii,Default,Oem,BigEndianUTF32"。
発生場所 行:1 文字:23
+ Get-Content -Encoding 932 text1.txt
+                       ~~~
    + CategoryInfo          : InvalidArgument: (:) [Get-Content]、ParameterBindingException
    + FullyQualifiedErrorId : CannotConvertArgumentNoMessage,Microsoft.PowerShell.Commands.GetContentCommand
```

このエラーメッセージには、PowerShell 5.1 の設計思想がそのまま書かれています。注目していただきたいのは**型名**です。

```
型 "Microsoft.PowerShell.Commands.FileSystemCmdletProviderEncoding" に変換できません
```

つまり、**PowerShell 5.1 の `-Encoding` の型は `System.Text.Encoding` ではなく、`FileSystemCmdletProviderEncoding` という PowerShell 専用の列挙型（enum）**なのです。エラーメッセージも「列挙値が無効なため」と明言しています。

enum である以上、定義された 12 個の値しか受け取れません。CodePage も WebName も、そもそも入り込む余地がありませんでした。**PowerShell 5.1 において、フレンドリ名の世界と .NET の名前の世界は完全に断絶していた**のです。

定食屋に例えるなら、5.1 は「メニューは 12 品。それ以外は作りません」という店でした。

このことから、5.1 には次のような不便が生じます。

1. **CP932 で書きたければ `Default` を使うしかない。** しかも `Default` の実体は「システムの ANSI コードページ」なので、日本語 Windows で動かす限りにおいて CP932 になる、という環境依存の回り道です。
2. **BOM なし UTF-8 を出力する手段がない。** `UTF8` は BOM 付きであり、BOM なしを指定するフレンドリ名が存在しません。
3. **`Default` は「既定」ではない。** 名前に反して、これは ANSI コードページのことです。OS のロケールで中身が変わります。

### PowerShell 7.x —— Encoding 型と引数変換

PowerShell 7.x（および 6.x）のフレンドリ名は次のとおりです。

| フレンドリ名 | 内容 |
|---|---|
| `ascii` | ASCII（7 ビット） |
| `ansi` | 現在のカルチャの ANSI コードページ（**PowerShell 7.4 で追加**） |
| `bigendianunicode` | UTF-16 ビッグエンディアン |
| `bigendianutf32` | UTF-32 ビッグエンディアン |
| `oem` | MS-DOS / コンソールの既定エンコーディング |
| `unicode` | UTF-16 リトルエンディアン |
| `utf7` | UTF-7（非推奨） |
| `utf8` | **UTF-8（BOM なし）** |
| `utf8BOM` | UTF-8（BOM 付き） |
| `utf8NoBOM` | UTF-8（BOM なし） |
| `utf32` | UTF-32 リトルエンディアン |

5.1 との違いのうち、フレンドリ名にかかわるものを挙げます。

**`utf8` の意味が変わりました。** 5.1 では BOM 付き、6.0 以降は BOM なしです。同じスクリプトを書いているつもりでも、実行するバージョンによって出力バイト列が変わります。移植時に最も事故が起きやすい箇所です。

これを避けるには、**`utf8` とだけ書かず、`utf8BOM` か `utf8NoBOM` を明示する**のが唯一確実な方法です。本記事のタイトルにもなっている `utf8NoBOM` が重要なのは、まさにこの「意図を明示できる」という点にあります。

**`Default` / `String` / `Unknown` / `Byte` が消えました。** `Byte` の代替は `-AsByteStream` パラメーターです。`Default` の代替は、7.4 以降であれば `ansi` が使えます。

**`utf7` は非推奨になりました。** UTF-7 は Unicode の標準エンコーディングではなく、使用は避けるべきとされています。

そして、構造そのものが変わりました。**7.x の `-Encoding` の型は `System.Text.Encoding` であり、引数変換の仕組みが文字列や数値を `Encoding` オブジェクトへ変換します。** 5.1 が enum だったのに対し、7.x は「型変換」なのです。

この違いが、次に述べる大きな変化を生みました。

### PowerShell 6.2 の変更 —— 2 つの世界が接続された

PowerShell 6.2 以降、`-Encoding` は **登録済みコードページの数値 ID と文字列名も受け付ける**ようになりました。

```powershell
Set-Content -Path a.txt -Value 'あ' -Encoding 932           # コードページ番号
Set-Content -Path a.txt -Value 'あ' -Encoding shift_jis     # 登録済みコードページ名
```

これは単なる利便性の向上ではありません。**フレンドリ名の世界と、CodePage / WebName の世界が接続された**ということです。

先ほどの定食屋の例えを続けるなら、7.x は「メニュー 12 品のほかに、食材を持ち込んでいただければ調理します」という店に変わりました。

### 決定的な実例 —— 一覧にない名前が通る

このことを最もよく示す実例があります。PowerShell 7.6 で試してみましょう。

```powershell
# UTF-16BE のファイルを読む
Get-Content -Encoding unicodeFFFE text1.txt   # 読めます
Get-Content -Encoding utf-16be    text1.txt   # 読めます
```

ここで先ほどのフレンドリ名一覧を見返してください。**`unicodeFFFE` も `utf-16be` も、どこにも載っていません。**

それでも通るのはなぜか。これらは CP1201（UTF-16BE）の **WebName とその別名**だからです。6.2 以降の `-Encoding` は登録済みコードページ名を受け付けるので、**フレンドリ名ではなく WebName 経由で解決されている**のです。

UTF-32BE も同様です。

```powershell
Get-Content -Encoding utf-32be text1.txt      # CP12001 / WebName = utf-32BE
```

そして当然ながら、**これらは PowerShell 5.1 では 1 つも通りません。** 5.1 の `-Encoding` は enum なのですから。

なお、`Encoding.GetEncoding(932)` のような呼び出しは、素の .NET では `CodePagesEncodingProvider` を明示的に登録しないと失敗します。しかし PowerShell では、次のようにモジュールを一切ロードしない新規プロセスでも通ります。

```powershell
pwsh -NoProfile -Command "[System.Text.Encoding]::GetEncoding(932).WebName"
# shift_jis
```

**PowerShell 自身が起動時にプロバイダーを登録してくれている**ということです。ありがたい親切ですが、.NET アプリを自分で書くときには自分で登録が必要になります（この話は .NET 側の記事で改めて扱います）。

---

## 三者の対応表と、WebName とフレンドリ名の違い

ここまでの内容を、一枚の表にまとめます。

| フレンドリ名（PS 7.x） | CodePage | WebName | BOM |
|---|---|---|---|
| `ascii` | 20127 | `us-ascii` | なし |
| `utf8` / `utf8NoBOM` | 65001 | `utf-8` | **なし** |
| `utf8BOM` | 65001 | `utf-8` | **あり** |
| `unicode` | 1200 | `utf-16` | あり |
| `bigendianunicode` | 1201 | `unicodeFFFE` | あり |
| `utf32` | 12000 | `utf-32` | あり |
| `bigendianutf32` | 12001 | `utf-32BE` | あり |
| `ansi`（日本語 Windows） | 932 | `shift_jis` | なし |
| `oem`（日本語 Windows） | 932 | `shift_jis` | なし |
| **（フレンドリ名なし）** | 51932 | `euc-jp` | なし |
| **（フレンドリ名なし）** | 50220 | `iso-2022-jp` | なし |

この表を眺めると、2 つのことに気づきます。

**ひとつめ。** `utf8NoBOM` と `utf8BOM` は、CodePage も WebName も同じです。つまり **フレンドリ名にしか表せない情報がある**ということです。

**ふたつめ。** EUC-JP や ISO-2022-JP には、対応するフレンドリ名が存在しません。つまり **CodePage / WebName にしか表せない範囲がある**ということです。

したがって、次のように結論できます。

> **WebName とフレンドリ名は、包含関係にはありません。互いに欠けを補い合う、別軸の名前空間です。**

- **WebName は「文字集合の名前」です。** どの文字がどのバイト列に対応するか、という規約そのものを指しています。だからこそ HTTP や HTML で通用します。しかし、ファイルの先頭に BOM を置くかどうかは、文字集合の規約の外側の話です。表現できるはずがありません。
- **フレンドリ名は「書き出し方の名前」です。** PowerShell がファイルを読み書きするときの、実際の振る舞いを選ぶキーワードです。だから BOM の有無を含められます。その代わり、PowerShell が用意した分しか存在しません。

本記事のタイトルの問い——**`-Encoding utf8NoBOM` は、なぜ WebName で代用できないのか**——への答えは、これです。

**BOM の有無は、文字集合の属性ではないからです。** `utf-8` という WebName は、UTF-8 という符号化方式を指し示す名前であって、「その符号化方式で書き出すときにファイル先頭へ 3 バイトを置くかどうか」という運用の選択までは含んでいません。だから WebName では代用できないのです。

そして PowerShell 6.2 以降、`-Encoding shift_jis` と WebName で書けるようになった今でも `utf8NoBOM` というフレンドリ名が残り続けているのは、この「WebName では表現できない一点」を担っているからにほかなりません。

---

## コラム —— 「読めている」は「正しく読めている」ではない

検証中に面白い現象に出会ったので、寄り道を一つ。

UTF-32BE で保存されたファイルを、`-Encoding unicodeFFFE`（UTF-16BE）で読んでみます。エンコーディングを間違えているのですから、当然文字化けするはずです。

```
PS> Get-Content -Encoding unicodeFFFE text1.txt
TESTストリングaaa日本語国債発行

TESTストリングbbb日本語

TESTストリングccc日本語

TESTストリング
```

読めてしまいました。改行の位置が少しおかしい気はしますが、文字は正しく見えます。

しかしこれは錯覚です。

UTF-32BE では、`T` は `00 00 00 54` という 4 バイトで表されます。これを UTF-16BE として 2 バイトずつ読むと、`U+0000`（NUL）と `U+0054`（T）という **2 文字**になります。そして NUL 文字はコンソールに表示されません。だから目には `T` しか見えないのです。

**4 バイトずつ梱包された荷物を、2 バイトずつ開封している**と考えてください。空箱と中身が交互に出てきます。空箱は目に見えないので、中身だけが並んでいるように錯覚するわけです。

確かめてみましょう。

```powershell
$s = Get-Content -Encoding unicodeFFFE -Raw text1.txt
$s.ToCharArray()[0..7] | ForEach-Object { [int]$_ }
# 0, 84, 0, 83, ...   ← NUL が挟まっている
```

改行が乱れて見えたのも同じ理由です。`CR LF` が `NUL CR NUL LF` になっていたのです。

このデータをそのまま処理に流したら、何が起きるでしょうか。文字列の長さは 2 倍になり、比較は一致せず、正規表現は空振りします。しかも画面で見る限り、何もおかしくないように見えるのです。

**目視で読めることは、正しく復号できていることを意味しません。** エンコーディングを正しい名前で指定することが、なぜこれほど大切なのか。その答えがここにあります。

---

## 落とし穴のまとめ

本記事で触れた注意点を整理しておきます。

**1. 「フレンドリ名」は `EncodingName` ではありません。**
`Encoding.EncodingName` は日本語 Windows では `日本語 (シフト JIS)` を返す、表示専用の文字列です。`-Encoding` に指定する名前とは無関係です。

**2. `utf8` の意味が 5.1 と 7.x で違います。**
5.1 では BOM 付き、6.0 以降は BOM なし。`utf8BOM` / `utf8NoBOM` を明示してください。

**3. `Default` は「既定」ではありません。**
実体は ANSI コードページで、環境依存です。7.x では廃止されました。代替は 7.4 以降の `ansi` です。

**4. UTF-16BE の WebName は `unicodeFFFE` です。**
IANA 標準名の `utf-16be` は別名としてのみ受け付けられます。

**5. `-Encoding shift_jis` は 5.1 では動きません。**
6.2 以降でのみ有効です。5.1 対応が必要なスクリプトでは使えません。

**6. `oem` と `ansi` が同じになるのは、日本語 Windows だからです。**
日本語 Windows 11 では両方とも CP932 になりますが、これは一般則ではありません。英語圏の Windows では `ansi` が 1252、`oem` が 437 と分かれます。また Windows の「ベータ: ワールドワイド言語サポートで Unicode UTF-8 を使用」を有効にすると、ANSI が 65001 になります。「oem と ansi は同じもの」と覚えないでください。

---

## 実務での使い分け

最後に、どの名前をどこで使うべきかをまとめます。

| 目的 | 使うべき名前 |
|---|---|
| PowerShell でのファイル出力 | **フレンドリ名**（`utf8BOM` / `utf8NoBOM` を明示） |
| HTTP / HTML / XML への埋め込み | **WebName** |
| 設定ファイルのキーや判定ロジック | **CodePage**（整数で安定している） |
| 画面表示・ログ出力 | **EncodingName**（表示専用） |

そして、最も重要な原則を一つだけ挙げるとすれば、これです。

> **`-Encoding utf8` とだけ書かないこと。`utf8BOM` か `utf8NoBOM` を明示すること。**

このひと手間が、PowerShell のバージョン差による事故を防いでくれます。

---

## まとめ

- **CodePage** は数値による識別子です。機械向けで、曖昧さがありません。
- **WebName** は IANA charset 名です。外の世界と通じる、文字集合の公式な名前です。
- **フレンドリ名** は PowerShell の `-Encoding` が受け付けるキーワードです。.NET の用語ではありません。
- PowerShell 5.1 の `-Encoding` は **enum** であり、12 個の値しか受け付けませんでした。
- PowerShell 6.2 以降の `-Encoding` は **`System.Text.Encoding` 型 + 引数変換**であり、CodePage や WebName も受け付けます。
- それでもなお `utf8BOM` / `utf8NoBOM` というフレンドリ名が必要なのは、**BOM の有無は文字集合の属性ではなく、WebName でも CodePage でも表現できない**からです。

WebName は「文字集合の名前」、フレンドリ名は「書き出し方の名前」。この一点を押さえておけば、迷うことはないはずです。

---

