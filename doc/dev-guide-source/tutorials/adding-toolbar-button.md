<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# ツールバーへのボタンの追加 #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

ツールバーにボタンを追加するには、[`widget`](packages/addon-kit/widget.html) モジュールを使用します。

`cfx init` で作成されるデフォルトのアドオンではウィジェットが使用されているので、それを例にとって説明しましょう。[`cfx init`](dev-guide/tutorials/getting-started-with-cfx.html#cfx-init) を紹介するチュートリアルをまだ完了していない場合は、まずそれを完了した後、このページに戻ってください。

新しいディレクトリを作成し、そのディレクトリに移動して `cfx init` を実行します。その後、「lib」ディレクトリで「main.js」ファイルを開きます。

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

ウィジェットが、ブラウザウィンドウの一番下の「アドオンバー」に追加されます。

<img class="image-right" src="static-files/media/screenshots/widget-mozilla.png"
alt="Mozilla icon widget" />

開発者はウィジェットを最初の場所から移動できませんが、ユーザーはウィジェットを別のツールバーに移動することができます。`id` はウィジェットの位置を記憶するための必須の属性なので、このアドオンの後続のバージョンでも変更しないでください。

ボタンをクリックすると、[http://www.mozilla.org](http://www.mozilla.org) が開きます。

<div style="clear:both"></div>

## アイコンの指定 ##

ウィジェットを使用してツールバーボタンを作成する場合、表示するアイコンを `contentURL` で指定します。アイコンには、上の例のようにリモートファイルを指定することも、ローカルファイルを指定することもできます。下の例では、アドオンの `data` ディレクトリから「my-icon.png」というアイコンファイルを読み込みます。

    var widgets = require("widget");
    var tabs = require("tabs");
    var self = require("self");

    var widget = widgets.Widget({
      id: "mozilla-link",
      label: "Mozilla website",
      contentURL: self.data.url("my-icon.png"),
      onClick: function() {
        tabs.open("http://www.mozilla.org/");
      }
    });

アイコンは、ウィジェットの `contentURL` プロパティを設定していつでも変更できます。

## ユーザーの操作への対応 ##

`click`、`mouseover`、`mouseout` イベントをリッスンするには、対応するコンストラクタオプションとしてハンドラ関数を渡します。上のウィジェットの例では、`onClick` オプションを使用して `click` イベントにリスナーを割り当てています。これと同様の `onMouseover` オプションや `onMouseout` オプションも用意されています。

ユーザー操作に対してより複雑な処理を行うには、ウィジェットにコンテンツスクリプトを付加します。アドオンスクリプトとコンテンツスクリプトは、互いの変数に直接アクセスすることも、互いの関数を呼び出すこともできませんが、相互にメッセージを送信することは可能です。

例を挙げて説明しましょう。ウィジェットに内蔵された `onClick` プロパティではマウスの右クリックと左クリックが区別されないので、これを区別するにはコンテンツスクリプトを使用する必要があります。このスクリプトは、以下のようになります。

    window.addEventListener('click', function(event) {
      if(event.button == 0 && event.shiftKey == false)
        self.port.emit('left-click');

      if(event.button == 2 || (event.button == 0 && event.shiftKey == true))
        self.port.emit('right-click');
        event.preventDefault();
    }, true);

このスクリプトでは、標準の DOM `addEventListener()` 関数を使用してクリックイベントがリッスンされ、イベントが発生した場合、対応するメッセージがメインのアドオンコードに送信されます。「left-click」および「right-click」の2つのメッセージは、ウィジェット API 自体で定義されているのではなく、アドオン作成者が定義したカスタムイベントであることに注意してください。

このスクリプトを `data` ディレクトリに「click-listener.js」という名前で保存します。

次に、`main.js` が以下の動作を行うように変更します。

<ul>
<li><code>contentScriptFile</code> プロパティを設定してスクリプトを渡します。</li>

<li>新しいイベントをリッスンします。</li>
</ul>

    var widgets = require("widget");
    var tabs = require("tabs");
    var self = require("self");

    var widget = widgets.Widget({
      id: "mozilla-link",
      label: "Mozilla website",
      contentURL: "http://www.mozilla.org/favicon.ico",
      contentScriptFile: self.data.url("click-listener.js")
    });

    widget.port.on("left-click", function(){
      console.log("left-click");
    });

    widget.port.on("right-click", function(){
      console.log("right-click");
    });

ここで `cfx run` を再度実行し、左右のマウスボタンをクリックしてみます。
対応する文字列が、コマンドシェルに書き込まれるのを確認してください。

## パネルの付加 ##

<!-- The icon the widget displays, shown in the screenshot, is taken from the
Circular icon set, http://prothemedesign.com/circular-icons/ which is made
available under the Creative Commons Attribution 2.5 Generic License:	
http://creativecommons.org/licenses/by/2.5/ -->

<img class="image-right" src="static-files/media/screenshots/widget-panel-clock.png"
alt="Panel attached to a widget">

ウィジェットのコンストラクタで `panel` オブジェクトを指定すると、ユーザーがウィジェットをクリックしたときにパネルが表示されます。

    data = require("self").data

    var clockPanel = require("panel").Panel({
      width:215,
      height:160,
      contentURL: data.url("clock.html")
    });

    require("widget").Widget({
      id: "open-clock-btn",
      label: "Clock",
      contentURL: data.url("History.png"),
      panel: clockPanel
    });

パネルの操作の詳細については、[ポップアップの表示](dev-guide/tutorials/display-a-popup.html) のチュートリアルを参照してください。

## さらに詳しく ##

widget モジュールの詳細については、[API リファレンスのドキュメント（英語）](packages/addon-kit/widget.html) を参照してください。

コンテンツスクリプトの詳細については、[コンテンツスクリプトガイド（英語）](dev-guide/guides/content-scripts/index.html)を参照してください。
