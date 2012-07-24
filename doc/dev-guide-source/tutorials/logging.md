# ロギング #

<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

[DOM `console`](https://developer.mozilla.org/en/DOM/console) は、JavaScript のデバッグに役立つオブジェクトです。しかし DOM オブジェクトはメインのアドオンコードで使用できないので、この SDK では独自のグローバルオブジェクトである `console` によって、エラーや警告、メッセージをログとして出力するメソッドを含め、DOM `console` とほぼ同じメソッドを提供しています。console を使用するために `require()` を使用する必要はありません。自動的に使用できるようになります。

`console.log()` メソッドは、メッセージを出力します。

    console.log("Hello World");

演習：

* 新しいディレクトリを作成し、そのディレクトリに移動します。
* `cfx init` を実行します。
* 「lib/main.js」を開き、内容を上のコードと置き換えます。
* `cfx run` を実行した後、`cfx run` を再度実行します。

Firefox が起動し、`cfx run` の実行に使用したコマンドウィンドウに以下の行が表示されます。

<pre>
info: Hello World!
</pre>

## コンテンツスクリプト内での `console` の使用 ##

console は、メインのアドオンコードだけでなく、[コンテンツスクリプト](dev-guide/guides/content-scripts/index.html) でも使用できます。以下のアドオンは、コンテンツスクリプトの内側で `console.log()` を呼び出して、ユーザーが読み込んだすべてのタブの HTML コンテンツをログとして出力します。

    require("tabs").on("ready", function(tab) {
      tab.attach({
        contentScript: "console.log(document.body.innerHTML);"
      });
    });

## `console` の出力 ##

アドオンをコマンドラインから（`cfx run` や `cfx test` を使用して）実行している場合、console のメッセージは使用したコマンドシェルに表示されます。

アドオンを Firefox にインストールしている場合、またはアドオンビルダーで実行している場合、メッセージは Firefox の[エラーコンソール](https://developer.mozilla.org/en/Error_Console)に表示されます。

## さらに詳しく ##

`console` API の詳細については、[API リファレンス：`console`（英語）](dev-guide/console.html)を参照してください。
