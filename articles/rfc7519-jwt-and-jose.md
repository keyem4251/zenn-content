---
title: "RFC7519: JWTとJOSE（JWS、JWE、JWK、JWA）"
emoji: ""
type: "tech"
topics: ["rfc", "authentication", "oauth", "jwt", "jose", "jws", "jwe", "jwk"]
published: true
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
JWTは3つの文字列で構成されています。JWTは基本的にJWS、JWEを利用されるので、中身の値にはそれらに使われる情報もここでは説明しないけど、例として入れておきます。
- Header: JWTがどういった署名アルゴリズムを使用しているかが記載される。トークンの種類も記載されているが、JWTの場合は `jwt`。また暗号化、署名に利用される鍵のIDを `kid` としてつける。
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
    "alg":"HS256",
    "kid": "key-001"
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
それぞれがJWS、JWE、JWKとして対応しており、RFC7515、RFC7516、RFC7517として定義されています。厳密にはさらに鍵をどういうアルゴリズムで使用するかを定めたJWA（RFC7518）もあります。

JWTとJOSEについての関係性はauth0の「[Demystifying JOSE, the JWT Family: JWS, JWE, JWA, and JWK Explained](https://auth0.com/blog/demystifying-jose-jwt-family/)」という記事が非常にイメージとしてわかりやすいです。
簡単に解説するとJOSEはデータを安全にやりとりするための規格で、JWTはそのデータ自体。具体的にはJWTを運びたい荷物だとすると、JWSあるいはJWEは荷物を梱包するコンテナや金庫のようなもので、JWKはコンテナ、金庫を開けるための鍵、JWAはダイヤルなのか、南京錠なのかなどの施錠の仕方。JOSEはそれらを取りまとめる荷物を安全に届けるための手法を定義していったものということです。

ではここからさらにJWS、JWE、JWKとJWAについて簡単に説明をしていきます。

## JWS
JWSはJWTでやり取りされるデータが改ざんされていないか、送った相手が正しいかを担保する仕組みで、RFC7515で定義されています。
JWTの概要でも説明しましたが、JWTのHeader部分に署名アルゴリズム、SignatureにHeader、Payloadの連結した署名とすることでクライアント、サーバー間でデータが改ざんされていないかを検証することでデータの整合性を担保します。
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
    "alg":"HS256",
    "kid": "key-001"
}
```
- HeaderとPayloadを`'.'`で繋いで鍵と `HS256` でハッシュ化する。（鍵については共通鍵暗号化方式、公開鍵暗号化方式をサポート）
- 送られてきたSignature（`B5Y3F93iWbTjV1B9_G6Q-e0sC8_b3kZ4R6wH_eW8bI0`）と一致するかを検証する。
  - ハッシュ化しているためPayloadやHeaderの中身が改ざんされている場合にはSignatureは異なる。

## JWE
JWEではJWTでやり取りされるデータが盗聴された場合に中身を見られても大丈夫なように暗号化する仕組みで、RFC7516で定義されています。
JWEはHTTPSによる通信時の暗号化がある、暗号化・復号化のパフォーマンスや鍵管理の手間などから利用される頻度は少ないですが、金融系やIDプロバイダーなどJWTの中身にマイナンバーや秘匿性の高い情報を扱う場合に利用されます。また複数のマイクロサービスやシステム間の連携の際に、仲介するシステムにはJWTの中身を知られたくない場合にも利用されます。（HTTPSが通信中を保護するだけなので、仲介するサーバーも中身が見れてしまうため）
JWEでは暗号化するための鍵の受け渡しも必要になるためデータは3つのパーツではなく、5つのパーツで構成されるようになります。
- Header: `enc`でPayloadを暗号化するためのアルゴリズム、 `alg`は鍵をどう受け渡すかのアルゴリズム（RSA公開鍵など）
- Encrypted Key: 暗号化された鍵。`alg`で指定された鍵（RSA公開鍵など）で暗号化されています。
- IV: 初期化ベクトル。ランダムな値が生成される。イメージとしてはハッシュ化におけるSaltのようなもの（厳密には違うので注意）
- Ciphertext: 暗号文。もとのPayload。
- Authentication Tag: 認証タグ。データが途中で改ざんされていないかを確かめるもの。JWSのSignatureと同じようなもの。ただしJWSでは送った相手が正しいかも検証するが、JWEでは改ざんだけが検証されます。
JWEではこれらの値を結合して送ることになります。そのためJWSに比べると文字列は長くなり、データ量が大きくなります。

## Nested JWT
ここでJWKなどの説明の前にNested JWTと呼ばれるものを説明します。
Nested JWTとはJWSした値をさらにJWEすることで、JWTを送った相手が正しいか、値は改ざんされていないか、値は盗聴されていないかということを検証することができるものです。
さきほどJWEはセキュリティ要件が高い場合に使用されると言いましたが、逆にJWEを行う場合はJWSと組み合わせることが多いです。

## JWK、JWA
ではここまででJWS、JWEで鍵やアルゴリズムが利用されているのが、わかったと思います。
次にこの鍵やアルゴリズムについて定義しているRFC7517、RFC7518について説明していきます。

### JWK
JWS、JWEで利用される鍵を公開するためのフォーマットが定義されています。
IdPなどが自身の認証・認可に関連するエンドポイントを公開している `/.well-known/openid-configuration` の中に `jwks_uri` という値でURLを記載しています。
このエンドポイントにアクセスするとIdPが認証・認可の際に利用する鍵が公開されています。
具体的にはJWSの場合は以下のようなイメージです。これを利用し、JWSの場合だとJWTを受け取った側はJWTのHeaderに入っている `kid` を見て該当する鍵を確認し、鍵の検証を行い、送った相手が正しいかを検証することができます。
```
{
  "keys": [
    {
      "kty": "RSA",
      "use": "sig",
      "kid": "idp-sig-key-2026",
      "alg": "RS256",
      "n": "0vx7agoebGcQSuuPiL...",
      "e": "AQAB"
    }
  ]
}
```
JWEの場合だと以下のようなイメージです。これを利用し、JWEの場合だとJWTを送る側が暗号化するための鍵として使用します。JWTを受け取る側が事前に自身の鍵をIdPに登録しておき、IdPは受け取る側の `kid` を確認し、JWTのHeaderにも `kid` をつけます。
```
{
  "keys": [
    {
      "kty": "RSA",
      "use": "enc",
      "kid": "rp-enc-key-001",
      "alg": "RSA-OAEP-256",
      "n": "v3c8bO_e-GcQSuuPiL...",
      "e": "AQAB"
    }
  ]
}
```

それぞれの値は `kty`: 鍵の暗号の種類、`use`: 用途、`kid`: 鍵のID、`alg`: アルゴリズム、`n`や`e`は鍵の詳細なパラメーターとなっています。

## JWA
JWAはこれまでのJWT、JWS、JWE、JWKで出てきたアルゴリズムの識別子を定義しているRFCになります。
以下のように対応が定義されています。
- `HS256`: HMAC（Hash-based Message Authentication Code）を利用した共通鍵を使うアルゴリズム。
- `RS256`: RSA暗号とSHA-256ハッシュ関数を組み合わせた公開鍵を利用したアルゴリズム。
- `none`: 署名なし。セキュアではないので実運用では利用されない想定。

その他にもいろいろなアルゴリズムが定義されています。

# まとめと所感
JWTというと当初はJWS、JWEなどとまとめたものをイメージしていたが、それぞれのRFCを見ていくと適切にそれぞれの責務ごとに定義されているということがわかります。
認証・認可の流れで[以前の記事でBasic認証](https://zenn.dev/keyem4251/articles/rfc7617-basic-auth)を紹介しましたが、パスワードなどをそのまま送っていた状態から比較するとJWTでは認証・認可に利用する値自体とそれ以外のセキュリティ要件を分割することで現代でも利用されるようになっているということがわかります。
