# Railsでサンプルアプリを作ってみる

- 作るアプリ
  - 毎日メールで送っている日報のテンプレートを作るアプリ
- 環境
  - Rails4
  - mysql
  - その他gem
- 前提
  - railsの開発環境があるものとする

## プロジェクトを作成

- 詳しいオプションはこちら
  - http://railsdoc.com/rails

```
$ rails new nippou --database sqlite3 --javascript jquery --skip-bundle
```

## setupファイルを編集

- 初期状態だと`rake db:setup`が実行されるので、変更しておく
  - `db:setup`は`db:create`、`db:schema:load`、`db:seed`を実行する
- この辺は新規のアプリかなど、どう開発を進めていくかによって変わってくる

## setupを実行する

```
$ bin/setup
== Installing dependencies ==
The Gemfile's dependencies are satisfied

== Preparing database ==

== Removing old logs and tempfiles ==

== Restarting application server ==
```

## 開発周りのgemをinstallする

### guardをインストール

- https://github.com/guard/guard
- Guardはファイルに変更があったら、その変更ファイルに合わせて処理を実行するもの
  - ユニットテストを実行したり
  - テンプレートを編集した際に、ライブリロードを実行したり
  - など

#### Gemfileを編集

`development`グループに追記する

```
group :development, :test do
  gem 'guard'
end
```

`bundle install`を実行（`bundle`だけでもOK）

```
$ bundle
```

guardのセットアップする

```
$ bundle exec guard init
17:23:06 - INFO - Writing new Guardfile to /Users/b07032/workspace/odajima-ca/nippou/Guardfile
```

動作確認

```
$ bundle exec guard
17:24:13 - ERROR - No plugins found in Guardfile, please add at least one.
17:24:14 - INFO - Guard is now watching at '/Users/b07032/workspace/odajima-ca/nippou'
[1] guard(main)>
```

### rspecをインストール

- https://github.com/rspec/rspec-rails
- https://github.com/guard/guard-rspec
- ユニットテストのgem

`development`グループに追記する

```
group :development, :test do
  gem 'rspec-rails'
  gem 'guard-rspec', require: false
end
```

`bundle install`を実行（`bundle`だけでもOK）

```
$ bundle
```

`rspec:install`を実行する

```
$ rails generate rspec:install
      create  .rspec
      create  spec
      create  spec/spec_helper.rb
      create  spec/rails_helper.rb
```

もともとある`test`ディレクトリを削除します

```
$ rm -rf test
```

rspecの動作確認

```
$ bundle exec rspec
No examples found.


Finished in 0.00035 seconds (files took 0.40606 seconds to load)
0 examples, 0 failures
```

rspecを自動実行するため、guardを設定する

```
$ bundle exec guard init rspec
17:33:58 - INFO - rspec guard added to Guardfile, feel free to edit it
```

### rubocopをインストールする

- https://github.com/bbatsov/rubocop
- https://github.com/yujinakayama/guard-rubocop
- コードが規約通り書かれているかの静的コード解析ツール

`development`グループに追記する

```
group :development, :test do
  gem 'rubocop', require: false
  gem 'guard-rubocop'
end
```

`bundle install`を実行（`bundle`だけでもOK）

```
$ bundle
```

rubocopの動作確認

- 初期設定だと、いろいろダメだよと言われます。
- なので`.rubocop.yml`を作成し、お好みの設定をします。

```
$ rubocop
Inspecting 4 files
....

4 files inspected, no offenses detected
```

rubocopを自動実行するため、guardを設定する

```
$ bundle exec guard init rubocop
17:40:12 - INFO - rubocop guard added to Guardfile, feel free to edit it
```

### factory_girlをインストールする

- https://github.com/thoughtbot/factory_girl_rails
- テストデータを作成するgem

```
group :development, :test do
  gem 'factory_girl_rails'
end
```

`bundle install`を実行（`bundle`だけでもOK）

```
$ bundle
```

その他設定は不要

### annotate_modelsをインストールする

- https://github.com/ctran/annotate_models
- 各種モデルファイルやテストファイルにテーブルのスキーマ情報をコメントするgem

```
group :development, :test do
  gem 'annotate'
end
```

`bundle install`を実行（`bundle`だけでもOK）

```
$ bundle
```

`rake db:migrate`のたびにコメントを更新するように、設定ファイルをインストールする

```
$ rails g annotate:install
create  lib/tasks/auto_annotate_models.rake
```

動作を確認

```
$ rake db:migrate
Nothing to annotate.
```

その他設定は不要


### rails_best_practicesをインストールする

- https://github.com/railsbp/rails_best_practices
- railsの使い方などをチェックするツール

```
group :development, :test do
  gem 'rails_best_practices'
  gem 'guard-rails_best_practices'
end
```

