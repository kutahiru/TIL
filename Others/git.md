# Git

## コマンド

ブランチを移動するコマンドです。-b オプションを付けると新しいブランチを作成し、切り替えます

```bash
git checkout master
```

ブランチの一覧表示

```bash
git branch
```

最新の取得

```bash
git pull origin master
```

ブランチの作成

```bash
git checkout -b 02_hello_world
```

作業ディレクトリ内の全ての変更をGitのステージングエリア（インデックス）に追加するコマンド
ステージングエリアは、作業ディレクトリとリポジトリの間に位置する中間的な場所
変更をコミットする前に、どの変更を次のコミットに含めるかを選択

```bash
git add -A
```

ステージングエリアに追加されたファイルをメッセージを設定してコミットする。

```bash
git commit -m "add hello world" 
```

コミット内容をプッシュする。

```bash
git push origin 02_hello_world
```

