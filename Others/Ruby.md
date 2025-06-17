## Ruby

------

### 配列・ハッシュ

------

- 配列.shift 
  配列.shift は、配列の先頭の要素を取得し、削除するメソッドのこと。

  ```ruby
  a = [1, 2, 3]
  a.shift # => 1が返る
  a = [2, 3]
  ```

- 配列.delete
  **指定した値と等しい全ての要素を配列から削除**

  ```ruby
  a = [10, 20, 30, 20, 40]
  a.delete(20) # => 20（配列は [10, 30, 40] になる、2つとも消える）
  a.delete(99) # => nil（配列は変わらない）
  ```

- 配列.join( )
  配列.join( )は、配列の要素を一つの文字列として出力するためのメソッド。繋げる時の「区切り文字」についても、配列.join("x")のように記述することで指定することができる。

  ```ruby
  a = [1, 2, 3]
  a.join(",")
  puts a # => "1, 2, 3" 
  ```

- 配列.include?(x)
  そのオブジェクトが指定した要素を含んでいるかどうかを判定し、
  含んでいれば`true`、含んでいなければ`false`を返します。
  
  ```ruby
  arr = [1, 2, 3, 4, 5]
  p arr.include?(3)   #=> true
  p arr.include?(9)   #=> false
  ```
  
- 配列.index(x)
  `arr.index(値)` は、**最初に値が現れるインデックス（0始まり）**を返します

  ```ruby
  arr = ["apple", "banana", "orange", "banana"]
  p arr.index("banana")   # => 1
  ```
  
- 
  
- 応用例
  ```ruby
  h.times { board << gets.chomp.chars }
  ```
  
  - **h行分の迷路や盤面を1行ずつ入力として受け取り**
  - **それぞれを「1文字ずつの配列」に変換**
  - **`board`配列に入れていく**

#### setについて

　Set（セット）は、**「重複しない値の集まり」を表現するデータ構造**を示す。配列と似ているが、「同じ値」を入れたとしても結果が一つだけになるのが特徴としてある。また、配列とは異なり、要素の順序が存在しない。

rubyでsetを使うには以下のように「 require "set" 」と記述する必要がある。
```ruby
require "set"

s = Set.new
s.add(1)
s.add(2)
s.add(1)  # 2回目の1は無視される

p s   # => #<Set: {1, 2}>
```

要素の重複を防ぐので検索対象などはset化して使われることがある。

| 役割         | データ型 | 理由                                        |
| :----------- | :------- | :------------------------------------------ |
| 検索対象(A)  | Set      | 含むかどうかの判定が高速＆重複がないから    |
| 検索したい値 | Array    | 順番を保ったまま1つずつ調べて出力したいから |



#### mapメソッドについて

- 配列のすべての要素に同じ処理をして、新しい配列を作るためのメソッド
- もとの配列の要素はそのままで、処理後の値を集めた新しい配列が返る

```ruby
numbers = [1, 2, 3]
doubled = numbers.map { |n| n * 2 }
puts doubled  # => [2, 4, 6]
```

`numbers.map { |n| n * 2 }` は、「numbersの各要素nを2倍した値を新しい配列にして返す」という意味

#### ＊ map(& : メソッド名 ) の意味

通常の書き方

```ruby
lines = ["abc\n", "def\n", "ghi\n"]
chomped = lines.map { |line| line.chomp }
# これは各要素に対してchompメソッドを実行しています
```

`map(&:chomp)` は、上のコードと**まったく同じ意味**（省略記法を用いている）（各要素に対してchompを呼び出して、という意味）

- `&:chomp` は、`{|x| x.chomp }` というブロックを作って、`map`に渡す省略記法
- `&`は「ブロックとして渡すよ」という意味、`:chomp`は「シンボル（メソッド名）」




### 条件分岐

------

・fizz buzz!

自己回答

```ruby
def chapter02_01(number)
  # コードを記載
  n = number
  for i in 1..n do
    if (i % 15 == 0)
      puts "FizzBuzz!"
    elsif ( i % 3 == 0)
     puts "Fizz!"
    elsif (i % 5 == 0)
      puts "Buzz!"
    else
      puts i
    end
  end
end
chapter02_01(15)
```

模範解答

```ruby
def chapter02_01(number)
  number.times do |num|
    n = num + 1
    if n % 3 == 0 && n % 5 == 0
      puts '"FizzBuzz!"'
    elsif n % 3 == 0
      puts '"Fizz!"'
    elsif n % 5 == 0
      puts '"Buzz!"'
    else
      puts n
    end
  end
end
chapter02_01(20)
```



### データの受け取り

------

#### getsとreadlineの違い

- gets

  -  標準入力から１行だけを読み込むメソッド
  - 戻り値は文字列
  - 改行文字も含めて出力される（必要な場合は .chompで削除）

  ```ruby
  input = gets      # 例: ユーザーが「hello」と入力→ inputには"hello\n"が入る
  input = gets.chomp # "hello"
  ```

