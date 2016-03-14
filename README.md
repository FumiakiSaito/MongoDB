# MongoDB

##コマンド

###バージョン確認

`# mongo -version`

###インタラクティブシェル起動
`mongo`

###dbを指定した状態でインタラクティブシェル起動
`mongo mydb`


###DB一覧確認
`show dbs`

###使用DBを変更(存在しない場合、コレクションを作成するとDBも作成される)
`use mydb`

###コレクション(エーブルのような概念)を作成する

`db.createCollection("users");`

###コレクション一覧を確認する

`show collections;`

###ドキュメント(レコードのような概念)を挿入する
`db.users.insert( {name: "saito", score: 30});`

###ドキュメント数を取得する

```
> db.users.count();
1
```

###ドキュメントを全件検索する
```
> db.users.find();
{ "_id" : ObjectId("56e613f473cd3c01e31f406d"), "name" : "saito", "score" : 30 }
>
```

###ドキュメントを全件削除する

```
> db.users.remove({}); ←空のオブジェクトを与える
WriteResult({ "nRemoved" : 1 })
>
```

###indexを確認する
```
> db.users.getIndexes();
[
        {
                "v" : 1,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_",
                "ns" : "mydb.users"
        }
]
```



###ドキュメントを条件付きで検索する
```
> db.users.find();
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "score" : 30 }
{ "_id" : ObjectId("56e6152373cd3c01e31f406f"), "name" : "yamada", "score" : 40 }
{ "_id" : ObjectId("56e6152b73cd3c01e31f4070"), "name" : "suzuki", "score" : 50 }

■nameがsaitoのドキュメントを検索
> db.users.find({name: "saito"});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "score" : 30 }

■scoreが40点以上のドキュメントを検索
> db.users.find({score: {$gte: 40}});
{ "_id" : ObjectId("56e6152373cd3c01e31f406f"), "name" : "yamada", "score" : 40 }
{ "_id" : ObjectId("56e6152b73cd3c01e31f4070"), "name" : "suzuki", "score" : 50 }

※条件種類
・gte(grater than equal)
・gt(grater than)
・lte(less than equal)
・lt(less than)
・eq(equal)
・ne(not equal)

■正規表現で検索
> > db.users.find({name: /sai/});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "score" : 30 }

■distinctで重複排除
> db.users.distinct("name");
[ "saito", "yamada", "suzuki" ]

■AND条件で検索(nameにsが含まれていてscoreが50点以上のドキュメント)
> db.users.find({name: /s/, score:{$gte: 50}});
{ "_id" : ObjectId("56e6152b73cd3c01e31f4070"), "name" : "suzuki", "score" : 50 }

■OR条件で検索(nameにyamadaが含まれている、もしくはscoreが40点以上のドキュメント)
> db.users.find({$or: [{name: /yamada/}, {score: {$gte: 40}}]});
{ "_id" : ObjectId("56e6152373cd3c01e31f406f"), "name" : "yamada", "score" : 40 }
{ "_id" : ObjectId("56e6152b73cd3c01e31f4070"), "name" : "suzuki", "score" : 50 }
>

■同フィールド内でOR条件で検索(scoreが30もしくは40のドキュメント)
> db.users.find({score: {$in: [30, 40]}});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "score" : 30 }
{ "_id" : ObjectId("56e6152373cd3c01e31f406f"), "name" : "yamada", "score" : 40 }
>

■フィールドの存在有無で検索
※satoさんのみageフィールドがある
> db.users.find();
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "score" : 30 }
{ "_id" : ObjectId("56e6152373cd3c01e31f406f"), "name" : "yamada", "score" : 40 }
{ "_id" : ObjectId("56e6152b73cd3c01e31f4070"), "name" : "suzuki", "score" : 50 }
{ "_id" : ObjectId("56e6198473cd3c01e31f4071"), "name" : "sato", "score" : 40, "age" : 23 }

・ageフィールドがあるドキュメントを検索
> db.users.find({age: {$exists: true}});
{ "_id" : ObjectId("56e6198473cd3c01e31f4071"), "name" : "sato", "score" : 40, "age" : 23 }

・ageフィールドがないドキュメントを検索
> db.users.find({age: {$exists: false}});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "score" : 30 }
{ "_id" : ObjectId("56e6152373cd3c01e31f406f"), "name" : "yamada", "score" : 40 }
{ "_id" : ObjectId("56e6152b73cd3c01e31f4070"), "name" : "suzuki", "score" : 50 }
>

■表示するフィールドを制限する(第1引数は件数条件なので全件の場合は空オブジェクト、第2引数にフィールド名true(もしくは1)を設定する)
> db.users.find({}, {name: true});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito" }
{ "_id" : ObjectId("56e6152373cd3c01e31f406f"), "name" : "yamada" }
{ "_id" : ObjectId("56e6152b73cd3c01e31f4070"), "name" : "suzuki" }
{ "_id" : ObjectId("56e6198473cd3c01e31f4071"), "name" : "sato" }
>
```


###抽出条件を加工する

