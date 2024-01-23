# My helloworld-html5 quickstart

# どこか適当なディレクトリに移動しておき、helloworld-html5のみをコピーする
$ cp -r $QUICKSTART_HOME/helloworld-html5 .
# 親ディレクトリにあるpom.xmlも必要になるので別名にしてコピーしておく
$ cp $QUICKSTART_HOME/pom.xml helloworld-html5/pom-parent.xml
$ cd helloworld-html5
$ vi pom.xml
# 28行目の<relativePath>を親からコピーしたpom.xmlを指すように編集する
'-: <relativePath>../pom.xml</relativePath>
'+: <relativePath>pom-parent.xml</relativePath>
# ビルド (`mvn install`でもよい)
$ mvn package
# デプロイ (すでにEAPを起動済みであること)
$ mvn wildfly:deploy
EAPのログにデプロイが行われたことを表すログが出力され、
http://localhost:8080/helloworld-html5 でこのアプリにアクセス可能になる
# GitHub画面右上の"+"で"New repository"を選び、"helloworld-html5"の名前で空のパブリックリポジトリを作成
# (pom-parent.xmlが存在する)自分のhelloworld-html5内で
$ echo target >> .gitignore
$ echo .idea >> .gitignore
$ echo .vscode >> .gitignore
$ echo "# My helloworld-html5 quickstart" > README.md
$ git init
$ git add .
$ git commit -m 'first commit'
$ git branch -M main
$ git remote add origin git@github.com:<自分のアカウント名>/helloworld-html5.git
$ git push -u origin main
GitHubのjboss-container-images/jboss-eap-openshift-templatesにEAPの公式
ImageStreamとTemplateがあるのでそれを現在のプロジェクトに導入する。
$ oc apply -f https://raw.githubusercontent.com/jboss-container-images/jboss-eap-openshift-templates/eap74/eap74-openjdk17-image-stream.json
imagestream.image.openshift.io/jboss-eap74-openjdk17-openshift created
imagestream.image.openshift.io/jboss-eap74-openjdk17-runtime-openshift created
$ oc get is
NAME IMAGE REPOSITORY TAGS UPDATED
jboss-eap74-openjdk17-openshift default-route-openshift-image-registry.apps-crc.testing/myproj/jboss-eap74-openjdk17-openshift 7.4,latest 29 seconds ago
jboss-eap74-openjdk17-runtime-openshift default-route-openshift-image-registry.apps-crc.testing/myproj/jboss-eap74-openjdk17-runtime-openshift 7.4,latest 25 seconds ago
$ for resource in eap74-amq-persistent-s2i.json eap74-amq-s2i.json eap74-basic-s2i.json eap74-https-s2i.json eap74-sso-s2i.json ; \
do oc apply -f https://raw.githubusercontent.com/jboss-container-images/jboss-eap-openshift-templates/eap74/templates/${resource}; done
template.template.openshift.io/eap74-amq-persistent-s2i created
template.template.openshift.io/eap74-amq-s2i created
template.template.openshift.io/eap74-basic-s2i created
template.template.openshift.io/eap74-https-s2i created
template.template.openshift.io/eap74-sso-s2i created
$ oc get template
NAME DESCRIPTION PARAMETERS OBJECTS
eap74-amq-persistent-s2i An example JBoss Enterprise Application Platform application using Red Hat JB... 39 (10 blank) 14
eap74-amq-s2i An example JBoss Enterprise Application Platform application using Red Hat JB... 37 (10 blank) 13
eap74-basic-s2i An example JBoss Enterprise Application Platform application. For more inform... 20 (5 blank) 8
eap74-https-s2i An example JBoss Enterprise Application Platform application configured with... 30 (11 blank) 10
eap74-sso-s2i An example JBoss Enterprise Application Platform application Single Sign-On a... 50 (21 blank) 10
これらは openshift 名前空間にすでに存在することもあるが古かったりするので独自
に取得する。また自分の名前空間に置くことで直接編集することもできる。
oc nw-appコマンドによるビルドとデプロイ
GitHubのリポジトリのアカウント名は適宜読み替えること。
$ oc new-app --template=eap74-basic-s2i \
-p APPLICATION_NAME=myapp \
-p IMAGE_STREAM_NAMESPACE=$(oc project -q) \
-p EAP_IMAGE_NAME=jboss-eap74-openjdk17-openshift:latest \
-p EAP_RUNTIME_IMAGE_NAME=jboss-eap74-openjdk17-runtime-openshift:latest \
-p SOURCE_REPOSITORY_URL=https://github.com/onagano-rh/helloworld-html5 \
-p SOURCE_REPOSITORY_REF=main \
-p CONTEXT_DIR=""
# `oc logs -f buildconfig/myapp-build-artifacts` でビルドの様子を、
# ビルドの正常終了後は `oc logs -f buildconfig/myapp` デプロイの様子を確認できる
