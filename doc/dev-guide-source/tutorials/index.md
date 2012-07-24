<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

# チュートリアル #

ここでは、SDK を使用したアドオンの開発方法を実践的に説明したページを一覧します。ハイレベル API の中には、チュートリアルが用意されていない API もあります。すべての API のリストについては、ページ左側のサイドバーを参照してください。

<hr>

<h2><a name="getting-started">はじめに</a></h2>

<table class="catalog">
<colgroup>
<col width="50%">
<col width="50%">
</colgroup>
  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/installation.html">インストール</a></h4>
      Windows、OS X および Linux 上で、SDK をダウンロード、インストール、および初期化します。
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/installation.html">cfx 入門</a></h4>
      アドオン作成を始めるために必要な基本の <code>cfx</code> コマンドを学習します。
    </td>

  </tr>
  <tr>

    <td>
      <h4><a href="dev-guide/tutorials/installation.html">トラブルシューティング</a></h4>
      よくある問題を解決する場合や、支援を求める場合のヒントを説明します。
    </td>

    <td>
    </td>

  </tr>

</table>

<hr>

<h2><a name="create-user-interfaces">ユーザーインターフェイスの作成</a></h2>

<table class="catalog">
<colgroup>
<col width="50%">
<col width="50%">
</colgroup>
  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/adding-toolbar-button.html">ツールバーボタンの追加</a></h4>
      Firefox アドオンツールバーにボタンを追加します。
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/display-a-popup.html">ポップアップの表示</a></h4>
      HTML および JavaScript を使用して実装したポップアップダイアログを表示します。
    </td>

  </tr>

  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/adding-menus.html">Firefox へのメニューアイテムの追加</a></h4>
      Firefox のメインメニューにアイテムを追加します。
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/add-a-context-menu-item.html">コンテキストメニューアイテムの追加</a></h4>
      Firefox のコンテキストメニューにアイテムを追加します。
    </td>

  </tr>

</table>

<hr>

<h2><a name="interact-with-the-browser">ブラウザの操作</a></h2>

<table class="catalog">
<colgroup>
<col width="50%">
<col width="50%">
</colgroup>
  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/open-a-web-page.html">Web ページを開く</a></h4>
      <code><a href="packages/addon-kit/tabs.html">tabs</a></code> モジュールを使用して、新しいブラウザタブまたはウィンドウでWebページを開き、そのコンテンツにアクセスします。
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/list-open-tabs.html">開いているタブの一覧表示</a></h4>
      <code><a href="packages/addon-kit/tabs.html">tabs</a></code> モジュールを使用して、現在開いているすべてのタブを反復処理し、それらのコンテンツにアクセスします。
    </td>

  </tr>

  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/listen-for-page-load.html">ページ読み込みのリッスン</a></h4>
      <code><a href="packages/addon-kit/tabs.html">tabs</a></code> モジュールを使用して、新しい Web ページが読み込まれたときに通知を受け取り、それらの Web ページのコンテンツにアクセスします。
    </td>

    <td>
    </td>

  </tr>

</table>

<hr>

<h2><a name="modify-web-pages">Web ページの変更</a></h2>

<table class="catalog">
<colgroup>
<col width="50%">
<col width="50%">
</colgroup>
  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/modifying-web-pages-url.html">URL に基づいた Web ページの変更</a></h4>
      URL に基づいて Web ページを検索するフィルタを作成し、フィルタに一致する URL の Web ページを読み込んだときに、フィルタ内の指定したスクリプトを実行します。
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/modifying-web-pages-tab.html">アクティブな Web ページの変更</a></h4>
      現在アクティブな Web ページに、動的にスクリプトを読み込みます。
    </td>

  </tr>

</table>

<hr>

<h2><a name="development-techniques">開発テクニック</a></h2>

<table class="catalog">
<colgroup>
<col width="50%">
<col width="50%">
</colgroup>
  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/logging.html">ログとして出力</a></h4>
      診断を行うために、メッセージをコンソールにログとして出力します。
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/load-and-unload.html">読み込みと読み込み解除のリッスン</a></h4>
      Firefox にアドオンが読み込まれたり、読み込み解除されたりしたときに通知を受け取ります。またコマンドラインからアドオンに引数を渡します。
    </td>

  </tr>

  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/reusable-modules.html">サードパーティモジュールの作成</a></h4>
      アドオンを別個のモジュールとして作成して、開発、デバッグ、および保守を容易にします。
      またモジュールが入った再利用可能なパッケージを作成して、他の開発者もそのモジュールを使用できるようにします。
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/adding-menus.html">サードパーティモジュールの使用</a></h4>
      SDK 自体に含まれていない追加のモジュールをインストールして使用します。
    </td>

  </tr>

  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/unit-testing.html">単体テスト</a></h4>
      SDK のテストフレームワークを使用して、単体テストを作成し実行します。
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/l10n.html">ローカリゼーション</a></h4>
      ローカライズ可能なコードを作成します。
    </td>

  </tr>

  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/chrome.html">Chrome 権限</a></h4>
      この権限を使用すると、アドオンが <a href="https://developer.mozilla.org/en/Components_object">Components</a> オブジェクトにアクセスできるので、どんな XPCOM オブジェクトでも読み込んで使用できるようになります。
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/mobile.html">モバイル開発</a></h4>
      Android 用 Firefox モバイルのアドオン開発を始める手順を説明します。
    </td>

  </tr>

</table>

<hr>

<h2><a name="putting-it-together">応用</a></h2>

<table class="catalog">
<colgroup>
<col width="50%">
<col width="50%">
</colgroup>
  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/annotator/index.html">アノテーターアドオン</a></h4>
      より複雑なアドオンの開発作業を順を追って説明します。
    </td>

    <td>
    </td>

  </tr>

</table>

