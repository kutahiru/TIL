# PWA対応

Rails8.0系での対応
app/views/pwaに以下のファイルが作成されている

- manifest.json.erb
- service-worker.js

```ruby
# config/routes.rb
get "service-worker" => "rails/pwa#service_worker", as: :pwa_service_worker
get "manifest" => "rails/pwa#manifest", as: :pwa_manifest
```

## アイコンを用意する

各サイズのアイコンを用意して以下を修正
app/views/pwa/manifest.json.erb

## アプリケーションレイアウトの更新

```erb
# app/views/layouts/application.html.erb
<link rel="manifest" href="<%= pwa_manifest_path(format: :json) %>">
<meta name="apple-mobile-web-app-capable" content="yes">
```

## サービスワーカーの登録

```js
# app/javascript/controllers/application.js
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/service-worker.js')
      .then(registration => {
        console.log('Service Worker registered:', registration);
      })
      .catch(error => {
        console.log('Service Worker registration failed:', error);
      });
  });
}
```

