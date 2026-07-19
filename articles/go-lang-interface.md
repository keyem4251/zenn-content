---
title: "Goらしいインターフェースの書き方: 利用する側に定義する"
emoji: "🐭"
type: "tech"
topics: ["go", "interface"]
published: false
---
# はじめに
Goを書き始めてGoではインターフェースを利用する側に定義するという慣習があるというのを知りました。いろいろな記事を見ると絶対というわけではないようですが、頭の片隅にはおいておいたほうが良さそうと思い自身の勉強も兼ねて簡単にまとめようと思います。

# 原典
[Go WikiのCodeReviewComment Interface](https://go.dev/wiki/CodeReviewComments#interfaces)を見てください。ちなみに以下の記事はほぼ日本語に訳したのみではあります。


> Go interfaces generally belong in the package that uses values of the interface type, not the package that implements those values. The implementing package should return concrete (usually pointer or struct) types: that way, new methods can be added to implementations without requiring extensive refactoring.

> Do not define interfaces on the implementor side of an API “for mocking”; instead, design the API so that it can be tested using the public API of the real implementation.

> Do not define interfaces before they are used: without a realistic example of usage, it is too difficult to see whether an interface is even necessary, let alone what methods it ought to contain.

# それぞれのセクションを要約
## Goのインターフェースは利用する側に定義する
１つ目のセクションの部分について、記事のタイトルにもある内容ですが、インターフェースは利用する側に定義するということが書かれています。また提供するパッケージでは構造体などの具体的な型を返すということが書かれています。
インターフェースを利用側に置くことで、利用側で必要最小限の抽象化されたインターフェースを利用することができるようになります。これにより適切に抽象化されていないインターフェースを提供側に置き、複数の利用者が参照している状況で、そのインターフェースに変更が入ると修正範囲が大きくなる。特定の利用者の都合でインターフェースを変更するが、他の利用者と調整が必要になるなどが起きることを懸念しているのかなと思いました。

## 実際の公開されているAPIを使用してテストする
提供側が具体的な構造体を返すことで、実際の振る舞いに基づいたテストを行うようにする。
これは1つめのセクションを守りつつ、インターフェースを作る際にモックを目的として作らないようにするということですね。

## インターフェースは必要になったら作る
実際に使用されるようになってからインターフェースを作るようにする。どういったメソッドが必要かは実際に使う段階にならないとわからない。

# まとめ
全体的にインターフェースを必要最小限にして、具体的な実装/現実の使用例に重きを置くという思想だなと感じました。
まだGo自体が触り始めたばかりで解釈が間違っているところもあると思うので、ここ違うよなどあればコメント頂けると大変助かります！
