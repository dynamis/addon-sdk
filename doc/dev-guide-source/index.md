<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

<h2 class="top">Add-on SDK 開発者ガイド</h2>

Using the Add-on SDK you can create Firefox add-ons using standard Web
technologies: JavaScript, HTML, and CSS. The SDK includes JavaScript APIs which you can use to create add-ons, and tools for creating, running, testing, and packaging add-ons.

<hr>

## <a href="dev-guide/tutorials/index.html"> チュートリアル </a> ##

<table class="catalog">
<colgroup>
<col width="50%">
<col width="50%">
</colgroup>
  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/index.html#getting-started">はじめに</a></h4>
      How to
      <a href="dev-guide/tutorials/installation.html">SDK のインストール</a> and
      <a href="dev-guide/tutorials/getting-started-with-cfx.html">cfx ツール入門</a> to develop, test, and package add-ons.
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/index.html#create-user-interfaces">ユーザーインターフェイスの作成</a></h4>
      Create user interface components such as
        <a href="dev-guide/tutorials/adding-toolbar-button.html">ツールバーボタン</a>,
        <a href="dev-guide/tutorials/adding-menus.html">メニューアイテム</a>, and
        <a href="dev-guide/tutorials/display-a-popup.html">ダイアログ</a>
    </td>
  </tr>

  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/index.html#interact-with-the-browser">ブラウザの操作</a></h4>
      <a href="dev-guide/tutorials/open-a-web-page.html">Web ページを開く</a>,
      <a href="dev-guide/tutorials/listen-for-page-load.html">ページ読み込みのリッスン</a>, and
      <a href="dev-guide/tutorials/list-open-tabs.html">開いているタブの一覧表示</a>.
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/index.html#modify-web-pages">Web ページの変更</a></h4>
      <a href="dev-guide/tutorials/modifying-web-pages-url.html">URL に基づいた Web ページの変更</a>
      or <a href="dev-guide/tutorials/modifying-web-pages-tab.html">アクティブな Web ページの変更</a>.
    </td>
  </tr>

  <tr>
    <td>
      <h4><a href="dev-guide/tutorials/index.html#development-techniques">開発テクニック</a></h4>
Learn about common development techniques, such as
<a href="dev-guide/tutorials/unit-testing.html">単体テスト</a>,
<a href="dev-guide/tutorials/logging.html">ログとして出力</a>,
<a href="dev-guide/tutorials/reusable-modules.html">サードパーティモジュールの作成</a>,
<a href="dev-guide/tutorials/l10n.html">ローカリゼーション</a>, and
<a href="dev-guide/tutorials/mobile.html">モバイル開発</a>.
    </td>

    <td>
      <h4><a href="dev-guide/tutorials/index.html#putting-it-together">応用</a></h4>
      Walkthrough of the <a href="dev-guide/tutorials/annotator/index.html">Annotator</a> example add-on.
    </td>
  </tr>

</table>

<hr>

## <a href="dev-guide/guides/index.html">Guides</a> ##

<table class="catalog">
<colgroup>
<col width="50%">
<col width="50%">
</colgroup>
  <tr>
    <td>
      <h4><a href="dev-guide/guides/index.html#sdk-infrastructure">SDK infrastructure</a></h4>
      Aspects of the SDK's underlying technology:
      <a href="dev-guide/guides/commonjs.html">CommonJS</a>, the
      <a href="dev-guide/guides/program-id.html">Program ID</a>, the
      <a href="dev-guide/guides/module-search.html">module search algorithm</a>
      and the rules defining
      <a href="dev-guide/guides/firefox-compatibility.html">Firefox compatibility</a>.
    </td>

    <td>
      <h4><a href="dev-guide/guides/index.html#sdk-idioms">SDK idioms</a></h4>
      The SDK's
      <a href="dev-guide/guides/events.html">event framework</a> and the
      <a href="dev-guide/guides/two-types-of-scripts.html">distinction between add-on scripts and content scripts</a>.
    </td>

  </tr>

  <tr>
    <td>
      <h4><a href="dev-guide/guides/index.html#content-scripts">Content scripts</a></h4>
      A <a href="dev-guide/guides/content-scripts/index.html">detailed guide to working with content scripts</a>,
      including: how to load content scripts, which objects
      content scripts can access, and how to communicate
      between content scripts and the rest of your add-on.
    </td>

    <td>
      <h4><a href="dev-guide/guides/index.html#xul-migration">XUL migration</a></h4>
      A guide to <a href="dev-guide/guides/xul-migration.html">porting XUL add-ons to the SDK</a>.
      This guide includes a
      <a href="dev-guide/guides/sdk-vs-xul.html">comparison of the two toolsets</a> and a
      <a href="dev-guide/guides/library-detector.html">worked example</a> of porting a XUL add-on.
    </td>

  </tr>

</table>

<hr>

## Reference ##

<table class="catalog">
<colgroup>
<col width="50%">
<col width="50%">
</colgroup>
  <tr>
    <td>
      <h4>API reference</h4>
      Reference documentation for the high-level SDK APIs found in the
      <a href="packages/addon-kit/index.html">addon-kit</a>
      package, and the low-level APIs found in the
      <a href="packages/api-utils/index.html">api-utils</a> package.
    </td>

    <td>
      <h4>Tools reference</h4>
      Reference documentation for the
      <a href="dev-guide/cfx-tool.html">cfx tool</a>
      used to develop, test, and package add-ons, the
      <a href="dev-guide/console.html">console</a>
      global used for logging, and the
      <a href="dev-guide/package-spec.html">package.json</a> file.
    </td>

  </tr>

</table>
