======================================
 AntBuilder による Ant 関連ツール利用
======================================

.. contents:: Contents
   :depth: 2


概要
====

GroovyにはAntの機能が食い込まれており、個々のAntタスクをAPI経由で呼び出せる。

Antのようなビルドツールの実装に利用できる他、タスクを独立したファイル操作APIの代わりに
利用することも可能。

使いでがあるのは、tar/zip/gzip/jarの圧縮展開、指定したURLからのファイル取得(Getタスク)、
複数ファイルの同期(Syncタスク)、SCPによるリモートファイルコピー(Scpタスク)あたりか。
メール送信も使えるかも。

使用するには、AntBuilderクラスをインスタンス化し、そのメソッドを呼び出す。
タスクに対応するメソッドには、タスクの属性値にあたるパラメータをマップ形式で
指定する。


コード例
========

指定したURLからZip形式のファイルを取得し、展開するメソッドの例。

.. code-block:: groovy

   def downloadAndExtractZip(String url, String toDir) {
     def tmpFile = File.createTempFile("ant", ".zip")
     def ant = new AntBuilder()
     ant.get(src:url, dest:tempFile)     // Getタスク
     ant.unzip(src:tmpFile, dest:toDir)  // Unzipタスク
   }


Antで使用するbuild.xmlと同様のことがGroovyでも書けるよう、要素をクロージャとしてネストさせ、
階層構造を記述できるようになっている。もちろん、本来のGroovyにある制御構造や他のAPIも
利用できるので、XMLでは難しい、フロー制御を持ったビルドスクリプトを記述できる。

以下のコードは、javacのコンパイルを試み、何らかの例外が発生した場合は開発チーム宛に
メールを送出する例。

.. code-block:: groovy

   ant = new AntBuilder()
   try{
     ant.javac(srcdir:"/src/dir/path", destdir:"/binary/dir/path" )
   }catch(Throwable thr){
     ant.mail(mailhost:"mail.example.com", subject:"ビルドに失敗しました"){
       from(address:"buildmaster@example.com", name:"buildmaster")
       to(address:"dev-team@example.com", name:"Development Team")
       message("コンパイル時にエラーが発生しました。[ ${thr.getMessage()} ]")
     }
   }
