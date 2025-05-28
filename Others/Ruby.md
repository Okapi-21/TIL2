## Ruby

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
ef chapter02_01(number)
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

