---
layout: post
title: "Có thể bạn đã biết - Ruby"
description: "Có thể bạn đã biết - Ruby"
category: ruby
modified: 2018-06-18
tags: [ruby]
comments: true
share: true
imagefeature: stupid-icon.png
imagefile: ruby
featured: true
---

Hôm nay nhân một ngày đẹp trời mình xin được giới thiệu cho các bạn top những tricks trong Ruby mà những bậc cao nhân hay dùng để tối ưu code hay đôi khi chỉ là làm cho nó nhìn vui hơn (khó đọc hơn) 

## Strings

Sử dụng **%{}** hay **%q{}**

```ruby
>> %{}
=> ""
>> %Q{}
=> ""
>> %{Tuan}
=> "Tuan"
>> %{Tuan #{2017 + 1}}
=> "Tuan 2018"
```

Bên cạnh đó **%q{}** là cách tương tự như việc tạo ra một single quote string, và do đó không thể chứa string interpolation.


```ruby
>> %q{}
=> ""
>> %q{Tuan}
=> "Tuan"
>> %q{Tuan #{2017 + 1}}
=> "Tuan \#{2017 + 1}"
```

## Arrays

**%w** có thể dùng tương tự như sử dụng **String#split**. Nó lấy một string và tách nó thành một mảng dựa vào các khoảng trắng (white space). Tuy nhiên khoảng trắng này cũng có thể được escape. **%W** cũng tương tự nhưng nó cho phép string interpolation.

```ruby
>> %w(a b c)
=> ["a", "b", "c"]
>> %w(a\ b c)
=> ["a b", "c"]
>> %w(a b c #{2017 + 1})
=> ["a", "b", "c", "\#{2017 + 1 }"]
>> %W(a b c #{2017 + 1})
=> ["a", "b", "c", "2018"]
```

## Regexes

Sử dụng **%r** có thể thay thế cho cặp **//** để tạo một đối tượng regexes. Thường cách viết này được dùng khi chúng ta không muốn phải escape dấu **/** chứa bên trong regexes.

```ruby
>> %r{a|b}
=> /a|b/
>> %r{a/b}
=> /a\/b/
```

## Symbol

**%s** có thể tạo symbol thay vì viết symbol như thế này **:foo**. Cũng như những trường hợp trên, cách viết này loại bỏ vấn đề escaping symbol, tuy nhiên không giống với **:""**, nó không cho phép string interpolation.

```ruby
>> %s(a)
=> :a
>> :"a #{2017 + 1}"
=> :"a 2018"
>> %s{a #{2017 + 1}}
=> :"a \#{2017 + 1}"
```

## Shelling out

