<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# 注釈の作成 #

ここでは注釈を作成するためのオブジェクトとして、注釈可能なページ内要素を検索する page-mod と、ユーザーが実際に注釈テキスト自体を入力するためのパネルを作成します。

## セレクタ page-mod ##

### セレクタコンテンツスクリプト ###

セレクタ page-mod のコンテンツスクリプトは、[jQuery](http://jquery.com/) を使用して DOM の調査と操作を行います。

このコンテンツスクリプトの主要な役割は「一致要素」、つまり現在の注釈対象候補であるページ内要素を保持することです。以下のコードでは、一致要素をハイライトし、クリックされればメインのアドオンコードにメッセージを送信するハンドラがバインドされます。

セレクタ page mod は、メインのアドオンコードからのメッセージによってオンとオフを切り替えることができます。初期状態はオフです。

    var matchedElement = null;
    var originalBgColor = null;
    var active = false;

    function resetMatchedElement() {
      if (matchedElement) {
        $(matchedElement).css('background-color', originalBgColor);
        $(matchedElement).unbind('click.annotator');
      }
    }

    self.on('message', function onMessage(activation) {
      active = activation;
      if (!active) {
        resetMatchedElement();
      }
    });

このセレクタは、[jQuery mouseenter](http://api.jquery.com/mouseenter/) イベントの発生をリッスンします。

mouseenter イベントが発生すると、セレクタはその要素が注釈可能かどうかを確認します。その要素自体、または DOM ツリーにあるその要素の先祖のいずれかが、「id」という属性を持っている場合、その要素は注釈可能です。このようなチェックは、ユーザーが注釈を付けた要素をアノテーターが後から正確に識別できるようにするために行われます。

ページ内要素が注釈可能な場合、セレクタはその要素をハイライトし、クリックハンドラがその要素にバインドされます。クリックハンドラは、`show` というメッセージをメインのアドオンコードに送信します。`show` メッセージには、ページの URL、ID 属性値、およびページ内要素のテキストコンテンツが含まれています。

    $('*').mouseenter(function() {
      if (!active || $(this).hasClass('annotated')) {
        return;
      }
      resetMatchedElement();
      ancestor = $(this).closest("[id]");
      matchedElement = $(this).first();
      originalBgColor = $(matchedElement).css('background-color');
      $(matchedElement).css('background-color', 'yellow');
      $(matchedElement).bind('click.annotator', function(event) {
        event.stopPropagation();
        event.preventDefault();
        self.port.emit('show',
          [
            document.location.toString(),
            $(ancestor).attr("id"),
            $(matchedElement).text()
          ]
       );
      });
    });

逆に、[mouseout](http://api.jquery.com/mouseout/) が発生すると、アドオンによって一致要素がリセットされます。

    $('*').mouseout(function() {
      resetMatchedElement();
    });

アドオンの `data` ディレクトリに `selector.js` という新しいファイルを作成し、このコードを保存します。

このコードは jQuery を使用するので、jQuery もあわせて[ダウンロード](http://docs.jquery.com/Downloading_jQuery) し、`data` に保存します。

### main.js の更新 ###

`main.js` に戻り、セレクタを作成するコードを `main` 関数に追加します。

    var selector = pageMod.PageMod({
      include: ['*'],
      contentScriptWhen: 'ready',
      contentScriptFile: [data.url('jquery-1.4.2.min.js'),
                          data.url('selector.js')],
      onAttach: function(worker) {
        worker.postMessage(annotatorIsOn);
        selectors.push(worker);
        worker.port.on('show', function(data) {
          console.log(data);
        });
        worker.on('detach', function () {
          detachWorker(this, selectors);
        });
      }
    });

jQuery の読み込みに使用する名前が、ダウンロードした jQuery のバージョンの名前と一致していることを確認してください。

この page-mod はすべてのページに一致するので、ユーザーがページを読み込むたびに page-mod で `attach` イベントが発生し、それによって、`onAttach` に割り当てたリスナー関数が呼び出されます。このハンドラには [worker](packages/api-utils/content/worker.html) オブジェクトが渡されます。各ワーカーは、アドオンコードとその特定のページコンテキストで実行されているコンテンツスクリプトの間の通信チャネルを表します。`page-mod` によるワーカーの使用方法の詳細については、[page-mod のドキュメント（英語）](packages/addon-kit/page-mod.html)を参照してください。.

アタッチハンドラでは、以下の 3 つの作業を行います。

* 現在のアクティブ状態を伝えるメッセージをコンテンツスクリプトに送信します。
* 後からワーカー宛にメッセージを送信できるように、ワーカーを `selectors` という配列に追加します。
* このワーカーからのメッセージに対して、メッセージハンドラを割り当てます。メッセージが `show` の場合、当面、コンテンツのログのみを記録します。メッセージが `detach` の場合、`selectors` 配列からワーカーを削除します。

ファイルの一番上で、`page-mod` モジュールをインポートし、ワーカーの配列を宣言します。

    var pageMod = require('page-mod');
    var selectors = [];

`detachWorker` を追加します。

    function detachWorker(worker, workerArray) {
      var index = workerArray.indexOf(worker);
      if(index != -1) {
        workerArray.splice(index, 1);
      }
    }

`toggleActivation` を以下のように編集します。これにより、ワーカーにアクティブ状態の変化が通知されます。

    function activateSelectors() {
      selectors.forEach(
        function (selector) {
          selector.postMessage(annotatorIsOn);
      });
    }

    function toggleActivation() {
      annotatorIsOn = !annotatorIsOn;
      activateSelectors();
      return annotatorIsOn;
    }

<span class="aside">このチュートリアルでは、これ以降のスクリーンショットに一貫して下に示すページを使用します。`cfx run` は閲覧履歴を保存しないので、他のページに移動する場合は、その前にこの URL を記録しておくことをお勧めします。</span>

ファイルを保存し、`cfx run` をもう一度実行します。ウィジェットをクリックしてアノテーターを起動し、ページを読み込みます。下のスクリーンショットでは、[http://blog.mozilla.com/addons/2011/02/04/ overview-amo-review-process/](http://blog.mozilla.com/addons/2011/02/04/overview-amo-review-process/) が使用されています。
マウスを移動して特定の要素に合わせると、ハイライトが表示されます。

<img class="image-center"
src="static-files/media/annotator/highlight.png" alt="Annotator Highlighting">

ハイライトをクリックすると、コンソール出力に以下のように表示されます。

<pre>
  info: show
  info: http://blog.mozilla.com/addons/2011/02/04/overview-amo-review-process/,
  post-2249,When you submit a new add-on, you will have to choose between 2
  review tracks: Full Review and Preliminary Review.
</pre>

## 注釈エディタパネル ##

ここまでの作業で、要素をハイライトし、それらの要素に関する情報をメインのアドオンコードに送信する page-mod を作成しました。次に、エディタパネルを作成します。これは選択された要素についてユーザーが注釈を入力するためのパネルです。

このチュートリアルでは、パネルのコンテンツが HTML ファイルで提供されます。またパネルのコンテキストで実行するコンテンツスクリプトも提供されます。

`data` の下に `editor` という名前のサブディレクトリを作成してください。このサブディレクトリには、以下で作成する HTML コンテンツファイルとコンテンツスクリプトファイルの 2 つを格納します。

### 注釈エディタ HTML ###

この HTML は非常に単純です。

<script type="syntaxhighlighter" class="brush: html"><![CDATA[
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
<head>
  <title>Annotation</title>
  <style type="text/css" media="all">
  body {
    font: 100% arial, helvetica, sans-serif;
    background-color: #F5F5F5;
  }
  textarea {
    width: 180px;
    height: 180px;
    margin: 10px;
    padding: 0px;
  }
  </style>

</head>

<body>

<textarea rows='10' cols='20' id='annotation-box'>
</textarea>

</body>

</html>

]]>
</script>

これを `data/editor` の中に `annotation-editor.html` という名前で保存します。

### 注釈エディタのコンテンツスクリプト ###

対応するコンテンツスクリプトでは、次の 2 つの処理を行います。

* アドオンコードからのメッセージに対する処理として、テキスト領域をフォーカスします。
* リターンキーをリッスンし、リターンキーが押されたときにテキスト領域のコンテンツをアドオンに送信します。

コードは以下のとおりです。

    var textArea = document.getElementById('annotation-box');

    textArea.onkeyup = function(event) {
      if (event.keyCode == 13) {
        self.postMessage(textArea.value);
        textArea.value = '';
      }
    };

    self.on('message', function() {
      var textArea = document.getElementById('annotation-box');
      textArea.value = '';
      textArea.focus();
    });


これを `data/editor` の中に `annotation-editor.js` という名前で保存します。

### main.js の更新 ###

ここで再度 `main.js` を更新してエディタを作成し、それを使用します。

まず、`panel` モジュールをインポートします。

    var panels = require('panel');

次に、`main` 関数に以下のコードを追加します。

    var annotationEditor = panels.Panel({
      width: 220,
      height: 220,
      contentURL: data.url('editor/annotation-editor.html'),
      contentScriptFile: data.url('editor/annotation-editor.js'),
      onMessage: function(annotationText) {
        if (annotationText) {
          console.log(this.annotationAnchor);
          console.log(annotationText);
        }
        annotationEditor.hide();
      },
      onShow: function() {
        this.postMessage('focus');
      }
    });

ここではエディタパネルを作成しますが、表示はしないでください。エディタパネルが表示されると、`focus` メッセージがエディタパネルに送信され、テキスト領域がフォーカスされます。エディタパネルからメッセージが送られると、メッセージがログに記録されてパネルが非表示になります。

最後に残った作業として、エディタをセレクタにリンクします。これには、セレクタに割り当てられたメッセージハンドラを以下のように編集します。これにより、`show` メッセージを受信したときに、新しいプロパティ「annotationAnchor」を使用してメッセージのコンテンツがパネルに割り当てられ、パネルが表示されます。

    var selector = pageMod.PageMod({
      include: ['*'],
      contentScriptWhen: 'ready',
      contentScriptFile: [data.url('jquery-1.4.2.min.js'),
                          data.url('selector.js')],
      onAttach: function(worker) {
        worker.postMessage(annotatorIsOn);
        selectors.push(worker);
        worker.port.on('show', function(data) {
          annotationEditor.annotationAnchor = data;
          annotationEditor.show();
        });
        worker.on('detach', function () {
          detachWorker(this, selectors);
        });
      }
    });

`cfx run` をもう一度実行し、アノテーターを起動します。要素にマウスを合わせ、ハイライトされたらその要素をクリックします。以下のように注釈入力用のテキスト領域を持つパネルが表示されます。

<img class="image-center"
src="static-files/media/annotator/editor-panel.png" alt="Annotator Editor Panel">
<br>

注釈を入力してリターンキーを押します。以下のようなコンソール出力が表示されます。

<pre>
  info: http://blog.mozilla.com/addons/2011/02/04/overview-amo-review-process/,
  post-2249,When you submit a new add-on, you will have to choose between 2
  review tracks: Full Review and Preliminary Review.
  info: We should ask for Full Review if possible.
</pre>

これが完全な注釈です。次のセクションでは、[注釈の保存](dev-guide/tutorials/annotator/storing.html)を行います。
