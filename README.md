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

# reuse
我々は、時々同じクエリを使いましたい場面に遭遇します。
pinqは、クエリの再利用を提供し、スマートなコードを提供します。

```
query = pinq.dummy().map(lambda x: x * 2)

for item in query.attach([1, 2, 3])
  print(item)
  # =>  2, 4, 6

for item in query.attach([4, 5, 6])
  print(item)
  # =>  8, 10, 12
```

