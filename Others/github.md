# Github

## SSHキーの作成

WSL2上で実行

```bash
cd ~/.ssh
```

↑フォルダがなければ作成する。

```bash
mkdir -p ~/.ssh
```

キーを作成

```bash
ssh-keygen -t rsa
```

以下を適切な応答

```bash
Enter file in which to save the key (/Users/(username)/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

公開鍵と秘密鍵が作成されるので、
公開鍵はGitHubに登録する。