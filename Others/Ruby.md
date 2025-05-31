## Ruby

------

### 配列

------

- 配列.shift 
  配列.shift は、配列の先頭の要素を取得し、削除するメソッドのこと。

  ```ruby
  a = [1, 2, 3]
  a.shift # => 1が返る
  a = [2, 3]
  ```

  

  

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

