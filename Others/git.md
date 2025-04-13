# Git

## コマンド

ブランチを移動するコマンドです。-b オプションを付けると新しいブランチを作成し、切り替えます

```bash
git checkout master
```

ブランチの一覧表示
オプション
-r リモートブランチの一覧

```bash
git branch
```

最新の取得

```bash
git pull origin master
```

ブランチの作成(リモートリポジトリに作成)

```bash
git checkout -b 02_hello_world
```

追加変更されたファイルの状態確認

```bash
git status
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

トラッキングを設定してプッシュする。

トラッキングを設定すると、該当のチェックアウトブランチとプッシュ先のブランチの紐づきがConfigに作成される。
以後、push先を明示的に指定する必要がなくなる。

```bash
git push --set-upstream origin 02_hello_world
```

マージ処理

```bash
git checkout master
git merge ブランチ名
```

ログ確認

```bash
git log
```

## 考え方

あるディレクトリに対して、紐づくブランチを切り替えて使用する。

