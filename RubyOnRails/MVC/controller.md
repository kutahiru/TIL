# controller

### レンダリングまでの仕組み

```ruby
def show
  #本来はこの処理で対応するviewをレンダリングしているが
  #Railsの規約により省略が可能となっている。
  render :show 
end
```

### 全アクションで共通する処理

全てのコントローラで共通する処理はapplicationコントローラにまとめる

```ruby
before_action :set_current_user

def set_current_user
  @current_user = User.find_by(id: session[:user_id])
end
```

### フィルタ(フック)

アクションの前後に処理を実施することができる。
アクションそれぞれで同じ実装をする必要がない。

```ruby
class BooksController < ApplicationController
  defore_action :set_book, only: [:show, :destroy]
end

private

def set_book
  @book = Book.find(params[:id])
end
```

### StrongParameters

ユーザーから送信されるParameterのうち、開発者が事前に定義したもののみをデータベースに保存する
MassAssignment機能の脆弱性への対応
StrongParameterはMassAssignmentで利用しても良いHashのキーを許可リストとして定義しておくことで、
想定していないパラメータの制限ができる。

```ruby
#MassAssignmentを使ってのデータの更新
def update
  user = current_user
  # params[:user] => {name: "bob", email: "bob@expample.com"}
  user.update(params[:user])
end

private

#外部からのパラメータをそのまま利用せずに、
#許可したキーのみ利用する。
def user_params
  params.require(:user).permit(:name, :email,)
  #mergeを使用することでフォームから渡されていない追加データの更新も可能。
  #以下はuserの名称、メールドレスがフォームから送信されたもの、現在のユーザーIDが追加データ
  params.require(:user).permit(:name, :email,).merge(user_id: current_user.id)
end
```