`bundle install`を実行（`bundle`だけでもOK）

```
$ bundle
```

動作確認

```
$ bundle exec rails_best_practices .
Source Codes: |==============================================================================================================|

Please go to http://rails-bestpractices.com to see more useful Rails Best Practices.

No warning found. Cool!
```

rails_best_practicesを自動実行するため、guardを設定する

- こんなエラーになるので、下記を参考にして対応
- https://github.com/logankoester/guard-rails_best_practices/issues/9

```
$ bundle exec guard init rails_best_practices
18:42:32 - ERROR - Could not load 'guard/rails_best_practices' or '~/.guard/templates/rails_best_practices' or find class Guard::Rails_best_practices
18:42:32 - ERROR - Error is: No such file or directory @ rb_sysopen - /Users/b07032/.guard/templates/rails_best_practices
```

guardでの動作を確認

```
$ bundle exec guard
18:46:29 - INFO - Running Rails Best Practices checklist with command
> [#3758BDCD300D] => rails_best_practices --vendor --spec --test --features
Source Codes: |==============================================================================================================|

Please go to http://rails-bestpractices.com to see more useful Rails Best Practices.

No warning found. Cool!
18:46:30 - INFO - Guard is now watching at '/Users/b07032/workspace/odajima-ca/nippou'
```

### 開発前準備のまとめ

- ここでとり上げたgem以外にも多数のgemがあります。
- railsの場合は、generatorを多様することが多いです。
- 出来る限り最初に、どのgemを使っていいくかなど決めておくとあとあと幸せになれるかと思います。
  - 例えばテンプレートは'erb'なのか、それとも`haml`、`slim`を使うのかなど

- 開発するときは`guard`を立ち上げながら、開発していきます。
  - テスト駆動開発(TDD)、振舞駆動開発(BDD)とか良くいいますが、それです。

## 日報テンプレートアプリを作成する

- アプリの内容
  - 日報メールを送るためのタスクをメモしておくアプリ
  - タスクは１日単位とする
- モデリング
  - タスクを管理するモデル
  - タスクは今日やるべきこと、後日やるべきことで分ける
  - タスクをカテゴリ分けするモデル
  - タスクの完了、未完了をチェックできるようにする
- その他機能は省く
  - ログイン機能
  - 見た目など

### カテゴリマスタを作成する

generateコマンドを実行

```
$ rails g model category name:string
      invoke  active_record
      create    db/migrate/20151008092735_create_categories.rb
      create    app/models/category.rb
      invoke    rspec
      create      spec/models/category_spec.rb
      invoke      factory_girl
      create        spec/factories/categories.rb
```

migrateionを修正する

```
$ vi db/migrate/20151008092735_create_categories.rb
class CreateCategories < ActiveRecord::Migration
  def change
    create_table :categories do |t|
-     t.string :name
+     t.string :name, null: false

      t.timestamps null: false
    end
  end
end
```

migrateを実行する

```
$ rake db:migrate
== 20151008092735 CreateCategories: migrating =================================
-- create_table(:categories)
   -> 0.0010s
== 20151008092735 CreateCategories: migrated (0.0011s) ========================

Annotated (1): Category
```

### カテゴリマスタを修正する

guardを起動しておくと下記のようになるはず

```
18:50:35 - INFO - Run all
18:50:35 - INFO - Running all specs
*

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) Category add some examples to (or delete) /Users/b07032/workspace/odajima-ca/nippou/spec/models/category_spec.rb
     # Not yet implemented
     # ./spec/models/category_spec.rb:14


Finished in 0.00046 seconds (files took 7.04 seconds to load)
1 example, 0 failures, 1 pending


18:50:43 - INFO - Inspecting Ruby code style of all files
Inspecting 8 files
......C.

Offenses:

spec/factories/categories.rb:13:10: C: Style/StringLiterals: Prefer single-quoted strings when you don't need string interpolation or special symbols. (https://github.com/bbatsov/ruby-style-guide#consistent-string-literals)
    name "MyString"
         ^^^^^^^^^^

8 files inspected, 1 offense detected

18:50:44 - INFO - Running Rails Best Practices checklist with command
> [#76EDE54007F4] => rails_best_practices --vendor --spec --test --features
Source Codes: |==============================================================================================================|

Please go to http://rails-bestpractices.com to see more useful Rails Best Practices.

No warning found. Cool!
```

要件を組み込みつつ、プログラムとして悪い点を修正していきます

下記のようになればOKです

```
Finished in 0.01085 seconds (files took 2.41 seconds to load)
1 example, 0 failures


18:59:06 - INFO - Inspecting Ruby code style of all files
Inspecting 8 files
........

8 files inspected, no offenses detected
18:59:07 - INFO - Running Rails Best Practices checklist with command
> [#1994E3BBF818] => rails_best_practices --vendor --spec --test --features
Source Codes: |==============================================================================================================|

Please go to http://rails-bestpractices.com to see more useful Rails Best Practices.

No warning found. Cool!
```

