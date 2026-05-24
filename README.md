# About Activity

## What's Activity
&nbsp;&nbsp;&nbsp; Activityには、2つの重要な側面がある。<br> 　1つ目は、UIを描画するウィンドウを提供する機能である。イメージとしてはキャンバスだろうか。<br>　2つ目は、起動の仕方である。通常のプログラミングでは、mainメソッドでアプリが起動するが、AndroidではActivityライフサイクルの特定の段階に対応したコールバックを呼び出すことで、アクティビティインスタンスのコードを開始している。また、アプリが別のアプリを呼び出す際には、別のアプリ全体を起動するのではなく、そのアプリの特定のアクティビティを起動する。<br>　アプリには複数のアクティビティを作れるが、通常メインアクティビティとして1つのアクティビティが指定される。Jetpack Composeでは、複数のアクティビティで画面遷移を実現するのではなく、コンポーザブルによってナビゲーションされる。

## Activity Lifecycle
&nbsp;&nbsp;&nbsp; Androidでは、Activityライフサイクの各段階に、コールバックが用意されているが、Composeアプリでは、そのコールバックに直接紐付けることは推奨されていない（参照：[「ライフサイクルを Compose と統合する」](https://developer.android.com/topic/libraries/architecture/lifecycle))。そのため、以下のコールバックは直接使うというより、ライフサイクルの概念を理解するために、記す。  

### Activity Lifecycleの図
<img width="205" height="225" alt="スクリーンショット 2026-05-24 23 01 18" src="https://github.com/user-attachments/assets/5497fe90-deaf-4cac-ac18-57f2b6e83a0f" /><br>
引用元：https://developer.android.com/codelabs/basic-android-kotlin-compose-activity-lifecycle#2

### onCreate()
&nbsp;&nbsp;&nbsp; このコールバックは、アクティビティを最初に作成した時に呼び出される。ここでは、アクティビティのライフサイクル全体を通して一度だけ実行される必要があるコードを実行する。ここの実行が完了すると、次に``onStart()``と``onResume()``が連続して呼び出される。

### onStart()
&nbsp;&nbsp;&nbsp; このコールバックにより、アクティビティはユーザーに表示される。しかし、画面に表示されるだけで、インタラクティブではない。``onCreate()``と同様に、このコールバックはすぐに完了し、次の``onResume()``が呼び出される。

### onRestart()
&nbsp;&nbsp;&nbsp; ``onStop()``コールバックが呼び出された後に、アクティビティを再起動する場合に呼び出される。

### onResume()
&nbsp;&nbsp;&nbsp; このコールバックが呼び出される状態は、アクティビティがフォアグラウンドに表示され、インタラクティブになる。アプリからフォーカスが外れるまで、この状態を維持する。割り込みイベントが発生すると、``onPause()``コールバックが呼び出される。<br>　例えば、マルチウィンドウというスマホの画面上に2つのアプリを表示する際に、操作していないアプリはstartの状態になる。そのため、マルチウィンドウで操作していない状況でも、カメラをアクティブな状態にしたい場合は、``onStart()``でカメラを初期化する必要がある。しかし、その場合、もう一つのアプリでカメラを起動させようとすると、二つのアプリがカメラを起動しようとしてしまい、クラッシュする。この問題を踏まえ、Android9の一部以降は、複数のアクティビティが、マルティウィンドウの際には、resume状態を維持できるようになった（参考：[Activity lifecycle in multi-window mode](https://developer.android.com/develop/ui/compose/layouts/adaptive/support-multi-window-mode#lifecycle)）。

### onPause()
&nbsp;&nbsp;&nbsp; このコールバックが呼び出される理由として、アプリの実行を中断するイベントが発生したこと、マルティウィンドウでフォーカスが外れたこと、ダイアログなど半透明のアクティビティが開かれたことが挙げられる。この段階であれば、アプリはフォアグラウンド上にはいないため、位置情報取得の停止やリソースの解放ができる。ただし、``onPause()``の実行時間は非常に短いため、データの保存やネットワークの呼び出しには使用できない。<br>　``onPause()``が重要なのは、アプリがフリーズしたように見えないように、UIを更新しておく必要があるからである。しかし、コードの実行が長いと、他の表示を妨げる可能性がある。例えば、電話がかかってきたときに、着信通知が遅れる可能性がある。アクティビティが完全に非表示になると、``onStop()``が呼び出される。

### onStop()
&nbsp;&nbsp;&nbsp; アクティビティがユーザーから見えなくなると、このコールバックが呼び出される。このstop状態では、CPU負荷の高い操作、例えばデータ保存もできる。さらに、状態やメンバ情報は保持される。Activityの実行が終了すると``onDestroy()``が呼び出される。

### onDestroy()
&nbsp;&nbsp;&nbsp; アクティビティが破棄される前、つまりユーザーがアプリを完全に終了した時に、このコールバックは呼び出される。さらに、デバイスの回転やマルチウィンドウへの移行など、構成変更の際にも呼び出されるが、ViewModelは保持され、次のアクティビティインスタンスに渡されるため、何もする必要がない。``onDestory()``では、先ほどの``onStop()``までのコールバックで解放されなかったリソースを全て解放する。

## When Activity Lifecycle
- デバイスの戻るボタンを使用したとき
    - アクティビティが削除されるため、``onDestroy()``が呼び出される。
- デバイスのホームボタンを使用したとき、アプリを切り替えたとき、電話が来たとき
    - アクティビティは破棄されず、``onPause()``、``onStop()``の順で呼び出され、アプリに戻ったら、``onRestart()``、``onStart()``、``onResume()``の順で呼び出される。

## Best Practice for Activity Lifecycle
### rememberとrememberSaveableの違い
&nbsp;&nbsp;&nbsp; アクティビティのライフサイクルと、コンポーザブルのライフサイクルは違う。例えば、コンポーザブルのライフサイクルは、コンポジションに入り、0回以上再構成され、コンポジションから出るという流れである。この再構成の際に、変更した値が初期値にならず、値を保持するには、``remember``が必要になる。しかし、``onDestroy()``からの``onCreate()``で構成変更が行われる際は、コンポジションが新たに作成されるため、``remember()``では対応できない。そのため、``rememberSaveable``を使用すると、アクティビティを破棄して再作成する場合でも、値を保存することができる。

### ライフサイクルを考慮したユースケース
- LifecycleStartEffectを用いて、位置情報更新の粒度を、onStartで詳細な更新をして、onStopで大まかな更新に切り替える。
- LifecycleResumeEffectを用いて、アプリがインタラクティブになるまで、onResumeで動画の再生を延期し、バックグラウンドになったら、onPauseで一時停止する。
- flowでネットワークから連続的にデータを受け取っている時に、collectAsStateWithLifecycleを用いると、アプリがフォアグラウンドにいる際は、リアルタイムで更新され、バックグラウンドになるとキャンセルされる。

参考：https://developer.android.com/topic/libraries/architecture/lifecycle#use-cases

### ライフサイクルの状態を収集
&nbsp;&nbsp;&nbsp; ``currentStateAsState``の返り値は、Lifecycle.State型で、CREATEDやRESUMEDという形で返ってくる。INITIALIZED→CREATED→STARTED→RESUMED→STARTED→CREATED→DESTROYEDというライフサイクルの各段階に合わせている。
```
val lifecycleOwner = LocalLifecycleOwner.current
val currentLifecycleState = lifecycleOwner.lifecycle.currentStateAsState()
```
参考：https://developer.android.com/reference/kotlin/androidx/lifecycle/compose/currentStateAsState.composable

### 単発のイベントに最適な書き方
&nbsp;&nbsp;&nbsp; lifecycleの各コールバックに対応した定数（例えば、ON_STARTやON_STOP）が用意されている。
```
@Composable
LifecycleEventEffect(Lifecycle.Event.ON_RESUME) {
    // ログや分析に使用
}
```
参考：https://developer.android.com/reference/kotlin/androidx/lifecycle/Lifecycle.Event

### 特定のステージで対応する場合の適切な書き方
&nbsp;&nbsp;&nbsp; 各コールバックは対応している。例えば、onStartとonStopやonResumeとonPauseという具合である。このように各ペアで開始と終了をすべきオペレーションがある場合、以下の書き方が適切になる。
#### LifecycleStartEffect
&nbsp;&nbsp;&nbsp; アプリの起動中（画面に表示されている間）に実行し、バックグラウンドの際には停止する必要のある操作に使う。``onStopOrDispose``は、Lifecycle.Event.ON_STOPが発生したり、LifecycleStartEffectがコンポジションから抜ける時に、実行される。通常のEffect同様に、引数にキーがあり、キーが変更されると、ブロック内のコードが再度実行される。ちなみに、原則として、ブロック内で使用される可変変数と不変変数は、キーに追加する必要がある（参考：[Restaring Effects](https://developer.android.com/develop/ui/compose/side-effects#restarting-effects)）。
```
@Composable
fun Example(fusedLocationClient: FusedLocationClient) {
    LifecycleStartEffect(fusedLocationClient) {
        scope.launch {
            withContext(Dispatchers.IO) {
                fusedLocationClient.lastLocation.await()
            }
        }
    }
}
```
参考：https://github.com/android/platform-samples/blob/main/samples/location/src/main/java/com/example/platform/location/currentLocation/CurrentLocationScreen.kt

#### LifecycleResumeEffect
&nbsp;&nbsp;&nbsp; アプリが表示されたときに起動し、画面から消えたときに停止する必要のある操作に使う。``onPauseOrDispose``は、Lifecycle.Event.ON_PAUSEが発生したり、LifecycleResumeEffectがコンポジションから抜けるときに、実行される。
```
@Composable
fun Example(cameraController: CameraController) {
    LifecycleResumeEffect(cameraController) {
        cameraController.startPreview()

        onPauseOrDispose {
            cameraController.stopPreview()
        }
    }
}
```
カメラ機能をアプリに追加する方法として、JetpackComposeでは(https://github.com/google/jetpack-camera-app)が参考になるが、上記の例では参照していないので気をつけること。

## Term
- フォアグラウンド
    - アプリが画面上に表示され、ユーザーのフォーカスを獲得している状態。フォーカスというのは、操作ができることを指す。

## Reference
- https://developer.android.com/codelabs/basic-android-kotlin-compose-activity-lifecycle#0
- https://developer.android.com/guide/components/activities/activity-lifecycle#activity-lifecycle-concepts
