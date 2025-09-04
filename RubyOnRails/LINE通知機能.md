# LINE通知機能

## ログイン時に友達追加オプションを有効化

```ruby
# config/initializers/devise.rb
config.omniauth :line,
                Rails.application.credentials.dig(:line, :channel_id),
                Rails.application.credentials.dig(:line, :channel_secret),
                scope: "profile openid",
                bot_prompt: "normal"  # 友だち追加オプションを有効化
```

## Messaging APIの登録をしておく

LINEログインのチャネルに「リンクされたLINE公式アカウント 」項目があるので、設定しておく
ログイン時の友達追加のための作業

## Messaging API用のアクセスキーを登録する

```bash
#LINE認証に必要なクライアントIDとクライアントシークレット
# credentialsファイルを編集コマンド
EDITOR=vi rails credentials:edit

# credentials.yml.enc の内容例
line_message:
  channel_id: "your_channel_id"
  channel_secret: "your_channel_secret"
  channel_access_token: "your_access_token"
```

## 実装

```
gem "line-bot-api"
```

```ruby
# app/services/line_notification_service.rb

class LineNotificationService
  def send_line_message(user_id, message_text)
    # メッセージリクエストを作成
    push_request = Line::Bot::V2::MessagingApi::PushMessageRequest.new(
      to: user_id,
      messages: [
        Line::Bot::V2::MessagingApi::TextMessage.new(text: message_text)
      ]
    )

    # メッセージ送信
    response, status_code, headers = client.push_message_with_http_info(
      push_message_request: push_request
    )

    status_code == 200
  end

  private

  def client
    @client ||= Line::Bot::V2::MessagingApi::ApiClient.new(
      channel_access_token: Rails.application.credentials.dig(:line_message, :channel_access_token)
    )
  end
end
```

```ruby
# 使用時
# app/jobs/line_notification_job.rb
class LineNotificationJob < ApplicationJob
  queue_as :notifications
  sidekiq_options retry: false

  def perform
    user = User.find(1)
    line_notification_service = LineNotificationService.new
    line_notification_service.send_line_message(user.uid, "テストメッセージ")
  end
end
```

