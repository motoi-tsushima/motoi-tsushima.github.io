---
title: "SnowStack.EncodingProbe.PowerShell を Windows PowerShell 5.1 へインストールする方法"
layout: single
classes: wide
permalink: /encodingprobe_powershell_install_ps51/
author_profile: true
---
2026/08/15 document

この記事は「SnowStack.EncodingProbe.PowerShell 解説」の補足説明です。

[SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)

本記事では、Windows PowerShell 5.1 の環境へ SnowStack.EncodingProbe.PowerShell をインストールする手順を解説します。

PowerShell 7.x をお使いの方は、この記事を読む必要はありません。本記事の手順は、Windows PowerShell 5.1 だけに必要な前作業です。

なお、PowerShell のパッケージ配布の仕組み（PowerShellGet・PSResourceGet・NuGet の関係）については、この記事では扱いません。インストールに必要な最小限の説明だけを行います。仕組みの解説は、別の記事で行いたいと思っています。

（この記事は、2026年8月15日時点の動作を元に書いています。PowerShell ギャラリー側の仕様は変更されることがありますので、その点は配慮して読んでください）

## Windows PowerShell 5.1 では Install-PSResource がそのまま使えない

「SnowStack.EncodingProbe.PowerShell 解説」では、以下のコマンドでインストールできると説明しました。

```
# PowerShell 7.x でのインストール
Install-PSResource SnowStack.EncodingProbe.PowerShell
```

しかし、このコマンドを Windows PowerShell 5.1 でそのまま実行しても、コマンドが見つからずエラーになります。

`Install-PSResource` は、`Microsoft.PowerShell.PSResourceGet` というモジュールが提供しているコマンドレットです。

このモジュールは PowerShell 7.4 以降には最初から同梱されていますが、Windows PowerShell 5.1 には同梱されていません。

| 項目 | Windows PowerShell 5.1 | PowerShell 7.4 以降 |
| :---- | :---- | :---- |
| 同梱されるパッケージ管理モジュール | PowerShellGet 1.0.0.1 | PowerShellGet ／ PSResourceGet |
| `Install-Module` が使えるか | 使えます | 使えます |
| `Install-PSResource` が使えるか | そのままでは使えません | 使えます |

表の通り、Windows PowerShell 5.1 に入っている `PowerShellGet` は 1.0.0.1 という古いバージョンで、`Install-PSResource` を含んでいません。

そのため、Windows PowerShell 5.1 では、先に `Microsoft.PowerShell.PSResourceGet` を手作業でインストールする必要があるのです。

## インストールの全体像

作業は大きく分けて2段階です。

1. `Install-PSResource` を使えるようにする（前作業。この記事の主題です）
2. `Install-PSResource` で SnowStack.EncodingProbe.PowerShell をインストールする

前作業は一度やってしまえば済みます。二度目以降は、PowerShell 7.x と同じ1行のコマンドでインストールできます。

## 前作業の手順

以下、手順を順番に説明します。

### 1. 管理者権限で Windows PowerShell 5.1 を起動する

スタートメニューから「Windows PowerShell」を右クリックし、「管理者として実行」を選んで起動してください。

これから実行する `Install-Module` は、標準では全ユーザー用のフォルダーへモジュールを書き込むため、管理者権限が必要になります。

### 2. スクリプトの実行を許可する

Windows PowerShell 5.1 の実行ポリシーが厳しいままだと、モジュールの取得に失敗することがあります。

以下のコマンドで、現在のユーザーの実行ポリシーを `RemoteSigned` に変更します。

```
# 現在のユーザーだけ、外部から取得したスクリプトの実行を許可する
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
```

`RemoteSigned` は、ローカルで作成したスクリプトはそのまま実行でき、インターネットから取得したスクリプトは署名を要求する設定です。

既にこの設定になっている場合は、このコマンドを実行する必要はありません。現在の値は `Get-ExecutionPolicy -List` で確認できます。

### 3. NuGet プロバイダーと PowerShellGet を更新する

