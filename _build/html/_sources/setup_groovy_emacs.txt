==============================================
 GroovyインストールとEmacsへのgroovy-mode設定
==============================================

Windows環境へのGroovy処理系インストールと、Emacs for Windowsへの組み込み方。

Groovy処理系のインストール
==========================

参考： http://groovy.codehaus.org/Installing+Groovy

#. Java SDKが入ってなければインストールします。古いバージョンは何かと危ないので、
   できれば最新版に上げておきましょう。

#. Java SDKのインストールディレクトリのフルパスを、環境変数JAVA_HOMEに設定します。

#. Groovy のバイナリディストリビューションを http://groovy.codehaus.org/Download から
   ダウンロードします。（「Download Windows-Installer」を選択）
   
#. ダウンロードした groovy-<version>.exe を実行します。
   x64環境の場合、デフォルトのインストールフォルダはWOW64用のフォルダ（Program Files (x86)）
   にインストールされるようですが、これでもx64版JavaSDKで動くようです。
   不安なら別のフォルダにインストールしても大丈夫。
   なお、最後に環境変数(GROOVY_HOME, PATH)の設定までやってくれますが、
   なぜか２～３分ほど待たされます。

#. インストール終了後、コマンドラインから「groovysh」と打ち込んでインタプリタが起動すればOK。


Emacsへの組み込み
=================

参考： http://groovy.codehaus.org/Emacs+Groovy+Mode

Emacs 24.3.1 for windows でやっていますが、Meadowでもいけるはず。多分。

#. emacs groovy-mode の入手
   https://launchpad.net/groovy-emacs-mode からダウンロード出来ます。
   tgzなファイルを解凍できない場合は、githubからZIP形式でも入手出来ます。
   （ https://github.com/russel/Emacs-Groovy-Mode から「ZIP」ボタンをクリック）

#. %home%\\.emacs.d 以下に、groovy-mode のファイルを展開する。
   環境変数 home を設定していない場合は、C:\\Users\\<ユーザ名>\\AppData\\Roaming\\.emacs.d 
   になるはず
   (Windows Vista/7の場合)

#. .emacs に以下のコードを追記する。

   ::

      ;;; Groovy-mode
      (add-to-list 'load-path "~/.emacs.d/emacs-groovy-mode")
      (autoload 'groovy-mode "groovy-mode" "Major mode for editing Groovy code." t)
      (add-to-list 'auto-mode-alist '("\.groovy$" . groovy-mode))
      (add-to-list 'interpreter-mode-alist '("grooy" . groovy-mode))

      ;;; バッファ上のスクリプトをGroovyで実行(C-c m)
      (add-hook 'groovy-mode-hook
		'(lambda ()
		   (require 'groovy-electric)
		   (groovy-electric-mode)))
      (defvar groovy-command "groovy %f"
	"groovy or compatible command. %f means buffer-file")
      (setq groovy-command-line nil)
      (defun groovy-execute-argument()
	(if (null groovy-command-line)
	    (setq groovy-command-line groovy-command))
	(setq groovy-command-line
	      (read-string "Groovy command: " groovy-command-line))
	(list groovy-command-line))
      (defun groovy-execute-buffer-file (arg)
	"Execute the groovy code of this buffer."
	(interactive (groovy-execute-argument))
	(save-some-buffers)
	(shell-command
	 (if (string-match "%f" groovy-command-line)
	     (replace-match (buffer-file-name) t t groovy-command-line nil)
	   groovy-command-line)))

      (add-hook 'groovy-mode-hook
		(lambda () (define-key groovy-mode-map "\C-cm" 'groovy-execute-buffer-file)))

      (setq groovy-command "groovy %f")

