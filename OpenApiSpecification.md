# OpenApi Specification (OAS)

## はじめに

FrontendとBackendエンジニア間のAPIの共有、FEやBEエンジニアにAPI明細を聞きたいときどうしてますか？

1. 口頭で聞く
   - いつか引き継ぎする日が来ると思います。  
2. コードを見るように
   - コードを見ると終わりがない
3. タイプを定義したDocument(ex. wiki, qitta)
   - いつも最新に更新されてますか？
4. Post Man Collection
   - これがベストなのか
   - pricing issue


## OpenApi Specificationとは？

RESTful APIを定義されている規則に合わせて`API spec`を`json`や`yaml`で表現する方法

### Example

``` yaml
openapi: 3.0.1 # openapi 버전
info:
  title: OpenAPI definition
  version: v0
servers:
- url: http://localhost:8080
  description: Generated server url
paths:
  /users:
    get:
      operationId: getUsers
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/users'
components:
  schemas:
    users:
      type: object
      properties:
        id:
          type: uuid
        age:
          type: integer
        address:
          type: string
        phone:
          type: string
        inserted_date:
          type: string
          format: date-time
```

### こういう事も書けます

- API path (dev, production)
- http method
- request parameter, header
- 状況別`response`
- コメントまで

#### 例

- API path (dev, production)

```yaml
servers:
- url: https://development.example.com/v1
  description: Development server
- url: https://api.example.com/v1
  description: Production server
```

- http method

```yaml
（前略）
paths:
  /users:
    get:
      operationId: getUsers
      responses:
        "200":
          description: OK
    post:
      operationId: postUsers
（後略）
```

- request parameter, header

```yaml
headers:
  X-Rate-Limit-Limit:
    description: The number of allowed requests in the current period
    schema:
      type: integer
```

- 状況別`response`

```yaml
responses:
  '200':
    description: your car appointment has been booked
    content: 
      application/json:
        schema:
          $ref: '#/components/schemas/SuccessResponse'
        examples:
          confirmation-success:
            $ref: '#/components/examples/confirmation-success'
```

- request/response example data

```yaml
parameters:
  - name: 'zipCode'
    in: 'query'
    schema:
      type: 'string'
      format: 'zip-code'
    examples:
      zip-example: 
        $ref: '#/components/examples/zip-example'
```

- コメント

```yaml
description: The number of allowed requests in the current period
```

### どこに使えますか？

- Postman Collectionに変換可能
- OpenApi-Generator
  - コードの自動作成

## OpenApi-Generator

`OpenAPI Generator`は`OpenAPI`仕様(2.0と3.0)から`API Client(SDK)`, Server Stub, ドキュメントを自動で生成できるツールです。

### OpenApi-Generatorを利用すると？

Backend

- APIベースの`Class`が自動で作成されます。
- **API-first Developmentが可能！**

Frontend

- API通信関数自動生成
- API requestとresponseよに使われるタイプを自動生成
- Sample Dataを返すStubサーバーコードを自動生成
- **サーバの開発が終わらなくても開発可能!**

## OASを導入した時の明るい未来

- ビジネスロジックやViewに集中可能
- 人が作成するコードの量が減るためヒューマンエラーが減る

## OASを導入した時の暗い未来

既存API明細をドキュメント化（json, yaml）するのに時間が...:pleading_face:

- 現実的にかなり時間が掛かる。

言語別に必ず違う部分がある（enum, date format）

- Generator毎に管理者が違うため、メンテされる頻度が違う
- 場合によってbraking-changeが多い
- そらか直接明示するしかない`--type-mappings date=string,object=any`

specの１行が影響を与える部分が多い

- `required: true` propertiesを忘れて全てが`Object?.{props}`になってしまう
- specのコードレビューに集中する必要がある 

generatorを100％信頼するのは難しい

## 意見

OASはとてもメリットが多いとても魅力的な記述だが、まだマイナーという問題があります。導入する場合コストをちゃんと検討した方が良いと思います。

## 参考記事

- [OpenAPI SpecificationでタイプセーフにAPIを開発しよう](https://www.youtube.com/watch?v=J4JHLESAiFk)
- [github/OpenAPI-Specification](https://github.com/OAI/OpenAPI-Specification)
- [swagger](https://swagger.io/specification/)