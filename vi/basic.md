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

To simplify this, you can directly provide a page template when creating the renderer. Most of the time we will put the page template in its own file, e.g. `index.template.html`:

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
  console.log(html) // will be the full page with app content injected.
})
```

### Template Interpolation

The template also supports simple interpolation. Given the following template:

``` html
<html>
  <head>
    <!-- use double mustache for HTML-escaped interpolation -->
    <title>{{ title }}</title>

    <!-- use triple mustache for non-HTML-escaped interpolation -->
    {{{ meta }}}
  </head>
  <body>
    <!--vue-ssr-outlet-->
  </body>
</html>
```

We can provide interpolation data by passing a "render context object" as the second argument to `renderToString`:

``` js
const context = {
  title: 'hello',
  meta: `
    <meta ...>
    <meta ...>
  `
}

renderer.renderToString(app, context, (err, html) => {
  // page title will be "Hello"
  // with meta tags injected
})
```

The `context` object can also be shared with the ứng dụng Vue instance, allowing components to dynamically register data for template interpolation.

In addition, the template supports some advanced features such as:

- Auto injection of critical CSS when using `*.vue` components;
- Auto injection of asset links và resource hints when using `clientManifest`;
- Auto injection và XSS prevention when embedding Vuex state for client-side hydration.

We will discuss these when we introduce the associated concepts later in the guide.
