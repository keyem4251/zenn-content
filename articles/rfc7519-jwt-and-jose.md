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
# JOSE

# まとめと所感
