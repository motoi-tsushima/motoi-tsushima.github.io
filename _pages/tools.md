---
title: "Tools"
layout: single
permalink: /tools/
author_profile: true
---
## Console tools

### rmsmf and txprobe

rmsmf は、複数のファイル内の複数の文字列を一括置換するコマンドラインツールです。文字エンコーディングの変換や、改行コードの変換に、BOM（Byte Order Mark）の制御も可能です。

txprobe は、複数のファイルの文字エンコーディング・BOM・改行コードを確認するコマンドツールです。

インストール方法:

以下の download から rmsmf.zip をダウンロードして解凍し、展開された 全てのファイルを 環境変数 Path の通ったフォルダーにコピーすれば、コンソールアプリとして使用できます。
単純な 実行ファイルだけで成り立つツールなので、インストーラーは用意していません。

使用方法は、それぞれ /h オプションで表示されます。

download [https://github.com/motoi-tsushima/rmsmf/releases/tag/v1.0.3.8](https://github.com/motoi-tsushima/rmsmf/releases/tag/v1.0.3.8)

repository [https://github.com/motoi-tsushima/rmsmf](https://github.com/motoi-tsushima/rmsmf)


※ rmsmf は以前から公開していましたが、今回 rmsmf の機能強化とリファクタリングをして、確認ツールの txprobe を追加して、正式版として再リリースしました。

