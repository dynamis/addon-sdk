<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# タブにホストされているページコンテンツの変更 #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

特定のタブによってホストされるページを変更するには、[tab](packages/addon-kit/tabs.html) オブジェクトの `attach()` メソッドを使用して、対象のページにスクリプトを読み込みます。このスクリプトは Web コンテンツとのやりとりを目的としているので、*コンテンツスクリプト*と呼ばれます。

以下に単純な例を示します。

    var widgets = require("widget");
    var tabs = require("tabs");

    var widget = widgets.Widget({
      id: "mozilla-link",
      label: "Mozilla website",
      contentURL: "http://www.mozilla.org/favicon.ico",
      onClick: function() {
        tabs.activeTab.attach({
          contentScript:
            'document.body.style.border = "5px solid red";'
          })
        }
    });

このアドオンでは、Mozilla ファビコンをアイコンとするウィジェットが作成されます。このウィジェットにあるクリックハンドラは、アクティブなタブを取得し、アアクティブなタブにホストされているページにスクリプトを読み込ませます。スクリプトは `contentScript` オプションで指定され、上記の例では、ページの周囲に赤い枠を表示する処理を行っています。演習：

* 新しいディレクトリを作成し、そのディレクトリに移動します。
* `cfx init` を実行します。
* `lib/main.js` ファイルを開き、その内容を上のコードと置き換えます。
* `cfx run` を実行した後、`cfx run` を再度実行します。

下の図のように、ブラウザの右下隅に Mozilla アイコンが表示されます。

<img class="image-center" src="static-files/media/screenshots/widget-mozilla.png"
alt="Mozilla icon widget" />

次に、同ブラウザ上で任意のページを開き、この Mozilla アイコンをクリックします。 
以下の図のように、ページの周囲に赤い枠が表示されます。

<img class="image-center" src="static-files/media/screenshots/tabattach-bbc.png"
alt="bbc.co.uk modded by tab.attach" />

## 別ファイルでのコンテンツスクリプトの管理 ##

上の例では、コンテンツスクリプトを文字列として渡しました。しかし、きわめて単純なスクリプトでない限り、スクリプトは別のファイルに保存して管理してください。これにより、コードの管理、デバッグ、レビューが容易になります。 

上のスクリプトを別ファイルとして管理する場合、例えば、アドオンの `data` ディレクトリの下の `my-script.js` というファイルに以下のコードを書き、保存します。

    document.body.style.border = "5px solid red";

このスクリプトを読み込むには、アドオンコードを以下のように変更します。

    var widgets = require("widget");
    var tabs = require("tabs");
    var self = require("self");

    var widget = widgets.Widget({
      id: "mozilla-link",
      label: "Mozilla website",
      contentURL: "http://www.mozilla.org/favicon.ico",
      onClick: function() {
        tabs.activeTab.attach({
          contentScriptFile: self.data.url("my-script.js")
        });
      }
    });

スクリプトは複数個読み込むことができます。またスクリプトは相互に直接やりとりすることが可能です。例えば、[jQuery](http://jquery.com/) を読み込んで、コンテンツスクリプトで使用することができます。

## コンテンツスクリプトとのやりとり ##

アドオンスクリプトとコンテンツスクリプトは、互いの変数に直接アクセスすることも、互いの関数を呼び出すこともできませんが、相互にメッセージを送信することは可能です。

アドオンスクリプトとコンテンツスクリプトの間でメッセージを送信するには、送信側が `port.emit()` を呼び出し、受信側が `port.on()` を使用してリッスンします。

* コンテンツスクリプトで、`port` はグローバルオブジェクト `self` のプロパティです。
* アドオンスクリプトでは、コンテンツスクリプトへのメッセージ送信に使用する `port` プロパティを格納したオブジェクトが、`tab-attach()` によって返されます。

上のコード例を書き換えて、アドオンからコンテンツスクリプトにメッセージを渡しましょう。書き換え後のコンテンツスクリプトは、以下のようになります。

    // "self" is a global object in content scripts
    // Listen for a "drawBorder"
    self.port.on("drawBorder", function(color) {
      document.body.style.border = "5px solid " + color;
    });

アドオンスクリプトで、`attach()` から返されたオブジェクトを使用して、コンテンツスクリプトに "drawBorder" メッセージを送信します。

    var widgets = require("widget");
    var tabs = require("tabs");
    var self = require("self");

    var widget = widgets.Widget({
      id: "mozilla-link",
      label: "Mozilla website",
      contentURL: "http://www.mozilla.org/favicon.ico",
      onClick: function() {
        worker = tabs.activeTab.attach({
          contentScriptFile: self.data.url("my-script.js")
        });
        worker.port.emit("drawBorder", "red");
      }
    });

`drawBorder` メッセージは内蔵のメッセージではなく、このアドオンの `port.emit()` 呼び出しで定義されたメッセージです。

## CSS のインジェクション ##

[`page-mod`](dev-guide/tutorials/modifying-web-pages-url.html) API と異なり、`tab.attach()` ではページに直接 CSS をインジェクションすることはできません。

ページのスタイルを変更するには、上の例のように JavaScript を使用する必要があります。

## さらに詳しく ##

SDK でのタブの操作方法の詳細については、[Web ページを開く](dev-guide/tutorials/open-a-web-page.html)と[開いているタブの一覧表示](dev-guide/tutorials/list-open-tabs.html) の各チュートリアル、および[API リファレンス：`tabs`（英語）](packages/addon-kit/tabs.html)を参照してください。

コンテンツスクリプトの詳細については、[コンテンツスクリプトガイド（英語）](dev-guide/guides/content-scripts/index.html)を参照してください。
