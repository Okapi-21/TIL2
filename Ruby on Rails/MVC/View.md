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



#### Xへのシェア機能

------

Xへのシェア機能は「ツイートするボタン」を設置し、クリック時にXの投稿画面を開くことによって簡単に実装することができる。

```ruby
<div class="mt-6">
    <% tweet_text = "#{@xxxxx}\n#{@xxxxx}\n#アプリ名称" %>
    <%= link_to "Xでシェアする", "https://twitter.com/intent/tweet?text=#{ERB::Util.url_encode(tweet_text)}", target: "_blank", rel: "noopener", class: "bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" %>
  </div>
```

1. 診断系アプリの場合、tweet_textへインスタンス変数を含めテキストを代入することで診断結果を示すことができる。
2. ボタン生成
   1. `ERB::Util.url_encode`で日本語や改行をURLエンコードしている。
      ＊URLエンコードとは：URLの中で使うことができない文字や日本語・記号などを、英数字の特別な形式に変換すること。
   2. `target: "_blank"`で新しいタブで開く。
   3. `rel: "noopener"`はセキュリティ対策。
   4. `class: ...`はボタンの見た目を整えるためのTailwind CSS例（変更OK）。



### slim

------

　webアプリケーションはブラウザで利用することになるので、最終的に画面をHTMLで出力することになる。railsを使った開発では、テンプレートエンジンを使用することにより、アプリケーションの画面をHTMLの構造が直観的にわかりやすい形でコードを書くことができる。テンプレートエンジンとは、HTMLのテンプレート（ひな形）とそこに記述された動的な処理から、最終的なHTMLを生成することのできる仕組みのことである。
　railsはデフォルトでERBというテンプレートエンジンを採用しているが、開発現場ではHTMLをツリー構造で簡潔に表現することができるHamlやSlimといった別のテンプレートエンジンが使用されることが多い。
　slimを利用するためには二つのgemをアプリケーションに追加しなくてはならない。一つはslimのジェネレータを提供してくれる slim-rails というgem、もう一つはERB形式のファイルをslim形式に変換してくれる erb2slimコマンドを提供してくれる html2slim というgemである。

