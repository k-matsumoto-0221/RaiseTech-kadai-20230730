# <ステップ1>
## **組み込みサーバにデプロイ**
**<参考にしたサイト>**
- https://pikawaka.com/rails/ec2_deplo  
- https://qiita.com/ysda/items/359ba0afa23741db192b  
- https://qiita.com/ysda/items/246009da5aea8cc29666  
---

### ①環境構築
1. **Swap領域を用意する**
~~~
$ sudo dd if=/dev/zero of=/swapfile1 bs=1M count=512
$ sudo chmod 600 /swapfile1
$ sudo mkswap /swapfile1
$ sudo swapon /swapfile1
$ grep Swap /proc/meminfo
~~~  

2. **必要なパッケージをインストールする**
- その他
~~~
$ sudo yum -y update
$ sudo yum -y install git make gcc-c++ patch libyaml-devel libffi-devel libicu-devel zlib-devel readline-devel libxml2-devel libxslt-devel ImageMagick ImageMagick-devel openssl-devel libcurl libcurl-devel curl
~~~

- Node.js
~~~  
$ sudo curl -sL https://rpm.nodesource.com/setup_14.x | sudo bash -
$ sudo yum -y install nodejs
~~~

- yarn
~~~  
$ curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
$ sudo yum -y install yarn
~~~  

- rbenvとruby-build
~~~
$ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
$ source .bash_profile
$ git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
~~~

- ruby
~~~
$ rbenv rehash
$ rbenv install 3.1.2
$ rbenv global 3.1.2
$ rbenv rehash
~~~

3. **Githubからサンプルアプリケーションをクローンする**
~~~
$ sudo mkdir /var/www/
$ sudo chown ec2-user /var/www/
$ cd /var/www/
$ git clone https://github.com/yuta-ushijima/raisetech-live8-sample-app.git
$ cd raisetech-live8-sample-app
~~~

4. **Gemをインストールする**
~~~
$ sudo yum -y install mysql-devel
$ gem install bundler -v 2.3.14
$ bundle install
~~~

5. **DB関連の設定**
~~~
$ sudo yum -y install mysql
$ cp -p config/database.yml.sample config/database.yml
~~~
「(省略)/config/database.yml」を編集
「default: &default」欄を追記・修正
- usernameを「admin」に
- RDSのパスワード
- host: 作成したrdsのエンドポイント 

~~~
bundle exec rails db:create
bundle exec rails db:migrate
~~~ 

6. **仕上げ**
~~~
$ bin/setup
$ sudo chmod 755 bin/dev
$ bin/dev
~~~

ブラウザから「http://パブリックIPv4:3000」でアクセスする。  

### ②動作確認 
![ステップ1_01)](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%971_01.png)  
![ステップ1_02](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%971_02.png)  

# <ステップ2>
## **「Nginx + Unicorn」で動作させる**
### ①環境構築
1. **Nginxをインストールする**
~~~
$ sudo amazon-linux-extras install -y nginx1
$ nginx -v
$ sudo touch /etc/nginx/conf.d/rails.conf
$ sudo chmod 777 /etc/nginx/conf.d/rails.conf
~~~

2. **Nginxの設定を行う**  
「/etc/nginx/conf.d/rails.conf」を以下のように編集。
~~~
upstream unicorn {
  server unix:/var/www/raisetech-live8-sample-app/unicorn.sock;
}

server {
  listen 80;
  root /var/www/raisetech-live8-sample-app/public;

  # location ^~ /assets/ {
  #   gzip_static on;
  #   expires max;
  #   add_header Cache-Control public;
  # }

  location @unicorn {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://unicorn;
  }

  try_files $uri/index.html $uri @unicorn;
  error_page 500 502 503 504 /500.html;
}
~~~

3. **Unicornをインストールする**  
既にインストール済みのため省略。

4. **Unicornの設定**  
「(省略)/config/unicorn.rb」の5,6行目を以下のように修正。
~~~
listen '/var/www/raisetech-live8-sample-app/unicorn.sock'
pid    '/var/www/raisetech-live8-sample-app/unicorn.pid'
~~~

5. **仕上げ**
~~~
$ sudo systemctl start nginx
$ bundle exec unicorn_rails -c config/unicorn.rb -E development
~~~

ブラウザから「http://パブリックIPv4:80」でアクセス。  

### ②動作確認 
![ステップ2_01](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%972_01.png)  
![ステップ2_02](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%972_02.png)  


# <ステップ3>
## **ロードバランサー(ALB)を追加する**
<ステップ2>までの手順で作成したEC2の前段にALBを配置し、ALB越しにアプリケーションにアクセスできるように設定を行う。
- ターゲットグループを作成し、EC2の「port:80」にヘルスチェックを行うように設定。
- EC2とALBのそれぞれが通信を行えるようにネットワーク設定を行う。(主にセキュリティグループ)
- アプリケーションが動作している状態でなければ、ヘルスチェックが「Unhealthy」となるため注意すること。 

ALBを作成後、ブラウザから「http://ALBのDNS名」でアクセス。  
その後、ブラウザに表示された「config.hosts << "(省略)"」を「(省略)/config/environments/development.rb」に記載し、再度ブラウザからアクセス。  

アプリケーションを再起動する。
~~~
$ bundle exec unicorn_rails -c config/unicorn.rb -E development
~~~

### 動作確認 
![ステップ3_01](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%973_01.png)  

# <ステップ4>
## **アプリケーションの画像の保存先をS3に変更する**
### ①環境構築
アップロードした画像がS3に格納されるように設定を変更するため、以下3つを作成する。
1. S3  
![ステップ4_01](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%974_01.png)  
2. エンドポイント  
![ステップ4_02](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%974_02.png)  
3. IAMロール(S3アクセス用)  
![ステップ4_03](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%974_03.png)  

以下2つの設定ファイルを編集する。  
1. config/environments/development.rb  
「config.active_storage.service = :local」を「amazon」に修正
  
2. config/storage.yml  
- 「access_key_id」と「secret_access_key」をそれぞれコメントアウト。  
- 「bucket: <%= Rails.application.credentials.dig(:aws, :active_storage_bucket_name) %>」
  を「S3のバケット名」に修正

アプリケーションを再起動する。
~~~
$ bundle exec unicorn_rails -c config/unicorn.rb -E development
~~~

### ②動作確認
ブラウザから「http://ALBのDNS名」でアクセスする。  
![ステップ4_04](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%974_04.png)  

ブラウザ上から画像を保存し、S3にアップロードされているかを確認する。  
![ステップ4_05](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%974_05.png)  
![ステップ4_06](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%974_06.png)  

ブラウザ上から画像を削除し、S3からも削除されるかを確認する。  
![ステップ4_07](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%974_07.png)  
![ステップ4_08](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%974_08.png)  


# <ステップ5>
## **構成図を作成する**
![ステップ５_01](img/第5回目/%E3%82%B9%E3%83%86%E3%83%83%E3%83%975_01.png)