- readline

  - `gets`よりも高度な入力機能（コマンド履歴、補完機能など）が使える（`readline`ライブラリをrequireする必要あり）。
  - デフォルトでは改行を含まない
  - getsと同様に標準入力から１行を読み取る
  - 複数行読み取りを行いたい場合は readlines を用いる必要がある

  ```ruby
  require "readline"
  
  input = Readline.readline  # ユーザーが「hello」と入力→ inputには"hello"が入る（改行は含まれません）
  ```

  

### 正規表現

------

正規表現は / パターン / という形で表される。（例 ↓）

```ruby
text = "abc123"
if text =~ /123/
  puts "数字が見つかった！"
end
```

#### よく使われる正規表現の記号

| 記号     | 意味                     | 例                       |
| -------- | ------------------------ | ------------------------ |
| `.`      | 何か1文字                | `/a.c/` → "abc"OK        |
| `*`      | 直前のものが0回以上      | `/a*/` → "", "a", "aa"OK |
| `+`      | 直前のものが1回以上      | `/a+/` → "a", "aa"OK     |
| `?`      | 直前のものが0回か1回     | `/a?/` → "", "a"OK       |
| `[abc]`  | aかbかcのどれか1文字     | `/[abc]/`                |
| `[^abc]` | a,b,c以外の1文字         | `/[^abc]/`               |
| `[0-9]`  | 0〜9のどれか1文字        | `/[0-9]/`                |
| `\d`     | 数字1文字（[0-9]と同じ） | `/\d/`                   |
| `\w`     | 英数字1文字              | `/\w/`                   |
| `\s`     | 空白文字1文字            | `/\s/`                   |
| `^`      | 行頭                     | `/^abc/`                 |
| `$`      | 行末                     | `/abc$/`                 |
| `{n}`    | 直前のものがn回          | `/a{3}/` → "aaa"OK       |
| `{n,m}`  | n回以上m回以下           | `/a{2,4}/`               |
| `( )`    | グループ化               | `/(abc)+/`               |

#### ちょっと応用

1. 文字列を三文字ずつ区切って出力したい場合

```ruby
"abcdefg".scan(/.{1,3}/) #=> ["abc", "def", "g"]
```

| 記号    | 意味                       | 例         |
| ------- | -------------------------- | ---------- |
| `.`     | 何か1文字（なんでもOK）    | `/a.c/`    |
| `{n,m}` | 直前のものがn回以上m回以下 | `/a{2,4}/` |

この二つの組み合わせで表現することができる。

2. 1.の文字数が３の倍数でないver

   ```ruby
   str = "1234567890"
   ary = str.reverse.scan(/.{1,3}/).map(&:reverse).reverse
   puts ary.join(",")  # => 1,234,567,890
   ```

   - まずreverseして後ろから3つずつにする
   - scanで3文字ずつ配列化
   - それぞれreverseして元に戻す（配列の要素内の数字の順番 + 配列要素の順番） 
   - 最後にjoinでカンマ区切り
     

#### Rubyでよく使う正規表現メソッド

- `=~` : パターンにマッチするか調べる
- `scan` : パターンに合う部分をすべて配列で抜き出す
- `sub` : 最初にヒットした部分だけ置換
- `gsub` : ヒットした全部を置換
- `match` : 詳しいマッチ結果を得る



#### 破壊的

------

「破壊的（はかいてき）」という言葉は、プログラミングの分野では**「元のデータやオブジェクトそのものを変更する」**という意味で使われる。

- **破壊的処理**：メソッドや関数が「呼び出されたオブジェクト自体」を直接書き換えること。

  　　　　　　これにより**元のデータが失われたり、変わったりする**。

- **非破壊的処理**：元のデータはそのままで、新しいデータ（コピーや新しいインスタンス）を返す。

　＊rubyはデフォルトで非破壊的処理を返すので、メソッドに「！」をつけて返すことで、破壊的処理を行うことができる。

　＊破壊的処理を使うときは、**元のデータが変わって困らないか**よく考える必要がある



### 構造体・クラス

------

クラスを活用しているコードの解釈

