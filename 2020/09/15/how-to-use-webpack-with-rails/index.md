# 如何在Rails裡使用Webpack?

簡單，但是有些小地方要注意...
<!--more-->
最近在寫一些練習的小專案，沒想到在設定Webpacker這個步驟卡了很久

所以打算把設定的步驟自己寫下來避免下次又忘了

## Webpack & Webpacker
[Webpack](https://webpack.js.org/)是一個靜態模組打包工具，幫你把有複雜依賴關係的js, css, scss...打包成靜態的檔案

[Webpacker](https://github.com/rails/webpacker)是一個Ruby gem，讓我們在Rails可以方便的使用Webpack處理Webpack可以處理的檔案

以下範例再建立一個Rails專案後，建立一個controller和一個view，並且設定Webpacker引用Bootstrap的CSS和JS到view裡

## 建立專案
* Ruby的版本是2.7.1

* Rails的版本是6.0.3.3

1. 首先輸入以下命令開新專案
```shell
rails new webpack_example
```
2. 接下來generate只有一個action的controller，記得先cd進去
```shell
❯ rails g controller page index
Running via Spring preloader in process 50838
      create  app/controllers/page_controller.rb
       route  get 'page/index'
      invoke  erb
      create    app/views/page
      create    app/views/page/index.html.erb
      invoke  test_unit
      create    test/controllers/page_controller_test.rb
      invoke  helper
      create    app/helpers/page_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    scss
      create      app/assets/stylesheets/page.scss
```
3. 打開config/routes.rb改成以下內容
```ruby
Rails.application.routes.draw do
  root 'page#index'
end
```
4. 打開app/views/page/index.html.erb貼上以下[Bootstrap Navbar](https://getbootstrap.com/docs/4.0/components/navbar/#supported-content)的範例code
```html
<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <a class="navbar-brand" href="#">Navbar</a>
  <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
    <span class="navbar-toggler-icon"></span>
  </button>

  <div class="collapse navbar-collapse" id="navbarSupportedContent">
    <ul class="navbar-nav mr-auto">
      <li class="nav-item active">
        <a class="nav-link" href="#">Home <span class="sr-only">(current)</span></a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Link</a>
      </li>
      <li class="nav-item dropdown">
        <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
          Dropdown
        </a>
        <div class="dropdown-menu" aria-labelledby="navbarDropdown">
          <a class="dropdown-item" href="#">Action</a>
          <a class="dropdown-item" href="#">Another action</a>
          <div class="dropdown-divider"></div>
          <a class="dropdown-item" href="#">Something else here</a>
        </div>
      </li>
      <li class="nav-item">
        <a class="nav-link disabled" href="#">Disabled</a>
      </li>
    </ul>
    <form class="form-inline my-2 my-lg-0">
      <input class="form-control mr-sm-2" type="search" placeholder="Search" aria-label="Search">
      <button class="btn btn-outline-success my-2 my-sm-0" type="submit">Search</button>
    </form>
  </div>
</nav>
```
5. 最後輸入以下命令啟動Rails伺服器後，打開瀏覽器輸入網址`http://localhost:3000/`就可以看到沒有JS也沒有CSS的Navbar了
```shell
rails s
```

## 升級Webpacker
目前Rails使用的Webpacker還是4.3版，目前已經有V5的版本可以使用

打開專案的Gemfile，把Webpacker的版本改為以下
```ruby
gem 'webpacker', '~> 5.2', '>= 5.2.1'
```
改完之後記得執行`bundle`套用版本升級

接下來執行`yarn upgrade @rails/webpacker --latest`命令升級node module裡的檔案。也可以使用`yarn upgrade @rails/webpacker@5.2.1`指定版本升級，但是這兩個目前是一樣的

> [Webpacker Git上升級的說明](https://github.com/rails/webpacker/tree/5-x-stable#upgrading)

## 安裝與設定Bootsrap
1. 執行`yarn add bootstrap jquery popper.js`從yarnpkg安裝Boorstrap和Bootstrap裡JS所依賴的jQuery, popper.js

2. 接下來要利用[Webpack的ProvidePlugin功能](https://webpack.js.org/plugins/provide-plugin/)把jQuery, popper.js自動載入到各個JS裡，就不用在JS檔裡寫`import $ from 'jquery'`

    打開config/webpack/environment.js，改成以下內容

    * `$: 'jquery/src/jquery'`的寫法可以在rails-ujs中也能使用jQuery
```js
const { environment } = require('@rails/webpacker')

const webpack = require('webpack')
environment.plugins.prepend('Provide',
  new webpack.ProvidePlugin({
    $: 'jquery/src/jquery',
    jQuery: 'jquery/src/jquery',
    Popper: ['popper.js', 'default']
  })
)

module.exports = environment
```
3. 打開app/javascript/packs/application.js，加入以下程式碼
```js
import 'bootstrap'
import 'bootstrap/scss/bootstrap.scss'
```
4. 打開app/views/layouts/application.html.erb，把`<%= stylesheet_link_tag 'application', ... %>`那行改成以下
```ruby
<%= stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
```
5. 最後再執行`rails s`，就可以在瀏覽器看到精美的Bootstrap Navbar🎉

## Webpack打包CSS和JS邏輯

在Webpacker設定下，Webpack打包時會進行以下步驟

1. 到app/javascript/packs尋找JS或SCSS檔案，作為[entry point](https://webpack.js.org/concepts/#entry)，並依據JS檔裡面的`import 'libary'`或SCSS檔裡面的`@import 'style.scss';`建立各檔案間的依賴性
  
   * 除了在JS裡import JS，你也可以在JS import SCSS, CSS

   * import檔案時，從資料夾名稱開始寫的話，會從node_module資料夾中開始找

   * 另外Webpack V5的新功能[Importing CSS as a multi-file pack](https://github.com/rails/webpacker/blob/5-x-stable/docs/css.md#importing-css-as-a-multi-file-pack-webpacker-v5)，可以直接在packs資料夾建立同名的JS和SCSS檔不會有問題

2. 依據entry point的檔名，打包出同名的JS檔或CSS檔。SCSS檔打包出CSS檔，JS檔打包出JS和CSS檔（如果有import CSS或SCSS）

3. 使用時，在html.erb使用`<%= stylesheet_pack_tag 'entry_point' %>`載入打包後的CSS檔，使用`<%= javascript_pack_tag 'entry_point' %>`載入打包後的JS檔

    * 如果在entry point同時引用JS和SCSS檔，記得要分別載入CSS和JS(stylesheet_pack_tag & javascript_pack_tag)

## Production環境會遇到的問題
讓我們用`rails s -e=production`命令啟動production環境的伺服器。然後網站就出錯了

### assets:precompile
打開log/production.log會看到以下錯誤訊息
```
ActionView::Template::Error (Webpacker can't find application.css in /home/jasonj/webpack_example/public/packs/manifest.json. Possible causes:
1. You want to set webpacker.yml value of compile to true for your environment
   unless you are using the `webpack -w` or the webpack-dev-server.
2. webpack has not yet re-run to reflect updates.
3. You have misconfigured Webpacker's config/webpacker.yml file.
4. Your webpack configuration is not creating a manifest.
Your manifest contains:
{
  "application.js": "/packs/js/application-9317054a6fa2affa21c0.js",
  "application.js.map": "/packs/js/application-9317054a6fa2affa21c0.js.map",
  "entrypoints": {
    "application": {
      "js": [
        "/packs/js/application-9317054a6fa2affa21c0.js"
      ],
      "js.map": [
        "/packs/js/application-9317054a6fa2affa21c0.js.map"
      ]
    }
  }
}
):
[3ac03c58-2f45-46f3-8997-611527716d59]      5:     <%= csrf_meta_tags %>
[3ac03c58-2f45-46f3-8997-611527716d59]      6:     <%= csp_meta_tag %>
[3ac03c58-2f45-46f3-8997-611527716d59]      7: 
[3ac03c58-2f45-46f3-8997-611527716d59]      8:     <%= stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
[3ac03c58-2f45-46f3-8997-611527716d59]      9:     <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
[3ac03c58-2f45-46f3-8997-611527716d59]     10:   </head>
[3ac03c58-2f45-46f3-8997-611527716d59]     11: 
[3ac03c58-2f45-46f3-8997-611527716d59]   
[3ac03c58-2f45-46f3-8997-611527716d59] app/views/layouts/application.html.erb:8
I,
```
簡單來說，就是在erb檔用了`stylesheet_pack_tag`和`javascript_pack_tag`，但是webpacker說找不到打包出來到CSS和JS檔

要產生production環境用的CSS和JS檔，需要先執行以下命令
```shell
RAILS_ENV=production rails assets:precompile
```
之後檢查public/packs/css和public/packs/js裡，就會有打包出來的檔案

在部署的時候，應該執行`rails assets:precompile`就好，`RAILS_ENV=production`應該會先設定在伺服器的環境變數中

> [部署在Heroku的話，應該會自動執行這一步的樣子](https://devcenter.heroku.com/articles/getting-started-with-rails6#deploy-your-application-to-heroku)

### RAILS_SERVE_STATIC_FILES
接下來再打開伺服器，怎麼又變成沒有引用CSS和JS的畫面了？

讓我們打開log看看
```
[2020-09-16T02:02:18.644064 #9016]  INFO -- : [87560b92-483f-43e3-a8eb-3089cb8fca12] Started GET "/packs/css/application-c6751772.css" for 127.0.0.1 at 2020-09-16 02:02:18 +0800
F, 
[2020-09-16T02:02:18.644927 #9016] FATAL -- : [87560b92-483f-43e3-a8eb-3089cb8fca12]   
[87560b92-483f-43e3-a8eb-3089cb8fca12] ActionController::RoutingError (No route matches [GET] "/packs/css/application-c6751772.css"):
...
...
...
[2020-09-16T02:02:18.667339 #9016]  INFO -- : [ef022040-9dd8-424b-beb7-ebaed172e8d0] Started GET "/packs/js/application-473b44d61307cd2ab54d.js" for 127.0.0.1 at 2020-09-16 02:02:18 +0800
F, 
[2020-09-16T02:02:18.668101 #9016] FATAL -- : [ef022040-9dd8-424b-beb7-ebaed172e8d0]   
[ef022040-9dd8-424b-beb7-ebaed172e8d0] ActionController::RoutingError (No route matches [GET] "/packs/js/application-473b44d61307cd2ab54d.js"):
```
原來是瀏覽器要求CSS和JS時，被Rails擋下來了

在config/environments/production.rb中
```ruby
# Disable serving static files from the `/public` folder by default since
# Apache or NGINX already handles this.
config.public_file_server.enabled = ENV['RAILS_SERVE_STATIC_FILES'].present?
```
一般部署Rails時，會搭配[Apache](https://httpd.apache.org/)或[NGINX](https://www.nginx.com/)使用，處理public資料夾裡的靜態檔案

所以如果沒有使用以上軟體，要使用以下命令啟動伺服器，
```shell
RAILS_SERVE_STATIC_FILES=true rails s -e=production
```
在部署時，要把`RAILS_SERVE_STATIC_FILES=true`加入環境變數中

**部署在Heroku時需要設定這個環境變數**

## webpack-dev-server
> [請領班幫忙](https://kaochenlong.com/2019/11/21/webpacker-with-rails-part-1/)

開發時，同時執行webpack-dev-server，可以加快CSS或JS檔重新載入的速度
