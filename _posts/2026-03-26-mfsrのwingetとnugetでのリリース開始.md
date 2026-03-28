---
title: "mfsr-mfprobe の winget と nuget.org でのリリース開始のお知らせ"
categories:
  - お知らせ
tags:
  - Character Encoding
  - 改行コード
  - BOM
  - mfprobe
  - mfsr
  - Windows ツール
  - macOS ツール
  - Linux OS ツール
---

これまで、mfsr と mfprobe は、実行ファイルを .zip で配布して、ユーザーさんに自分でPathの通ったフォルダーに配置してもらっていました。

## winget でのインストールが可能になりました

先日、Windows 版は winget によるリリースを開始し、Windows標準の winget install コマンドで、簡単に mfsr と mfprobe をインストールできるようになりました。

rmsmf と txprobe は、これまでと同じで .zip ファイルでのインストールする方式のままです。

私としては、.NET Framework 4.8 でビルドしている rmsmf と txprobe よりも、.NET10 でビルドしている mfsr と mfprobe を極力使用して頂きたいので、このようにしました。

 ## macOS と Linux OS へ対応しました

元々、.NET10 で開発していますから、macOS と Linux OS でもリリースすれば動作するように開発しています。

しかし、これまでは Mac と Linux へのリリース手段を提供していませんでした。

リリースに時間がかかった理由として、macOS と Linux OS はCPUの種類やカーネルのバージョンなどの種類が多く、各種の環境に合わせたネイティブコードでのリリースが困難になるため、どのようなリリース方法が良いか判断に迷ったからです。

Windows版は、Native AOT によりネイティブコードでリリースしています。
winget でインストールする mfsr と mfprobe はネイティブコード版です。

今回、macOS と Linux OS に対しては、.NET10 のマネージドコードのアセンブリで提供することにしました。

マネージドコードの実行には .NET10 の Runtime（ランタイム） が必要になります。

しかし、.NET10 Runtime が、CPUの違いやカーネル・glibc などの違いを吸収してくれるので、OS に .NET10 Runtime さえインストールされていれば、どこの環境でも稼働するメリットがあります。

Mac には、インテルCPU版・Apple Silicon版・A18 Pro版と三種類のCPUがあり、macOSのバージョンも新旧でいくつか並行して提供されています。

Linuxはディストリビューションが多数ありカーネルやglibcのバージョンも多数並行して提供されています。

これら全てにネイティブコードを提供することは、難しいと思います。

これらの違いは、 .NET10 Runtime に吸収してもらい、mfsr と mfprobe は Runtime だけに合わせる形式での提供が、一番効率が良く信頼性も高いと考えました。

## アセンブリは nuget.org で提供しています

 Windows版のネイティブコードは、winget で提供していますが、macOS と Linux OS へのマネージドコードでの提供は、nuget.org で提供しています。

インストールには dotnet コマンドで dotnet tool install によってインストールします。

### Windowsでは二つの提供がある

マネージドコードでの提供でお気づきの方もいると思いますが、nuget.org でのマネージドコードでの提供は、Windowsでもインストールすることが可能です。
つまり、Windowsでは winget 提供のネイティブコード版と、nuget.org 提供のマネージドコード版の二種類を提供していることになります。

私としては、Windows ではネイティブコード版を利用して欲しいと考えいますが、マネージドコード版もWindowsで同様に動作します。

正直なところ、どちらを使っても違いはほとんど無いです。

違いはマネージドコード版には .NET Runtime が必要なことぐらいです。
好きな方を利用してください。

なお、環境変数 Path の関係で両方使用することはできないと思います。

## .zip のダウンロードは必要無い

winget install でも dotnet tool install でも、事前に mfsr と mfprobe の .zip ファイルのダウンロードは必要無いです。

それぞれのコマンドを実行すればインストールできます。

具体的なインストール方法については、以下のページで解説しています。



[**mfsr-mfprobe インストール解説**](https://snow-stack.net/mfsr_install_guide/)



使い方の解説は、以前から紹介していますように、以下の二つのページで解説しています。

[rmsmf-txprobe & mfsr-mfprobe 使い方の分かりやすい解説](https://snow-stack.net/tool_rmsmf_guide/)

[PowerShellの文字エンコーディング問題に終止符を打つ：BOM・改行コードを自在に扱う方法](https://snow-stack.net/%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0/2026/03/15/PowerShell%E3%81%AE%E6%96%87%E5%AD%97%E3%82%A8%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E5%95%8F%E9%A1%8C%E3%81%AB%E7%B5%82%E6%AD%A2%E7%AC%A6%E3%82%92%E6%89%93%E3%81%A4-BOM-%E6%94%B9%E8%A1%8C%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E8%87%AA%E5%9C%A8%E3%81%AB%E6%89%B1%E3%81%86%E6%96%B9%E6%B3%95.html)



以上、「お知らせ」でした。
