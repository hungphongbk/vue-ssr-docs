# Hướng dẫn cơ bản

## Cài đặt

``` bash
npm install vue vue-server-renderer --save
```

Chúng tôi sử dụng hệ thống quản lý gói NPM xuyên suốt hướng dẫn này, nhưng thay vào đó bạn cũng có thể sử dụng [Yarn](https://yarnpkg.com/en/).

#### Ghi chú

- Khuyến khích việc sử dụng Node.js phiên bản 6+.
- `vue-server-renderer` và `vue` phải có cùng phiên bản.
- `vue-server-renderer` relies on some Node.js native modules và therefore can only be used in Node.js. We may provide a simpler build that can be run in other JavaScript runtimes in the future.

## Render một đối tượng Vue

``` js
// Bước 1: Tạo một đối tượng Vue
const Vue = require('vue')
const app = new Vue({
  template: `<div>Xin chào thế giới</div>`
})

// Bước 2: Tạo một đối tượng renderer
const renderer = require('vue-server-renderer').createRenderer()

// Bước 3: Render đối tượng Vue thành chuỗi HTML
renderer.renderToString(app, (err, html) => {
  if (err) throw err
  console.log(html)
  // => <div data-server-rendered="true">Xin chào thế giới</div>
})

// Trong phiên bản 2.5.0+, trả về một Promise nếu không có hàm callback nào được truyền vào:
renderer.renderToString(app).then(html => {
  console.log(html)
}).catch(err => {
  console.error(err)
})
```

## Tích hợp với Server

Sử dụng Vue SSR với server thuần Node.js cũng rất đơn giản, ví dụ như [Express](https://expressjs.com/):

``` bash
npm install express --save
```
---
``` js
const Vue = require('vue')
const server = require('express')()
const renderer = require('vue-server-renderer').createRenderer()

server.get('*', (req, res) => {
  const app = new Vue({
    data: {
      url: req.url
    },
    template: `<div>URL bạn đang truy cập là: {{ url }}</div>`
  })

  renderer.renderToString(app, (err, html) => {
    if (err) {
      res.status(500).end('Internal Server Error')
      return
    }
    res.end(`
      <!DOCTYPE html>
      <html lang="en">
        <head><title>Xin chào</title></head>
        <body>${html}</body>
      </html>
    `)
  })
})

server.listen(8080)
```

## Sử dụng template trang

Khi bạn render một ứng dụng Vue, đối tượng renderer sẽ chỉ tạo ra chuỗi kết xuất HTML cho ứng dụng. Trong ví dụ dưới đây chúng ta cần bao bọc kết xuất với một khung sườn HTML để tạo nên một nội dung trang HTML hoàn chỉnh.

``` html
<!DOCTYPE html>
<html lang="en">
  <head><title>Hello</title></head>
  <body>
    <!--vue-ssr-outlet-->
  </body>
</html>
```

Lưu ý đến comment `<!--vue-ssr-outlet-->` -- bắt buộc phải có comment này trong template bởi đây sẽ là nơi kết xuất HTML của renderer được chèn vào.

Chúng ta có thể đọc và truyền nội dung tệp vào đối tượng Vue renderer như sau:

``` js
const renderer = createRenderer({
  template: require('fs').readFileSync('./index.template.html', 'utf-8')
})

renderer.renderToString(app, (err, html) => {
  console.log(html) // biến `html` chứa toàn bộ nội dung HTML hoàn chỉnh với kết xuất của ứng dụng được chèn vào.
})
```

### Template Interpolation

> **Ghi chú**: trong Toán học, *"interpolation"* được dịch là *"phép nội suy"*. Tuy nhiên nội suy trong toán học lại khá khác biệt về định nghĩa từ *"interpolation"* trong lập trình, nó vốn dùng để chỉ một kĩ thuật tạo chuỗi bằng cách nhúng trực tiếp biến dữ liệu vào trong chuỗi. Vì sự khác biệt đó, xuyên suốt tài liệu này chúng tôi cũng sẽ giữ nguyên từ *"interpolation"*

Template cho phép hỗ trợ kĩ thuật interpolation khá đơn giản. Ví dụ như một template HTML phía dưới

``` html
<html>
  <head>
    <!-- sử dụng kí pháp ngoặc nhọn kép (double-mustache) cho phép interpolation với dữ liệu HTML-escaped -->
    <title>{{ title }}</title>

    <!-- sử dụng kí pháp ngoặc nhọn ba (triple-mustache) cho phép interpolation với dữ liệu không-HTML-escaped -->
    {{{ meta }}}
  </head>
  <body>
    <!--vue-ssr-outlet-->
  </body>
</html>
```

Chúng ta cung cấp dữ liệu nhằm cho phép interpolation một template bằng cách truyền vào một "đối tượng render context" tại tham số thứ hai trong hàm `renderToString`:

``` js
const context = {
  title: 'hello',
  meta: `
    <meta ...>
    <meta ...>
  `
}

renderer.renderToString(app, context, (err, html) => {
  // Tiêu đề trang sẽ là "Hello"
  // với các thẻ `meta` được truyền vào
})
```

Đối tượng `context` cũng có thể được chia sẻ với đối tượng ứng dụng Vue, cho phép các components có thể đăng kí động dữ liệu cho template interpolation.

Ngoài ra, template còn hỗ trợ một số tính năng cao cấp như:

- Tự động chèn vào các đoạn CSS tối quan trọng khi sử dụng `*.vue` component;
- Tự động chèn vào các đường dẫn và gợi ý tài nguyên tĩnh (resource hints) khi sử dụng `clientManifest`;
- Chống tấn công XSS khi nhúng trực tiếp đối tượng Vuex state vào kết xuất trong quá trình client-side hydration.

We will discuss these when we introduce the associated concepts later in the guide.
