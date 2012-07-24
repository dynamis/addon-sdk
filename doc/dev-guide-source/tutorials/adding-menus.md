<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# Firefox へのメニューアイテムの追加 #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

この SDK では、Firefox に新しいメニューアイテムを追加する API がまだ提供されていません。しかしこの SDK は拡張性に優れ、誰もがアドオン開発者向けのモジュールを作成して公開できるように設計されています。幸い、メニューアイテムの追加については、Erik Vold 氏の作成による [`menuitems`](https://github.com/erikvold/menuitems-jplib) パッケージが利用できます。

このチュートリアルでは、外部のサードパーティパッケージをアドオンで使用する一般的な方法と、特に `menuitems` パッケージを使用してメニューアイテムを追加する方法の両方について説明します。

## `menuitems` のインストール##

まず `menuitems` を [https://github.com/erikvold/menuitems-jplib](https://github.com/erikvold/menuitems-jplib/zipball/51080383cbb0fe2a05f8992a8aae890f4c014176) からダウンロードします。

次に、SDK の `packages` ディレクトリ上で `menuitems` を展開します。

<pre>
cd packages
tar -xf ../erikvold-menuitems-jplib-d80630c.zip
</pre>

ここで `cfx docs` を実行すると、サイドバーに「Third-Party APIs」という新しいセクションが追加され、`menuitems` パッケージが表示されます。パッケージに含まれるモジュールは、その下に表示されます。`menuitems` には、同じく `menuitems` という名前のモジュールが 1 つだけ入っていることがわかります。

モジュール名をクリックすると、そのモジュールの ＡＰＩ ドキュメントが表示されます。またパッケージ名をクリックすると、そのパッケージの基本的なドキュメントが表示されます。

パッケージページに記載されている重要事項の 1 つに、以下のようなパッケージの依存関係があります。

<pre>
Dependencies             api-utils, vold-utils
</pre>

この記述は、このパッケージを使用するためには、`vold-utils` パッケージ（[https://github.com/erikvold/vold-utils-jplib](https://github.com/voldsoftware/vold-utils-jplib/zipball/1b2ad874c2d3b2070a1b0d43301aa3731233e84f) からダウンロードできます）をインストールし、`packages` ディレクトリの下に `menuitems` とともに保存する必要があることを示しています。

## `menuitems` の使用 ##

`menuitems` の使用方法は、内蔵モジュールの使用方法とまったく同じです。

`menuitems` モジュールのドキュメントの記述によると、メニューアイテムの作成には `MenuItem()` を使用します。`MenuItem()` のオプションの中から、ここでは以下の最小セットを使用します。

* `id`：このメニューアイテムの識別子（固有名）
* `label`：このメニューアイテムに表示されるテキスト
* `command`：ユーザーがこのメニューアイテムを選択したときに呼び出される関数
* `menuid`：このメニューアイテムの親要素の識別子
* `insertbefore`：このメニューアイテムの前に表示されるアイテムの識別子

次に、新しいアドオンを作成します。任意の場所に 'clickme' という名前のディレクトリを作成し、そのディレクトリに移動して `cfx init` を実行します。`lib/main.js` を開き、内容を以下と置き換えます。

    var menuitem = require("menuitems").Menuitem({
      id: "clickme",
      menuid: "menu_ToolsPopup",
      label: "Click Me!",
      onCommand: function() {
        console.log("clicked");
      },
      insertbefore: "menu_pageInfo"
    });

次に、 `menuitems` パッケージに対する依存関係を宣言する必要があります。 
アドオンの `package.json` に、次の行を追加します。

<pre>
"dependencies": "menuitems"
</pre>

[bug 663480](https://bugzilla.mozilla.org/show_bug.cgi?id=663480) のバグのため、`package.json` に `dependencies` 行を追加し、[`addon-kit`](packages/addon-kit/index.html) などの内蔵パッケージにあるモジュールを使用する場合でも、以下のように依存関係を宣言する必要があることに注意してください。

<pre>
"dependencies": ["menuitems", "addon-kit"]
</pre>

これで作業は完了です。アドオンを実行すると、「ツール」メニューに新しいアイテムが表示されます。そのアイテムを選択すると、コンソールに `info: clicked` と表示されます。

## 注意 ##

Mozilla は、サードパーティ製パッケージの豊富さが、いずれはこの ＳＤＫの大きな魅力の 1 つになると考えています。現在これらのパッケージは、面倒な低レベル API を使用せずに、現行の API では対応できない機能を実現する優れた方法です。しかし同時に、以下の点に注意する必要があります。

* Mozilla によるサードパーティ製パッケージのサポートは、まだ万全とはいえません。その一例として、SDK の GitHub Wiki にある [Community Developed Modules](https://github.com/mozilla/addon-sdk/wiki/Community-developed-modules) のページにパッケージの一覧が用意されてはいますが、サードパーティ製パッケージが容易に見つからないこともあります。

* サードパーティ製パッケージでは、通常、低レベル API が使用されているので、Firefox の新規リリースによって機能しなくなることがあります。
