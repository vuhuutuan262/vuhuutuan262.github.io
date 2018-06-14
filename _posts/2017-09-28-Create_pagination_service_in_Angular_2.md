---
layout: post
title: "Tạo Service Pagination trong Angular 2"
description: "Tạo Service Pagination trong Angular 2"
category: angular2
modified: 2017-09-28
tags: [angular2]
comments: true
share: true
imagefeature: pagination.jpg
imagefile: angular_2
featured: true
---

## Vấn đề
Mình mới làm quen với angular 1 thời gian, phải sử lý đến pagination.

Những thư viện của angular đều là load tất cả item vào pagination rồi xử lý, nên mình quyết định custom 1 cái service pagination dùng cho sướng ..

### Bắt tay làm nào

Mình làm với Ruby On Rails nên mình sẽ làm demo sử dụng gem Kaminari

### Follow sẽ như sau

![follow](/images/posts/pagination_angular_2/follow_pagination_service.jpg)

### Đầu tiên sẽ tạo 1 component tạo view và sử lý chung: **pagination.component.ts**

```typescript
  @Component({
    selector: 'app-pagination',
    templateUrl: 'pagination.component.html',
    styleUrls: ['pagination.component.scss']
  })
```

### File view pagination.component.html

```html
  <ul *ngFor="let page of pages" class="pagination">
    <li (click)="selectPagination(page)">{{ page }}</li>
  </ul>
```

Xây tạm xong phần view của pagination

Bây giờ sẽ sang phần **service paginate**

```typescript
 getPagesInPaginate(pages, currentPage:  number = 1,
   pageDistance: number = valSetting.DEFAULT_PAGE_DISTANCE,
   pageSize: number = valSetting.DEFAULT_PAGE_SIZE) {

    const listPages = [];
    const totalPages: number = Math.ceil(pages / pageSize);
    let startPage: number, endPage: number;

    if (totalPages < 2 * pageDistance) {
        startPage = 1;
        endPage = totalPages;
      } else {
        if (currentPage <= pageDistance) {
          startPage = 1;
          endPage = 2 * pageDistance + 1;
        } else if (currentPage + pageDistance > totalPages) {
          startPage = totalPages - 2 * pageDistance;
          endPage = totalPages;
        } else {
          startPage = currentPage - pageDistance;
          endPage = currentPage + pageDistance;
        }
      }
    for (let i = startPage ; i <= endPage; i++) {
      listPages.push(i);
    }
  }
```

**service sẽ trả ra 1 Array có số thứ tự của page( trên view sẽ lấy ra Array này để hiển thị pagination )** 

#### Ví dụ: 
Ở đây có pageDistance = 5 và có 15 trang thì kết quả sau khi đi qua service sẽ như sau

* [1] 2 3 4 5 6 7 8 9 10
* 1 [2] 3 4 5 6 7 8 9 10
* 1 2 [3] 4 5 6 7 8 9 10
* 1 2 3 [4] 5 6 7 8 9 10
* 1 2 3 4 [5] 6 7 8 9 10
* 1 2 3 4 5 [6] 7 8 9 10
* 2 3 4 5 6 [7] 8 9 10 11
* 3 4 5 6 7 [8] 9 10 11 12
* 4 5 6 7 8 [9] 10 11 12 13
* 5 6 7 8 9 [10] 11 12 13 14
* 6 7 8 9 10 [11] 12 13 14 15
* 6 7 8 9 10 11 [12] 13 14 15
* 6 7 8 9 10 11 12 [13] 14 15
* 6 7 8 9 10 11 12 13 [14] 15
* 6 7 8 9 10 11 12 13 14 [15]

Ở đây ta sử dụng Samples Component đế sử dụng service pagination Có 1 vấn đề là khi Samples truyền record.size để build pagination thì nó đã được build xong với pages = [ ], nên ta phải có sự kiện luôn lắng nghe khi pagination service trả lại số lượng pages

```typescript
  // Pagination.component.ts
  this.paginationService.getPages.subscribe(pages => {
    this.pages = pages;
  });
```

## Vậy là đã build xong, để sử dụng thế nào nhỉ

Tạo 1 **Samples.component** rồi import pagination.service

```typescript
  import { PaginationService } from '../services/pagination.service';
```

**Add pagination.component vào declarations và exports trong module để sử dụng**

Trong view Samples gọi seletor của pagination

```typescript
  <app-pagination></app-pagination>
```

Truyền record size vào pagination.service là xong

```typescript
  constructor(private paginationService: PaginationService) { }
  
  this.paginationService.getPagesInPaginate(totalPages);
```

Vậy là đã hiển thị xong pagination

**Bây giờ là event khi clicked Ta sẽ cần 1 sự kiện lắng nghe mỗi khi pagination được clicked tại pagination.service**

```typescript
  this.paginationService.getCurrentPages.subscribe(currentPage => {
    //To do ...
   });
```

Ta có thể lấy được pages mà vừa được click, truyền lên sever để lấy lại item mới

Nhớ **unsubscribe** sự kiện lắng nghe đi :D

### Bonus:

* Mình có sinh ra các nút go first page or go last page, các bạn có thể sử dụng or custom lại
* Vấn đề khi sử dụng pagination + selec , mình có giải quyết bằng 1 hàm reset current pages trong service, mỗi khi select xong nó sẽ để current_page đúng giá trị mong muốn

```typescript
  this.paginationService.resetCurrenPage(:number);
```

Các bạn có thể xem full [tại đây](https://github.com/vuhuutuan262/angular_pagination_with_kaminari)
