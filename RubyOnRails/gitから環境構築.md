 # gitから環境構築

## gitからcloneする

githubのCodeのSSHからコピー

![image-20250422145453267](C:\Users\kota\AppData\Roaming\Typora\typora-user-images\image-20250422145453267.png)

```bash
git@github.com:runteq/curriculum_base_environment.git
```

## git clone後、cdコマンドを使用してcloneしたディレクトリに移動してください。

```
$ cd xxxx_xxxx_beginner_rails
```

## コンテナの立ち上げ
以下のコマンドを順番に実行してコンテナを立ち上げてください。
```
$ docker compose build

$ docker compose up
```

## gem・JavaScriptパッケージをインストールする
docker compose up が実行されているターミナルとは別のターミナルを立ち上げて、以下のコマンドを順番に実行してください。
```
$ docker compose run web bundle install

$ docker compose run web yarn install
```

## データベースの初期化・マイグレーションファイル適応
docker compose up が実行されているターミナルとは別のターミナル上で、以下のコマンドを順番に実行してください。
```
$ docker compose exec web rails db:create

# 以下は課題を取り組んでいる中でデータベースを初期化する必要になった際に実行してください。
$ docker compose exec web rails db:drop
$ docker compose exec web rails db:create
$ docker compose exec web rails db:migrate
```

## CSS, JavaScript用のサーバー起動
docker compose up が実行されているターミナルとは別のターミナル上で、以下のコマンドを実行してください。
```
$ docker compose exec web bin/dev
```

### Dockerコンテナを終了する際の操作

```bash
$ docker compose down
```
