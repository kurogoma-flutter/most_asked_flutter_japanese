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

![image](https://user-images.githubusercontent.com/67848399/205405267-90d66f54-a86d-45d5-b63e-10466be79d11.png)

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
例外として、GlobalKeyがStateful Widgetで使用される場合です。これについては、質問3.で詳しく説明しています。

### 2.Stateオブジェクトとは何か、dispose()メソッドが実行された後はどうなるのか？
ウィジェットは不変であると聞いたことがありますが、ユーザーのアクションによってUIが変化するのはなぜでしょうか？もしWidgetsが不変なら、誰がUIの動的な動作に責任を持つのでしょうか？

**StatefulWidgetのStateオブジェクトです！**

StatefulWidgetがElementツリーに挿入されると、そのcreateState()メソッドは対応するElementのコンストラクタから呼び出されます。

Stateがツリーに挿入されると、initState()が実行されますが、これはStateインスタンスごとに一度だけ実行されます。

initState()の後、build()が呼び出されます。setState()を呼び出すことで、複数回呼び出すことができる。

つまり、StatefulWidgetの親が再描画された場合、Widgetがゼロから作成されても、StateオブジェクトのinitState()は呼ばれません（一度だけ呼ばれるから！）。なぜ、このようなことが起こるのでしょうか？

FlutterはStatefulWidgetの状態を保持しようとし、要素ツリー内の古い場所にある場合のみWidgetを再作成します。

a.) runtime Type または
b.) Key
に変化があった場合、ウィジェットが再作成されるのです。

Widgetが同じタイプで、同じキー（またはキーなし）であれば、ElementはWidgetを更新（再作成）するだけで、古いStateオブジェクトは保存されます。

![image](https://user-images.githubusercontent.com/67848399/205405239-9f65526d-c55e-4e71-8576-a27999869ad4.png)

![image](https://user-images.githubusercontent.com/67848399/205405249-4334b8b7-7b24-4b25-822f-3be471fada12.png)

Streamをの検知をするのを止めたり、または別のものを登録したり、ウィジェットを更新（再作成）するにはどうすれば良いでしょうか？
以下がそのメソッドです。

```Dart
@override
void didUpdateWidget(covariant StatefulText oldWidget) {
```

これは、Widgetが新しいものと入れ替わるたびに呼び出される。このメソッドでは、新しくアタッチされたWidgetに由来する、必要なものすべてを更新することができます。このメソッドが実行された後、build()メソッドが呼び出されます。

しかし、もしStateを再利用せず、親Widgetが再構築されるたびに新しい要素（新しいWidgetと新しいState）を作成したい場合は、そのWidgetにUniqueKeyを追加します（しかし、これは高価であることに留意してください！）。

では、エレメントツリーから削除された後(エレメント全体が別のものに置き換えられるとき)、Stateはどうなるのでしょうか？

![image](https://user-images.githubusercontent.com/67848399/205405879-f4f71b67-1b5d-4215-9817-d4b1f0b0b802.png)

この TimerWidget は、ユーザがクリックすると、UniqueKey() を使って新しいものに置き換えられます。

https://miro.medium.com/max/1400/1*2iveunD_P7qGDu4rrzOwEg.webp

で、2回クリックした後のコンソールログはこんな感じです。

![image](https://user-images.githubusercontent.com/67848399/205406626-ef37095f-06d1-410b-8006-0d0063e4a549.png)

これは、State要素がツリーから削除された後も生きている（メモリのどこかに生きている）ことを意味するので、dispose()ではStreamの購読を解除したり、Timerをキャンセルしたりすることに注意してください。

... あるいは、dispose()の前に呼ばれるdeactivate()でさえ、そうするかもしれません。場合によっては、ツリーから削除されたエレメントは、他の場所に再挿入されることがあります（例えば、グローバルキーが使用されている場合）。Elementが再挿入されると、activate()メソッドが呼ばれ（deactivate()でオフにしたものをすべて起動する時間を与える）、その後、build()が呼ばれます。

この例では、dispose()が終了すると、Stateはunmounted(mounted == false)され、ツリーから永久に削除されます!

![image](https://user-images.githubusercontent.com/67848399/205406949-e9a97192-5e56-437f-a576-063b23b2bf3d.png)

...これは実際には、このStateオブジェクトがそのElementから切り離されたことを意味する。

### 3.Widget Keyとは何ですか、何に使うのですか？
### 4.Dartはシングルスレッド言語ですか？非同期タスクはどのように実行されるのですか？
### 5.constとfinalの違いは何ですか？
### 6.abstract classとmixinの違いを説明してください。
### 7.状態管理ライブラリをいくつかリストアップし、それらがどのように機能するかを説明してください。
### 8.静的コード解析とは何か、そして開発者としてどのように役立つのか。
### 9.extension methodsとは何ですか？
### 10.null safetyとは何ですか？
