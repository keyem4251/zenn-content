---
title: "RFC7519: JWTとJOSEワーキンググループ（JWS、JWE、JWK）"
emoji: ""
type: "tech"
topics: ["rfc", "authentication", "oauth", "jwt", "jose", "jws", "jwe", "jwk"]
published: false
---

# はじめに
OIDC、OAuthで利用されるJWTについてRFCを調べたのでまとめようと思います。
RFC7519のJWTだけでなく、JSON形式のデータに署名や暗号化を使用するための要件を定めた全体をまとめたJOSE、署名、暗号化などのJWS、JWE、JWKについても一緒に説明をしようと思います。

# 概要
[以前の記事でRFC6750: Bearerトークンについて](https://zenn.dev/keyem4251/articles/rfc-6750-oauth-beaer-token)まとめましたが、具体的に利用されるトークンであるJWTについてまずは記載して、その後JOSEなどについて記載していきます。
JWTは当初は認可を扱うOAuth2.0で利用されましたが、その後で認証を扱うOIDCでも利用されています。これはそれまでの認証認可で懸念されていたセキュリティ的な対策をJOSEで行うことができるために、認可だけでなく認証でも同様のフローを踏襲しているのかなと思います。（同様のフローを踏襲していること自体は認証・認可が同じようなタイミングでリクエスト/レスポンスされるなど他にも理由はあると思うので、あくまでJWTに関するところ）

# JWT
JWTは二者間（クライアント、サーバー）の間でデータをURLセーフな形式で安全にやり取りするためのデータフォーマットです。
RFC7519自体に記載されている内容はシンプルで、JWTがリクエスト時にどのような形式で送られるか？中身がどのように定義されているか？といったことが書かれています。
そのデータを安全に受け渡すためにどう署名するか、暗号化するかなどは後述するJOSE、JWS、JWEなどに定義されているという関係性になっています。
JWTは3つの文字列で構成されています。
- Header: JWTがどういった署名アルゴリズムを使用しているかが記載される。（トークンの種類も記載されているが、JWTの場合は `jwt`）
- Payload: クライアントがサーバーに渡したいデータの本体。`iss`、`sub`、`exp`などどういった値が入るかの代表的なものはRFC7519で定義されている。
- Signature: 改ざん検知のための暗号化署名。HeaderとPayloadを結合して鍵で署名した値。

これらのパーツをbase64urlでエンコードして、ピリオド（`'.'`）で区切った値を送ります。

## 具体的なJWTの例
具体的にJWTがどういった値のものなのかを簡単な例を使って紹介します。
それぞれの詳細な意味についてはここでは説明は省くので、こんな値に変換されて、最終的にこんな値になるんだというイメージを掴んでもらえたらと思います。

Headerのもとの値
```
{
    "typ":"JWT",
    "alg":"HS256"
}
```
Headerのエンコード後
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

Payloadのもとの値
```
{
    "iss":"joe",
    "exp":1300819380,
    "http://example.com/is_root":true
}
```
Payloadのエンコード後（スペースなどは取り除いてます）
```
eyJpc3MiOiJqb2UiLCJleHAiOjEzMDA4MTkzODAsImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ
```
Signatureのもとの値（Header、Payloadのエンコード後の値を（`'.'`）でつなぐ
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLCJleHAiOjEzMDA4MTkzODAsImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ
```
Signatureの最終的な値（鍵を用いてHS256でハッシュ化し、エンコードする。ここでは適当な文字列を鍵としています）
```
B5Y3F93iWbTjV1B9_G6Q-e0sC8_b3kZ4R6wH_eW8bI0
```

これにより最終的なJWTは以下のようになります。
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLCJleHAiOjEzMDA4MTkzODAsImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.B5Y3F93iWbTjV1B9_G6Q-e0sC8_b3kZ4R6wH_eW8bI0
```
HeaderとSignatureについてはRFC7519では中身の詳細は定義されず、JWTをどう安全にやり取りするか？を決めているJWS、JWEのために使用されます。JWS、JWEについては後ほど説明をしようと思います。
PayloadについてはJWTでやり取りしたいデータそのものなので、RFC7519で主要な値が定義されており、このまま簡単に説明したいと思います。

## Payloadの主要な値
JWTの中の値はクレーム（Claim）と呼ばれており、主要な値についてはRFC7519で定義されています。
具体的には以下のものになります。
- iss: 発行者。JWTを発行したのが誰か？という値が入る。
  - `"[https://auth.example.com](https://auth.example.com)"`のような認可サーバーのエンドポイントなど
- sub: JWTが誰？あるいは何に関するものなのか？
  - 一般的にはJWTでやり取りしたいユーザーIDなど
- aud: JWTを受け取るサーバー。APIサーバーやリソースサーバーが受け取ったJWTが自分宛てのものかを確認する。
  - `https://api.example.com/v1`のようなAPIサーバーのエンドポイントなど
- exp: JWTの有効期限。Unixタイムスタンプが入る。
他にもRFC7519には `iat`: 発行日時、`jti`: JWTの識別子（ID）、`nbf`: JWTの有効開始日時などが定義されています。
またRFCに定義されている以外の情報を受け渡したい場合にはJSONに独自のキー、バリューを入れて受け渡すこともして良いと定義されています。具体的な説明はここでは割愛します。

## JWTのまとめ
このようにJWTは3つのパーツを使い、データを渡すための仕組みになっています。
ただRFC7519のJWTだけでは「安全に」という仕組みは定義されおらず、それを実現するのがJOSEなどになります。
ここからはそのJOSEについて説明していきます。

# JOSE
JOSEではここまで説明してきたJWTをどう安全にやり取りするかということが定義されています。
RFC7165で定義されており、JSON Object Signing and Encryptionの略でJSONを署名・暗号化して安全にやり取りするための仕様をまとめたものになります。IETFというインターネットの技術標準化の機関でワーキンググループがあり、そこで定義されました。
JOSEではJSONを「整合性」、「機密性」、これらを実現するための「鍵」の3つの構成要素でどうやり取りするかということをまとめています。
それぞれがJWS、JWE、JWKとして対応しており、RFC7515、RFC7516、RFC7157として定義されています。厳密にはさらに鍵をどういうアルゴリズムで使用するかを定めたJWA（RFC7518）もあります。

ではここからJWS、JWE、JWKとJWAについて簡単に説明をしていきます。

## JWS
JWSはJWTでやり取りされるデータの整合性を担保する仕組みで、RFC7515で定義されています。
JWTの具体的でも説明しましたが、JWTのHeader部分に署名アルゴリズム、SignatureにHeader、Payloadの連結した署名とすることでクライアント、サーバー間でデータが改ざんされていないかを検証することでデータの整合性を担保します。
先ほどのJWTの例を見ると `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLCJleHAiOjEzMDA4MTkzODAsImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.B5Y3F93iWbTjV1B9_G6Q-e0sC8_b3kZ4R6wH_eW8bI0` という値をサーバーが受け取った場合には
- トークンを`'.'`でHeader、Payload、Signatureに分割する。
```
Header: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
Payload: eyJpc3MiOiJqb2UiLCJleHAiOjEzMDA4MTkzODAsImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ
Signature: B5Y3F93iWbTjV1B9_G6Q-e0sC8_b3kZ4R6wH_eW8bI0
```
- Headerのアルゴリズムを確認する。
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```
をbase64urlデコードして以下に戻して `HS256` というのを確認。
```
{
    "typ":"JWT",
    "alg":"HS256"
}
```
- HeaderとPayloadを`'.'`で繋いで鍵と `HS256` でハッシュ化する。（鍵については共通鍵暗号化方式、公開鍵暗号化方式をサポート）
- 送られてきたSignature（`B5Y3F93iWbTjV1B9_G6Q-e0sC8_b3kZ4R6wH_eW8bI0`）と一致するかを検証する。
  - ハッシュ化しているためPayloadやHeaderの中身が改ざんされている場合にはSignatureは異なる。

## JWE


# まとめと所感
