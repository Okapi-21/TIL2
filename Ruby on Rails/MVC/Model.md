## Model

------



### Validation（データ制約）

------

```
lass User < ApplicationRecord
  authenticates_with_sorcery!

  validates :password, length: { minimum: 3 }, if: -> { new_record? || changes[:crypted_password] }
  validates :password, confirmation: true, if: -> { new_record? || changes[:crypted_password] }
  validates :password_confirmation, presence: true, if: -> { new_record? || changes[:crypted_password] }
  validates :first_name, presence: true, length: { maximum: 255 }
  validates :last_name, presence: true, length: { maximum: 255 }
  validates :email, presence: true, uniqueness: true
end
```



| 検証内容                                                     |          ヘルパーの使い方の例          |
| :----------------------------------------------------------- | :------------------------------------: |
| 必須のデータが入っているか                                   |     validates :foo, presence: true     |
| 数値を期待しているところに数値以外の値が入っていないか       |   validates :foo, numerically: true    |
| （数値の）範囲が適切か                                       |  validates :foo, inclusion: {in 0..9}  |
| 文字列の長さは適切か                                         | validates :foo, length: { maximum: 30} |
| データは一意か                                               |    validates :foo, uniqueness: true    |
| パスワードやメールアドレスがその確認用と内容が一致しているか |   validates :foo, confirmation: true   |

