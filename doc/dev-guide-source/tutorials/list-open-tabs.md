<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# 開いているタブの一覧表示 #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

開いているタブを一覧表示するには、[`tabs`](packages/addon-kit/tabs.html) オブジェクトを反復処理します。

以下のアドオンは、ユーザーがクリックしたときに、開いているタブの URL をログとして出力する[ウィジェット](packages/addon-kit/widget.html) を追加します。

    var widget = require("widget").Widget({
      id: "mozilla-link",
      label: "Mozilla website",
      contentURL: "http://www.mozilla.org/favicon.ico",
      onClick: listTabs
    });

    function listTabs() {
      var tabs = require("tabs");
      for each (var tab in tabs)
        console.log(tab.url);
    }

このアドオンを実行し、タブを 2、3 個読み込み、ウィジェットをクリックすると、[コンソール](dev-guide/console.html)に以下のような出力が表示されます。

<pre>
info: http://www.mozilla.org/en-US/about/
info: http://www.bbc.co.uk/
</pre>

このタブでホストされるコンテンツには直接アクセスできません。タブのコンテンツにアクセスするには、`tab.attach()` を使用してタブにスクリプトを付加する必要があります。以下のアドオンは、開いているすべてのタブにスクリプトを付加します。付加されたスクリプトは、タブの文書に赤い枠を追加します。

    var widget = require("widget").Widget({
      id: "mozilla-link",
      label: "Mozilla website",
      contentURL: "http://www.mozilla.org/favicon.ico",
      onClick: listTabs
    });

    function listTabs() {
      var tabs = require("tabs");
      for each (var tab in tabs)
        runScript(tab);
    }

    function runScript(tab) {
      tab.attach({
        contentScript: "document.body.style.border = '5px solid red';"
      });
    }

## さらに詳しく ##

SDK でのタブの操作方法の詳細については、[API リファレンス：`tabs`（英語）](packages/addon-kit/tabs.html)を参照してください。

タブでスクリプトを実行する方法の詳細については、[`tab.attach()` の使用方法についてのチュートリアル](dev-guide/tutorials/modifying-web-pages-tab.html)を参照してください。
