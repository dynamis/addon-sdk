<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# URL に基づいた Web ページの変更 #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

特定のパターン（例えば、「http://example.org/」）に一致するページを読み込んだときにそのページを変更するには、[`page-mod`](packages/addon-kit/page-mod.html) モジュールを使用します。

page-mod を作成するには、以下の 2 つを指定する必要があります。

* 実行する 1 つまたは複数のスクリプト。Web コンテンツとのやりとりを目的としたスクリプトなので、*コンテンツスクリプト*と呼ばれます。
* 変更するページの URL を検出するための 1 つまたは複数の一致パターン

以下に単純な例を示します。コンテンツスクリプトは `contentScript` オプションとして指定します。 

    // Import the page-mod API
    var pageMod = require("page-mod");

    // Create a page mod
    // It will run a script whenever a ".org" URL is loaded
    // The script replaces the page contents with a message
    pageMod.PageMod({
      include: "*.org",
      contentScript: 'document.body.innerHTML = ' +
                     ' "<h1>Page matches ruleset</h1>";'
    });

演習：

* 新しいディレクトリを作成し、そのディレクトリに移動します。
* `cfx init` を実行します。
* `lib/main.js` ファイルを開き、その内容を上のコードと置き換えます。
* `cfx run` を実行した後、`cfx run` を再度実行します。
* 開いたブラウザウィンドウで、[ietf.org](http://www.ietf.org) を開きます。

これにより、以下が表示されます。

<img  class="image-center" src="static-files/media/screenshots/pagemod-ietf.png"
alt="ietf.org eaten by page-mod" />

## 一致パターンの指定 ##

一致パターンの指定には、[`match-pattern`](packages/api-utils/match-pattern.html) 構文を使用します。match-pattern 文字列は個別に渡すことも、配列として渡すこともできます。

## 別ファイルでのコンテンツスクリプトの管理 ##

上の例では、コンテンツスクリプトを文字列として渡しました。しかし、きわめて単純なスクリプトでない限り、スクリプトは別のファイルに保存して管理してください。これにより、コードの管理、デバッグ、レビューが容易になります。

スクリプトを別のファイルに保存するには、以下の手順に従います。

* アドオンの `data` ディレクトリにスクリプトを保存します。
* `contentScript` の代わりに `contentScriptFile` オプションを使用し、スクリプトの URL を渡します。URL は `self.data.url()` を使用して取得できます。 

上のスクリプトを別ファイルとして管理する場合、例えば、アドオンの `data` ディレクトリの下の `my-script.js` というファイルに以下のコードを書き、保存します。

    document.body.innerHTML = "<h1>Page matches ruleset</h1>";

このスクリプトを読み込むには、page-mod コードを以下のように変更します。

    // Import the page-mod API
    var pageMod = require("page-mod");
    // Import the self API
    var self = require("self");

    // Create a page mod
    // It will run a script whenever a ".org" URL is loaded
    // The script replaces the page contents with a message
    pageMod.PageMod({
      include: "*.org",
      contentScriptFile: self.data.url("my-script.js")
    });

## 複数のコンテンツスクリプトの読み込み ##

スクリプトは複数個読み込むことができます。またスクリプトは相互に直接やりとりすることが可能です。例えば、jQuery を使用して `my-script.js` を以下のように書き換えることができます。

    $("body").html("<h1>Page matches ruleset</h1>");

次に、アドオンの `data` ディレクトリに jQuery をダウンロードし、スクリプトと jQuery を一緒に読み込みます（必ず jQuery を先に読み込んでください）。

    // Import the page-mod API
    var pageMod = require("page-mod");
    // Import the self API
    var self = require("self");

    // Create a page mod
    // It will run a script whenever a ".org" URL is loaded
    // The script replaces the page contents with a message
    pageMod.PageMod({
      include: "*.org",
      contentScriptFile: [self.data.url("jquery-1.7.min.js"),
                          self.data.url("my-script.js")]
    });

`contentScript` と `contentScriptFile` は、同じ page-mod 内で使用できます。その場合、`contentScript` で指定したスクリプトが先に読み込まれます。

    // Import the page-mod API
    var pageMod = require("page-mod");
    // Import the self API
    var self = require("self");

    // Create a page mod
    // It will run a script whenever a ".org" URL is loaded
    // The script replaces the page contents with a message
    pageMod.PageMod({
      include: "*.org",
      contentScriptFile: self.data.url("jquery-1.7.min.js"),
      contentScript: '$("body").html("<h1>Page matches ruleset</h1>");'
    });

スクリプトを Web サイトから読み込めないことに注意してください。スクリプトは `data` ディレクトリから読み込む必要があります。

## コンテンツスクリプトとのやりとり ##

アドオンスクリプトとコンテンツスクリプトは、互いの変数に直接アクセスすることも、互いの関数を呼び出すこともできませんが、相互にメッセージを送信することは可能です。

アドオンスクリプトとコンテンツスクリプトの間でメッセージを送信するには、送信側が `port.emit()` を呼び出し、受信側が `port.on()` を使用してリッスンします。

* コンテンツスクリプトで、`port` はグローバルオブジェクト `self` のプロパティです。
* アドオンスクリプトでは、`onAttach` イベントをリッスンして、`port` を含むオブジェクトを渡す必要があります。

上のコード例を書き換えて、アドオンからコンテンツスクリプトにメッセージを渡しましょう。メッセージにはドキュメントに挿入する新しいコンテンツが格納されます。書き換え後のコンテンツスクリプトは、以下のようになります。

    // "self" is a global object in content scripts
    // Listen for a message, and replace the document's
    // contents with the message payload.
    self.port.on("replacePage", function(message) {
      document.body.innerHTML = "<h1>" + message + "</h1>";
    });

アドオンスクリプトで、コンテンツスクリプトに対し、`onAttach` の中のメッセージを送信します。

    // Import the page-mod API
    var pageMod = require("page-mod");
    // Import the self API
    var self = require("self");

    // Create a page mod
    // It will run a script whenever a ".org" URL is loaded
    // The script replaces the page contents with a message
    pageMod.PageMod({
      include: "*.org",
      contentScriptFile: self.data.url("my-script.js"),
      // Send the content script a message inside onAttach
      onAttach: function(worker) {
        worker.port.emit("replacePage", "Page matches ruleset");
      }
    });

`replacePage` メッセージは内蔵のメッセージではなく、アドオンの `port.emit()` 呼び出しによって定義されたメッセージです。

<div class="experimental">

## CSS のインジェクション ##

**このセクションで説明する機能は、現時点ではまだ実験的です。この機能はおそらく今後も引き続きサポートされますが、API の詳細が変更されることがあります。**

ページに JavaScript をインジェクションするのではなく、page-mod の `contentStyle` オプションを設定して CSS をインジェクションすることができます。

    var pageMod = require("page-mod").PageMod({
      include: "*",
      contentStyle: "body {" +
                    "  border: 5px solid green;" +
                    "}"
    });

`contentScript` と同様に、対応する `contentStyleFile` オプションを使用して「data」ディレクトリにある CSS の URL を指定することができます。CSS が非常に複雑な場合は、`contentStyle` よりもこのオプションを優先して使用することをお勧めします。

    var pageMod = require("page-mod").PageMod({
      include: "*",
      contentStyleFile: require("self").data.url("my-style.css")
    });

</div>

## さらに詳しく ##

page-mod の詳細については、[API リファレンスページ（英語）](packages/addon-kit/page-mod.html) を参照してください。

コンテンツスクリプトの詳細については、[コンテンツスクリプトガイド（英語）](dev-guide/guides/content-scripts/index.html)を参照してください。
