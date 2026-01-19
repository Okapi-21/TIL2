## Controller

------

#### コントローラー生成

```
docker compose exec web rails g controller コントローラー名
```

なお、コントローラー名に関しては命名規則により、通常は複数形で入力される。



1. #### コントローラーメソッド


------

```
before_action :xxxx
```

アクションが実行される前に指定されたメソッドを呼び出す。親クラスに指定しておくことで、その子にも適応可能。

```
require_login
```

ログインしているかどうかを確認するためのメソッド

```
def not_authenticated
    redirect_to login_path
  end
```

ログインしていない場合に指定されたパスにリダイレクトする。（今回はログインパス）

```
skip_before_action :require_login, only: %i[new create]
```

コントローラーにおいて特定のアクションが実行される前に設定されたフィルターをスキップするために使用される。今回はログイン前に new と create の動作を行う事ができる。



#### ユーザー登録のためのcontroller設定

------

```
class UsersController < ApplicationController
  skip_before_action :require_login, only: %i[new create]

  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
    if @user.save
      redirect_to root_path
    else
      render :new
    end
  end

  private

  def user_params
    params.require(:user).permit(:first_name, :last_name, :email, :password, :password_confirmation)
  end
end
```

ストロングパラメーター

 Railsでデータを一括で更新する際に、安全に更新するための仕組み。許可されていないデータの入力を防ぐために用いられる。許可されるデータは params により指定される。

#### ユーザーログインのためのcontroller設定

------

```
class UserSessionsController < ApplicationController
  skip_before_action :require_login, only: %i[new create]

  def new; end

  def create
    @user = login(params[:email], params[:password])

    if @user
      redirect_to root_path
    else
      render :new
    end
  end

  def destroy
    logout
    redirect_to root_path, status: :see_other
  end
end
```

status: :see_otherについて

　HTTPステータスコード303 "See Other"はリダイレクトを指示するコード。status: :see_otherを指定しない場合、リダイレクト後もブラウザがPOSTメソッドを保持し続け、予期せぬ挙動に繋がる危険がある。　
