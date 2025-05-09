# rails new

## Docker 関連のファイル生成と編集

```
touch Dockerfile.dev
touch compose.yml
```

### Dockerfile.dev

```
FROM ruby:3.3.6
ARG UID=1000
ENV LANG C.UTF-8
ENV TZ Asia/Tokyo
ENV BUNDLE_APP_CONFIG /myapp/.bundle
RUN apt-get update -qq \
&& apt-get install -y ca-certificates curl gnupg \
&& mkdir -p /etc/apt/keyrings \
&& curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
&& NODE_MAJOR=20 \
&& echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
&& curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor -o /etc/apt/keyrings/yarn.gpg \
&& echo "deb [signed-by=/etc/apt/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs yarn vim
RUN useradd -m -u $UID rails
RUN mkdir /myapp
WORKDIR /myapp
RUN gem install bundler
RUN mkdir -p /myapp/node_modules && chown -R rails:rails /myapp/node_modules
COPY . /myapp
RUN chown rails:rails -R /myapp
```

### compose.yml

```
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
      test: ["CMD-SHELL", "pg_isready -U postgres"]
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
    user: "1000:1000"
    depends_on:
      db:
        condition: service_healthy
volumes:
  bundle_data:
  postgresql_data:
  node_modules:
```

### ビルド

```bash
docker compose build

# まずGemfileを作成
docker compose run --rm web /bin/bash -c "bundle init"

# Railsを追加する前にvendor/bundleの設定
docker compose run --rm web /bin/bash -c "bundle config set --local path vendor/bundle"

# Railsを追加
docker compose run --rm web /bin/bash -c "bundle add rails -v '~> 7.2.1'"
```

```bash
# 全てのgemをインストール
docker compose run --rm web /bin/bash -c "bundle install"
```

```bash
# カスタマイズ性を考慮してこちらを使用する
# TailwindCSSをCSSフレームワークとして使用する場合は以下のコマンドを実行してください。
docker compose run --rm web bundle exec rails new . -d postgresql -j esbuild --css=tailwind --force
```

```bash
# vendorディレクトリをGitから除外
echo '/vendor/bundle' >> .gitignore
```

```bash
#Gemfileにforemanを追加する。
#追加しないとシステム全体にインストールしようとしてエラーになるから
group :development do
  # Use console on exceptions pages [https://github.com/rails/web-console]
  gem "web-console"

  gem "foreman"
end
```

```bash
#bin/devスクリプト以下のように修正して、bundle exec foremanを使用するようにする
#!/usr/bin/env sh

# Default to port 3000 if not specified
export PORT="${PORT:-3000}"

exec bundle exec foreman start -f Procfile.dev --env /dev/null "$@"
```

```bash
#Procfile.devを修正してコンテナ外部からの接続を可能にする
web: env RUBY_DEBUG_OPEN=true bin/rails server -b 0.0.0.0
js: yarn build --watch
css: yarn build:css --watch
```

```bash
# Bootstrapは使用しないので、不要
# BootstrapをCSSフレームワークとして使用する場合は以下のコマンドを実行してください。
docker compose run --rm web rails new . -d postgresql -j esbuild --css=bootstrap
```