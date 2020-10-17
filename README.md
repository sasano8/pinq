# pinq
pythonにおけるlinqの実装を目指したライブラリです。
イテレータに対して汎用的な機能を提供します。

# filter
builtinのfilterと違い、複数の条件を位置限定引数として受け入れ可能です。（空も可能）
これにより、動的な条件組み立てなど高度なクエリの表現が可能です。

```
query = pinq([-1, 0, 5, 10]).filter(
  lambda x: 0 <= x,
  lambda x: x < 10,
)
list(query)  # => [0, 5]

query = pinq("abcdefg").filter_or(
  lambda x: x == "a",
  lambda x: x == "c",
  lambda x: x == "f",
)
list(query)  # => ["a", "c", "f"]

condition = dict(
  contry="tokyo",
  age=20,
  sex=None,
)
query = Linq(condition).items().filter(lambda x: not x[1] is None).map(lambda x: op[x[0]] == x[1])
result = pinq(customers).filter_or(**query)
```

# stateless iterator
mapやfilterといったビルドインのイテレータhook関数は一度イテレータを消費すると、再消費ができません。（statefull iterator）
これにより、以下のような問題を引き起こします。

```
query = map(lamdba x: x, [1, 2, 3])

for item in query:
  print(item)
  # => 1, 2, 3

for item in query:
  print(item)
  # =>

```

上記の挙動は非直感的であり、immutableでありません。
pinqが提供する関数は、__iter__要求時にイテレータを生成し、一貫的な挙動を提供します。

```
query = pinq([1,2,3]).map(lambda x: x)
for item in query:
  print(item)
  # => 1, 2, 3

for item in query:
  print(item)
  # => 1, 2, 3
```

ただし、イテレータはルートイテレータの挙動を引き継ぐことに注意してください。消費するイテレータをルートイテレータに利用すべきではありません。

```
query = Linq(map(lamdba x: x, [1, 2, 3]))

for item in query:
  print(item)
  # => 1, 2, 3

for item in query:
  print(item)
  # =>
```


# reuse
我々は、時々同じクエリを使いましたい場面に遭遇します。
pinqは、クエリの再利用を提供し、スマートなコードを提供します。

```
# way 1
query = pinq.dummy().map(lambda x: x * 2)

for item in query.attach([1, 2, 3])
  print(item)
  # =>  2, 4, 6

for item in query([4, 5, 6])  # attachと同義
  print(item)
  # =>  8, 10, 12

# way 2
filter_even = pinq.dummy().filter(lambda x: x % 2 == 0)

for item in filter_even([7, 8, 9])
  print(item)
  # =>  8
```

# generator
generator関数でイテレータを取得するには、generator関数を実行する必要があります。
Linqを使うことで、generator関数を隠蔽し、イテレートを要求時に何度でもイテレータを生成するオブジェクトを簡単に作成することができます。

```
def generate():
  yield 1
  yield 2

# way 1
for item in generate():
  print(item)

# way 2
generator = Linq(generate)

for item in generator:
  print(item)

# way 3
@Linq
def generate():
  yield 1
  yield 2

for item in generate:
  print(item)
```

# observable

```
import random
_is_cancelled = False

def is_cancelled():
  return _is_cancelled

def update_cancel():
  val = random.random()
  if val > 0.95:
    _is_cancelled = True

def get_data():
    return {"name": "test", "age": 20}

database1 = []
database2 = []

observer = (
  Linq.dummy()
  .dispatch(lambda x: update_cancel())
  .until(is_cancelled)
  .repeat_infinity()
  .take(1000)
  .dispatch(
    database1.append,
    database2.append
  )
)
observer.subscribe(get_data)
```

# observable
``` py
event = Linq.event(
  on_error=lambda e: print("error"),
  on_success=
  on_finally: print("complete")
)
event_even = Linq.dummy().filter(e => e % 2 == 0).dispatch(print)  # observer event handler
event.add_handler(event_even)

event.occur(1)
# =>
event.occur(2)
# => 2 

event.take(5).map(int).stream(["1", "2", "3", "4", "5", "6"])
# => 2 4 complete

event.take(10).stream([2, "a"])
# => 2 error


どんなパターンがあるか

```
query_dispatch_even = as_query = Linq.dummy().filter(e => e % 2 == 0).each(print)
func_dispatch_even = as_query = Linq.dummy().filter(e => e % 2 == 0).each(print).to_func()
func_dispatch_even = as_query = Linq.dummy().filter(e => e % 2 == 0).each(print).to_list
result = query_dispatch_even([1, 2, 3, 4])
print(result) # => type lazylinq
for item in query_dispatch_even([1, 2, 3, 4]):  # => 2, 4
  pass

result = func_dispatch_even([1, 2, 3, 4])  => 2, 4
print(result) # => type func
for item in func_dispatch_even([1, 2, 3, 4]):  # => 2,4 戻り値はイテレータか、ファイナライザに応じた型がくるため実行可能
  pass

# オブサーバブルに流用することができる
event.add_handler(query_dispatch_even)
event.stream([1, 2, 3, 4, 5])

for item in 

```
