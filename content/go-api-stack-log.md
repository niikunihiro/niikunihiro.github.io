+++
title="Go の errors パッケージを使ってエラースタックを出力する"
date=2020-10-10
category="Go"
+++

どうも Nii です。2020年もあいかわらず Web 屋をやってるので Go の errors パッケージを使ってエラーログをスタックしていく方法を記します。

Go でエラーのスタックトレースを取るときは github.com/pkg/errors か xerrors を使って、出力時のフォーマットで %+v を使うのが定石なんだなぁと思ってたんですが、[『Go祭2020』](https://techbookfest.org/product/5410766247165952)の Go で作ろう! Web アプリのカスタムエラーの章で Go 1.13 からエラーのラッピングがサポートされてる事を知ったので早速やってみようと思います。それなりに実践ぽくしたいので、[github.com/go-chi/chi](https://github.com/go-chi/chi) を使って API を実装してみます。

リクエストでエラーが発生したときに、内部処理のエラーはスタックしたログを残し、レスポンスにはエラーコードを返す。ということをやります。


まずは、アプリエラーを定義します。


```Go
type AppError struct {
	Err         error
	ErrorID     ErrorID
	Description string
}

func (err *AppError) Error() string {
	return err.Description
}

func (err *AppError) Unwrap() error {
	return err.Err
}
```

Unwrap を実装しておくとスタックしたエラーを全部ポップできるようになります。

```Go
type ErrorID uint

func (eid ErrorID) Wrap(err error, description string) error {
	return &AppError{
		Err:         err,
		ErrorID:     eid,
		Description: description,
	}
}
```

ここうまく説明できないのですが、AppError をラップしたエラーを持つ。つまりスタックするところ、くらいの理解です。

次は、クライアントに返すエラーコードをエラーIDと結びつけて定義します。

```Go
const (
	Unknown ErrorID = iota
	Maintenance
	OutOfService
	ServerError
)

var ErrorCode = map[ErrorID]string{
	Unknown:      "E0000",
	Maintenance:  "E0001",
	OutOfService: "E0002",
	ServerError:  "E0003",
}
```

エラーコードでもいいし、エラーテキストを返すのもありだと思います。

次は、スタックしたエラーを出力するところです。

```Go
func init() {
	render.Respond = func(w http.ResponseWriter, r *http.Request, v interface{}) {
		if appErr, ok := v.(*AppError); ok {
			fmt.Printf("Logging err: %s\n", appErr.Error())
			err := appErr.Err
			for err != nil {
				fmt.Printf("Logging err: %s\n", err.Error())
				err = errors.Unwrap(err)
			}
			render.DefaultResponder(w, r, render.M{"code": ErrorCode[appErr.ErrorID]})
			return
		}
		render.DefaultResponder(w, r, v)
	}
}
```

レスポンスを返すときに、エラーがあればスタックしてるエラーを出力し、レスポンスにエラーコードを指定します。ログ出力が2か所になってるのはご愛嬌ということで。

あとは、API の処理内でエラーを起こして確認です。

```Go
func main() {
	r := chi.NewRouter()
	r.Get("/", func(w http.ResponseWriter, r *http.Request) {
		err := errors.New("xxx connection error")
		err = Maintenance.Wrap(err, "under maintenance")
		err = OutOfService.Wrap(err, "not available now")
		err = ServerError.Wrap(err, "server error")
		render.Respond(w, r, err)
	})
	http.ListenAndServe(":3000", r)
}
```

スタックされたログが出力されてることが確認できます。

```
Logging err: server error
Logging err: not available now
Logging err: under maintenance
Logging err: xxx connection error
```

ということで標準パッケージの errors でエラーをスタックして出力してみました。

---

1. 『Go 祭 2020』Women Who Go Tokyo 2020年9月
