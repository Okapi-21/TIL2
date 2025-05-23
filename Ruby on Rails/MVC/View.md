## View

------

### helperメソッド

### form_with

------

フォームを作成するためのメソッド。以下のように記述される。

```
form_with model:@xxx, local:true do |f|
	f.label :name, class: xxxx
	f.text-field :name, class: xxxx
	
	f.label :description, class: xxxx
	f.text-area :description, class: xxxxx
	
	f.submit class: "btn btn-primary" 
```

- form_withはmodelオプションで指定したオブジェクト（＠xxx）の属性に基づいて、必要なフォームを自動生成してくれる。
- 「local:true」に関しては、フォームが通常のHTTPリクエストとして送信される。これにより、ページ全体がリロードされて、サーバーからのレスポンスを待つことになる。
- 「`do |f|`」の部分はブロック形式であり、`f`という変数を使ってフォームフィールドを生成するためのメソッドにアクセスするためのもの。（慣習的にf が使われているが、f以外でもOK）
- 「f.label :name」はフィールドのラベルを設定するためのもの
- 「f.text-field :name」ではフィールドの入力ボックスを作成している
- 「f.submit」では submit ボタンの生成を行なっている。



#### link_to 

------

HTMLのアンカータグ（aタグ）を生成を行う。このヘルパーを使用することで、Railsアプリケーション内でのページ間ナビゲーションが容易になる。

```
<%= link_to '表示名', パス名, class: "nav-link" %>
```

第一引数として、リンクが表示される名前を、第二引数として、パスを指定する事ができる（rails により指定されたものでも可能）。また、第二引数にはHTTPのメソッドも追加することもできる。
