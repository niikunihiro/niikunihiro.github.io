+++
title="Go のバイナリには静的ファイルは含まれないことを知った41歳の夏"
date=2020-08-04
category="Go"
+++

Go の文法をなんとなく理解して、API サーバを開発したときの話で、ワシャワシャとコードを書いて、さぁサーバにデプロイだぜってときに Go の素人が陥った罠というか、無知ゆえのドハマリの記録です。
あと41歳ではないです。

で、本題なんですが、超シンプルにこんなディレクトリ構成で開発してたとします。

```bash
.
├── Makefile
├── README.md
├── etc
│   └── conf.yaml
├── go.mod
├── go.sum
└── main.go
```

基本的には main.go があって、設定ファイルの conf.yaml があるだけです。

---

main.go もシンプルにこんな感じとします。

```go
func main() {
	conf := flag.String("config", "etc/conf.yaml", "configuration file path")
	port := flag.Int("port", 3333, "server port")
	flag.Parse()

	file, dir := filepath.Base(*conf), filepath.Dir(*conf)

	fp, err := os.Open(fmt.Sprintf("%s/%s", dir, file))
	if err != nil {
		panic(err)
	}
	defer fp.Close()

	b, _ := ioutil.ReadAll(fp)

	r := chi.NewRouter()
	r.Get("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write(b)
	})
	err = http.ListenAndServe(fmt.Sprintf(":%d", *port), r)
	if err != nil {
		panic(err)
	}
}
```

ルーターに [chi](https://github.com/go-chi/chi) を使用していて、[localhost:3333](http://localhost:3333) にアクセスしたら、設定ファイルの中身を出力するだけのサンプルです。タイトルに有った静的ファイルはこの設定ファイルを指しています。

---

Makefile も Go らしく作っておきます

```Makefile
GO_CMD=go
GO_BUILD=$(GO_CMD) build
GO_CLEAN=$(GO_CMD) clean
GO_TEST=$(GO_CMD) test
GO_GET=$(GO_CMD) get
BIN_NAME=goskeleton

all: test build
build:
	$(GO_BUILD) -i -o $(BIN_NAME) -v .
test:
	$(GO_TEST) -v ./...
run:
	$(GO_BUILD) -o $(BIN_NAME) -v .
	./$(BIN_NAME)

```

これで make run あるいは go build して、開発中のディレクトリでバイナリを実行しても、何も問題は起きないです。なので無知だったあの頃の私は何も気づかずに幸せに暮らしていました。

---

そして「ときは来た！それだけだ…」状態で、Dockerfile を作りました。  
Go で Docker コンテナを作る時はマルチステージ・ビルドというやり方が推奨されてるっぽいので、なんとなくやってみました。  
マルチステージ・ビルドをざっくり説明すると、ビルドするコンテナと運用するコンテナを分けることで、コンテナサイズを小さくできるというメリットが得られるというものです。  
ビルドするコンテナから運用するコンテナにバイナリを COPY で送るんですが、設定ファイルはバイナリに含まれないことを知らない私は次のログを見てパニックに陥ってました。

```sh
Attaching to gochiapi
gochiapi    | panic: open etc/conf.yaml: no such file or directory
gochiapi    | 
gochiapi    | goroutine 1 [running]:
gochiapi    | main.main()
gochiapi    |   /go/src/github.com/niikunihiro/goskeleton/main.go:23 +0x4d1
gochiapi exited with code 2
```

沈思黙考し、Dockerfile 内で 
```sh
RUN ls いろんなディレクトリ
```
というローラー作戦を展開して、バイナリからコンテナのファイルシステムに静的ファイルを読み込みにいって指定されたパスにファイルが無いぜ panic だぜと、Go のバイナリが教えてくれてることに気づけました。こんなことに小一時間費やしてしまったんですが、忘れられない夏のメモリーとなりました。

しかし、これって Go やってる人には当たり前なんですよね。当たり前過ぎて情報が見つけられず辛かった。

最後に、マルチステージ・ビルドの Dockerfile も載せておきます。

```dockerfile
FROM golang:latest as builder

# for alpine
ENV CGO_ENABLED=0
ENV GOOS=linux
ENV GOARCH=amd64

WORKDIR /go/src/github.com/niikunihiro/goskeleton

ENV GO114MODULE="on"
COPY go.mod go.sum ./
RUN go mod download -x

COPY . ./
RUN make setup
RUN make build

# runtime image
FROM alpine:latest
RUN apk add --no-cache ca-certificates

WORKDIR /app

COPY --from=builder /go/src/github.com/niikunihiro/goskeleton/goskeleton .
COPY --from=builder /go/src/github.com/niikunihiro/goskeleton/etc/go/conf.yaml ./etc/go/

EXPOSE 3333
CMD ["/app/goskeleton"]
```

おわり
