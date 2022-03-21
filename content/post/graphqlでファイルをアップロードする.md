+++
author = "すかい"
title = "graphqlでファイルをアップロードする"
date = "2018-08-26"
description = "graphqlでファイルをアップロードする"
tags = [
    "Graphql",
]
+++

最近graphqlを触り始めました．
書いてて多分ぶち当たる案件にファイルのアップロードをどうするのかという問題があります．
この件についてこちらの記事が参考になります．

- [GraphQLの規格とプロダクトの隙間をどう埋めるか 〜ファイルアップロード〜](http://hshimoyama.hatenablog.com/entry/2018/01/31/153140)

オチから書くと今回はbase64で実装しました．

記事中やGoogleで検索してると出てくるapollo-upload-server/clientが使いやすそうで良いのですが今回の実行環境がFirebase Functionsということがありそのままでは素直に動いてくれませんでした．
というのもapollo-upload-server/clientは以下で仕様が説明されている通りファイルをmultipartで送信します．

- [jaydenseric/graphql-multipart-request-spec](https://github.com/jaydenseric/graphql-multipart-request-spec)

Firebase functionsではユーザーが使用しているexpressの前にGoogle側のexpressがいるらしくこいつがRequestを先に読み出してしまうようです．
そのため内部で使用されているbusboyがRequestを正常にパースすることが出来ず結果として

```
Missing multipart field ‘operations’
```

とエラーが返ってきてアップロードすることができません．

multipart自体は以下の記事の説明にある通りGoogle Functionsのページには説明がありましたが上手く組み込めなかったので諦めました．

- [Firebaseの小ネタ集](https://qiita.com/shora_kujira16/items/95216245ecf06c4cd16d#cloud-functions-%E3%81%A7-multipartform-data-%E3%81%AE%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%82%92%E6%89%B1%E3%81%86)

これについては
https://github.com/jaydenseric/apollo-upload-server/issues/19
https://github.com/jaydenseric/apollo-upload-server/issues/21
でも取り上げられていますが自分の技術力では解決出来なかったので誰か上手く行った方がいれば教えて欲しいです．

apollo-upload-server/client自体は以下のようなコードで動きます（多分
シンプルで良いですね．

server

```js
...
import { GraphQLUpload } from "apollo-upload-server";
import { apolloUploadExpress } from "apollo-upload-server"

const schema = new GraphQLSchema({
  mutation: new GraphQLObjectType({
    name: "Mutation",
    fields: {
      uploadImage: {
        type: GraphQLBool,
        args: {
          image: {type: GraphQLUpload},
        },
        resolve: async (_, args) => {
          return await uploadImage(args);
        }
      },
    }
  })
});
const app = express();
...
app.use("/graphql", apolloUploadExpress());
```

clinet

```js
import HttpLink from "apollo-link-http";
import fetch from "node-fetch";
import ApolloClient from "apollo-client";
import gql from "graphql-tag";
import { createUploadLink } from 'apollo-upload-client'
import { InMemoryCache } from 'apollo-cache-inmemory'
import { ApolloLink } from 'apollo-link';

const client = new ApolloClient({
  cache: new InMemoryCache(),
  link: createUploadLink({uri: "/path/to/graphql"}),
  fetch: fetch,
});

document.getElementById("mutate").addEventListener("change", ((event) => {
  const file = event.target.files[0];
  client.mutate({
    mutation: gql`
      mutation uploadImageFile($file: Upload!) {
        uploadImage(image: $file){
          created
        }
      }
    `,
    variables: {file},
  })
    .then(data => console.log(data))
    .catch(error => console.error(error));
}));
```
