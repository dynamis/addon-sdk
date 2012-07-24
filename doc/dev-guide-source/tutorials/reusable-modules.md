<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# 再利用可能なモジュールの作成 #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

この SDK では、アドオンの全てのコードを「main.js」ファイルに書く必要はありません。1 つのコードを別個のモジュールに分割し、それぞれに明確なインターフェイスを定義することができます。その上で、アドオンの他の部分から `require()` 文によってこれらのモジュールをインポートして使用します。つまり、[`widget`](packages/addon-kit/widget.html) や [`panel`](packages/addon-kit/panel.html) などの SDK のコアモジュールをインポートする場合と同じ方法が使用できます。

多くの場合、大きなアドオンや複雑なアドオンは、モジュールの集まりとして構成する方が便利です。アドオンの設計がわかりやすくなる上、各モジュールでは選択した要素のみをエクスポートするという一種のカプセル化が可能になるので、ユーザーが使用できる状態を保ったままモジュールの内部を変更できます。

このように開発されたアドオンは、モジュールをパッケージ化し、アドオンから独立して配布できます。これにより、他のアドオン開発者がそのモジュールを利用できるとともに、SDK 自体が事実上拡張されることになります。

このチュートリアルでは、Firefox の位置情報 API を使用したモジュールにより、以上のことを正確に実現します。

## アドオンでの位置情報の使用 ##

