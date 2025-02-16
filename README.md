# Swagger（OpenAPI）仕様のAPI開発を行うための環境をdockerで構築した覚書  

あちこちのサイトから拾って以前に作ったが今となってはネタ元不明。  
久々に引っ張り出したら、動かなかったの修正した。


* 分割して編集したファイルを結合
* APIドキュメントをブラウザ表示
* APIモックサーバを起動
* APIドキュメントを静的HTMLで出力
* コード生成

# 環境構築

### 動かした環境
`macOS Sonoma + colima + docker`

### コンテナ起動
`docker compose up -d`

これで、SwaggerUIによるドキュメント表示、ReDocによるドキュメント表示、モックサーバーが利用可能。  

# 分割して編集したファイルを結合
`/workspace/documents/`下でにAPI仕様を記述する。 
`/workspace/documents/main.yaml`は必須。 
各ファイルの変更を検知して自動で`/workspace/dist/openapi.yaml`が作成される。  

openapi.yamlは結合結果のファイルなので、このファイルを直接編集しないこと。  

# APIドキュメントをブラウザ表示
UIは、SwaggerUIとReDocの両方で表示可能なので、好みの方を利用する。  
ReDoc `http://localhost:8010`  
SwaggerUI `http://localhost:8011`  

ホストPCでyamlファイルを編集後、ブラウザリロードで画面に反映される。  

# APIモックサーバ
`http://localhost:8012`でモックAPIにリクエスト可能。  
（GET）`http://localhost:8012/pets` にリクエストすれば、サンプルAPIドキュメントのレスポンスを確認可能。  

# APIドキュメントを静的HTMLで出力
本プロジェクトのルートディレクトリで、下記コマンドを実行し静的HTMLを出力可能。  
```bash
docker run --rm --mount type=bind,source="$(pwd)"/workspace,target=/workspace redocly/cli \
build-docs /workspace/dist/openapi.yaml -o /workspace/html/openapi-redoc.html
```

```/workspace/html/openapi-redoc.html```いうファイルが出力されるので、それをそのままブラウザで表示可能。  
単一のhtmlファイルのみで、他のcssやjsファイルに依存していないため、1枚のhtmlファイルのみでAPIドキュメントとして配布可能。  

# コード生成
`openapi-generator-cli` でコード生成が可能。
`-g` の後に言語を指定する。

```bash
docker run --rm --mount type=bind,source="$(pwd)"/workspace,target=/workspace openapitools/openapi-generator-cli \
generate -i /workspace/dist/openapi.yaml -g java-helidon-server -o /workspace/generate/java-helidon-server
```
対応言語は
[READMEのOverview](https://github.com/OpenAPITools/openapi-generator?tab=readme-ov-file#overview)
を参照。

# コンテナの停止と削除
```bash
docker rm $(docker stop $(docker ps -a -q --filter="name=openapi"))
```
```openapi```を含むコンテナは全て停止され削除されるので注意