```
■昇順表示
> db.users.find({}, {_id: 0}).sort({score: 1});
{ "name" : "saito", "score" : 30 }
{ "name" : "yamada", "score" : 40 }
{ "name" : "sato", "score" : 40, "age" : 23 }
{ "name" : "suzuki", "score" : 50 }

■降順表示
> db.users.find({}, {_id: 0}).sort({score: -1});
{ "name" : "suzuki", "score" : 50 }
{ "name" : "yamada", "score" : 40 }
{ "name" : "sato", "score" : 40, "age" : 23 }
{ "name" : "saito", "score" : 30 }
>

■表示件数を指定
> db.users.find({}, {_id: 0}).sort({score: 1}).limit(3);
{ "name" : "saito", "score" : 30 }
{ "name" : "yamada", "score" : 40 }
{ "name" : "sato", "score" : 40, "age" : 23 }

■前方n件を非表示にする
> db.users.find({}, {_id: 0}).sort({score: 1}).skip(3);
{ "name" : "suzuki", "score" : 50 }

```

###ドキュメントを更新する

```
■nameがsaitoのscoreを100に更新する
※第一パラメータのオブジェクトは条件、第二パラメータのオブジェクトは更新する値
> db.users.update({name: "saito"}, {$set: {score: 100}});

■nameがsaitoのscoreを100にかつ、ageを30に更新する
> db.users.update({name: "saito"}, {$set: {score: 100, age: 30}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.find({name: /saito/});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "score" : 100, "age" : 30 }

■nameがsaitoのドキュメントをまるっと入れ替える
> db.users.update({name: "saito"}, {name: "saito", score: 100});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.find({name: /saito/});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "score" : 100 }
>

※updateは最初の1件のみ更新される
■条件に合致するドキュメントを全て更新するには第三パラメータにmultiを設定する
> > db.users.update({name: /s/}, {$set: {score: 100}}, {multi: true});
WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 2 })
> db.users.find();
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "score" : 100 }
{ "_id" : ObjectId("56e6152373cd3c01e31f406f"), "name" : "yamada", "score" : 40 }
{ "_id" : ObjectId("56e6152b73cd3c01e31f4070"), "name" : "suzuki", "score" : 100 }
{ "_id" : ObjectId("56e6198473cd3c01e31f4071"), "name" : "sato", "score" : 100, "age" : 23 }

■現在の値から相対的に更新する(scoreを+5する)
> db.users.update({name: "saito"}, {$inc: {score: 5}});
> db.users.find({name: "saito"});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "score" : 105 }

■フィールド名を更新する(score→pointにする)
> db.users.update({name: "saito"}, {$rename: {score: "point"}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.find({name: "saito"});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "point" : 105 }
>

■フィールドを追加する
> db.users.update({name: "saito"}, {$set: {score: 200}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.find({name: "saito"});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "point" : 105, "score" : 200 }
>

■フィールドを削除する
> db.users.update({name: "saito"}, {$unset: {score: ""}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.users.find({name: "saito"});
{ "_id" : ObjectId("56e6151b73cd3c01e31f406e"), "name" : "saito", "point" : 105 }

■存在したら更新、存在しなかったらinsertする
> db.users.update({name: "kato"}, {name: "kato", score: 48}, {upsert: true});
> db.users.find({name: "kato"});
{ "_id" : ObjectId("56e634579b2c4bb43b7ec62a"), "name" : "kato", "score" : 48 }
>

```

###ドキュメントを削除する

```
■nameがkatoのドキュメントを削除する
> > db.users.remove({name: "kato"});
```

###索引を使う
```

■scoreの降順？用indexを作成する
> db.users.createIndex({score: -1});
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
・index確認
> db.users.getIndexes();
[
        {
                "v" : 1,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_",
                "ns" : "mydb.users"
        },
        {
                "v" : 1,
                "key" : {
                        "score" : -1
                },
                "name" : "score_-1",
                "ns" : "mydb.users"
        }
]

■indexを削除する(index名を指定する)
> db.users.dropIndex("score_-1");
{ "nIndexesWas" : 2, "ok" : 1 }

・index確認
> db.users.getIndexes();
[
        {
                "v" : 1,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_",
                "ns" : "mydb.users"
        }
]


■ユニークindexを作成する
> db.users.createIndex({name: 1}, {unique: true});
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
> db.users.getIndexes();
[
        {
                "v" : 1,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_",
                "ns" : "mydb.users"
        },
        {
                "v" : 1,
                "unique" : true,
                "key" : {
                        "name" : 1
                },
                "name" : "name_1",
                "ns" : "mydb.users"
        }
]
・同じsaitoをインサートしようとするとエラーとなる
> db.users.insert({name: "saito"});
WriteResult({
        "nInserted" : 0,
        "writeError" : {
                "code" : 11000,
                "errmsg" : "E11000 duplicate key error collection: mydb.users index: name_1 dup key: { : \"saito\" }"
        }
})


```

###バックアップと復元
```
■ダンプを取得する
# mongodump -d mydb
2016-03-14T13:01:40.046+0900    writing mydb.users to
2016-03-14T13:01:40.052+0900    done dumping mydb.users (4 documents)
[root@fffumi tmp]# ll
合計 12
drwxr-xr-x 3 root     root     4096  3月 14 13:01 2016 dump

■リストアする(DBが存在する場合は--dropで上書きする)
# mongorestore --drop
2016-03-14T13:04:26.583+0900    using default 'dump' directory
2016-03-14T13:04:26.583+0900    building a list of dbs and collections to restore from dump dir
2016-03-14T13:04:26.595+0900    reading metadata for mydb.users from dump/mydb/users.metadata.json
2016-03-14T13:04:26.610+0900    restoring mydb.users from dump/mydb/users.bson
2016-03-14T13:04:26.613+0900    restoring indexes for collection mydb.users from metadata
2016-03-14T13:04:26.620+0900    finished restoring mydb.users (4 documents)
2016-03-14T13:04:26.620+0900    done

```
