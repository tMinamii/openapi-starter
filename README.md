# Usage #

- `npm run build` YAMLを結合して単一のjsonを`build`に生成する
- `npm run redoc` ReDocのHTMLを`build`生成する
- `npm run mock` PrismでMockサーバーを建てる
- `npm run ui` SwaggerUIサーバーを建てる

## VSCodeではじめるOpenAPI - 1 ##

**本記事はOpenAPI3.0を使用しています**

最近のAPIドキュメント作成といえばOpenAPIですが、意外にもOpenAPIの開発環境やプラクティスに関する記事があまりありません。
OpenAPIは素晴らしいツールなのですが、 APIの数を増やしていくと、あっという間に数千行のYAMLが誕生してしまいます。
長期的にドキュメントを保守していくには、開発環境を整え、ディレクトリ構造を整理してファイル分割するなど戦略を練らないといけません。
今回は、実開発にて手探りで調査・探求して、プラクティスがそれなりにまとまったので、それらを説明したいと思います。

これから紹介する必要な設定やツールとOpenAPIのサンプルを下記のリポジトリにまとめています。
https://github.com/tMinamii/openapi-starter

## 本記事のキーワード ##

- OpenAPIの開発環境
- OpenAPI関連のツール
- 数十のAPIを長期間保守することを意識したディレクトリ構造

## 開発環境 ##

### VSCodeの拡張 ###

```json:extensions.json
{
    "recommendations": [
        // OpenAPI (Swagger) Editor
        "42crunch.vscode-openapi",
        // OpenApi Designer
        "philosowaffle.openapi-designer",
        // Prettier
        "esbenp.prettier-vscode"
    ]
}
```

#### OpenAPI (Swagger) Editor ####

