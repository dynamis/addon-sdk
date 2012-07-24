<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# ウィジェットの実装 #

ウィジェットには、次の 2 つの機能を実装します。

<span class="aside">
[Bug 634712](https://bugzilla.mozilla.org/show_bug.cgi?id=634712) のバグのため、ウィジェット API は左右のマウスクリックに対して別個の（または少なくとも区別可能な）イベントを発生させる必要があります。このバグが修正されれば、このウィジェットのコンテンツスクリプトは不要になります。</span>

* ウィジェットを左クリックすると、アノテーターのオンとオフが切り替わります。
* 右クリックすると、ユーザーが作成したすべての注釈のリストが表示されます。

ウィジェットの `click` イベントは左右のマウスクリックを区別しないので、コンテンツスクリプトを使用してクリックイベントを取得し、対応するメッセージをアドオンに送り返します。

ウィジェットでは、アドオンがアクティブであることを示すアイコンと、非アクティブであることを示すアイコンの 2 つのアイコンが使用されます。

したがって、ウィジェットのコンテンツスクリプトと 2 つのアイコンの合計 3 つのファイルを作成する必要があります。

`data` サブディレクトリの中に、別のサブディレクトリ `widget` を作成します。ウィジェットのファイルはここに保存します（このことは必須ではありません。すべてのファイルを `data` の下に置くことも可能です。しかしディレクトリを分けた方が、整理された感じがします）。

## ウィジェットのコンテンツスクリプト ##

ウィジェットのコンテンツスクリプトは、以下のように左右のマウスクリックをリッスンして、対応するメッセージをアドオンコードに送信するだけのスクリプトです。

    this.addEventListener('click', function(event) {
      if(event.button == 0 && event.shiftKey == false)
        self.port.emit('left-click');

      if(event.button == 2 || (event.button == 0 && event.shiftKey == true))
        self.port.emit('right-click');
        event.preventDefault();
    }, true);

このスクリプトを `data/widget` ディレクトリに「widget.js」という名前で保存します。

## ウィジェットのアイコン ##

ウィジェットのアイコンは、ここからコピーできます。

<img style="margin-left:40px; margin-right:20px;" src="static-files/media/annotator/pencil-on.png" alt="Active Annotator">
<img style="margin-left:20px; margin-right:20px;" src="static-files/media/annotator/pencil-off.png" alt="Inactive Annotator">

（もちろん、独自のアイコンを作成して使用することもできます。）アイコンファイルは `data/widget` ディレクトリに保存してください。

## main.js ##

次に `lib` ディレクトリで `main.js` を開き、内容を以下と置き換えます。

    var widgets = require('widget');
    var data = require('self').data;

    var annotatorIsOn = false;

    function toggleActivation() {
      annotatorIsOn = !annotatorIsOn;
      return annotatorIsOn;
    }

    exports.main = function() {

      var widget = widgets.Widget({
        id: 'toggle-switch',
        label: 'Annotator',
        contentURL: data.url('widget/pencil-off.png'),
        contentScriptWhen: 'ready',
        contentScriptFile: data.url('widget/widget.js')
      });

      widget.port.on('left-click', function() {
        console.log('activate/deactivate');
        widget.contentURL = toggleActivation() ?
                  data.url('widget/pencil-on.png') :
                  data.url('widget/pencil-off.png');
      });

      widget.port.on('right-click', function() {
          console.log('show annotation list');
      });
    }

アノテーターは、デフォルトで非アクティブです。アクティブ化の状態を切り替えることで、ウィジェットが作成され、ウィジェットのコンテンツスクリプトからのメッセージに対して応答が行われます。 

<span class="aside">[bug 626326](https://bugzilla.mozilla.org/show_bug.cgi?id=626326) のバグのために、ウィジェットのコンテンツスクリプトで `event.preventDefault()` 呼び出しても、アドオンバーのコンテキストメニューが表示されます。</span>

まだ注釈を表示するコードが存在しないので、右クリックイベントのみがコンソールにログとして出力されます。

ここで、`annotator` ディレクトリで `cfx run` を入力します。以下の図のように、アドオンバーにウィジェットが表示されます。

<div align="center">
<img src="static-files/media/annotator/widget-icon.png" alt="Widget Icon">
</div>
<br>

左右のボタンをクリックすると、それぞれに対応するデバッグ出力が生成されます。また左クリックした場合は、同時にウィジェットアイコンがアクティブに変化します。

次のチュートリアルでは、[注釈を作成](dev-guide/tutorials/annotator/creating.html)するコードを追加します。
