<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

<div class="warning">Chrome アクセスを取得するための API は現在 SDK の実験的な機能であり、将来のリリースで変更されることがあります。</div>

# Chrome 権限 #

## Chrome 権限の使用 ##

最も強力な低レベルモジュールは、「Chrome 権限」を使用して実行されます。この権限を使用すると、ホストシステムへの無制限のアクセス権を付与する、悪名高い <code>Components</code> オブジェクトにアクセスできます。モジュールは、Components オブジェクトから、ブラウザが提供するほとんどの機能を実行できます。このような権限を取得するには、モジュールで以下のような文を使用して取得の意図を宣言する必要があります。

    var {Cc, Ci} = require("chrome");

<code>require("chrome")</code> によって返されるオブジェクトを、Mozilla JS 環境の「分割代入（destructuring assignment）」機能を使用して展開すると、通常の <code>Components.*</code> エイリアスが提供されます。

<code>**Cc**</code>

`Components.classes` のエイリアス

<code>**Ci**</code>

`Components.interfaces` のエイリアス

<code>**Cu**</code>

`Components.utils` のエイリアス

<code>**Cr**</code>

`Components.results` のエイリアス

<code>**Cm**</code>

`Components.manager` のエイリアス

<code>**components**</code>

`Components` 自体のエイリアス（小文字であることに注意してください）。このエイリアスから、通常あまり使用されない `Components.stack` や `Components.isSuccessCode` などのプロパティにアクセスできます。

注：`require("chrome")` 文は、Chrome 機能や `Components` API にアクセスする**唯一の**方法です。`Components` オブジェクトには、モジュールから**絶対アクセスしないでください**。SDK ツールがモジュールコードの中に `Components` への直接参照を発見した場合、警告が発生します。

Chrome 権限は、絶対に必要な場合以外、モジュールで使用しないでください。Chrome 権限を使用したモジュールは、セキュリティ面を慎重に確認する必要があります。このようなモジュールのバグは、ほとんどがセキュリティ上の危険要因となります。

## マニフェストの生成 ##

**マニフェスト**は、生成後の XPI に含まれているリストで、`require()` アクセスを要求したモジュールとアクセスを要求されたモジュールが記載されます。このリストにはまた、chrome アクセスを要求したモジュールも記録されています。このリストの生成にあたっては、含まれているすべてのモジュールで `require(XYZ)` 文がスキャンされ、それらが参照する "XYZ" 文字列が記録されます。

マニフェストの実装が完了すると、ランタイムローダーが機能して、マニフェストにリストされていないモジュールに対する `require()` が実際に禁止されます。同様に、Chrome 権限を要求したことがマニフェストに記録されているモジュール以外、Chrome 権限を取得できないようになります。これにより、実行コードに適用されているのと同じ権限の制約が確実に実現されるので、アドオンのレビュー効率が向上します（この作業が完了していない場合、モジュールがこの権限の制約を回避できることがあります）。

マニフェストは、完全な Javascript パーサではなく、単純な regexp ベースのスキャナによって作成されます。そのため、開発者は単一の静的文字列を持つ単純な `require` 文を 1 行に 1 つだけ記述するという規則に従う必要があります。スキャナが `require` の指定を検出できなかった場合、その指定はマニフェストに追加されず、実装後にランタイムコードで例外が発生することになります。

例えば、以下のコードはいずれもマニフェストスキャナによって検出されないので、`require()` 呼び出しによって指定したモジュールをインポートできず、実行時に例外が発生します。

    // all of these will fail!
    var xhr = require("x"+"hr");
    var modname = "xpcom";
    var xpcom = require(modname);
    var one = require("one"); var two = require("two");

つまり開発者は、モジュールで使用する権限を（セキュリティレビューアに対して）宣言する、それらの権限をモジュールのローカルのネームスペースにマッピングする方法を制御するという 2 つの目的で require() を使用します。したがってその点で、`require()` 文を明確にし、解析しやすくする必要があります。将来的には、権限のより詳細な表現を可能にするため、マニフェストのフォーマットから宣言部分を別ファイルに移すことが考えられています。

「`cfx xpi`」や「`cfx run`」など、マニフェストを作成するコマンドを実行すると、含まれているすべてのモジュールで、`Cc`/`Ci` エイリアス（または拡張形式 `Components.classes`）がスキャンされます。拡張形式が検出された場合、または require("chrome") の記述と対応していない「Cc」などを検出した場合、警告が発生します。これらの警告は、開発者が適切なパターンを使用できるようにするためのガイドです。モジュール開発者は、警告に注意し、警告が発生しなくなるようにコードを訂正してください。
