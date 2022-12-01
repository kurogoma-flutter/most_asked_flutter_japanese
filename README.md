## Most asked Flutter and Dart interview questions with detailed answers
[Most asked Flutter and Dart interview questions with detailed answers](https://jelenaaa.medium.com/most-asked-flutter-and-dart-interview-questions-with-detailed-answers-89025bf69037#af3c)を軽く読んでみて、かなり面白そう＋タメになりそうなので頑張って日本語訳をしてみた。

### 書き出し
> What every developer should know before applying for a Flutter job?

Flutterの仕事に応募する前に、すべての開発者が知っておくべきこととは？

<br>

### 原作者
素敵な記事を書いてくださったのでぜひフォローしてください！

https://jelenaaa.medium.com/membership

<br>

### 項目一覧
この記事は、Flutterエンジニアの技量を図るための質問がまとめられています。全てで１０項目です。

> 1. Can you explain what is the connection between an Element and a Widget?

ElementとWidgetの関係性について説明できますか？

> 2. What is a State object and what happens to it after its dispose() method gets executed?

Stateオブジェクトとは何か、dispose()メソッドが実行された後はどうなるのか？

> 3. What are Widget Keys and what are they used for?

Widget Keyとは何ですか、何に使うのですか？

> 4. Is Dart single threaded language? How are async tasks executed?

Dartはシングルスレッド言語ですか？非同期タスクはどのように実行されるのですか？

> 5. What is the difference between const and final?

constとfinalの違いは何ですか？

> 6. Explain difference between an abstract class and a mixin.

abstract classとmixinの違いを説明してください。

> 7. List couple of state management libraries and explain how they work.

状態管理ライブラリをいくつかリストアップし、それらがどのように機能するかを説明してください。

> 8. What is static code analysis and how does it help you as a developer?

静的コード解析とは何か、そして開発者としてどのように役立つのか。

> 9. What are extension methods?

extension methodsとは何ですか？

> 10. What is null safety?

null safetyとは何ですか？

### 1.ElementとWidgetの関係性について説明できますか？
FlutterはElementとRenderの2つのツリーを使います。
Renderツリーには画面上にピクセルを描画するために使用されるUIコンポーネントの座標が格納されます。
ElementツリーにはWidgetsとその状態（Statefulの場合）、そして時間の経過とともに変化する親子関係が格納されます。

*BuildContext*

```dart
///...略...
/// [BuildContext] objects are actually [Element] objects. The [BuildContext]
/// interface is used to discourage direct manipulation of [Element] objects.
abstract class BuildContext {
  /// The current configuration of the [Element] that is this [BuildContext].
  Widget get widget;
  //...略...
}
```

*Element*

```dart
abstract class Element extends DiagnosticableTree implements BuildContext {
  /// Creates an element that uses the given widget as its configuration.
  ///
  /// Typically called by an override of [Widget.createElement].
  Element(Widget widget)
    : assert(widget != null),
      _widget = widget;

  Element? _parent;
  //...略... 
}
```
つまり、BuildContextの正体はElementである。
（Element全体ではなく、BuildContextのインターフェース部分なので、ツリーそのものをいじることはできない）

各Elementは、`_parent`フィールドで自分の親が誰なのか知っている。

＜＜TODO エレメントツリーの画像＞＞

そして、Elementツリーを検索していくと、ルートElement（親はnull）に辿り着くことができます。
上の画像からわかるように、builderContextはContainerのElementの親Element[ノードであり](https://wa3.i-3-i.info/word1300.html)、デバッガが停止した場所（21行目）である。
では、これらのオブジェクトはどのような順序で作成されるのだろうか？

まず、Widgetを作成します。
そして、そのWidgetをアプリのUIに組み込むと決めたら、そのWidgetのElementを作成します。
その要素がコンストラクタの中に作成されます。
そのコンストラクタ内で、State(Statefulの場合)が生成され、最後にビルドメソッドが実行されます。

```dart
/// An [Element] that uses a [StatefulWidget] as its configuration.
class StatefulElement extends ComponentElement {
  /// Creates an element that uses the given widget as its configuration.
  StatefulElement(StatefulWidget widget)
      : _state = widget.createState(),
        super(widget) {
  //...略...
}
```

要約すると、ElementはWidgetのインスタンスと、StatefulWidgetの場合その状態のインスタンスを保持します。そのため、Widgetが不変となるためUIは簡単に変更ができます。

興味深いことに、StatefulWidgetの同じインスタンスをUIの複数の場所に挿入すると、Widgetのインスタンスは1つになりますが、Stateのインスタンスは複数（ツリーの異なるElementに配置）存在することになります。
例外として、GlobalKeyがStateful Widgetで使用される場合です。これについては、質問3.で詳しく説明しています。

*Stateクラスのコメント抜粋*
```dart
///
/// [State] objects are created by the framework by calling the
/// [StatefulWidget.createState] method when inflating a [StatefulWidget] to
/// insert it into the tree. Because a given [StatefulWidget] instance can be
/// inflated multiple times (e.g., the widget is incorporated into the tree in
/// multiple places at once), there might be more than one [State] object
/// associated with a given [StatefulWidget] instance. Similarly, if a
/// [StatefulWidget] is removed from the tree and later inserted in to the tree
/// again, the framework will call [StatefulWidget.createState] again to create
/// a fresh [State] object, simplifying the lifecycle of [State] objects.
///
```

### 2.Stateオブジェクトとは何か、dispose()メソッドが実行された後はどうなるのか？
### 3.Widget Keyとは何ですか、何に使うのですか？
### 4.Dartはシングルスレッド言語ですか？非同期タスクはどのように実行されるのですか？
### 5.constとfinalの違いは何ですか？
### 6.abstract classとmixinの違いを説明してください。
### 7.状態管理ライブラリをいくつかリストアップし、それらがどのように機能するかを説明してください。
### 8.静的コード解析とは何か、そして開発者としてどのように役立つのか。
### 9.extension methodsとは何ですか？
### 10.null safetyとは何ですか？
