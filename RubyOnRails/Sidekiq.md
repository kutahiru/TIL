# Sidekiq

http://localhost:3000/sidekiq/

```
# ログ確認
docker-compose logs sidekiq
docker-compose logs -f sidekiq | grep -E "(start|done|GetNotificationTargetsJob)"
```

```ruby
# Gemfile
gem "sidekiq"
gem "sidekiq-scheduler"
gem "redis"
```

```ruby
# config/initializers/sidekiq.rb
Sidekiq.configure_server do |config|
  config.redis = { url: "redis://redis:6379/0" }
end

Sidekiq.configure_client do |config|
  config.redis = { url: "redis://redis:6379/0" }
end
```

```yml
# compose.yml
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
    command: bash -c "bundle install && bundle exec rails db:create db:migrate && rm -f tmp/pids/server.pid && tail -f /dev/null"
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
      redis:
        condition: service_started
  redis:
    image: redis
    volumes:
      - redis_volume:/data
    command: redis-server --appendonly yes
    ports:
      - "6379:6379"
  sidekiq:
    build:
      context: .
      dockerfile: Dockerfile.dev
    command: bundle exec sidekiq
    volumes:
      - .:/myapp
      - bundle_data:/usr/local/bundle:cached
    environment:
      TZ: Asia/Tokyo
      PGHOST: db
      PGUSER: postgres
      PGPASSWORD: password
      REDIS_URL: redis://redis:6379/0
    depends_on:
      - db
      - redis
volumes:
  bundle_data:
  postgresql_data:
  node_modules:
  redis_volume:
```

```ruby
# config/routes.rb
# http://localhost:3000/sidekiq/
  if Rails.env.development?
    require "sidekiq/web"
    require "sidekiq-scheduler/web"
    mount Sidekiq::Web => "/sidekiq"
  end
```

```yml
# config/sidekiq.yml
:concurrency: 5
:queues:
  - default
:scheduler:
  :schedule:
    get_notification_targets:
      cron: '*/30 * * * *'
      class: GetNotificationTargetsJob
      queue: default
      description: '30分毎に通知対象を取得'
```

```ruby
# app/jobs/get_notification_targets_job.rb
# 定期実行でmedication_schedulesテーブルからデータを取得し、以下の処理を実施する。
# ・1回目のジョブキューを作成する
# ・管理テーブルを作成する
# 2回目以降のリマインド実行は別処理でキューを作成
class GetNotificationTargetsJob < ApplicationJob
  sidekiq_options retry: false

  def perform
    # 通知対象を取得
    notification_targets = NotificationDatabaseService.get_notification_data
    notification_targets.each do |target|
      # 管理テーブルを作成
      medication_management = NotificationDatabaseService.insert_medication_management(target)
      target.medication_management_id = medication_management.id

      # 全属性をハッシュに変更
      # ハッシュ経由でしか渡せないため
      target_hash = target.attributes

      if target.medication_taker?
        # 服薬者の通知をJOBキューに登録
        # LineNotificationJob.perform_at(target.medication_date_time, target)
        LineNotificationJob.set(wait_until: target.medication_date_time).perform_later(target_hash)
      else
        # 見守り家族の通知をJOBキューに登録
        # LineNotificationJob.perform_at(target.family_notification_date_time, target)
        LineNotificationJob.set(wait_until: target.family_notification_date_time).perform_later(target_hash)
      end
    end
  end
end
```