### カテゴリマスタを登録する

- マスタデータを登録する方法はいろいろありますが、ここでは`seed`を使います。

db/seeds.rbを修正する

```
 #
 #   cities = City.create([{ name: 'Chicago' }, { name: 'Copenhagen' }])
 #   Mayor.create(name: 'Emanuel', city: cities.first)
+
+p Category.create(name: '開発')
+p Category.create(name: 'MTG')
+p Category.create(name: 'その他')
```

seedを実行する

```
$ rake db:seed
#<Category id: 1, name: "開発", created_at: "2015-10-08 10:03:54", updated_at: "2015-10-08 10:03:54">
#<Category id: 2, name: "MTG", created_at: "2015-10-08 10:03:54", updated_at: "2015-10-08 10:03:54">
#<Category id: 3, name: "その他", created_at: "2015-10-08 10:03:54", updated_at: "2015-10-08 10:03:54">
```

### タスク機能を作成する

generatorでベースを作成します

```
$ rails g scaffold task category:belongs_to name:string description:text status:string completed_at:datetime
```

migrationを修正し、`db:migrate`を実行します

```
$ rake db:migrate
== 20151008102949 CreateTasks: migrating ======================================
-- create_table(:tasks)
   -> 0.0020s
== 20151008102949 CreateTasks: migrated (0.0020s) =============================

Annotated (1): Task
```

### タスクモデルを修正する

- 先ほどのカテゴリマスタと同じ要領で修正します
- 下記のような出力になればOK

```
19:41:16 - INFO - Running: spec/models/task_spec.rb
...

Finished in 0.24851 seconds (files took 4.68 seconds to load)
3 examples, 0 failures
```

### rubocopの警告に対応する

- ここはどこまで規約に則っていくかによります
- 必要に応じて`.rubocop.yml`を修正することもありです。

```
24 files inspected, 210 offenses detected
↓
24 files inspected, no offenses detected
```

### タスクのCRUD機能を修正する

- 先ほどのカテゴリマスタと同じ要領で修正します
- 修正内容はコミットログを参照

### RailsBestPracticesに対応する

下記のような警告が出ているので対応します

```
20:08:01 - INFO - Running Rails Best Practices checklist with command
> [#638152C1D275] => rails_best_practices --vendor --spec --test --features
Source Codes: |==============================================================================================================|
/Users/b07032/workspace/odajima-ca/nippou/app/helpers/tasks_helper.rb:1 - remove empty helpers
/Users/b07032/workspace/odajima-ca/nippou/app/views/tasks/_form.html.erb:1 - replace instance variable with local variable
/Users/b07032/workspace/odajima-ca/nippou/app/views/tasks/_form.html.erb:2 - replace instance variable with local variable
/Users/b07032/workspace/odajima-ca/nippou/app/views/tasks/_form.html.erb:4 - replace instance variable with local variable
/Users/b07032/workspace/odajima-ca/nippou/app/views/tasks/_form.html.erb:7 - replace instance variable with local variable

Please go to http://rails-bestpractices.com to see more useful Rails Best Practices.

Found 5 warnings.
```

こうなればOK

```
20:11:22 - INFO - Running Rails Best Practices checklist with command
> [#EABF2F35F5D6] => rails_best_practices --vendor --spec --test --features
Source Codes: |==============================================================================================================|

Please go to http://rails-bestpractices.com to see more useful Rails Best Practices.

No warning found. Cool!
```

## 動作を確認してみる

```
$ rails s
=> Booting WEBrick
=> Rails 4.2.3 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
[2015-10-08 20:12:45] INFO  WEBrick 1.3.1
[2015-10-08 20:12:45] INFO  ruby 2.2.3 (2015-08-18) [x86_64-darwin14]
[2015-10-08 20:12:45] INFO  WEBrick::HTTPServer#start: pid=80574 port=3000
```

`http://localhost:3000`にアクセスしてみます。



## 開発開始

### トップページをタスク一覧に変更する

- 下記のように修正
- コードを追記したら、specにも追記を心がけること

```
$ git diff
diff --git a/config/routes.rb b/config/routes.rb
index 3343391..7a42208 100644
--- a/config/routes.rb
+++ b/config/routes.rb
@@ -1,4 +1,5 @@
 Rails.application.routes.draw do
+  root 'tasks#index'
   resources :tasks
   # The priority is based upon order of creation: first created -> highest priority.
   # See how all your routes lay out with "rake routes".
diff --git a/spec/routing/tasks_routing_spec.rb b/spec/routing/tasks_routing_spec.rb
index 09533db..4719759 100644
--- a/spec/routing/tasks_routing_spec.rb
+++ b/spec/routing/tasks_routing_spec.rb
@@ -4,6 +4,7 @@ RSpec.describe TasksController, type: :routing do
   describe 'routing' do

     it 'routes to #index' do
+      expect(get: '/').to route_to('tasks#index')
       expect(get: '/tasks').to route_to('tasks#index')
     end
```