ID: [42crunch.vscode-openapi](https://marketplace.visualstudio.com/items?itemName=42Crunch.vscode-openapi)

- インストールしておくと`$ref`の参照先YAMLにコードジャンプできます。 APIの数が増えるとファイルが数十・数百と増えていくのコードジャンプできるのは非常に便利です
- 他にもナビゲーション機能がありますが、分割すると使用できないので実質コードジャンプのためだけに使用しています

[![OpenAPI (Swagger) Editor](https://i.gyazo.com/ca46eed18452a73b419a0c39f880257b.gif)](https://gyazo.com/ca46eed18452a73b419a0c39f880257b)

#### OpenApi Designer ####

ID: [philosowaffle.openapi-designer](https://marketplace.visualstudio.com/items?itemName=philosowaffle.openapi-designer)

- プレビュー表示する拡張
- ファイル分割したopenapi.ymlにも対応しています
- yamlファイルを編集して保存すると、プレビューも自動更新されます
- ビューワーはコマンドパレットから起動します `ctrl-shft-p > OpenApi Designer: Preview`

[![Image from Gyazo](https://i.gyazo.com/f70f176c95c7025692979877fc2fa2b3.gif)](https://gyazo.com/f70f176c95c7025692979877fc2fa2b3)


#### Prettier ####

ID: [esbenp.prettier-vscode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

- YAMLのフォーマッタ(https://prettier.io/)

### 便利なツール ###

#### ファイル結合 Swagger/OpenAPI CLI ####

https://github.com/APIDevTools/swagger-cli

OpenAPIには分割されたYAMLに非対応なツールもあります。
次に説明する静的HTML生成ツールReDocもその一つです。
またエンジニア間で共有する場合も分割されていると不便なのでswagger-cliを用いて結合しています。

```sh
swagger-cli bundle -r openapi/openapi.yml -o build/openapi.json
```

出力形式はyamlとjsonどちらも対応していますがjsonのほうが汎用的かなと思いデフォルトのjsonにしています。

[![Swagger/OpenAPI CLI](https://i.gyazo.com/c77cc1a64d9f0d233c606bd690db7cce.png)](https://gyazo.com/c77cc1a64d9f0d233c606bd690db7cce)

#### 静的HTML生成 ReDoc ####

https://github.com/Redocly/redoc

openapi.jsonから静的なHTMLを生成するツールの中でもっともキレイで見やすいとおもったのがReDocです。
成果物が単一のHTMLファイルというのも魅力です。
納品物としてドキュメントを提出したり、社内で情報共有する際にとても重宝します。

```sh
redoc-cli bundle build/openapi.json --options.menuToggle --options.pathInMiddlePanel  -o build/redoc-static.html
```

自分はオプションで以下の2つをONにしています

- `--options.menuToggle`:  左のmenuをトグルできるようにする
- `--options.pathInMiddlePanel`: 中央にAPIのエンドポイントパスを表示する

[![ReDoc](https://i.gyazo.com/d2ff37f3cb9fd1543997a3dc2e964bec.png)](https://gyazo.com/d2ff37f3cb9fd1543997a3dc2e964bec)

#### Viewer(Swagger UI) ####

ReDocではなくSwaggerUIのほうが見やすいという方は、dockerを使用してローカルにSwaggerUIをホストしましょう

```sh
docker run -p 8081:8080 -v $(pwd)/openapi:/usr/share/nginx/html/openapi -e API_URL=openapi/openapi.yml swaggerapi/swagger-ui
```

[![SwaggerUI](https://i.gyazo.com/8cf28cdc6a41c0e6fe55ba44990eb556.png)](https://gyazo.com/8cf28cdc6a41c0e6fe55ba44990eb556)

#### Mockサーバー Prism ####

https://github.com/stoplightio/prism

ドキュメント通りにAPIのMockサーバーを建てることができます。戻り値はexampleを返します。

```sh
prism mock openapi/openapi.yml
```

[![Prism](https://i.gyazo.com/f7e066a1c8e1a5bcc709ac7c30f0b4e9.png)](https://gyazo.com/f7e066a1c8e1a5bcc709ac7c30f0b4e9)

## ファイル分割とディレクトリ構造 ##

使い回すという観点よりも**長期的にドキュメントを保守すること**を考えて末にたどり着いたディレクトリ構造です。

OpenAPIは単一のファイルにまとめるとあっという間に数千行になります。
単一ファイル内でcomponent化して再利用することもでいますが、$ref文はgoto文と同じなため、数千行スパゲッティコードと格闘することになります。
$ref文の参照先が、なるべくが散らばらないようにし、APIを追加するときは毎回同じ作業になるよう意識して分けました。

```text
./
 +- openapi
    |- openapi.yml
    +- components       分割したYAMLファイルの置き場
         |- errors        エラー系の戻り値の構造とExample
         |- examples      正常系の戻り値の例
         |- parameters    APIのパラメタ
         |    |- header     headerのパラメタ
         |    |- path       pathパラメタ
         |    +- query      queryStringパラメタ
         |
         |- paths         openapi.ymlのpaths
         |- requestBody   リクエストボディ(POST/PUTのBody)
         +- schemas       各APIのスキーマ
```

- openapi.ymlに新しいエンドポイントの一覧になります。openapi.ymlではエンドポイントを俯瞰しやすくするため、なるべく記述せず分割・参照します
- components/pathsににはエンドポイントが処理するGET/POST/PUT/DELETEなどの記述をしていきます
- 戻り値は複雑になりがちで行数が多くなるためpathsに書かずschemaとexampleに分割して参照します
  - schemasには戻り値の構造を記述します
  - examplesに戻り値の例を記述します
- errors, requestBody, parametersは使います機会が多いので使いやすい粒度で分割しておきます
  - parametersはheader, path, queryに分けています。これは同じパラメタ名がpathとquery両方に存在するケースがあり、間違えて参照しないよう分けています

実際にファイルを設置するとこのようになります。
[![Image from Gyazo](https://i.gyazo.com/776ea363e685ba2a9375f377e0aafbf1.png)](https://gyazo.com/776ea363e685ba2a9375f377e0aafbf1)

## まとめ ##

- OpenAPI3.0対応の便利な拡張・Toolをつかおう
- 長期運用する場合はかならずファイル分割しよう
- $refによるスパゲッティコード化を防ぐためディレクトリ構造を工夫しよう