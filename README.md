# pinq
pythonにおけるlinqの実装を目指したライブラリです。
イテレータに対して汎用的な機能を提供します。


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