### タスクとカテゴリを紐つける

- 詳細はコミットログを参照

### ステータスを選択項目に変更する

- 詳細はコミットログを参照
- rails4.1の機能（Enum）

statusをstringで作ってしまったので、migrationを作成する

```
$ rails g migration change_status_type_in_tasks
```

カラムの変更内容を記載します

```ruby
class ChangeStatusTypeInTasks < ActiveRecord::Migration
  def change
    change_column :tasks, :status, :integer, null: false
  end
end
```

```
$ rake db:migrate
== 20151009020109 ChangeStatusTypeInTasks: migrating ==========================
-- change_column(:tasks, :status, :integer, {:null=>false})
   -> 0.0082s
== 20151009020109 ChangeStatusTypeInTasks: migrated (0.0084s) =================

Annotated (1): Task
```

statusにenum機能を付与します

- 詳細はコミットログを参照

### タスクの作業開始機能を追加する

- tasksコントローラーに`doing`アクションを追加し、ステータスをdoingに変更するようにします
- 詳細はコミットログを参照

### タスクの作業完了機能を追加する

- tasksコントローラーに`done`アクションを追加し、ステータスをdoneに変更するようにします
- doneにした際にcompleted_atに日付を入れるようにする
- 詳細はコミットログを参照

### タスクの作業完了機能を追加する

### 「本日の日報」テンプレートを作成する

- 日報メールを送る際のテンプレートを作成する
- コピペしてメールに添付するようなイメージ

generateする

```
$ rails g controller daily_reports index
      create  app/controllers/daily_reports_controller.rb
       route  get 'daily_reports/index'
      invoke  erb
      create    app/views/daily_reports
      create    app/views/daily_reports/index.html.erb
      invoke  rspec
      create    spec/controllers/daily_reports_controller_spec.rb
      create    spec/views/daily_reports
      create    spec/views/daily_reports/index.html.erb_spec.rb
      invoke  helper
      create    app/helpers/daily_reports_helper.rb
      invoke    rspec
      create      spec/helpers/daily_reports_helper_spec.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/daily_reports.coffee
      invoke    scss
      create      app/assets/stylesheets/daily_reports.scss
```

まずはroutesを修正します

```
get 'daily_reports/index'
↓
resources :daily_reports, only: [:index]
```

routing用のspecファイルを作成

```
$ vi spec/routing/daily_reports_routing_spec.rb

require 'rails_helper'

RSpec.describe DailyReportsController, type: :routing do
  describe 'routing' do

    it 'routes to #index' do
      expect(get: '/daily_reports').to route_to('daily_reports#index')
    end

  end
end
```

まずは、最低限でspecとコードの警告などがないように修正する

```
12:30:27 - INFO - Run all
12:30:27 - INFO - Running all specs
...................................................

Finished in 0.67117 seconds (files took 2.4 seconds to load)
51 examples, 0 failures


12:30:31 - INFO - Inspecting Ruby code style of all files
Inspecting 27 files
...........................

27 files inspected, no offenses detected
12:30:32 - INFO - Running Rails Best Practices checklist with command
> [#4F047C8D5AB0] => rails_best_practices --vendor --spec --test --features
Source Codes: |=======================================================================================================================|

Please go to http://rails-bestpractices.com to see more useful Rails Best Practices.

No warning found. Cool!
```

### レポート内容を実装する

- 実行日（リクエスト日）を基準にタスクを一覧表示する
  - その日に完了したタスク
  - 仕掛中のタスク
  - 未着手のタスク
- 詳細はコミットログを参照

#### ポイント

- テストしやすいコードを書くこと

- 例えば下記のように変数の値をテストしやすく記述する
  - 例が少し悪いですが。

```ruby
class DailyReportsController < ApplicationController
  def index
    @today = Date.today
    @todo_tasks = Task.todo
    @doing_tasks = Task.doing
    @done_tasks = Task.done.where(completed_at: @today.beginning_of_day..@today.end_of_day)
  end
end
```

```ruby
    it 'returns http success' do
      get :index
      expect(response).to have_http_status(:success)
      expect(assigns(:today)).to eq Date.today
      expect(assigns(:todo_tasks)).to match_array todo_tasks
      expect(assigns(:doing_tasks)).to match_array doing_tasks
      expect(assigns(:done_tasks)).to match_array done_tasks
    end
```