Windows PowerShell 5.1 に同梱されている `PowerShellGet` 1.0.0.1 は、現在の PowerShell ギャラリーから `PSResourceGet` を直接取得できません。

そこで、土台となるパッケージ管理の機能を先に更新します。

```
# パッケージ取得に使用する NuGet プロバイダーを導入する
Install-PackageProvider -Name NuGet -Force

# PowerShellGet を新しいバージョンへ更新する
Install-Module -Name PowerShellGet -Force -AllowClobber
```

`-AllowClobber` は、既存のコマンド名を新しいモジュール側で上書きすることを許可するオプションです。`PowerShellGet` の更新では、同名のコマンドレットが置き換わるため、このオプションが必要になります。

途中で「信頼されていないリポジトリ」の確認メッセージが出た場合は、`[Y]` または `[A]` を入力して進めてください。

### 4. PowerShell を開き直す

ここが、忘れやすい重要な手順です。

**更新した `PowerShellGet` は、今開いている PowerShell では有効になりません。**

一度 PowerShell のウィンドウを閉じ、改めて管理者権限で Windows PowerShell 5.1 を起動し直してください。

同じセッションのまま次の手順へ進むと、更新前の古いモジュールが読み込まれたままになり、インストールに失敗することがあります。この点は、間違えないように注意してください。

### 5. Microsoft.PowerShell.PSResourceGet をインストールする

開き直した PowerShell で、`Install-PSResource` を提供するモジュールをインストールします。

```
# Install-PSResource コマンドレットを含むモジュールを導入する
Install-Module -Name Microsoft.PowerShell.PSResourceGet -Repository PSGallery -Force
```

`-Repository PSGallery` は、取得元を PowerShell ギャラリーに指定するオプションです。

### 6. Install-PSResource が使えることを確認する

インストールが終わったら、コマンドレットが認識されているか確認します。

```
# Install-PSResource が使えるか確認する
Get-Command Install-PSResource

# 実行結果（例）
CommandType     Name                  Version    Source
-----------     ----                  -------    ------
Cmdlet          Install-PSResource    1.1.1      Microsoft.PowerShell.PSResourceGet
```

このように `Install-PSResource` が表示されれば、前作業は完了です。

`Version` の値は、インストールした時期によって変わります。表示されていること自体が確認できれば問題ありません。

コマンドが見つからないというエラーが出る場合は、手順4の「PowerShell を開き直す」を行っていない可能性が高いと思われます。

## SnowStack.EncodingProbe.PowerShell のインストール

ここから先は、PowerShell 7.x と同じ手順です。

```
# SnowStack.EncodingProbe.PowerShell を導入する
Install-PSResource SnowStack.EncodingProbe.PowerShell
```

初めて PowerShell ギャラリーを利用する環境では、以下の確認メッセージが表示されます。

```
Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its Trusted value by running the
Set-PSResourceRepository cmdlet. Are you sure you want to install the PSResource from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"):
```

信頼済みとして登録されていないリポジトリからの取得なので、警告が出ます。

インストールするには、ここで `[Y]` か `[A]` を入力して、信頼していただく必要があります。

`[Y]` か `[A]` を入力して `[Enter]` キーを押すと、SnowStack.EncodingProbe.PowerShell がインストールされます。

インストール後、以下のコマンドで動作を確認できます。

```
# コマンドレットが読み込めているか確認する
Get-Command Resolve-Encoding

# バージョンを表示して動作を確認する
Resolve-Encoding -Version
```

### アンインストール方法

削除する場合は、以下のコマンドを使用してください。

```
# SnowStack.EncodingProbe.PowerShell を削除する
Uninstall-PSResource SnowStack.EncodingProbe.PowerShell
```

## インストール先とターゲットフレームワークについて

Windows PowerShell 5.1 で `Install-PSResource` を実行すると、`net472`（.NET Framework 4.7.2）向けの構成が選択され、モジュールは従来の Windows PowerShell 用フォルダーへ配置されます。

| 実行環境 | モジュールの配置先（全ユーザー向けの場合） |
| :---- | :---- |
| Windows PowerShell 5.1 | `C:\Program Files\WindowsPowerShell\Modules` |
| PowerShell 7.x | `C:\Program Files\PowerShell\Modules` |

