======================
 関数型プログラミング
======================

.. contents:: Contents
   :depth: 2


クロージャ（関数）
==================

*クロージャ* は、Groovyで使用出来る *第一級関数* であり、メソッドとは区別される。

メソッドは、 Java文法または ``def`` キーワードによって定義される。

.. code-block:: groovy

   // Javaと同様の文法によるメソッド定義
   int methodA(int arg1, String arg2) {
     // ... 処理
     return res
   }

   // 型を省略する場合は def キーワードを使用する。
   def methodA(arg1, arg2) {
     // ... 処理
     return res
   }


クロージャは、以下のように定義する。

.. code-block:: groovy 

   // クロージャの例
   // ここでの def キーワードは、メソッドではなく
   // 変数 closureA の宣言を行なっていることに注意
   def closureA = {arg1, arg2 ->
     // ... 処理
     return res
   }

   // 引数リストおよび「->」を省略すると、引数を一つとるクロージャとなる。
   // 仮引数名は自動的に「it」となる。
   def closureB = {
     if (it == "hoge") 
     // ... 処理
   }

   // 引数を取らないクロージャの場合、引数リストを書かず「->」のみ書く
   def closureC = { ->
     // ... 処理
   }


また、 ``.&`` 演算子でメソッドからクロージャを生成することができる。

.. code-block:: groovy

   def closure = a.&method1


高階関数
========

クロージャの主な用途としては、 *高階関数* への利用が挙げられる。

高階関数は、引数で他の関数を受け取り内部で利用する関数、あるいは戻り値として値ではなく
関数を返す関数のこと。StrategyパターンにおけるContextに相当する。

Groovyでは、制御構造やコレクションメソッドなどの標準APIの多くが、高階関数として提供されている。

.. code-block:: groovy

   def closureA = { println "hello" }

   // 高階関数「(Integer値).times」は、引数で渡したクロージャをInteger値回数繰り返す
   5.times(closureA)


一度きりの利用であれば、上記のように変数に代入する必要はなく、高階関数の引数に
直接クロージャを記述できる。

.. code-block:: groovy

   5.times { println "hello"}


なお、クロージャは実行時はClosure型のインスタンスとなるので、自分で高階関数を定義する場合は、
Closure型の引数を受け取るように定義するとよい。

.. code-block:: groovy

   def higher(Closure cl) {
     // ... 前処理
     cl.call()     // 引数で渡されたクロージャの実行
     // ... 後処理
   }


関数合成
========

クロージャ同士での関数合成は以下のように書ける。

.. code-block:: groovy

   def f = { x -> x * 2 }    // 関数 f(x)
   def g = { x -> x + 3 }    // 関数 g(x)

   def fg = (f << g)   // f(g(x)) と等価
   def gf = (f >> g)   // g(f(x)) と等価


SAM (Single Abstract Method) 型のクロージャによる実装
=====================================================

メソッド一つだけからなるinterface( *SAM型* )の実装は、クロージャで簡単に記述できる。

Javaの場合：

.. code-block:: java

   Runnable runnable = new Runnable() {
     void run() {
       // 処理
     }
   }


Groovyでクロージャを使用した場合：

.. code-block:: groovy

   def runnable = {
     // 処理
   } as Runnable


クロージャとして記述した関数は、SAM型内で定義されたメソッドに自動的にマップされる。

これを利用して、SAM型のオブジェクトを引数にとるJavaのAPIを
高階関数のように利用することができる。例えばJava6のConcurrentAPIによる
並行処理を、以下のように記述できる。

.. code-block:: groovy

   import java.util.concurrent.Executors

   def ex = Executors.newSingleThreadExecutor()
   def future = ex.submit {
     // 並行タスク
   }      // ※SAM型を引数にとるメソッド渡すクロージャは、asによる型指定を省略可

   // 主タスク ...

   future.get()  // 並行タスクの終了を待つ
   ex.shutdown()


永続データ構造、副作用
======================

finalキーワードの扱い
---------------------

Groovyでは、変数への再代入を制限できない。

Javaと同様、Groovyでも変数宣言・初期化時にfinalキーワードを
付与することができるが、無視される。（finalを指定された変数も再代入可能）


イミュータブルコレクション
--------------------------

Groovyのコレクション型（List、Map等）は、デフォルトではミュータブル（変更可能）だが、
``asImmutable`` メソッドでイミュータブル（変更不可）のオブジェクトを取得できる。

ここで取得されるイミュータブルオブジェクトは、JavaAPIの ``java.util.Collections`` に含まれる
``unmodifiableCollection`` メソッドで返されるものと同一。従って、 ``asImmutable`` で取得した
オブジェクトに対し、中身を書き換えるような操作を行うと、 ``UnsupportedOperationException`` が
throwされる。


Immutableアノテーションを使ったイミュータブル型の定義
-----------------------------------------------------

クラス定義にImmutableアノテーションを付与することで、
そのクラスのインスタンスは変更不可能であることが保証される。

（実際は、インスタンス変数が書き換わるコードが実行される時点で
``ReadOnlyPropertyException`` がthrowされる）

なお、C++のconstのように、ミュータブルな型のインスタンスを
イミュータブルなものとして扱うことはできない模様（残念）。

.. code-block:: groovy

   import groovy.transform.Immutable
   
   @Immutable class Hoge {
     String a
     int b
   }

   def h = new Hoge(a: "fuga", b: 3)
   h.a = "baa"                       // ReadOnlyPropertyExceptionがthrowされる


再帰
====

Groovyでは、クロージャ内で ``call`` キーワードを用いて自分自身の再帰呼び出しができる。
（つまり、匿名関数を匿名のまま再帰化できる）

Groovyでは、暗黙的な末尾再帰最適化は行われないが、処理の最後で明示的に 
``call`` の代わりに ``trampoline`` メソッドを呼び出すことで、末尾再帰最適化が行われる。
ただし、末尾再帰になる（継続が不要になる）ケース以外で ``trampoline`` を使用すると、
``groovy.lang.MissingMethodException`` がthrowされる。


遅延評価
========

Groovyは正格評価を基本とし、Scalaのlazyのような遅延評価指定もないが、
Schemeのように、計算結果の代わりにラムダ（クロージャ）を返して
呼出元で任意のタイミングで評価する手法が使える。

