<h1>Lazy Loading Module - Angular 2</h1>

<h2>1. Vấn đề</h2>

<hr>
Cấu trúc một simple app của chúng ta sẽ là như thế này:

<ul>
  <li>
    Chúng ta sẽ có một trang Home và trang Elements. Trang Elements sẽ chứa hết tất cả các components của bootstrap Bootstrap components. Đương nhiên trang Elements sẽ rất nặng, chủ đích đễ dễ thấy dự khác biệt sau khi áp dụng các kĩ thuật thô Apply lazy load cho Element component
  </li>
  <li>
    Đầu tiền chúng ta sẽ tách Elements component ra là một module riêng (Lưu ý: Một module có thể chứa một hoặc nhiều component nhé). Trong thực tế chúng ta nên module lại nhiều component cùng chung 1 chức năng.
  </li>
  <li>
    Ví dụ như module cho thông tin user (các component sửa thông tin, đổi mật khẩu, xem lịch sử,…) nó sẽ không phải lúc nào cũng cần tải lên để người dùng sử dụng. Chúng ta sẽ module nó và chỉ load khi nào người dùng gọi đến nó.
  </li>
</ul>

<pre style="display: block; margin-top: 0; margin-bottom: 1rem; font-size: 90%; color: white; background-color: black">
  <code class="language-JavaScript">
    import { NgModule, ModuleWithProviders } from '@angular/core';
    import { CommonModule } from '@angular/common';
    import { Routes, RouterModule } from '@angular/router';
    import { ElementsComponent } from './elements.component';

    //Định nghĩa router riêng cho module này
    const routing: Routes = [
      { path: '', component: ElementsComponent }
    ];

    //forChild -> Vì router này sẽ được load như một router con
    //cho nên chúng ta định nghĩa forChild cho router này
    const Routing: ModuleWithProviders = RouterModule.forChild(routing);

    @NgModule({
      imports: [
        CommonModule,
        Routing
      ],
      declarations: [
        ElementsComponent
      ]
    })
    export class ElementsModule { }
  </code>
</pre>
<h3>Các bạn chú ý chỗ loadChildren nhé. Đại loại khi trỏ tới link elements nó sẽ load module từ Elements Module.</h3>
<pre style="display: block; margin-top: 0; margin-bottom: 1rem; font-size: 90%; color: white; background-color: black">
  <code class="language-JavaScript">
    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule, ModuleWithProviders } from '@angular/core';
    import { FormsModule } from '@angular/forms';
    import { HttpModule } from '@angular/http';
    import { Routes, RouterModule } from '@angular/router';

    import { AppComponent } from './app.component';
    import { HomeComponent } from './home/home.component';
    import { ElementsComponent } from './elements/elements.component';

    const routing: Routes = [
      { path: '', component: HomeComponent },
      { path: 'home', component: HomeComponent },
      //Here: Chúng ta sử dụng loadChildren
      //và đưa đường dẫn đến module mà chúng ta sẽ apply lazyloading
      { path: 'elements', loadChildren: 'app/elements/elements.module#ElementsModule' }
    ];

  const Routing: ModuleWithProviders = RouterModule.forRoot(routing);

  @NgModule({
    declarations: [
      AppComponent,
      HomeComponent,
      ElementsComponent
    ],
    imports: [
      BrowserModule,
      FormsModule,
      HttpModule,
      Routing
    ],
    providers: [],
    bootstrap: [AppComponent]
  })
  export class AppModule { }
  </code>
</pre>
<ul>
  <li>
    Vì do khi app được load thì thằng element component không được load cùng nên => tốc độ được cải thiện. Do đó trong dự án trước khi bắt đầu code chúng ta nên module hóa hết tất cả các chức năng. Và càng chi tiết càng tốt.
  </li>
  <li>
    Vì để sau này khi dự án đã lớn gom lại theo module vẫn được nhưng sẽ rất tốn thời gian. Vì khi đó sẽ phát sinh lỗi vì khi dự án lớn các component khá chồng chéo, các services nhiều nơi, các thirt party controls sử dụng cho nhóm chức năng nào,…
  </li>
  <li>
    Nhưng việc này sẽ tạo ra một issue khác, đó chính là khoảng thời gian mà người dùng phải đợi sau khi click vào elements module. Đây là khoảng thời gian chết mà chúng ta đưa cho người dùng.
  </li>
  <li>
    Từ đó => chúng ta không nên để người dùng phải nhất thiết phải click vào thì chúng ta mới tải và execute chúng. => Khái niệm preload a module.
  </li>
</ul>

<h2>2. Áp dụng preload cho một module</h2>
Tiếp tục lại rất vui vì Angular đã support cho chúng ta làm việc này một cách thật đơn giản khi dùng chung với lazyloading.
<h3>2.1. Add PreloadAllModules từ @angular/router</h3>
<pre style="display: block; margin-top: 0; margin-bottom: 1rem; font-size: 90%; color: white; background-color: black">
  <code class="language-JavaScript">
    import { Routes, RouterModule, PreloadAllModules } from '@angular/router';
  </code>
</pre>
<h3>2.2 Định nghĩa lại cho router</h3>
<pre style="display: block; margin-top: 0; margin-bottom: 1rem; font-size: 90%; color: white; background-color: black">
  <code class="language-JavaScript">
    const Routing: ModuleWithProviders = RouterModule.forRoot(routing, { preloadingStrategy: PreloadAllModules });
  </code>
</pre>
<h4>
  Chú ý về angular complier Angular2 có hai cách biên dịch là JIT(just-in-time) và AOT(Ahead-of-time). Đại khái JIT sẽ biên dịch khi runtime còn AOT sẽ biên dịch trước toàn bộ và khi app run thì không cần phải tốn time để execute nữa. AOT được đưa ra sau này để cải thiện performace khi load page và nó được dùng để deploy app lên production. Các bạn có thể xem thêm ở đây Angular complier.
</h4>
Tài Liệu tham khảo <a href = "https://angular-2-training-book.rangle.io/handout/modules/lazy-loading-module.html">tại đây</a>
