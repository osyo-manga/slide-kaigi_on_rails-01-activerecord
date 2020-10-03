## Kaigi on Rails
- - -

## ActiveRecord の歩み方

---

### 突然ですが、
### 皆さんは ActiveRecord のコードを
### 読んだことはありますか？

---

### 知らない人もいると思うんですが
### ActiveRecord って Ruby で
### 書かれているんですよ

---

### つまり Ruby を知っていれば
### ActiveRecord のコードを
### 読むことができるんです！

---

## ちょっと ActiveRecord コードを
## 覗いてみましょう…

---

```ruby
Relation::VALUE_METHODS.each do |name|
  method_name = \
    case name
    when *Relation::MULTI_VALUE_METHODS then "#{name}_values"
    when *Relation::SINGLE_VALUE_METHODS then "#{name}_value"
    when *Relation::CLAUSE_METHODS then "#{name}_clause"
    end
  class_eval <<-CODE, __FILE__, __LINE__ + 1
    def #{method_name}                   # def includes_values
      default = DEFAULT_VALUES[:#{name}] #   default = DEFAULT_VALUES[:includes]
      @values.fetch(:#{name}, default)   #   @values.fetch(:includes, default)
    end                                  # end

    def #{method_name}=(value)           # def includes_values=(value)
      assert_mutability!                 #   assert_mutability!
      @values[:#{name}] = value          #   @values[:includes] = value
    end                                  # end
  CODE
end
```

---

```ruby
QUERYING_METHODS = [
  :find, :find_by, :find_by!, :take, :take!, :first, :first!, :last, :last!,
  :second, :second!, :third, :third!, :fourth, :fourth!, :fifth, :fifth!,
  :forty_two, :forty_two!, :third_to_last, :third_to_last!, :second_to_last, :second_to_last!,
  :exists?, :any?, :many?, :none?, :one?,
  :first_or_create, :first_or_create!, :first_or_initialize,
  :find_or_create_by, :find_or_create_by!, :find_or_initialize_by,
  :create_or_find_by, :create_or_find_by!,
  :destroy_all, :delete_all, :update_all, :touch_all, :destroy_by, :delete_by,
  :find_each, :find_in_batches, :in_batches,
  :select, :reselect, :order, :reorder, :group, :limit, :offset, :joins, :left_joins, :left_outer_joins,
  :where, :rewhere, :preload, :extract_associated, :eager_load, :includes, :from, :lock, :readonly, :extending, :or,
  :having, :create_with, :distinct, :references, :none, :unscope, :optimizer_hints, :merge, :except, :only,
  :count, :average, :minimum, :maximum, :sum, :calculate, :annotate,
  :pluck, :pick, :ids
].freeze # :nodoc:
delegate(*QUERYING_METHODS, to: :all)
```

---

```ruby
module ActiveRecord
  class Base
    extend ActiveModel::Naming

    extend ActiveSupport::Benchmarkable
    extend ActiveSupport::DescendantsTracker

    extend ConnectionHandling
    extend QueryCache::ClassMethods
    extend Querying
    extend Translation
    extend DynamicMatchers
    extend Explain
    extend Enum
    extend Delegation::DelegateCache
    extend Aggregations::ClassMethods

    include Core
    include Persistence
    include ReadonlyAttributes
    include ModelSchema
    include Inheritance
    include Scoping
    include Sanitization
    include AttributeAssignment
    include ActiveModel::Conversion
    include Integration
    include Validations
    include CounterCache
    include Attributes
    include AttributeDecorators
    include Locking::Optimistic
    include Locking::Pessimistic
    include DefineCallbacks
    include AttributeMethods
    include Callbacks
    include Timestamp
    include Associations
    include ActiveModel::SecurePassword
    include AutosaveAssociation
    include NestedAttributes
    include Transactions
    include TouchLater
    include NoTouching
    include Reflection
    include Serialization
    include Store
    include SecureToken
    include Suppressor
  end
```

---

## こんなのぼくが知ってる
## Ruby じゃない！！！

---

#### 自己紹介
- - -

