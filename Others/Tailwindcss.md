## Tailwindcss

------

公式 : https://tailwindcss.com/

チートシート

英語版：https://umeshmk.github.io/Tailwindcss-cheatsheet/
　＊breakpoint などがひとまとめに載っているので便利

日本語版：https://telehakke.github.io/tailwindcss-japanese-cheat-sheet/

#### よく使うもの

------

##### padding  （余白調整）

###### https://tailwindcss.com/docs/padding

![img](https://popshub.s3.amazonaws.com/uploads/ckeditor/pictures/15867/content_image-1695009349241.jpg)

![img](https://popshub.s3.amazonaws.com/uploads/ckeditor/pictures/15868/content_image-1695014463575.jpg)

##### margin　（余白調整）

https://tailwindcss.com/docs/margin
![img](https://popshub.s3.amazonaws.com/uploads/ckeditor/pictures/15871/content_image-1695017660267.jpg)

![img](https://popshub.s3.amazonaws.com/uploads/ckeditor/pictures/15872/content_image-1695019029675.jpg)

##### container　（コンテンツ幅の設定）

https://tailwindcss.com/docs/container

##### Background Color　（背景色の設定）

https://tailwindcss.com/docs/background-color

##### flex

https://tailwindcss.com/docs/display

##### width　（グリッド分割）

https://tailwindcss.com/docs/width
![img](https://popshub.s3.amazonaws.com/uploads/ckeditor/pictures/15887/content_image-1695103395179.jpg)



#### 画面いっぱいに表示したいとき（フッターが上に来すぎてしまう）

------

1. min-h-screen
   TailwindのCSSクラス min-h-screen を一番外側のdivタグにつけて、「最低でも画面の高さ」まで要素を広げる

2. flex flex-col
   flex ： flex ボックスというCSSレイアウトの設定方法の指定
   flex-col：縦方向に要素を並べる指定

   ＊これも一番外側のdivタグにつけて画面表示のルールを作る

3. flex-1
   `flex-1`は「余ったスペースを全部使って伸びる」指定
   例えば、`flex flex-col`の親の中で、`flex-1`を付けた子要素は**残りの高さを全部使って広がる**。
   その結果、フッターが常に一番下に来て、下の余白が自然に埋まる



#### Responsive

------

https://skillhub.jp/courses/261/lessons/2083

rails ではデフォルトで ApplicationController に 「allow_browser versions: :modern」が設定されているので削除するとレスポンシブに
（modern（モダン）なブラウザのみ許可する」という制限。一部のスマートフォンやタブレット、または古いブラウザでは406エラー（unsupported browser）が表示される原因になる）



