================
 ファイル入出力
================

.. contents:: Contents
   :depth: 2

概要
====

ファイル入出力は、Java同様 InputStream/OutputStream や Reader/Writer を使っても書けるが、
拡張された File クラスとクロージャを使うと、スッキリ書ける。

参考：

* http://groovy.codehaus.org/groovy-jdk/java/io/File.html

テキストファイルの読み込み
===========================

例えば、テキストファイルを一行ずつ読み込んで標準出力に出力するコードは以下のようになる。

.. code-block:: groovy

   new File("./sample.txt").eachLine {
     println it
   }
   

また、ファイルを丸ごと読み込んで、一つの String オブジェクトとするには以下のコードで可能。

.. code-block:: groovy

   def alltext = new File("./sample.txt).getText()


テキストファイルへの追記
========================

ファイルが書き込み可能であれば、以下のコードでファイルへの追記ができる。

.. code-block:: groovy

   new File("./sample.text").append("hogehoge")
   // 出力する文字エンコードをを指定したい場合は、appendの第二引数で指定可能
   new File("./sample.text").append("にほんご", "UTF-8")


ディレクトリ以下の全ファイルの処理
==================================

ディレクトリに存在する全ファイルに対し順次処理を行いたい場合は、
ディレクトリパスで File オブジェクトを作成し、eachFile に処理を記述したクロージャを渡す。

.. code-block:: groovy

   new File("./temp_dir").eachFile {
     it.append("newline")     // クロージャの引数（it）に個々のFileオブジェクトが設定される
   }


InputStream/OutputStreamのLoanパターンでの利用
==============================================

*Loanパターン* は、高階関数を用いてリソース確保・解放とリソースを利用した処理を分離する
デザインパターン。

Groovyには、InputStream/OutputStreamに追加されている高階関数メソッドによって、
Streamのクローズ処理をAPIに任せてしまう事ができる。

以下のように、withStreamメソッドに対し、目的のStreamを引数を取るクロージャを渡すと、
クロージャの実行を行い、完了後にStreamのクローズを行なってくれる。

.. code-block:: groovy

   new FileInputStream("hoge.txt").withStream { inputStream ->
     // inputStreamによるデータ読み込み
   }


InputStream/OutputStream/Reader/Writerの取得
============================================

File ファイルオブジェクトの newReader や newInputStream のようなメソッドを利用すると、
これらのオブジェクトを簡単に生成することができる。

.. code-block:: groovy

   def reader = new File("./test.txt").newReader()


これは、Javaの以下のコードと同じ動作となる。

.. code-block:: java

   BufferedReader reader =
     new BufferedReader(
       new FileReader("./test.txt"));