[Firefox に内蔵の位置情報 API](https://developer.mozilla.org/en/using_geolocation) を使用する場合を考えてみましょう。SDK は位置情報にアクセスするための API を提供していませんが、[`require("chrome")` を使用して XPCOM API にアクセス](dev-guide/guides/xul-migration.html#xpcom)できます。

次のアドオンでは、[ツールバーにボタンが追加](dev-guide/tutorials/adding-toolbar-button.html) され、ユーザーがそのボタンをクリックすると、[XPCOM nsIDOMGeoGeolocation](https://developer.mozilla.org/en/XPCOM_Interface_Reference/NsIDOMGeoGeolocation) オブジェクトが読み込まれてユーザーの現在の位置が取得されます。

    var {Cc, Ci} = require("chrome");

    // Implement getCurrentPosition by loading the nsIDOMGeoGeolocation
    // XPCOM object.
    function getCurrentPosition(callback) {
      var xpcomGeolocation = Cc["@mozilla.org/geolocation;1"]
                          .getService(Ci.nsIDOMGeoGeolocation);
      xpcomGeolocation.getCurrentPosition(callback);
    }

    var widget = require("widget").Widget({
      id: "whereami",
      label: "Where am I?",
      contentURL: "http://www.mozilla.org/favicon.ico",
      onClick: function() {
        getCurrentPosition(function(position) {
          console.log("latitude: ", position.coords.latitude);
          console.log("longitude: ", position.coords.longitude);
        });
      }
    });

演習：

* 「whereami」という名前の新しいディレクトリを作成し、そのディレクトリに移動します。
* `cfx init` を実行します。
* 「lib/main.js」を開き、内容を上のコードと置き換えます。
* `cfx run` を実行した後、`cfx run` を再度実行します。

ボタンが、ブラウザウィンドウの一番下の「アドオンバー」に追加されます。

<img class="image-center" src="static-files/media/screenshots/widget-mozilla.png"
alt="Mozilla icon widget" />

ボタンをクリックすると、少ししてコンソールに以下のような出力が表示されます。

<pre>
info: latitude:  29.45799999
info: longitude:  93.0785269
</pre>

ここまでは順調です。しかし、MDN の位置情報ガイドによると、API を使用する前に [ユーザーに許可を求める](https://developer.mozilla.org/en/using_geolocation#Prompting_for_permission)必要があります。

そこで、MDN のページにある対応バージョンのコードを使用してアドオンを拡張します。

<pre><code>
var activeBrowserWindow = require("window-utils").activeBrowserWindow;
var {Cc, Ci} = require("chrome");

// Ask the user to confirm that they want to share their location.
// If they agree, call the geolocation function, passing the in the
// callback. Otherwise, call the callback with an error message.
function getCurrentPositionWithCheck(callback) {
  let pref = "extensions.whereami.allowGeolocation";
  let message = "whereami Add-on wants to know your location."
  let branch = Cc["@mozilla.org/preferences-service;1"]
               .getService(Ci.nsIPrefBranch2);
  if (branch.getPrefType(pref) === branch.PREF_STRING) {
    switch (branch.getCharPref(pref)) {
    case "always":
      return getCurrentPosition(callback);
    case "never":
      return callback(null);
    }
  }
  let done = false;

  function remember(value, result) {
    return function () {
      done = true;
      branch.setCharPref(pref, value);
      if (result) {
        getCurrentPosition(callback);
      }
      else {
        callback(null);
      }
    }
  }

  let self = activeBrowserWindow.PopupNotifications.show(
               activeBrowserWindow.gBrowser.selectedBrowser,
               "geolocation",
               message,
               "geo-notification-icon",
    {
      label: "Share Location",
      accessKey: "S",
      callback: function (notification) {
        done = true;
        getCurrentPosition(callback);
      }
    }, [
      {
        label: "Always Share",
        accessKey: "A",
        callback: remember("always", true)
      },
      {
        label: "Never Share",
        accessKey: "N",
        callback: remember("never", false)
      }
    ], {
      eventCallback: function (event) {
        if (event === "dismissed") {
          if (!done)
            callback(null);
          done = true;
          PopupNotifications.remove(self);
        }
      },
      persistWhileVisible: true
    });
}

// Implement getCurrentPosition by loading the nsIDOMGeoGeolocation
// XPCOM object.
function getCurrentPosition(callback) {
  var xpcomGeolocation = Cc["@mozilla.org/geolocation;1"]
                      .getService(Ci.nsIDOMGeoGeolocation);
  xpcomGeolocation.getCurrentPosition(callback);
}

var widget = require("widget").Widget({
  id: "whereami",
  label: "Where am I?",
  contentURL: "http://www.mozilla.org/favicon.ico",
  onClick: function() {
    getCurrentPositionWithCheck(function(position) {
      if (!position) {
        console.log("The user denied access to geolocation.");
      }
      else {
       console.log("latitude: ", position.coords.latitude);
       console.log("longitude: ", position.coords.longitude);
      }
    });
  }
});

</code></pre>

これでボタンをクリックすると、許可を求める通知ボックスが表示され、ユーザーの選択に応じて位置情報かエラーメッセージのいずれかがログとして出力されるようになりました。

しかし拡張したことでコードが大きく複雑になってしまいました。このアドオンに他の機能を追加しようとすると管理が困難になります。そこで、位置情報コードを別のモジュールに分割することにします。

## 別のモジュールの作成 ##

### `geolocation.js` の作成 ###

まず、「lib」に「geolocation.js」という名前の新しいファイルを作成し、ウィジェットのコード以外のすべてのコードを新しいファイルにコピーします。

次に、新しいファイルのどこかに、以下の行を追加します。

    exports.getCurrentPosition = getCurrentPositionWithCheck;

これにより、新しいモジュールのパブリックインターフェイスが定義されます。ここでは、ユーザーに許可を求め、ユーザーが同意した場合には現在の位置を取得する関数のみをエクスポートします。

これにより、「geolocation.js」は以下のようになります。

<pre><code>
var activeBrowserWindow = require("window-utils").activeBrowserWindow;
var {Cc, Ci} = require("chrome");

// Ask the user to confirm that they want to share their location.
// If they agree, call the geolocation function, passing the in the
// callback. Otherwise, call the callback with an error message.
function getCurrentPositionWithCheck(callback) {
  let pref = "extensions.whereami.allowGeolocation";
  let message = "whereami Add-on wants to know your location."
  let branch = Cc["@mozilla.org/preferences-service;1"]
               .getService(Ci.nsIPrefBranch2);
  if (branch.getPrefType(pref) === branch.PREF_STRING) {
    switch (branch.getCharPref(pref)) {
    case "always":
      return getCurrentPosition(callback);
    case "never":
      return callback(null);
    }
  }
  let done = false;

  function remember(value, result) {
    return function () {
      done = true;
      branch.setCharPref(pref, value);
      if (result) {
        getCurrentPosition(callback);
      }
      else {
        callback(null);
      }
    }
  }

  let self = activeBrowserWindow.PopupNotifications.show(
               activeBrowserWindow.gBrowser.selectedBrowser,
               "geolocation",
               message,
               "geo-notification-icon",
    {
      label: "Share Location",
      accessKey: "S",
      callback: function (notification) {
        done = true;
        getCurrentPosition(callback);
      }
    }, [
      {
        label: "Always Share",
        accessKey: "A",
        callback: remember("always", true)
      },
      {
        label: "Never Share",
        accessKey: "N",
        callback: remember("never", false)
      }
    ], {
      eventCallback: function (event) {
        if (event === "dismissed") {
          if (!done)
            callback(null);
          done = true;
          PopupNotifications.remove(self);
        }
      },
      persistWhileVisible: true
    });
}

// Implement getCurrentPosition by loading the nsIDOMGeoGeolocation
// XPCOM object.
function getCurrentPosition(callback) {
  var xpcomGeolocation = Cc["@mozilla.org/geolocation;1"]
                      .getService(Ci.nsIDOMGeoGeolocation);
  xpcomGeolocation.getCurrentPosition(callback);
}

exports.getCurrentPosition = getCurrentPositionWithCheck;
</code></pre>

### `main.js` の更新 ###

最後に、"main.js" を更新します。まず、新しいモジュールをインポートするための行を追加します。

    var geolocation = require("./geolocation");

SDK の内蔵モジュール以外のモジュールをインポートする場合、可能であればモジュールローダーによって対象のモジュールを検索するのではなく、上のように明示的にモジュールのパスを指定してください。

ウィジェットの `getCurrentPositionWithCheck()` への呼び出しを変更して、代わりに `getCurrentPosition()` 関数が呼び出されるように指定します。

    geolocation.getCurrentPosition(function(position) {
      if (!position) {

これにより、「geolocation.js」は以下のようになります。

<pre><code>
var geolocation = require("./geolocation");

var widget = require("widget").Widget({
  id: "whereami",
  label: "Where am I?",
  contentURL: "http://www.mozilla.org/favicon.ico",
  onClick: function() {
    geolocation.getCurrentPosition(function(position) {
      if (!position) {
        console.log("The user denied access to geolocation.");
      }
      else {
       console.log("latitude: ", position.coords.latitude);
       console.log("longitude: ", position.coords.longitude);
      }
    });
  }
});
</code></pre>

## 位置情報モジュールのパッケージ化 ##

これまで見てきたように、モジュール化はアドオンを構成する上で便利なテクニックです。モジュールはさらにパッケージ化して、アドオンと別個に配布することもできます。これにより、他のアドオン開発者がそのモジュールをダウンロードして、SDK の内蔵モジュールとまったく同じように使用できるようになります。

### コードの変更 ###

まず、コードを少し変更します。現在、プロンプトに表示されるメッセージおよびユーザーの選択内容の格納に使用されるプリファレンス名は、以下のようにハードコードされています。

    let pref = "extensions.whereami.allowGeolocation";
    let message = "whereami Add-on wants to know your location."

これを以下のように変更し、`self` モジュールを使用して、それらのプリファレンス名がアドオンに固有になるようにします。

    var addonName = require("self").name;
    var addonId = require("self").id;
    let pref = "extensions." + addonId + ".allowGeolocation";
    let message = addonName + " Add-on wants to know your location."

### 再パッケージ化 ###

次に、位置情報モジュールを再パッケージ化します。

* 「geolocation」という名前の新しいディレクトリを作成し、そのディレクトリで `cfx init` を実行します。
* `cfx` によって生成された「main.js」を削除し、ここで作成した「geolocation.js」をコピーします。

### ドキュメントの作成 ###

パッケージとそれに含まれるモジュールに関するドキュメントを作成しておけば、他の開発者がそのパッケージをインストールし、`cfx docs` を実行したときに SDK 自体のドキュメントと統合されて表示されます。

位置情報モジュールを含むパッケージに関するドキュメントは、パッケージルートで `cfx init` によって生成された「README.md」ファイルを編集して作成することができます。このドキュメントは、[Markdown](http://daringfireball.net/projects/markdown/syntax) 形式です。

位置情報モジュール自体のドキュメントを用意するには、パッケージの「doc」ディレクトリに「geolocation.md」というファイルを作成します。このファイルも Markdown 形式で記述されますが、必要に応じて、API の記述に[拡張構文](https://wiki.mozilla.org/Jetpack/SDK/Writing_Documentation#APIDoc_Syntax)を使用することができます。

演習：

* 「README.md」を編集し、「doc」の下に「geolocation.md」を追加します。
* SDK ルートの「packages」ディレクトリの下に、作成した位置情報パッケージをコピーします。
* `cfx docs` を実行します。

`cfx docs` の実行が完了すると、「Third-Party APIs」というサイドバーに、位置情報パッケージとそれに含まれるモジュールが一覧表示されます。

### 「package.json」の編集 ###

パッケージのルートディレクトリの「package.json」ファイルには、パッケージ用のメタデータが格納されています。詳細については、[パッケージ仕様（英語）](dev-guide/package-spec.html) を参照してください。パッケージを配布する場合は、ここで作成者の名前を追加したり、配布ライセンスを選択したりするとよいでしょう。

## さらに詳しく ##

他の開発者が作成したモジュールの例については、[コミュニティ開発モジュール（英語）](https://github.com/mozilla/addon-sdk/wiki/Community-developed-modules) のページを参照してください。開発中のコードでサードパーティ製のモジュールを使用する方法の詳細については、[メニューアイテムの追加についてのチュートリアル](dev-guide/tutorials/adding-menus.html)を参照してください。
