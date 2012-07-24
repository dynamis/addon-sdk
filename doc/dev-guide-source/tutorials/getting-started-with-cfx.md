<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

<span class="aside">このチュートリアルでは、[インストールガイド](dev-guide/tutorials/installation.html)の説明を読み、それに従って SDK をインストールし、起動していると想定しています。</span>

# cfx 入門 #

SDK を使用してアドオンを作成するには、`cfx` コマンドライン ツールを使いこなす必要があります。`cfx` は、アドオンのテストとパッケージングに使用します。

`cfx` の機能については、[reference documentation（リファレンスガイド）](dev-guide/cfx-tool.html) に詳細な説明がありますが、このチュートリアルでは、最低限必要な次の 3 つのコマンドについて説明します。

* [`cfx init`](dev-guide/tutorials/getting-started-with-cfx.html#cfx-init)
：アドオンのスケルトン構造を作成します。
* [注釈の保存](dev-guide/tutorials/annotator/storing.html)
：作成したアドオンをインストールした状態で、Firefox の新しいインスタンスを起動します。
* [注釈の保存](dev-guide/tutorials/annotator/storing.html)
：作成したアドオンを配布するための、インストール可能な [XPI](https://developer.mozilla.org/en/XPI) ファイルを作成します。

## <a name="cfx-init">cfx init</a> ##

`cfx init` を使用すると、アドオンの基本的なスケルトンが作成されます。

新しいディレクトリを作成し、コマンドシェルでそのディレクトリに移動して `cfx init` を実行します。

<pre>
mkdir my-addon
cd my-addon
cfx init
</pre>

このディレクトリは必ずしも SDK ルートの下に作成する必要はありません。`cfx` はいったん SDK ルートから起動されれば、その後 SDK の場所を記憶し続けるので、どのディレクトリからでも使用できます。

以下に、出力例を示します。

<pre>
* lib directory created
* data directory created
* test directory created
* doc directory created
* README.md written
* package.json written
* test/test-main.js written
* lib/main.js written
* doc/main.md written

  Your sample add-on is now ready for testing:
      try "cfx test" and then "cfx run". Have fun!"
</pre>

## <a name="cfx-run">cfx run</a> ##

`cfx run` を使用すると、作成したアドオンをインストールした状態で、Firefox の新しいインスタンスが起動されます。
これは、開発中のアドオンをテストするためのコマンドです。

先ほど `cfx init` の実行によって非常に基本的なアドオンがすでに作成されているので、コードを新たに作成しなくても、次のように入力するだけで `cfx run` の動作を試すことができます。

<pre>
cfx run
</pre>

このコマンドを初めて実行すると、以下のようなメッセージが表示されます。

<pre>
No 'id' in package.json: creating a new ID for you.
package.json modified: please re-run 'cfx run'
</pre>

<img class="image-right" src="static-files/media/screenshots/widget-mozilla.png"
alt="Mozilla icon widget" />

`cfx run` をもう一度実行すると、Firefox のインスタンスが起動され、ブラウザの右下隅に、Firefox ロゴのアイコンが表示されます。アイコンをクリックすると、新しいタブが開き、[http://www.mozilla.org/](http://www.mozilla.org/) が読み込まれます。

アドオンのメインコードは、常にアドオンの `lib` ディレクトリの `main.js` ファイルに保存されます。このアドオンの `main.js` を開いてみましょう。

    const widgets = require("widget");
    const tabs = require("tabs");

    var widget = widgets.Widget({
      id: "mozilla-link",
      label: "Mozilla website",
      contentURL: "http://www.mozilla.org/favicon.ico",
      onClick: function() {
        tabs.open("http://www.mozilla.org/");
      }
    });

このアドオンでは、`widget` と `tabs` の 2 つの SDK モジュールが使用されています。[`widget`](packages/addon-kit/widget.html) モジュールによってブラウザにボタンが追加され、[`tabs`](packages/addon-kit/tabs.html) モジュールによってタブの基本的な操作が実行されます。上のアドオンでは、Mozilla ファビコン（お気に入りアイコン）をアイコンに持つウィジェットが作成された後、Mozilla のホームページを新しいタブに読み込むクリックハンドラが追加されます。

それでは、このファイルを編集してみましょう。例えば以下のように、表示するアイコンや読み込む URL を変更することができます。

    const widgets = require("widget");
    const tabs = require("tabs");

    var widget = widgets.Widget({
      id: "jquery-link",
      label: "jQuery website",
      contentURL: "http://www.jquery.com/favicon.ico",
      onClick: function() {
        tabs.open("http://www.jquery.com/");
      }
    });

<img class="image-right" src="static-files/media/screenshots/widget-jquery.png"
alt="jQuery icon widget" />

コマンドプロンプトで `cfx run` をもう一度実行すると、今度はアイコンとして jQuery ファビコンが表示され、それをクリックすると [http://www.jquery.com](http://www.jquery.com) が表示されます。

<div style="clear:both"></div>

## <a name="cfx-xpi">cfx xpi</a> ##

`cfx run` がアドオンの開発中に使用するコマンドであるのに対し、`cfx xpi` は開発完了後に [XPI](https://developer.mozilla.org/en/XPI) ファイルをビルドするために使用します。XPI は、Firefox アドオンのインストール可能ファイル形式です。作成したファイルは、自分で配布することも、他のユーザーがダウンロードできるように [http://addons.mozilla.org](http://addons.mozilla.org) に公開することもできます。

XPI のビルドは、以下のようにアドオンのディレクトリから `cfx xpi` コマンドを実行するだけで完了します。

<pre>
cfx xpi
</pre>

以下のようなメッセージが表示されます。

<pre>
Exporting extension to my-addon.xpi.
</pre>

`my-addon.xpi` ファイルは、`cfx xpi` コマンドを実行したディレクトリに作成されます。

作成したファイルをテストするには、自分の Firefox にアドオンをインストールします。

これには、Firefox で Ctrl キー（Mac の場合は Cmd キー）を押しながら O キーを押すか、「ファイル」メニューの「ファイルを開く」を選択します。

ファイルを選択するダイアログが表示されます。`my-addon.xpi` ファイルに移動してファイルを開き、プロンプトに従ってアドオンをインストールします。

これで基本的な `cfx` コマンドを習得できました。ここからは、[SDK の各機能](dev-guide/tutorials/index.html) を実際に使用してみましょう。
