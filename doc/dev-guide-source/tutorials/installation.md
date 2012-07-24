<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# インストール #

## 前提条件

アドオン SDK を使用して開発を行うには、以下が必要です。

* [Python](http://www.python.org/) 2.5 または 2.6。Python バージョン 3.0 と 3.1 はサポートされていません。環境変数 PATH に Python のパスを設定してください。

* [対応するバージョンの Firefox](dev-guide/guides/firefox-compatibility.html)。
SDK が公開された時点で公開されているバージョンの Firefox か、または SDK が公開された時点のベータバージョンの Firefox です。SDK リリースと Firefox のリリースの対応については、[SDK のリリース予定（英語）](https://wiki.mozilla.org/Jetpack/SDK_2012_Release_Schedule)を参照してください。

* SDK 本体。最新の安定したバージョンの SDK は、[tarball](https://ftp.mozilla.org/pub/mozilla.org/labs/jetpack/jetpack-sdk-latest.tar.gz) または [zip ファイル](https://ftp.mozilla.org/pub/mozilla.org/labs/jetpack/jetpack-sdk-latest.zip) として入手できます。あるいは、最新の開発バージョンを [GitHub リポジトリ](https://github.com/mozilla/addon-sdk)から入手することもできます。

## Mac OS X / Linux でのインストール ##

適切な場所を選択してファイルを展開します。シェル / コマンドプロンプトで SDK のルートディレクトリに移動します。以下に例を示します。

<pre>
tar -xf addon-sdk.tar.gz
cd addon-sdk
</pre>

次に、以下を実行します。

<pre>
source bin/activate
</pre>

以下のように、コマンドプロンプトの前に SDK のルートディレクトリ名が追加されます。

<pre>
(addon-sdk)‾/mozilla/addon-sdk >
</pre>

## Windows でのインストール ##

適切な場所を選択してファイルを展開します。シェル / コマンドプロンプトで SDK のルートディレクトリに移動します。以下に例を示します。

<pre>
7z.exe x addon-sdk.zip
cd addon-sdk
</pre>

次に、以下を実行します。

<pre>
bin¥activate
</pre>

以下のように、コマンドプロンプトの前に SDK の絶対パスが追加されます。

<pre>
(C:¥Users¥mozilla¥sdk¥addon-sdk) C:¥Users¥Work¥sdk¥addon-sdk>
</pre>

## SDK 仮想環境 ##

上記のようにコマンドプロンプトが変更されていれば、シェルで仮想環境が起動し、アドオン SDK コマンドラインツールにアクセスできます。

仮想環境は、`deactivate` を実行していつでも終了することができます。

仮想環境は、この特定のコマンドプロンプトでのみ有効です。このコマンドプロンプトを閉じると、仮想環境が終了するので、新しいコマンドプロンプトを起動するたびに `source bin/activate` または `bin¥activate` と入力する必要があります。新しいコマンドプロンプトを開くだけでは、SDK は起動されません。

ディスク上の異なる場所に SDK の複数のコピーを置き、切り替えて使用することもできます。さらには、別個のコマンドプロンプトで、両方のコピーを同時に起動することも可能です。

### `activate` の永続化 ###

`activate` が行う処理は、最上位レベルの `bin` ディレクトリにあるスクリプトを使用して、現在のコマンドプロンプトに関する複数の環境変数を設定することだけです。そこで、使用する環境でこれらの変数を永続的に設定すれば、新しくコマンドプロンプトを開くだけでそれらの変数が読み込まれ、仮想環境が常に使用できます。これにより、新しいコマンドプロンプトを開くたびに `activate` と入力する必要がなくなります。

ただし、コマンドプロンプトに関する変数が、新しい SDK のリリース時に変更されることがあるので、SDK の起動スクリプトを参照して、設定が必要な変数を確認してください。bash 環境（Linux および Mac OS X）と Windows 環境では、起動に使用するスクリプトも、それによって設定される変数も異なります。

#### Windows ####

Windows では、`bin¥activate` を実行すると `activate.bat` が使用されます。SDK を常に有効にするには、コマンドラインから `setx` ツールを使用するか、コントロール パネルを使用します。

#### Linux / Mac OS X ####

Linux および Mac OS X では、`source bin/activate` により `activate` bash スクリプトが使用されます。SDK を常に有効にするには、`‾/.bashrc`（Linux）または `‾/.bashprofile`（Mac OS X）を使用します。

あるいは、`‾/bin` ディレクトリにある `cfx` プログラムへのシンボリックリンクを作成する方法もあります。

<pre>
ln -s PATH_TO_SDK/bin/cfx ‾/bin/cfx
</pre>

## サニティチェック ##

シェルプロンプトで以下を実行します。

<pre>
cfx
</pre>

これにより、使用状況の情報が大量に出力されますが、以下では最初の数行のみを示します。

<pre>
Usage: cfx [options] [command]
</pre>

これが `cfx` コマンドラインプログラムです。`cfx` は、アドオン SDK の主要なインターフェイスで、Firefox の起動とアドオンのテスト、アドオンを配布するためのパッケージング、文書の表示、単体テストの実行に使用します。

## cfx docs ##

cfx docs` を実行すると、SDK のドキュメントがビルドされ、ブラウザに表示されます。

## 問題が発生した場合 ##

[トラブルシューティング](dev-guide/tutorials/troubleshooting.html)のページを参照して、解決を試みてください。
 

## 次のステップ ##

[cfx 入門](dev-guide/tutorials/getting-started-with-cfx.html)のチュートリアルで、`cfx` ツールによってアドオンを作成する方法を学習してください。