```ruby
# 社員クラスの定義
class Employee
 #インスタンス変数を作成するための記述
  def initialize(number, name)
    @number = number
    @name = name
  end

  # メンバメソッド
  def getnum
    @number
  end

  def getname
    @name
  end
end

# 社員のリストを保存する配列
employees = []

# コマンド受付

while line = gets #コマンドが入力されている限りはtrue
  cmd = line.chomp.split(" ") #１行で入力されているコマンドを空白区切りの配列とみなして変数 cmd に格納
  #cmdの先頭の要素によって場合分けを行う。
  if cmd[0] == "make"　#makeの場合
    number = cmd[1].to_i　#二つ目の要素を整数型に変更して、number変数に代入
    name = cmd[2]　#三つ目の要素をname変数に代入
    employees << Employee.new(number, name)　#Employeeにより(number, name)の要素を持つ構造体にして、配列employeesに代入する
  elsif cmd[0] == "getnum"　#getnumの場合
    n = cmd[1].to_i　#二つ目の要素を整数型に変更して、n変数に代入
    puts employees[n - 1].getnum #makeにより生成したインスタンス変数はemployeesに格納されているため、getnumを用いて出力
  elsif cmd[0] == "getname"　#getnameの場合
    n = cmd[1].to_i　#二つ目の要素を整数型に変更して、n変数に代入
    puts employees[n - 1].getname #makeにより生成したインスタンス変数はemployeesに格納されているため、getnameを用いて出力
  end
end
```

#### 継承

paiza問題（https://paiza.jp/works/mondai/class_primer/class_primer__inheritance）

```ruby
class Customer
    def initialize(age)
        @age = age 
        @fee = 0
        @alcohol_ordered = false 
    end 
    
    def softdrink(fee)
        @fee += fee 
    end
    
    def food(fee)
        @fee += fee 
    end
    
    def alcohol(fee) 

    end 
    
    def total
        @fee 
    end 
end

class AdultCustomer < Customer 
    def alcohol(fee) 
        @fee += fee 
        @alcohol_ordered = true 
    end 
    
    def food(fee)
        if @alcohol_ordered
            @fee += [fee - 200, 0].max
        else 
            @fee += fee 
        end
    end
end 
        
num, orders = gets.split(" ").map(&:to_i) 

customers = []

num.times do 
    age = gets.to_i 
    if age >= 20 
        customers << AdultCustomer.new(age)
    else 
        customers << Customer.new(age)
    end 
end 


orders.times do 
    order = gets.split(" ")
    customer_num = order[0].to_i 
    order_content = order[1]
    order_fee = order[2].to_i
    if order_content == "food" 
        customers[customer_num -1].food(order_fee)
    elsif order_content =="softdrink"
        customers[customer_num -1].softdrink(order_fee)
    elsif order_content =="alcohol"
        customers[customer_num -1].alcohol(order_fee)
    end 
end

(0...num).each do |i| 
    puts customers[i].total 
end 
    
```



#### 静的メンバ

```ruby
class Customer 
    def initialize(age)
        @age = age 
        @fee = 0 
        @order_alcohol = false 
    end 
    
    def food(fee) 
        @fee += fee 
    end 
    
    def softdrink(fee) 
        @fee += fee 
    end 
    
    def alcohol 
        
    end 
    
    def total 
        @fee 
    end 
end 

class AdultCustomer < Customer 
    def alcohol(fee)
        @fee += fee
        @order_alcohol = true 
    end 
    
    def food(fee)
        if @order_alcohol 
            @fee += fee -200
        else 
            @fee += fee 
        end 
    end
end 

num, orders = gets.split(" ").map(&:to_i)
customers = []
leaved_customer = 0

num.times do
    age = gets.to_i 
    if age >= 20 
        customers << AdultCustomer.new(age) 
    else 
        customers << Customer.new(age) 
    end 
end


orders.times do 
    order = gets.split(" ")
    order_num = order[0].to_i - 1
    order_content = order[1]
    next if customers[order_num].nil?
      case order_content
      when "softdrink"
        customers[order_num].softdrink(order[2].to_i)
      when "food"
        customers[order_num].food(order[2].to_i)
      when "alcohol"
        customers[order_num].alcohol(order[2].to_i)
      when "0"
        customers[order_num].alcohol(500)
      when "A"
        puts customers[order_num].total
        leaved_customer += 1
        customers[order_num] = nil
    end
end

puts leaved_customer
   
```



### 計算量の意識

------

計算量を考えない場合

```ruby
num, events = gets.split(" ").map(&:to_i)
ary = []

num.times do 
    ary << gets.chomp
end 

events.times do 
    comand, name = gets.split(" ")
    if comand == "join"
        ary << name 
    elsif comand == "leave"
        ary.delete(name)
    elsif comand == "handshake"
        puts ary.sort 
    end 
end 
```

計算量を考える場合（保留）

```ruby
require 'set' #Setの呼び出し

n, k = gets.split.map(&:to_i)
names = Set.new #配列ではなくSetで要素を管理
n.times { names << gets.chomp }

k.times do
  event = gets.chomp
  if event == "handshake"
    names.to_a.sort.each { |name| puts name }　#集合.to_a（配列化）.sort（並び替え）.each { |name| puts name}
  else
    command, name = event.split
    if command == "join"
      names << name
    else # leave
      names.delete(name)
    end
  end
end
```

