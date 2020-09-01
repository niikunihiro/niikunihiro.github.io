+++
title="UUID version4 が衝突しないのを実験する"
date=2020-09-02
category="UUID"
+++

UUID の衝突は気にしなくていいという記事を読んで、なるほどなーと思いつつも、数式分からぬ実際にコードで確かめてみたい。と思ったので実験です。

方法はプログラムで UUID ver4 を生成して UNIQUE 制約を設定したデータベースカラムに挿入し、UNIQUE 制約エラーが起きないかを確かめるというものです。  
あとはバイナリを生成して、実行するマシンのコア数だけ実行するという方法にしました。  
特に理由はないですが goroutine は敢えて使ってません。

DDL(MySQL)

```sql
create table uuids
(
	id int auto_increment
		primary key,
	uuid varchar(36) not null,
	constraint uuids_uuid_uindex
		unique (uuid)
);

```

Source Code(Go)

試す場合は、userName、password、databaseName をご自身の環境に合わせてもらえれば動くかと。

```go
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"github.com/google/uuid"
	"log"
)

var query = "INSERT INTO uuids(uuid) VALUES(?)"

func main() {
	dataSourceName := fmt.Sprintf("%s:%s@tcp(%s)/%s?charset=utf8&parseTime=true",
		"userName", "password", "127.0.0.1:3306", "databaseName",
	)
	db, err := sql.Open("mysql", dataSourceName)
	if err != nil {
		panic(err)
	}
	defer db.Close()

	tx, err := db.Begin()
	if err != nil {
		panic(err)
	}
	stmt, err := tx.Prepare(query)
	if err != nil {
		panic(err)
	}
	defer stmt.Close()

	for i := 0; i < 100000; i++ {
		if i%10000 == 0 && i != 0 {
			log.Printf("now index: %d\n", i)
		}
		id, err := uuid.NewRandom()
		if err != nil {
			log.Fatalf("error in uuid generate %v\n", err.Error())
		}

		if _, err = stmt.Exec(id.String()); err != nil {
			tx.Rollback()
			log.Fatalf("rollback %v\n", err)
		}
	}

	tx.Commit()

	fmt.Println("finished")
}
```


で、結果ですが、今のところ衝突は起こせていないです。2コアはちょっと頼りないか。

今回はサーバで生成する前提で書いてるんですけど、用途によっては MACアドレスを元に生成する ver1 使って、サーバで生成せずにクライアント端末で生成した UUID を集めたほうが限りなく衝突しないんじゃないかなと思ったり。
