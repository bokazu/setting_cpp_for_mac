# M1 Macでclangからg++へ変更する方法

## 1. 概要
Macは標準でclangのみを搭載している。例えば、ターミナルで
```
g++ --version
```
と打つと
```
Configured with: --prefix=/Applications/Xcode.app/Contents/Developer/usr --with-gxx-include-dir=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/c++/4.2.1
Apple clang version 11.0.3 (clang-1103.0.32.59)
Target: x86_64-apple-darwin19.4.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
```
のように返ってくる。
別にこれで困ることはないがlinuxのほうではg++を使用しているため、両者で使用しているものを揃えておきたい。clangとg++の違いについては以下の記事を参考にした。

[clangとg++の違い](https://qiita.com/pochiMasahiro/items/eb9c1a95fc01228e1dd7)

ここでは、homebrewはすでにインストールされているものとして説明を進める。また、M1 Mac向けに書いているものなのでIntel製のCPUのMacには適用できない部分がある。

#### 参考にした記事
この文章は以下の記事に書いてある内容をまとめる形で作成した。

[Mac mini (M1, 2020) で競プロ環境構築 (VSCode)](https://qiita.com/cubinglover/items/5b4d05ca0f77c60f1d79)

[MacでのC言語の実行環境変更メモ](https://zenn.dev/peg/articles/7c9ab2c4901c80)

[初心者向け　MacでOperation not permittedの解決方法](https://qiita.com/iwaseasahi/items/9d2e29b02df5cce7285d)

## 2.homebrewでgccをインストールする
ターミナルで次のことを実行し、GCCをインストールする。
```
brew install gcc
```

## 3.リンクの設定
さきほど導入したGCCを使用するためにシンボリックリンクを貼る。
```
sudo ln -sf $(ls -d /opt/homebrew/bin/* | grep "/g++-" | sort -r | head -n1) /usr/local/bin/gcc
sudo ln -sf $(ls -d /opt/homebrews/bin/* | grep "/g++-" | sort -r | head -n1) /usr/local/bin/g++
```
ここで、ネット上にあがっている記事には`/opt/homebrew/`の部分が`/usr/local/`となっているものもあるがこれはIntel製のCPUに対するものなので注意が必要である。M1ではhomebrewによるinstall先として`/opt/homebrew/bin`が推奨されているため違いが生じる。


以上で設定は完了しているはずである。

#### 注意
mac OSのX 10.11 El Capitanより追加されたセキュリティ機能、SIP（System Integrity Protection）によって以下の領域がガードされている。
- `/bin/`
- `/sbin/`
- `/System`
そこで、上記のコマンドを実行すると
```
ln : Operation not permitted
```
というエラーが発生する場合がある。これを解決するには「設定」→「セキュリティとプライバシー」→「フルディスクアクセス」からターミナルに対して許可を与える必要がある。許可を与えたらmacを再起動すればよい。

## 4. g++のversionの確認
上記の手順を踏んだ後で、ターミナルでg++のversionを確認してみる。すると、以下のようになるはずだ。
```
$ g++ --version
  g++ (Homebrew GCC 11.2.0_3) 11.2.0
  Copyright (C) 2021 Free Software Foundation, Inc.
  This is free software; see the source for copying conditions.  There is NO
  warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
このように表示されれば設定は完了している。

## 5. VSCodeでの設定
エディタとしてVScodeを使用している場合は、そちらのほうの設定も少しばかりいじる必要がある。
まずは、`c_cpp_properties.json`を開いて以下のように設定する。
```
{
    "configurations": [
        {
            "name": "Mac",
            "includePath": [
                "${workspaceFolder}/**",
                "/opt/homebrew/include/**",
                ],
            "defines": [],
            "macFrameworkPath": [
                "System/Library/Frameworks"
            ],
            "compilerPath": "/usr/local/bin/g++",
            "cStandard": "c17",
            "cppStandard": "c++17",
            "intelliSenseMode": "macos-gcc-arm64",
            "browse": {
                "limitSymbolsToIncludedHeaders": false
            }
        }
    ],
    "version": 4
}
```
このように設定すればVsCodeでg++としてGCC版が使えるようになっているはずである。