#　Dockerファイル権限

Docker内からWSL上にファイルを作成した場合、VSCodeでファイルを修正できず、
以下のコマンドが必要な状態

```
sudo chown -R $USER:$USER .
```

## 都度実行を回避する

これを実行すると、rubocopがエラーになってしまった。
おそらく、Gemのインストール先をデフォルトではなく、プロジェクトにすることで解決できそう。

### docker-compose.override.ymlファイルの作成

```
version: '3'
services:
  app:  # メインのdocker-compose.ymlで使用しているサービス名と同じにする
    user: "${UID:-1000}:${GID:-1000}"
```

### 環境変数の設定

```bash
nano ~/.bashrc #設定ファイルを開く
```

```
# Docker用環境変数設定
export UID=$(id -u)
export GID=$(id -g)
```

```bash
source ~/.bashrc #設定の反映
```

### .gitignoreへの追加

```bash
echo "docker-compose.override.yml" >> .gitignore
```







