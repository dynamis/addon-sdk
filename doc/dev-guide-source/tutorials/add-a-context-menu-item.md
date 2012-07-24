<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# コンテキストメニューアイテムの追加 #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ[SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

Firefox のコンテキストメニューにアイテムやサブメニューを追加するには、[`context-menu`](packages/addon-kit/context-menu.html) モジュールを使用します。

以下は、コンテキストメニューアイテムを新しく追加するアドオンです。このアイテムは、ページ内で任意の要素を選択したときに表示されます。このアイテムをクリックすると、選択した内容がメインのアドオンコードに送信されてログに記録されます。

    var menuItem = contextMenu.Item({
      label: "Log Selection",
      context: contextMenu.SelectionContext(),
      contentScript: 'self.on("click", function () {' +
                     '  var text = window.getSelection().toString();' +
                     '  self.postMessage(text);' +
                     '});',
      onMessage: function (selectionText) {
        console.log(selectionText);
      }
    });

演習：上のアドオンを実行した後、Web ページをロードし、任意のテキスト文字列を選択して右クリックしてください。
以下の図のように、新しいアイテムが表示されます。

<img class="image-center" src="static-files/media/screenshots/context-menu-selection.png"></img>

このアイテムをクリックすると、選択した文字列が[コンソールにログとして出力](dev-guide/tutorials/logging.html)されます。

<pre>
info: elephantine lizard
</pre>

このアドオンが行うのは、コンテキストメニューアイテムの作成だけであって、作成したコンテキストメニューアイテムを追加する必要はありません。メニューアイテムを作成するだけで、適切なコンテキストに自動的にそのアイテムが追加されるからです。上のアドオンでは、コンストラクタが、`label`、`context`、`contentScript`、`onMessage` の 4 つのオプションを取っています。

### label ###

`label` は、アイテムとして表示される文字列です。

### context ###

`context` は、メニューアイテムをどのような状況で表示するかを指定します。`context-menu` モジュールには、単純なコンテキストがいくつか内蔵されています。ここで使用した `SelectionContext()` はその 1 つで、「ページで何らかの要素を選択したときにアイテムを表示する」ことを意味します。

単純なコンテキストだけでは不十分な場合、スクリプトを使用してさらに高度なコンテキストを定義できます。

### contentScript ###

このオプションは、アイテムにスクリプトを結合します。上のアドオンの場合、スクリプトは、ユーザーがアイテムをクリックするのをリッスンし、選択したテキストが入ったメッセージをアドオンに送信します。

### onMessage ###

`onMessage` プロパティは、コンテキストメニューアイテムに結合されたスクリプトからのメッセージを、アドオンコード側のハンドラ関数を指定します。上のスクリプトでは、選択したテキストがログに記録されます。

つまり、以下のように処理が実行されます。

1. ユーザーがアイテムをクリックします。
2. コンテンツスクリプトの `click` イベントが起動し、選択したテキストがコンテンツスクリプトによって取得され、メッセージがアドオンに送信されます。
3. アドオンの `message` イベントが起動し、選択したテキストがアドオンコードのハンドラ関数に渡され、ログに記録されます。

## さらに詳しく ##

`context-menu` モジュールの詳細については、[API リファレンス:`context-menu`（英語）] (packages/addon-kit/context-menu.html)を参照してください。
