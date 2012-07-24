<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# 単体テスト #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)および[再利用可能なモジュールの作成](dev-guide/tutorials/reusable-modules.html)のチュートリアルを学習してください。
</span>

SDK では、コードの単体テストの作成と実行を支援するフレームワークが提供されています。ここでは、このフレームワークの機能を理解するために、単純な [Base64](http://en.wikipedia.org/wiki/Base64) エンコードモジュール用の単体テストを作成します。

## 単純な Base64 モジュール ##

Web ページでは、`btoa()` 関数と `atob()` 関数を使用して、Base64 のエンコードとデコードを実行できます。しかし残念なことにこれらの関数は `window` オブジェクトに付加され、`window` オブジェクトはメインのアドオンコードで使用できないため、`atob()` と `btoa()` もメインのアドオンコードで使用できません。低レベルの [window-utils](packages/api-utils/window-utils.html) モジュールを使用して、`window` にアクセスし、これらの関数を呼び出すことは可能です。

しかし、`window-utils` に直接アクセスするコードを別のモジュールとしてカプセル化し、`atob()` 関数と `btoa()` 関数のみをエクスポートする方が適切な方法です。そこでここでは、それを行う Base64 モジュールを作成します。

まず新しいディレクトリを作成し、そのディレクトリに移動して `cfx init` を実行します。「lib」に「base64.js」という名前の新しいファイルを作成し、以下の内容をコピーします。

    var window = require("window-utils").activeBrowserWindow;

    exports.atob = function(a) {
      return window.atob(a);
    }

    exports.btoa = function(b) {
      return window.btoa(b);
    }

このコードは 2 つの関数をエクスポートし、それらの関数が `window` オブジェクトの対応する関数を呼び出します。モジュールの動作を確認するため、「main.js」ファイルを以下のように編集してください。

    var widgets = require("widget");
    var base64 = require("base64");

    var widget = widgets.Widget({
      id: "base64",
      label: "Base64 encode",
      contentURL: "http://www.mozilla.org/favicon.ico",
      onClick: function() {
        encoded = base64.btoa("hello");
        console.log(encoded);
        decoded = base64.atob(encoded);
        console.log(decoded);
      }
    });

これで「main.js」は base64 モジュールをインポートし、それがエクスポートする 2 つの関数を呼び出すようになりました。このアドオンを実行し、ウィジェットをクリックすると、以下のログ出力が表示されます。

<pre>
info: aGVsbG8=
info: hello
</pre>

## 作成した Base64 モジュールのテスト ##

アドオンの `test` ディレクトリに移動し、`test-main.js` ファイルを削除します。このファイルの代わりに `test-base64.js` というファイルを作成し、以下の内容をコピーします。

<pre><code>
var base64 = require("base64")

function test_atob(test) {
  test.assertEqual(base64.atob("aGVsbG8="), "hello");
  test.done();
}

function test_btoa(test) {
  test.assertEqual(base64.btoa("hello"), "aGVsbG8=");
  test.done();
}

function test_empty_string(test) {
  test.assertRaises(function() {
    base64.atob();
  },
  "String contains an invalid character");
};

exports.test_atob = test_atob;
exports.test_btoa = test_btoa;
exports.test_empty_string = test_empty_string;
</code></pre>

このファイルでは 3 つの関数が呼び出され、それぞれが `test` オブジェクトである引数を 1 つずつ受け取ります。`test` は [`unit-test`](packages/api-utils/unit-test.html) モジュールによって提供され、単体テストを簡単に実行するための関数を提供します。

* 最初の 2 つの関数は `atob()` と `btoa()` を呼び出し、[`test.assertEqual()`](packages/api-utils/unit-test.html#assertEqual(a, b, message)) を使用して予期したとおりの出力が得られることを確認します。

* 3 番目の test_empty_string 関数は、モジュールのエラー処理コードのテストとして、空の文字列を `atob()` に渡し、[`test.assertRaises()`](packages/api-utils/unit-test.html#assertRaises(func, predicate, message)) を使用して予期したとおりに例外が発生することを確認します。

この時点で、アドオンは以下のようになります。

<pre>
  /base64
      package.json
      README.md
      /doc
          main.md
      /lib
          main.js
          base64.js
      /test
          test-base64.js
</pre>

ここでアドオンのルートディレクトリから、`cfx --verbose test` を実行します。以下のような出力が表示されます。

<pre>
Running tests on Firefox 10.0/Gecko 10.0 ({ec8030f7-c20a-464f-9b0e-13a3a9e97384}) under darwin/x86.
info: executing 'test-base64.test_atob'
info: pass: a == b == "hello"
info: executing 'test-base64.test_btoa'
info: pass: a == b == "aGVsbG8="
info: executing 'test-base64.test_empty_string'
info: pass: a == b == "String contains an invalid character"

3 of 3 tests passed.
Total time: 1.691787 seconds
Program terminated successfully.
</pre>

ここでの `cfx test` の動作は以下のとおりです。

<span class="aside">モジュール名の「test」の後にハイフンが付いていることに注意してください。例えば「test-myCode.js」というモジュールは `cfx test` に読み込まれますが、「test_myCode.js」や「testMyCode.js」は読み込まれません。</span>

* パッケージの `test` ディレクトリを調べます。
* 名前が `test-` で始まるすべてのモジュールを読み込みます。
*  それらのモジュールによってエクスポートされるすべての関数を呼び出します。このとき、`test` オブジェクトの実装を唯一の引数として渡します。

もちろん、`--verbose` オプションを `cfx` に渡すことは必須ではありませんが、このオプションを使用すると出力がわかりやすくなります。
