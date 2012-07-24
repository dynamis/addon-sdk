<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# Web ページを開く #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

新しい Web ページを開くには、以下のように [`tabs`](packages/addon-kit/tabs.html) モジュールを使用します。

    var tabs = require("tabs");
    tabs.open("http://www.example.com");

この関数は非同期なので、[`tab` オブジェクト](packages/addon-kit/tabs.html#Tab) をすぐに受け取って調べることはできません。`tab` オブジェクトがすぐに返されるようにするには、コールバック関数を `open()` に渡します。これにより、コールバックが `onReady` プロパティに割り当てられ、タブが引数として渡されます。

    var tabs = require("tabs");
    tabs.open({
      url: "http://www.example.com",
      onReady: function onReady(tab) {
        console.log(tab.title);
      }
    });

この場合も、このタブにホストされているコンテンツには直接アクセスできません。

タブのコンテンツにアクセスするには、`tab.attach()` を使用してタブにスクリプトを付加する必要があります。以下のアドオンでは、ページを読み込んだ後、そのページに赤い枠を追加するスクリプトが付加されます。

    var tabs = require("tabs");
    tabs.open({
      url: "http://www.example.com",
      onReady: runScript
    });

    function runScript(tab) {
      tab.attach({
        contentScript: "document.body.style.border = '5px solid red';"
      });
    }

## さらに詳しく ##

SDK でのタブの使用方法の詳細については、[API リファレンス：`tabs`（英語）](packages/addon-kit/tabs.html)を参照してください。

タブでスクリプトを実行する方法の詳細については、[`tab.attach()` の使用方法についてのチュートリアル](dev-guide/tutorials/modifying-web-pages-tab.html)を参照してください。
