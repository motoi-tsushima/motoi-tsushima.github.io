---
title: "mfsr-mfprobe インストール解説"
layout: single
permalink: /mfsr_install_guide/
author_profile: true
---

mfsr と mfprobe は、rmsmf と txprobe とは異なるインストール手段を提供しています。

## インストールの前提の解説

インストール方法を解説する前に、mfsr と mfprobe のリリース方針を理解してもらう必要があります。

mfsr-mfprobe は、Windows 環境に対しては .NETランタイムを必要としないネイティブコードでのリリースを行っています。

これに対し、macOS と Linux OS に対しては、.NETランタイムを必要とする.NETマネージドコードでのリリースを行っています。

.NETマネージドコードは、.NETランタイムがあれば、どこでも稼働するので、Windowsでもマネージドコードが動作します。

つまり、Windowsに対しては、ネイティブコードとマネージドコードの二つのリリースモジュールが存在することになります。

どちらを使用するかは、ユーザーの自由ですが、私としては、Windowsではネイティブコードの使用を推奨します。

## 各種インストール方法の解説

### winget によるインストール（Windows のみ）

先の説明にあるように、Windowsに対しては Native AOT でビルドしたネイティブコードを提供しています。

ネイティブコードの mfsr-mfprobe 実行ファイルは、Windows 標準のパッケージマネージャである winget コマンドで、簡単にインストールできます。
（winget は、プレインストールされています：[WinGet を使用したアプリケーションのインストールと管理](https://learn.microsoft.com/ja-jp/windows/package-manager/winget/#install-winget)）

インストールは以下のコマンド入力でできます。

```

winget install motoi.tsushima.mfprobe

winget install motoi.tsushima.mfsr

```

winget によるインストールは、mfprobe と mfsr がそれぞれ別々に行います。

### dotnet tool install によるインストール

mfsr-mfprobe は、.NET の C# で開発しており、.NETアセンブリ（マネージドコード）でのリリースも行っています。

.NETアセンブリは、.NET Runtime 上で稼働する実行ファイルです。.NET Runtime が OS の違いを吸収してくれるので、Windows , macOS , Linux OS のいずれの環境でも動作します。

故に .NETアセンブリは実行する前に、.NET Runtime をインストールしておく必要があります。

.NET Runtime は以下のMicrosoft公式サイトからダウンロードしインストールできます。

[**.NET 10.0 のダウンロード**](https://dotnet.microsoft.com/ja-jp/download/dotnet/10.0)

インストールするのは、Runtime でも SDK でもどちらでも良いです。

Runtime は.NETアセンブリを使用する為に必要最小限のモジュールです。

SDK は Runtime 含み、.NETアプリの開発者が必要とする機能全般を提供するモジュールです。

ここからダウンロードした Runtime をインストールしても良いですが、他にも Runtime のインストール方法があるので、ＯＳごとに解説します。

#### macOS への .NET Runtime のインストール方法

```
brew install dotnet-sdk
```

#### Linux OS への .NET Runtime のインストール方法

```
# Ubuntu/Debian
sudo apt-get install dotnet-runtime-10.0
```

#### Windows への .NET Runtime のインストール方法

```
winget install Microsoft.DotNet.SDK.10
```

#### mfsr-mfprobe アセンブリのインストール

.NET Runtime をインストールし終えたら、dotnet コマンドで mfsr と mfprobe の.NETアセンブリをインストールできます。

```

dotnet tool install -g mfprobe

dotnet tool install -g mfsr

```

インストールは、mfprobe と mfsr それぞれ別々に行います。

`-g` オプションを忘れずに付けてください。これが無いとコマンド名だけでコマンドを起動できなくなります。

## 各種アンインストール方法の解説

mfsr と mfprobe のアンインストール方法は、インストール方法と同じようにできます。

インストールしたコマンドと同じコマンドで、アンインストールします。

### winget によるアンインストール

winget でインストールした場合は、winget でアンインストールします。

```

winget uninstall mfprobe

winget uninstall mfsr

```

### dotnet tool によるアンインストール

dotnet tool でインストールした場合は、dotnet tool でアンインストールします。

Windows , macOS , Linux OS の、どの OS でも操作は同じです。

```

dotnet tool uninstall -g mfprobe

dotnet tool uninstall -g mfsr

```

## 手動でダウンロードしてインストールする方法

以前解説していた手動のインストール方法も、一応解説しておきます。

ただ、既に winget と dotnet tool （NuGet.org）による自動インストールを提供しているので、従来の手動インストールはお勧めしません。

以下の Download から mfsr.win.zip をダウンロードして解凍し、展開された .exe ファイルを 環境変数 Path の通ったフォルダーにコピーすれば、コンソールアプリとして使用できます。

**Download** [https://github.com/motoi-tsushima/mfsr/releases/tag/v1.0.2.6](https://github.com/motoi-tsushima/mfsr/releases/tag/v1.0.2.6)

リポジトリは以下の GitHub リポジトリで公開しています。

**Repository** [https://github.com/motoi-tsushima/mfsr](https://github.com/motoi-tsushima/mfsr)

git clone コマンドで、ソースコードごとダウンロードして、ローカルPCでビルドして手動インストールするのも、自己責任で可能です。

```
git clone https://github.com/motoi-tsushima/mfsr
```

## 補足

現在、インストール方法によって、コマンドのバージョンが違いますが、どれもコマンドの機能や不具合対応の内容に違いはありません。

リリースの都合で、バージョン番号がバラバラになっているだけです。

そのうち、バージョン番号を揃えます。

