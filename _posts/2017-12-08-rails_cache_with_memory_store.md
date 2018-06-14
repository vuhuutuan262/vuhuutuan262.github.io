---
layout: post
title: "Rails cache với memory store"
description: "Rails cache với memory store"
category: rails
modified: 2017-12-08
tags: [rails]
comments: true
share: true
imagefeature: rails-cache.jpg
imagefile: rails
featured: true
---

## Trong vài ngày gần đây nảy sinh 1 vấn đề nhỏ với việc truy vấn dư thừa, sau khi đã giải quyết được vấn đề mình viết bài viết này.

## Vấn đề đặt ra:
*Giả sử như bạn có một menu gồm list các category kèm theo là số phần tử trong mỗi category đó, và bạn luôn hiển thị nó, tuy nhiên số phần tử trong mỗi category biến động theo khoảng thời gian nhất định (30 phút, 2h, hoặc có thể là 1 ngày, etc). Việc truy vấn để tính size trong category có lẽ sẽ lãng phí, vì thường chúng ta sẽ đặt hàm tính này trong before_fillter (tùy trường hợp thì có thể giải quyết bài toán này với counter cache hoặc không thể) và mỗi lần request thì việc truy vấn xuống DB sẽ xảy ra. Các bạn có thể xem hình để hình dùng rõ hơn*.

Ở đây dữ liệu của mình sẽ được crawling về 1 ngày 1 lần vì vậy mình chắc chắn rằng trong 1 ngày nếu có đăng nhập vào và làm bất kỳ việc gì để xem thì không làm thay đổi các con số này, vậy tại sao phải có quá nhiều truy vấn lãng phí nhỉ?

==> **Vấn đề này có thể được giải quyết với Cache key trên Rails dùng Memory Store.**

### Kỹ thuật:
* Việc ta hi sinh memory để chứa 1 vài biến sẽ tốt hơn nhiều so với việc truy suất db để tính toán và trả về giá trị (tốt về thuật toán và cả tốc độ, etc…)

* Hiện nay memcache đang được sử dụng ở hầu hết các site lớn, nên việc chúng ta áp dụng vào các ứng dụng không có gì là độc đáo nữa, mà dần trở thành 1 việc nên làm.

### Cách thực hiện:

Mình sẽ trình bày code của mình làm ví dụ thực tế để có thể giải thích rõ hơn cho các bạn:

– Trước tiên muốn thực hiện cache cần vào application.rb và thêm dòng sau:


```ruby
	
config.cache_store = :memory_store
```

Đây là default thì cache sẽ chiếm 32 MB, nếu bạn muốn thêm có thể thêm option vào:
```ruby
config.cache_store = :memory_store, {size: 64.megabytes}
```


Đoạn code lúc đầu khi thực hiện load menu (gây lãng phí) như sau:
```ruby 
def load_menu
   @tags_menu = {}
   tags = Tag.all
   tags.each do |tag|
     @tags_menu[tag.name] = tag.pod_casts.size
   end
   @tags_menu = @tags_menu.to_a
 end
```

Đoạn code sau khi dùng cache (cách read, write bình thường):
```ruby
 def load_menu
    @tags_menu = {}
    tags = Tag.all
    tags.each do |tag|
      if Rails.cache.read("#{tag.name}") == nil
        Rails.cache.write("#{tag.name}", tag.pod_casts.size, :expires_in => 12.hours)
        @tags_menu[tag.name] = Rails.cache.read("#{tag.name}")
      else
        @tags_menu[tag.name] = Rails.cache.read("#{tag.name}")
      end
    end
    @tags_menu = @tags_menu.to_a
  end
```
  
OK. Vấn đề đã được giải quyết rồi, giờ mình giải thích một số để các bạn hiểu thêm về phương thức read, write, fetch, delete.

Như tên gọi của chúng

* **read** là đọc dữ liệu trong cache ra, tham số nhận vào là key lúc bạn ghi xuống, nếu có giá trị thì trả về, không trả về nil.

* **write** là ghi dữ liệu vào cache, tham số truyền vào là key, value, và option (ở đây bạn có thể thấy rằng option mình chỉ định là thời gian hết hạn của key này – 12 tiếng sau khi nó tạo ra).

Vậy fetch là gì nhỉ?

* **fetch** thì giống với **read** tuy nhiên fetch có cái hay là khi bạn **fetch** 1 key và trả về nil thì bạn có thể thực hiện code trong block.

giờ mình sử dụng block cho **fetch** như sau:

```ruby
 def load_menu
    @tags_menu = {}
    tags = Tag.all
    tags.each do |tag|
      Rails.cache.fetch('test') do
        Rails.cache.write("#{tag.name}", tag.pod_casts.size, :expires_in => 12.hours)
      end
      @tags_menu[tag.name] = Rails.cache.read("#{tag.name}")
    end
    @tags_menu = @tags_menu.to_a
  end
```
  
* Còn **delete** thì quá rõ rồi, xóa key trong cache bạn nhé, phương thức nhận đối số truyền vào là key
