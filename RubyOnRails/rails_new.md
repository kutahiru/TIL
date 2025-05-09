# rails new

## Docker 関連のファイル生成と編集

通常権限、DevContainers、LSPを使用する

動かない拡張機能に対する対応
https://tech.gamewith.co.jp/entry/2024/12/18/102455

```bash
#Dockerfile.dev
FROM ruby:3.3.6
ENV LANG C.UTF-8
ENV TZ Asia/Tokyo
RUN apt-get update -qq \
&& apt-get install -y ca-certificates curl gnupg \
&& mkdir -p /etc/apt/keyrings \
&& curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
&& NODE_MAJOR=20 \
&& echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
&& curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor -o /etc/apt/keyrings/yarn.gpg \
&& echo "deb [signed-by=/etc/apt/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs yarn vim
RUN mkdir /myapp
WORKDIR /myapp
RUN gem install bundler
COPY . /myapp
```

```bash
#compose.yml
services:
  db:
    image: postgres
    restart: always
    environment:
      TZ: Asia/Tokyo
      POSTGRES_PASSWORD: password
    volumes:
      - postgresql_data:/var/lib/postgresql
    ports:
      - 5432:5432
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -d myapp_development -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    command: bash -c "bundle install && bundle exec rails db:create db:migrate && rm -f tmp/pids/server.pid && ./bin/dev"
    tty: true
    stdin_open: true
    volumes:
      - .:/myapp
      - bundle_data:/usr/local/bundle:cached
      - node_modules:/myapp/node_modules
    environment:
      TZ: Asia/Tokyo
      PGHOST: db
      PGUSER: postgres
      PGPASSWORD: password
    ports:
      - "3000:3000"
    depends_on:
      db:
        condition: service_healthy
volumes:
  bundle_data:
  postgresql_data:
  node_modules:

```



```bash
#.devcontainer\devcontainer.json
{
  "name": "IdeaStorage",
  "dockerComposeFile": ["../compose.yml"],
  "service": "web",
  "workspaceFolder": "/myapp",
  "customizations": {
    "vscode": {
      "settings": {
        "extensions.verifySignature": false
      },
      "extensions": [
        "Shopify.ruby-lsp",
        "KoichiSasada.vscode-rdbg",
        "saoudrizwan.claude-dev",
        "streetsidesoftware.code-spell-checker",
        "dbaeumer.vscode-eslint",
        "mhutchie.git-graph",
        "eamodio.gitlens",
        "oderwat.indent-rainbow",
        "ionutvmi.path-autocomplete",
        "humao.rest-client",
        "bradlc.vscode-tailwindcss",
        "shardulm94.trailing-spaces"
      ]
    }
  }
}
```

```bash
#.vscode\extensions.json
{
  "recommendations": ["ms-vscode-remote.vscode-remote-extensionpack"]
}
```

```bash
#vscode\launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "rdbg",
      "name": "Attach with rdbg",
      "request": "attach"
    }
  ]
}
```

```bash
#.vscode\setting.json
{
  "[ruby]": {
    "editor.defaultFormatter": "Shopify.ruby-lsp",
    "editor.formatOnSave": true,
    "editor.insertSpaces": true,
    "editor.semanticHighlighting.enabled": true,
    "editor.formatOnType": true
  },
  "rubyLsp.formatter": "rubocop",
  "rubyLsp.testTimeout": 60
}
```

```bash
docker compose build

docker compose run --rm web gem install rails -v '~> 7.2.1'

docker compose run --rm web rails new . -d postgresql -j esbuild --css=tailwind
```

```bash
#Procfile.devを修正してコンテナ外部からの接続を可能にする
web: env RUBY_DEBUG_OPEN=true bin/rails server -b 0.0.0.0
js: yarn build --watch
css: yarn build:css --watch
```