表の通り、Windows PowerShell 5.1 と PowerShell 7.x では、モジュールの置き場所そのものが別になっています。

よって、片方にインストールしても、もう片方では使用できません。両方で使いたい場合は、それぞれの環境でインストールしてください。

なお、自分専用にインストールしたい場合は `-Scope CurrentUser` を、全ユーザー向けにインストールしたい場合は `-Scope AllUsers` を指定できます。`-Scope AllUsers` を指定する場合は管理者権限が必要です。

## うまくいかない場合に確認すること

ここまでの手順で失敗する場合、原因は SnowStack.EncodingProbe.PowerShell 側ではなく、PowerShell ギャラリーへの接続にあることがほとんどです。

以下の3点を確認してください。

**通信プロトコルの設定**

古い Windows 環境では、既定の通信プロトコルが PowerShell ギャラリー側の要求を満たさず、接続に失敗する場合があります。その場合は、以下のコマンドを実行してから、もう一度インストールを試してください。

```
# このセッションの間だけ TLS 1.2 で通信する
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```

この設定は、PowerShell を閉じると元に戻ります。恒久的に設定を変える方法もありますが、環境全体に影響するため、ここでは扱いません。

**NuGet プロバイダーのバージョン**

手順3の `Install-PackageProvider` が正しく完了しているかを確認します。

```
# 導入済みのパッケージプロバイダーを一覧表示する
Get-PackageProvider
```

**社内プロキシ・制限された回線**

企業のネットワークでは、プロキシや通信制限によって PowerShell ギャラリーへ到達できないことがあります。この場合は、ネットワーク管理者に確認していただくしかありません。

整理すると、失敗の原因は「実行ポリシー」「古いパッケージ管理モジュール」「ギャラリーへの通信」の3つに、ほぼ集約されます。エラーメッセージを読んで、どれに当たるのかを見分けてください。

## Windows PowerShell 5.1 で使用するときの注意点

インストールできても、Windows PowerShell 5.1 と PowerShell 7.x では、`Resolve-Encoding` の返す `PSEncodingName` の値が異なります。

Windows PowerShell 5.1 では BOM の無い UTF-8 を扱えず、Shift_JIS のフレンドリ名も用意されていないためです。

この違いは、SnowStack.EncodingProbe.PowerShell の都合ではなく、PowerShell 側の仕様の違いによるものです。

詳しくは「SnowStack.EncodingProbe.PowerShell 解説」の「PSEncodingName のフレンドリ名について」と、以下の記事をご覧ください。

[Windows PowerShell 5.1 と PowerShell 6.2 以降の違い](https://snow-stack.net/windows-powershell-51-vs-pwsh-62/)

[EncodingProbe から見た .NET / PowerShell の文字コード環境差](https://snow-stack.net/encoding-across-dotnet-powershell/)

## 最後に

以上、Windows PowerShell 5.1 へ SnowStack.EncodingProbe.PowerShell をインストールする方法の解説でした。

手順が多く見えますが、前作業が必要なのは最初の一度だけです。二度目以降は、PowerShell 7.x と同じ1行のコマンドでインストールできます。

`PowerShellGet` と `PSResourceGet` の違いや、PowerShell のモジュール配布の仕組みについては、別の記事で改めて解説したいと思っています。

## 関連資料

- [SnowStack.EncodingProbe.PowerShell 解説](/encodingprobe_powershell_guide/)
- [Windows PowerShell 5.1 と PowerShell 6.2 以降の違い](https://snow-stack.net/windows-powershell-51-vs-pwsh-62/)
- [EncodingProbe から見た .NET / PowerShell の文字コード環境差](https://snow-stack.net/encoding-across-dotnet-powershell/)
- [PowerShell ギャラリーの概要](https://learn.microsoft.com/ja-jp/powershell/gallery/getting-started?view=powershellget-3.x)
- [Microsoft.PowerShell.PSResourceGet ドキュメント](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.psresourceget/?view=powershellget-3.x)
