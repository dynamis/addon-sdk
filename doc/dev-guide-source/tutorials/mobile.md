<!-- This Source Code Form is subject to the terms of the Mozilla Public
   - License, v. 2.0. If a copy of the MPL was not distributed with this
   - file, You can obtain one at http://mozilla.org/MPL/2.0/. -->

<div class="warning">Firefox Mobile 対応のアドオンの開発は、いまだ SDK の実験的機能です。使用されている SDK モジュールは安定していますが、セットアップ手順や cfx コマンドは変更される可能性があります。
</div>

# Firefox Mobile 対応の開発 #

<span class="aside">
このチュートリアルに沿って学習するには、あらかじめ [SDK をインストール](dev-guide/tutorials/installation.html)し、[`cfx` 入門](dev-guide/tutorials/getting-started-with-cfx.html)を学習してください。
</span>

Mozilla は先ごろ、[Firefox Mobile on Android の UI の再実装](http://starkravingfinkle.org/blog/2011/11/firefox-for-android-native-android-ui/) を決定しました。これにより、従来の XUL に代わってネイティブな Android ウィジェットを使用した UI が実装されます。アドオン SDK では、この新しいバージョンの Firefox Mobile とデスクトップバージョンの Firefox の両方で動作するアドオンを開発できます。

デスクトップの Firefox と Firefox Mobile には同じコードを使用でき、`cfx run`、`cfx test`、`cfx xpi` で追加のオプションを指定するだけで Firefox Mobile をターゲットとすることができます。

現在、完全に機能するのは、以下のモジュールのみです。

* [page-mod](packages/addon-kit/page-mod.html)
* [page-worker](packages/addon-kit/page-worker.html)
* [request](packages/addon-kit/request.html)
* [self](packages/addon-kit/self.html)
* [simple-storage](packages/addon-kit/simple-storage.html)
* [timers](packages/addon-kit/timers.html)

Mozilla では、他のモジュールについてもサポートを追加できるよう取り組んでいます。

このチュートリアルでは、開発マシンに USB 接続された Android デバイスで SDK アドオンを実行する方法について説明します。
アドオン SDK とデバイス間の通信には、[Android Debug Bridge](http://developer.android.com/guide/developing/tools/adb.html)（adb）を使用します。

<img class="image-center" src="static-files/media/mobile-setup-adb.png"/>

[Android エミュレータ](http://developer.android.com/guide/developing/tools/emulator.html) を使用すれば、デバイスにアクセスせずにアドオンを開発することができます。しかしエミュレータは低速なので、当面、ここで説明する方法を使用する方が簡単です。

## 環境の設定 ##

まず、[ネイティブバージョンの Firefox Mobile を実行できる Android デバイス](https://wiki.mozilla.org/Mobile/Platforms/Android#System_Requirements) を用意します。
Android デバイスで、以下の手順を実行します。

* デバイスに [Firefox Mobile の Nightly ビルド](https://wiki.mozilla.org/Mobile/Platforms/Android#Download_Nightly)をインストールします。
* [デバイスで USB デバッグを有効にします（このリンクの手順 3 のみ）](http://developer.android.com/guide/developing/device.html#setting-up) 

開発マシンで、以下の手順を実行します。

* アドオン SDK のバージョン 1.5 以上をインストールします。
*  デバイスに適したバージョンの [Android SDK](http://developer.android.com/sdk/index.html) をインストールします。
* Android SDK を使用して、[Android Platform Tools](http://developer.android.com/sdk/installing.html#components) をインストールします。

次に、デバイスを開発マシンに USB で接続します。

ここでコマンドシェルを開きます。Android SDK をインストールしたディレクトリの下にある「platform-tools」ディレクトリに、Android Platform Tools によって `adb` がインストールされています。環境変数 PATH に「platform-tools」のパスを設定してください。その後、以下を入力します。

<pre>
adb devices
</pre>

以下のような出力が表示されます。

<pre>
List of devices attached
51800F220F01564 device
</pre>

（上の長い 16 進数は、表示される数値とは異なります）

これで `adb` によってデバイスが検出され、開発の準備が整いました。

## Android 上でのアドオンの実行 ##

サポートされている範囲でモジュールを使用する限り、通常どおりアドオンを開発できます。

アドオンを実行する必要がある場合、まずそのデバイスで Firefox が実行されていないことを確認します。次に、オプションを指定して `cfx run` を実行します。

<pre>
cfx run -a fennec-on-device -b /path/to/adb --mobile-app fennec --force-mobile
</pre>

このコマンドの詳細については、[「モバイル開発のための cfx オプション」](dev-guide/tutorials/mobile.html#cfx-options)を参照してください。

コマンドシェルに、以下のように表示されます。

<pre>
Launching mobile application with intent name org.mozilla.fennec
Pushing the addon to your device
Starting: Intent { act=android.activity.MAIN cmp=org.mozilla.fennec/.App (has extras) }
--------- beginning of /dev/log/main
--------- beginning of /dev/log/system
Could not read chrome manifest 'file:///data/data/org.mozilla.fennec/chrome.manifest'.
info: starting
info: starting
zerdatime 1329258528988 - browser chrome startup finished.
</pre>

この後、多くのデバッグ出力が続きます。

デバイス上では、インストールしたアドオンとともに Firefox が起動します。

アドオンからの `console.log()` 出力は、デスクトップ開発の場合と同様に、コマンドシェルに書き込まれます。しかし、シェルには他の多くのデバッグ出力が書き込まれるので、理解するのは容易ではありません。`adb logcat` コマンドを実行すると `adb` のログが表示され、アドオンの実行後にデバッグ出力をフィルタ処理することができます。例えば、Mac OS X または Linux では以下のようなコマンドを使用できます。

<pre>
adb logcat | grep info:
</pre>

これは `cfx test` を実行しても同じです。

<pre>
cfx test -a fennec-on-device -b /path/to/adb --mobile-app fennec --force-mobile
</pre>

## <a name="cfx-options">モバイル開発のための cfx オプション</a> ##

上のコード例が示すように、`cfx run` と `cfx test` を Android デバイスで使用するには、以下の 4 つのオプションが必要です。

<table>
<colgroup>
<col width="30%">
<col width="70%">
</colgroup>

<tr>
  <td>
    <code>-a fennec-on-device</code>
  </td>
  <td>
    Add-on SDK に対して、アドオンをホストするアプリケーションを指定するオプションです。デバイス上の Firefox Mobile でアドオンを実行する場合は、「fennec-on-device」に設定します。
   </td>
</tr>
<tr>
  <td>
    <code>-b /path/to/adb</code>
  </td>
  <td>
    <p>前述のように、<code>cfx</code> は Android Debug Bridge（adb）を使用して Android デバイスと通信します。このオプションは、<code>cfx</code> に対して <code>adb</code> 実行可能ファイルの場所を指定します。</p>
    <p><code>adb</code> 実行可能ファイルへの絶対パスを指定する必要があります。</p>
  </td>
</tr>
<tr>
  <td>
    <code>--mobile-app</code>
  </td>
  <td>
    <p><a href="http://developer.android.com/reference/android/content/Intent.html"> Android インテント</a>の名前を指定するオプションです。この値は、デバイスで実行している Firefox Mobile のバージョンによって異なります。</p>
    <ul>
      <li><code>fennec</code>：Nightly を実行している場合</li>
      <li><code>fennec_aurora</code>：Aurora を実行している場合</li>
      <li><code>firefox_beta</code>：Beta を実行している場合</li>
      <li><code>firefox</code>：Release を実行している場合</li>
    </ul>
    <p>バージョンが不明な場合は、以下のようなコマンドを実行します（OS X または Linux の場合。Windows ではこれに相当するコマンドを実行)。</p>
    <pre>adb shell pm list packages | grep mozilla</pre>
    <p>「package」の後に「org.mozilla.」と表示され、その後に文字列が続きます。
    最後の文字列が、ここで使用する名前です。例えば、以下が表示された場合、</p>
    <pre>package:org.mozilla.fennec</pre>
    <p>以下を指定します。</p>
    <pre>--mobile-app fennec</pre>
    <p>デバイスに 1 つの Firefox のみをインストールしている場合、このオプションは不要です。</p>
  </td>
</tr>
<tr>
  <td>
    <code>--force-mobile</code>
  </td>
  <td>
    <p>Firefox Mobile との互換性を強制するために使用するオプションです。Firefox Mobile で実行する場合は、常に使用する必要があります。</p>
  </td>
</tr>
</table>

## モバイルアドオンのパッケージ化 ##

モバイルアドオンを XPI としてパッケージ化するには、以下のコマンドを使用します。

<pre>
cfx xpi --force-mobile
</pre>

実際、デバイス上に XPI をインストールするには、少し注意が必要です。最も簡単な方法は、おそらく XPI をデバイスのどこかにコピーすることです。

<pre>
adb push my-addon.xpi /mnt/sdcard/
</pre>

次に Firefox Mobile を開き、アドレスバーに以下を入力します。

<pre>
file:///mnt/sdcard/my-addon.xpi
</pre>

ブラウザで XPI が開き、インストールするかどうかを尋ねるメッセージが表示されます。

その後、以下のように `adb` を使用して XPI を削除できます。

<pre>
adb shell
cd /mnt/sdcard
rm my-addon.xpi
</pre>
