# 在Rails專案設定Travis-CI或Github action

<!-- more -->

# 在Rails專案設定Travis-CI或Github action

## 什麼是CI？
> [偷偷用一下Gitlab CI/CD的文件](https://docs.gitlab.com/ee/ci/README.html)
~~結束~~

簡單來說，CI(Continuous Integration, 持續整合)可以幫你自動化push一個新分支到Github等網站後的步驟

* 以Rails程式來說，至少有跑測試(RSpec)這個步驟

CD(Continuous Delivery, 持續交付),可以幫你自動化新分支merge到主分支後需要執行的步驟。
* 以Rails等程式來說，至少有部署這個步驟

## 怎麼設定？
以一個存放在Github的專案，部署在Heroku伺服器上的網路應用程式來說。

如果只要符合最少CI/CD的步驟

* CI： 這裡會示範Travis-CI和Github action這兩個工具
* CD： 在Heroku中，鏈接到Github之後就可以設定自動部署了

以下會用一個，使用Postgresql資料庫，以Webpack做為打包工具的Rails6應用，示範Travis-CI和Github action的設定要怎麼寫

## 為什麼需要設定？
我們可以試著回想一下，在一個全新的電腦上，如果需要把一個Rails的Rspec測試跑起來，需要什麼步驟？
1. 安裝Ubuntu
2. 安裝Ruby
3. 安裝Postgresql資料庫
4. 安裝Node
5. bundle安裝Ruby gems
6. yarn安裝node modules
7. 跑rspec

因為在Travis-CI或Github action執行時，就等於是在一台新電腦上執行。

所以這些步驟都需要寫在一個檔案中，告訴Travis-CI或Github action怎麼設定和執行

## Travis-CI
Travis-CI的設定檔在專案根目錄的.travis.yml
```yml
os: linux
dist: bionic
language: ruby
cache:
  - bundler
  - yarn
services:
  - postgresql
before_install:
  - nvm install --lts
before_script:
  - bundle install --jobs=3 --retry=3
  - yarn
  - bundle exec rake db:create
  - bundle exec rake db:schema:load
script:
  - bundle exec rspec
```
`os`設定作業系統

`dist`設定Linux發行版

`cache`設定那些資料需要被快取暫存

`services`設定需要的服務

`before_install`需要另外安裝的程式

`before_script`執行script前執行的指令

`script`執行script

## Github Action
Github action的設定檔在專案.github/workflows資料夾中YAML設定檔(檔名不限)
> [參考來源](https://hai3.net/blog/rails-rspec-postgres-github-action/)
```yml
name: RSpec
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    # if: "!contains(github.event.head_commit.message, 'ci skip')"
    services:
      postgres:
        image: postgres:12
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    steps:
    - uses: actions/checkout@v2
    - name: Set up Ruby
      uses: ruby/setup-ruby@v1
      # with:
      #   ruby-version: 2.7.1 # Not needed with a .ruby-version file
    - name: Setup Node # delete this step will use node v12
      uses: actions/setup-node@v1
      with:
        node-version: 14
    - name: Cache gems
      uses: actions/cache@v2
      with:
        path: vendor/bundle
        key: ${{ runner.os }}-gem-${{ hashFiles('**/Gemfile.lock') }}
        restore-keys: |
          ${{ runner.os }}-gem-
    - name: Cache node modules
      uses: actions/cache@v2
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/yarn.lock') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - name: Install dependencies
      run: |
        gem install bundler
        bundle install --path vendor/bundle --without production
        yarn install
    - name: Set env
      run: |
        echo '::set-env name=RAILS_ENV::test'
        echo '::set-env name=DATABASE_URL::postgres://postgres:postgres@localhost:5432/'
    - name: setup DB
      run: |
        bundle exec rails db:create
        bundle exec rails db:schema:load
    - name: Run tests
      run: bundle exec rspec
```
`name`設定顯示在Action中的名稱
`on`設定底下Job觸發條件

```yml
job:
  test:
```
設定job名稱

`runs-on`設定機器的作業系統

`services`設定需要的服務

```yml
env:
  POSTGRES_PASSWORD: postgres
```
設定環境變數，以設定資料庫的使用者密碼

`setps`設定各個步驟，可以使用現有的action或執行指令

```yml
- name: Set up Ruby
  uses: ruby/setup-ruby@v1
  # with:
  #   ruby-version: 2.7.1 # Not needed with a .ruby-version file
```
設定安裝Ruby，不設定版本會從.ruby-version讀取

```yml
- name: Setup Node # delete this step will use node v12
  uses: actions/setup-node@v1
  with:
    node-version: 14
```
設定安裝Node，沒有這一步驟的話會用Node V12

```yml
- name: Cache gems
  uses: actions/cache@v2
  with:
    path: vendor/bundle
    key: ${{ runner.os }}-gem-${{ hashFiles('**/Gemfile.lock') }}
    restore-keys: |
      ${{ runner.os }}-gem-
- name: Cache node modules
  uses: actions/cache@v2
  with:
    path: node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/yarn.lock') }}
    restore-keys: |
      ${{ runner.os }}-node-
```
設定哪些資料需要被快取

```yml
- name: Install dependencies
  run: |
    gem install bundler
    bundle install --path vendor/bundle --without production
    yarn install
```
安裝gem，安裝node modules

```yml
- name: Set env
  run: |
    echo '::set-env name=RAILS_ENV::test'
    echo '::set-env name=DATABASE_URL::postgres://postgres:postgres@localhost:5432/'
```
設定環境變數，RAILS_ENV，DATABASE_URL

```yml
- name: setup DB
  run: |
    bundle exec rails db:create
    bundle exec rails db:schema:load
```
初始化資料庫

```yml
- name: Run tests
  run: bundle exec rspec
```
執行測試
