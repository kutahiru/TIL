# Supabase

## 認証

### テーブル設計

ユーザーがログインすると、
auth.usersテーブルが自動で作成される。
トリガーでpublic.usersに同じidでレコードが作成される。
　対象テーブルはpublic.usersの外部キー参照で判断している