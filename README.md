# My helloworld-html5 quickstart
ロジェクトだけを自分用にコピーして、自由に編集・使い捨てできるように
する。ここでは helloworld-html5 を例にするが、基本的にどのプロジェクトでもよ
い。
# どこか適当なディレクトリに移動しておき、helloworld-html5のみをコピーする
$ cp -r $QUICKSTART_HOME/helloworld-html5 .
# 親ディレクトリにあるpom.xmlも必要になるので別名にしてコピーしておく
$ cp $QUICKSTART_HOME/pom.xml helloworld-html5/pom-parent.xml
$ cd helloworld-html5
$ vi pom.xml
# 28行目の<relativePath>を親からコピーしたpom.xmlを指すように編集する
-: <relativePath>../pom.xml</relativePath>
+: <relativePath>pom-parent.xml</relativePath>
