<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# ページ読み込みのリッスン #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

[`tabs`](packages/addon-kit/tabs.html) モジュールを使用して、新しいページが読み込まれたときに通知を受け取ることができます。以下のアドオンは、タブに内蔵されている `ready` イベントをリッスンし、ユーザーがタブに URL を読み込んだときにその URL をコンソールに出力します。

    require("tabs").on("ready", logURL);

    function logURL(tab) {
      console.log(tab.url);
    }

タブにホストされているコンテンツには直接アクセスできません。

タブのコンテンツにアクセスするには、`tab.attach()` を使用してタブにスクリプトを付加する必要があります。以下のアドオンは、開いているすべてのタブにスクリプトを付加します。付加されたスクリプトは、タブ内の文書に赤い枠を追加します。

    require("tabs").on("ready", logURL);

    function logURL(tab) {
      runScript(tab);
    }

    function runScript(tab) {
      tab.attach({
        contentScript: "if (document.body) document.body.style.border = '5px solid red';"
      });
    }

（上のアドオンは、概念をわかりやすく示すための例に過ぎません。このような動作を実際に実装するには、[`page-mod`](dev-guide/tutorials/modifying-web-pages-url.html) を代わりに使用し、一致パターンとして「*」を指定します）

## さらに詳しく ##

SDK でのタブの使用方法の詳細については、[API リファレンス：`tabs`（英語）](packages/addon-kit/tabs.html)を参照してください。この他にも、`open`、`close`、`activate` などのタブイベントをリッスンできます。

タブでスクリプトを実行する方法の詳細については、[`tab.attach()` の使用方法についてのチュートリアル](dev-guide/tutorials/modifying-web-pages-tab.html)を参照してください。
