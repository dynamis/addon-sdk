<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# アドオンのロードとアンロードのリッスン #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

## exports.main() ##

アドオンの `main.js` コードは、アドオンが読み込まれるとすぐに実行されます。`main.js` コードは、インストールされたとき、有効化されたとき、または Firefox が起動したときに読み込まれます。

アドオンが `main()` という関数をエクスポートする場合、`main.js` 全体が評価され、すべての最上位レベルの `require()` 文が実行されると（したがって、通常、すべての従属モジュールが読み込まれると）すぐにその関数が呼び出されます。

    exports.main = function (options, callbacks) {};

`options` は、アドオンのロードに使用したパラメータを示すオブジェクトです。

### options.loadReason ###

`options.loadReason` は、アドオンが読み込まれた理由を示す以下のいずれかの文字列です。 

<pre>
install
enable
startup
upgrade
downgrade
</pre>

### options.staticArgs ###

[`cfx`](dev-guide/cfx-tool.html) `--static-args` オプションを使用すると、任意のデータをプログラムに渡すことができます。

`--static-args` の値には、JSON 文字列を指定します。JSON によってエンコードされたオブジェクトは、`options` オブジェクトの `staticArgs` メンバーとなり、最初の引数としてプログラムの `main` 関数に渡されます。`--static-args` のデフォルトの値は `「{}」`（空のオブジェクト）なので、`options` に `staticArgs` が存在するかどうかを確認する必要はありません。

例えば、`main.js` が以下のような場合、

    exports.main = function (options, callbacks) {
      console.log(options.staticArgs.foo);
    };

以下のように cfx を実行すると、

<pre>
  cfx run --static-args="{ ¥"foo¥": ¥"Hello from the command line¥" }"
</pre>

コンソールに、以下のように表示されます。

<pre>
info: Hello from the command line
</pre>

`--static-args` オプションは、`cfx run` と `cfx xpi` のいずれによっても認識されます。`cfx xpi` で使用した場合、JSON が XPI の harness options を伴ってパッケージ化されるので、XPI 内のプログラムを実行するたびに使用されるようになります。

## exports.onUnload() ##

アドオンが `onUnload()` という関数をエクスポートする場合、その関数はアドオンがアンロードされたときに呼び出されます。

    exports.onUnload = function (reason) {};

`reason` は、アドオンがアンロードされた理由を示す以下のいずれかの文字列です。

<span class="aside">ただし、[bug 627432](https://bugzilla.mozilla.org/show_bug.cgi?id=627432) のバグのため、`onUnload` リスナーが `uninstall` を理由として呼び出されることはなく、`disable` でのみ呼び出されます。特に[そのバグのコメント 12](https://bugzilla.mozilla.org/show_bug.cgi?id=627432#c12) を参照してください。</span>

<pre>
uninstall
disable
shutdown
upgrade
downgrade
</pre>

`exports.main()` や `exports.onUnload()` の使用は必須ではありません。関数でラップして `exports.main()` に割り当てなくても、アドオンのコードを最上位レベルに置くだけで、同じように読み込まれます。ただしその場合、`options` 引数や `callbacks` 引数にアクセスすることはできません。