Như ta đã biết, khi text được bao bọc bởi backquotes (kí tự **`**, hay còn gọi là backticks), thì đoạn text được xem như là một literal của double-quoted string. Giá trị của literal đó được truền tới một phương thức đặc biệt tên là Kernel. Phương thức này thực thi dòng lệnh như một câu lệnh shell trên hệ điều hành và trả về kết quả là một string. Thay vì sử dụng backticks ta có thể dùng **%x**, và cũng giống như backticks nó cung cấp cơ chế string interpolation.

```ruby
>> `echo hi`
=> "hi\n"
>> %x{echo hi}
=> "hi\n"
>> %x{echo hi #{2017 + 1}}
=> "hi 2018\n"
```

## Array#join

Chúng ta đã nhiều lần được nhìn thấy việc sử dụng Array#* với một số, để nhân kích thước mảng lên bằng cách thêm vào duplicate của nó.

```ruby
>> %w{a b c} * 2
=> [a, b, c, a, b, c]
```

Nhưng bạn có biết rằng thay vì sử dụng số mà dùng một string làm đối số của Array#* thì nó sẽ thực hiện một phép kết nối?

```ruby
>> %w{a b c} * "2"
=> "a2b2c"
```

## Lambda literal

Một cách đơn giản và được sử dụng tương đối nhiều trong thời gian gần đây để định nghĩa scope trong Rails, đó là sử dụng kí hiệu ->, hay chính là hiện diện của Lambda Literal, cho phép bạn dễ dàng tạo ra một biểu thức lambda.

```ruby
>> a = ->{2017 + 1}
=> #<Proc:0x0000000009121458@(pry):4 (lambda)>
>> a.call
=> 2018
```
## Tham số cho method

```ruby
  def method  a, *b, **c
      return a, b, c
  end
```

Trong đó a sẽ là một tham số bình thường. *b sẽ nhận tất cả các tham số truyền vào đứng phía sau cái đầu tiên và đặt chúng vào một mảng. **c sẽ nhận bất cứ tham số nào truyền vào dưới dạng key: value ở cuối của lời gọi hàm.

```ruby
method 1
# => [1, [], {}]
method 1, 2, 3, 4
# => [1, [2, 3, 4], {}]
method 1, 2, 3, 4, a: 1, b: 2
# => [1, [2, 3, 4], {:a=>1, :b=>2}]
```

## Xử lý một object đơn giống như một mảng

Đôi khi trong metaprogramming bạn sẽ gặp nhiều trường hợp phải handle dữ liệu một cách linh hoạt, ví dụ với trường hợp này ta muốn xử lý một object đơn hoặc một mảng các objects. Thay vì phải kiểm tra trường hợp loại option nào được truyền vào, bạn có thể sử dụng [*something] hoặcArray(something).

```ruby
num = 1
num_arr = [1, 2, 3]
```

Giả sử rằng bạn muốn lặp qua các phần tử dù cho cái nhận vào chỉ là một object đi chăng nữa, bạn có thể dùng:

```ruby
[*num].each { |s| s }
[*num_arr].each { |s| s }

//OR

Array(num).each { |s| s }
Array(num_arr).each { |s| s }
```

## Tham số Hash dạng bắt buộc

Cái này có bắt đầu từ Ruby 2.0. Thay vì chỉ định nghĩa ra mọt phương thức nhận một hash làm tham số như thế này

```ruby
def method {}
end
```

Bạn có thể chỉ định keys bắt buộc thậm chí cả giá trị mặc định cho chúng. Như ví dụ dưới đây thì a và b là các khóa bắt buộc, trong khi c là optional.

```ruby
def method a:, b:, c: \'default\'
  return a, b, c
end
```

Chúng ta thử gọi hàm chỉ với a: 1 thì sẽ thấy không được vì còn thiếu khóa b:

```ruby
method a: 1
# => ArgumentError: missing keyword: b
```

Bạn cũng có thể thay đổi giá trị của c bằng cách truyền vào tất cả chúng

```ruby
method a: 1, b: 2, c: 3
# => [1, 2, 3]

hash = {a: 1, b: 2, c: 3}
method hash
# => [1, 2, 3]
```

## String concatenation

Bạn nên bỏ thói quen nối string bằng += mà hãy dùng << thay thế. Mặc dù kết quả là hoàn toàn giống nhau nhưng điểm khác biệt ở đây là gì?

```ruby
str1 = "first"
str2 = "second"
str1.object_id       # => 16241320

str1 += str2    # str1 = str1 + str2
str1.object_id  # => 16241240, id is changed

str1 << str2
str1.object_id  # => 16241240, id is the same
```
Khi bạn dùng **+=** Ruby tạo ra một đối tượng tạm thời là kết quả của str1 + str2. Sau đó nó thay thế str1 bằng reference đến đối tượng vừa được tạo. Trong khi đó **<<** sẽ chỉnh sửa trên chính xâu gốc.

Do vậy việc sử dụng **+=** tạo ra một vài bất lợi sau đây:

* Thêm công việc tính toán để nối string
* Một đối tượng string bị thừa trong bộ nhớ (giá trị cũ của str1)

Hy vọng giúp ích cho các bạn