* 名前 : osyo
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* 趣味で Ruby にパッチを投げたり、気になった Ruby のチケットをブログにまとめたりしてる
* Ruby で一番好きな機能は Refinements
* 普段は [activerecord-bitemporal](https://github.com/kufu/activerecord-bitemporal) を開発してる
* エディタは Vim 
* 質問等があれば Twitter まで！

---

## ActiveRecord の歩み方

---

#### アジェンダ
- - -

* ActiveRecord を読む前に                         <!-- .element: class="fragment" -->
    * Ruby の知識を蓄える
    * 知っておくとよい Ruby の知識
    * コードを読むときに使う道具の紹介
* ActiveRecord を読むときのコツ                          <!-- .element: class="fragment" -->
    * ActiveRecord の構造を知る
    * 読む目的を明確にする
* ライブリーディング                          <!-- .element: class="fragment" -->
    * コードを読む前の下準備
    * ActiveRecord を読んでみる
* まとめ              <!-- .element: class="fragment" -->
* 話さないこと                              <!-- .element: class="fragment" -->
    * ActiveRecord 自体の機能解説はしない
    * 今日の話を聞いて実際に自分で読んでみよう！！               <!-- .element: class="fragment" -->

---

## ActiveRecord を読む前に

---

#### Ruby の知識を蓄える
- - -

* ActiveRecord は Ruby で書かれている                      <!-- .element: class="fragment" -->
    * なので Ruby が読めれば ActiveRecord も読むことはできる
* しかし ActiveRecord では普段はあまり使わないような Ruby の機能が使われている                      <!-- .element: class="fragment" -->
    * メタプロだったり動的なオブジェクト生成だったり
* などでちゃんと読むためにはある程度 Ruby に精通した知識が必要になってくる                      <!-- .element: class="fragment" -->
* 知らない Ruby の書き方が出てきてもギョッとせずに冷静に対応する事が大事                      <!-- .element: class="fragment" -->
    * 知らない機能やメソッドが出てきたらちゃんと調べる！

---

#### 知っておくとよい Ruby の知識
- - -

* メタプログラミングの知識         <!-- .element: class="fragment" -->
    * `#define_method` や `#respond_to?`, `#method_missing`, `#send` 
* 動的なクラス/モジュール生成        <!-- .element: class="fragment" -->
    * `Class.new` や `Module.new` が出てきてもびっくりしない
* コンテキストを切り替えたブロックの評価        <!-- .element: class="fragment" -->
    * `instance_exec` や `instance_eval`, `class_eval`
* 継承関連                                    <!-- .element: class="fragment" -->
    * いろんなところで include /prepend や extend してる
    * User.ancestors とか見るとめっちゃ継承してる…
* ActiveSupport::Concern や delegate の機能        <!-- .element: class="fragment" -->
    * Ruby ではなくて Rails の拡張機能
    * ActiveRecord で広く使われているので知っておくとよい

---

#### コードを読むときに使う道具の紹介
- - -

* Object#method                           <!-- .element: class="fragment" -->
    * メソッドの定義位置などを調べる
* メタ情報などをデバッグ出力                           <!-- .element: class="fragment" -->
    * `__method__` `caller` `.class`
* 自作デバッグ出力                          <!-- .element: class="fragment" -->
    * gem : [binding-debug](https://github.com/osyo-manga/gem-binding-debug)
* 実行時デバッグ機能                           <!-- .element: class="fragment" -->
    * binding.irb や binding.pry や [ruby_jard](https://rubyjard.org/)
* $debug = true と pp hoge if $debug の組み合わせ                           <!-- .element: class="fragment" -->
    * 必要なときのみデバッグ出力を ON にする
* ActiveRecord の #to_sql                            <!-- .element: class="fragment" -->
    * 実行される SQL 文を取得する
* エディタを便利にする                           <!-- .element: class="fragment" -->
    * エディタ上でファイラをつかったり grep したり

>>>

#### メソッド定義位置を調べる
- - -

* `Object#method` を使用してメソッドの定義位置を調べられる

```ruby
# test.rb
class Super
  def hoge; end
end
class Sub < Super
  def hoge; end
end

obj = Sub.new
# hoge メソッドのオブジェクトを取得する
meth = obj.method(:hoge)

# メソッドの定義位置が出力される
pp meth.source_location
# => ["/home/hoge/test.rb", 7]
# こうすると親メソッドも参照する事ができる
pp meth.super_method.source_location
# => ["/home/hoge/test.rb", 3]
```

>>>

#### メタ情報をデバッグ出力
- - -

* よく利用するデバッグ出力情報

```ruby
def plus(a, b)
  # メソッド名を出力
  pp __method__    # => :plus
  # クラス名を出力
  pp a.class.name  # => "Integer"
  # コールスタックを出力
  pp caller

  a + b
end

plus 1, 2
```

>>>

#### 自作デバッグ出力 gem
- - -

* pp { expr } を実行すると "expr" とその結果の両方が出力される
    * gem install binding-debug

```ruby
require "binding/debug"

using BindingDebug

def plus(a, b)
  # メソッド名を出力
  pp { __method__ }
  # output: __method__ # => :plus

  # クラス名を出力
  pp { a.class.name }
  # output: a.class.name # => "Integer"

  a + b
end

plus 1, 2
```

>>>

#### $debug = true と pp hoge if $debug の組み合わせ
- - -

* pp でデバッグ出力を仕込むと調べたい箇所以外のタイミングでも出力される事がある
* pp だけではなくて pp hoge if $debug で制御することで任意のタイミングでのみデバッグ出力できる

```ruby
def twice(a)
  pp __method__ if $debug
  pp a if $debug
  a + a
end

twice(42)

$debug = true
# ここで呼び出された場合のみデバッグ出力する
twice("homu")
$debug = false

twice(42)
```

>>>

#### #to_sql で SQL 文を取得
- - -

* #to_sql で実行される SQL 文を取得する事ができる

```ruby
# puts で出力する
puts User.where(active: true).to_sql
# output: SELECT "users".* FROM "users" WHERE "users"."active" = 1

# p や pp だと " がエスケープされるので注意
p User.where(active: true).to_sql
# => "SELECT \"users\".* FROM \"users\" WHERE \"users\".\"active\" = 1"
```

>>>

#### エディタを便利にする
- - -

* 使い勝手をよくするためにエディタを拡張する
    * 道具にこだわるのは大事
* エディタ上でコードを実行させる
    * [vim-quickrun](https://github.com/thinca/vim-quickrun)
* カーソル下の単語をハイライトさせる
    * [vim-brightest](https://github.com/osyo-manga/vim-brightest)
* 任意の単語をハイライトする
    * [vim-quickhl](https://github.com/t9md/vim-quickhl)
* 検索数を表示する
    * [anzu.vim](https://github.com/osyo-manga/vim-anzu)
* SQL を整形する
    * [SQLUtilities](https://github.com/vim-scripts/SQLUtilities)

---

## ActiveRecord を読むときのコツ

---

#### ActiveRecord の構造を知る
- - -

* ActiveRecord にはいろいろな機能が実装されている                           <!-- .element: class="fragment" -->
    * モデル定義
    * 関連付けの設定
    * バリデーション
    * SQL の実行・読み込み・モデルのインスタンス化
    * etc...
* 調べたい機能に対してどこを読むべきなのかを調べるのが難しい                           <!-- .element: class="fragment" -->
    * 単純に ActiveRecord 自体のコード量が多い
* ある程度 ActiveRecord の構造を知っておくとあたりをつけて読みやすい                           <!-- .element: class="fragment" -->
    * 慣れの部分もあるがある程度全体像を把握しておくとよい

>>>

* 例えば…
* ActiveRecord::Persistence                           <!-- .element: class="fragment" -->
    * モデルの生成、更新、削除
    * `.create`, `#update`, `#destroy`
* ActiveRecord::Relation や ActiveRecord::QueryMethods                           <!-- .element: class="fragment" -->
    * クエリメソッドの定義や Arel の構築処理
    * `.where`, `.order`, `.limit`
* ActiveRecord::Reflection                           <!-- .element: class="fragment" -->
    * 関連付けの設定定義
    * `has_many`, `has_one`, `belongs_to`
* ActiveRecord::Associations                           <!-- .element: class="fragment" -->
    * `#associations` や関連先の読み込みの処理
* ActiveRecord::Migration                           <!-- .element: class="fragment" -->
    * migration 関連の処理
* などなど…                           <!-- .element: class="fragment" -->
* ActiveRecord 以外で定義されている処理もあるので注意する                           <!-- .element: class="fragment" -->

---

#### 読む目的を明確にする
- - -

* ActiveRecord というのはいろいろなことをやっている                           <!-- .element: class="fragment" -->
* ActiveRecord を読む！と言っても漠然としてて迷走しがち                           <!-- .element: class="fragment" -->
* まずは『ActiveRecord のこういう機能を読む！』という目的を見つけると入り口が明確になってよみやすい                          <!-- .element: class="fragment" -->
    * User.where(name: "Tom").to_sql で SQL がどう構築されるのか
    * has_many :comments を行うと何が起きるのか
    * バリデーションの仕組みって…？コールバックの仕組みって…？ 
* 特に『このメソッドを呼ぶと何が起きるのか』みたいな目的だとコードリーディングに入りやすい                           <!-- .element: class="fragment" -->
    * has_many がなにをやっているのか調べる、ってなると has_many のメソッドを調べるところから始められる

---

## ライブリーディング

---

#### コードを読む前の下準備
- - -

* 何を読むのか目的を明確にする                           <!-- .element: class="fragment" -->
    * 今回は scope の実装を読んでみる
* 調べたい機能の挙動を調べておく                           <!-- .element: class="fragment" -->
    * その挙動がどうやって Ruby で表現されているのか見るのが大事
    * `scope` のドキュメントは[こちら](https://api.rubyonrails.org/classes/ActiveRecord/Scoping/Named/ClassMethods.html#method-i-scope)
    * ドキュメントを参照する場合は[ここ](https://api.rubyonrails.org/)がおすすめ
* 最小構成のコードを用意する                           <!-- .element: class="fragment" -->
    * 最小構成のほうが余計な処理がなく実行も早いし、意図しない動作が発生することも少ない
    * Rails が[バグレポート用のテンプレート](https://github.com/rails/rails/blob/master/guides/bug_report_templates/active_record_gem.rb)を用意しているのでそれを使うと1ファイルだけで簡単に ActiveRecord のコードを動かすことができる
    * ActiveRecord 以外のテンプレートもあるのでよしなに利用できる

---

## ActiveRecord を読んでみる

>>>

#### 01. その機能を使ってみる
- - -

* まず実際にどういう挙動をするのか使ってみる

```ruby
class User < ActiveRecord::Base
  # こんな感じで where を scope として定義する
  scope :actives, -> { where(active: true) }
  scope :owners, -> { where(owner: true) }
end

# 定義した scope はクラスメソッドのような形で呼び出すことができる
puts User.actives.to_sql
# => SELECT "users".* FROM "users" WHERE "users"."active" = 1

# また User. だけではなくて scope をチェーンすることもできる
puts User.actives.owners.to_sql
# => SELECT "users".* FROM "users" WHERE "users"."active" = 1 AND "users"."owner" = 1
puts User.order(:created_at).owners.to_sql
# => SELECT "users".* FROM "users" WHERE "users"."owner" = 1 ORDER BY "created_at" ASC
```

>>>

#### 02. その機能（メソッド）の定義位置を調べる
- - -

* 実際にどこでメソッドが定義されているのか Object#method で確認する
* Ruby の場合は動的にメソッドが定義されていたりすることがあるので Object#method を使うのが一番確実

```ruby
class User < ActiveRecord::Base
  # 実際にメソッドがどこで定義されるのか調べる
  # これは Ruby の標準の機能かつ実行時に値を取得するので一番確実
  pp method(:scope).source_location
  # => ["/home/worker/.rbenv/versions/2.7.1/lib/ruby/gems/2.7.0/gems/activerecord-6.0.3/lib/active_record/scoping/named.rb",
  #     170]

  # ちなみに Ruby 2.7 以降だと p や pp でそのまま出力しても表示される
  pp method(:scope)
  # => #<Method: #<Class:ActiveRecord::Base>(ActiveRecord::Scoping::Named::ClassMethods)#scope(name, body, &block) /home/worker/.rbenv/versions/2.7.1/lib/ruby/gems/2.7.0/gems/activerecord-6.0.3/lib/active_record/scoping/named.rb:170>
end
```

>>>

#### 03. 実装を読む
- - -

* Object#method で取得した情報を元にファイルを開いて実際に定義されているメソッドの中身を読んでいく
    * [このあたり](https://github.com/rails/rails/blob/v6.0.3/activerecord/lib/active_record/scoping/named.rb#L170-#L205)

```ruby
def scope(name, body, &block)
  unless body.respond_to?(:call)
    raise ArgumentError, "The scope body needs to be callable."
  end

  if dangerous_class_method?(name)
    raise ArgumentError, "You tried to define a scope named \"#{name}\" " \
      "on the model \"#{self.name}\", but Active Record already defined " \
      "a class method with the same name."
  end

  if method_defined_within?(name, Relation)
    raise ArgumentError, "You tried to define a scope named \"#{name}\" " \
      "on the model \"#{self.name}\", but ActiveRecord::Relation already defined " \
      "an instance method with the same name."
  end

  valid_scope_name?(name)
  extension = Module.new(&block) if block

  if body.respond_to?(:to_proc)
    singleton_class.define_method(name) do |*args|
      scope = all._exec_scope(name, *args, &body)
      scope = scope.extending(extension) if extension
      scope
    end
  else
    singleton_class.define_method(name) do |*args|
      scope = body.call(*args) || all
      scope = scope.extending(extension) if extension
      scope
    end
  end

  generate_relation_method(name)
end
```

>>>

* 実装を読んでいくと以下のような事がわかってくる

```ruby
# body で call メソッドが呼び出せないとエラー
unless body.respond_to?(:call)
  raise ArgumentError, "The scope body needs to be callable."
end

# dangerous_class_method? とは…？
if dangerous_class_method?(name)
  raise ArgumentError, "You tried to define a scope named \"#{name}\" " \
    "on the model \"#{self.name}\", but Active Record already defined " \
    "a class method with the same name."
end

# method_defined_within? とは…？
if method_defined_within?(name, Relation)
  raise ArgumentError, "You tried to define a scope named \"#{name}\" " \
    "on the model \"#{self.name}\", but ActiveRecord::Relation already defined " \
    "an instance method with the same name."
end
# 省略
```

>>>

#### 04. 気になった部分を掘り下げていく
- - -

* 先程の実装でよく知らないメソッドが出てきました
    * dangerous_class_method? と method_defined_within?
* このメソッドが何をやっているのか気になるので次はこの実装を…という感じで気になったメソッドをどんどん掘り下げていきコードを読んでく
    * また #method で掘り下げていくこともあればそのファイルで検索したり gem 全体で grep することもある
* もちろん関係なさそうな処理をすっ飛ばして次のコードを読んでいくのも OK
    * コードリーディングは自由

---

## まとめ

---

#### まとめ
- - -

* ActiveRecord はいろんなことをしているので全部を読むのは難しい                           <!-- .element: class="fragment" -->
    * なのでまずは気になる機能など小さな目的を見つけて読んでいくと入りやすい
* ActiveRecord は Ruby で書かれているので Ruby がわかっていれば読むことはできる                           <!-- .element: class="fragment" -->
    * 複雑なコードを見てもパニックにならず冷静に読んでみるのが大事
    * 実際 Ruby がわかっていれば（処理は複雑だが）読み解くのはそこまで難しくはない
* 読んでて意図しない動作をした場合は #method で定義位置を確認したりデバッグ出力する                           <!-- .element: class="fragment" -->
    * たまに全然違うメソッドの実装を読んでいる事がある
    * ちゃんと意図するメソッドが呼ばれているかどうか確認しよう
* ActiveRecord を読みたい場合はまずは Ruby の気持ちを理解するところから！！                           <!-- .element: class="fragment" -->
    * ActiveRecord なにもわからないけど Ruby はチョットデキルのでなんとか読むことはできる
* 記述したデバッグコードはちゃんと消そう…                           <!-- .element: class="fragment" -->

---


## ご清聴
## ありがとうございました

