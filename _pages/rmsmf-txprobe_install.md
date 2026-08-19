---
title: "rmsmf-txprobe インストール解説"
layout: single
permalink: /rmsmf_install_guide/
author_profile: true
---

## インストール方法:

以下の Download から rmsmf.zip をダウンロードして解凍し、展開された 全てのファイルを 環境変数 Path の通ったフォルダーにコピーすれば、コンソールアプリとして使用できます。
単純な 実行ファイルだけで成り立つツールなので、インストーラーは用意していません。

使用方法は、それぞれ /h オプションで表示されます。

**Download** [https://github.com/motoi-tsushima/rmsmf/releases/tag/v1.1.1.0](https://github.com/motoi-tsushima/rmsmf/releases/tag/v1.1.1.0)

**Repository** [https://github.com/motoi-tsushima/rmsmf](https://github.com/motoi-tsushima/rmsmf)

## 重要なお知らせ（2026年8月19日）

前回、リリースした rmsmf のリリースファイルに異常があり、起動できない状態で配布しておりました。

原因として、rmsmf の参照している標準NuGetパッケージのUpgradeに失敗しており、起動時にバージョンが合わずエラーになっていたようです。

.NET Framework 4.8 の実行ファイルは、最新の .NET10 のNuGetパッケージを参照できないので、NuGetパッケージを最新にしてしまうと、リンクできずに起動時にエラーになってしまうのです。

自分では確認したつもりでしたが、テストミスにより別のモジュールと勘違いしていたのかもしれません。

重ね重ね、失敗が続き、大変申し訳ありません。
