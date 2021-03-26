+++
title="Goによるデータ構造 連結リスト"
date=2021-03-17
category="Go"
+++

年始にデータ構造の記事を書いてから、はや二ヶ月が過ぎたので続きを書く。

テキストによると連結リストを学ぶにあたっての目標とポイントは以下の通りです。

* 連結リストに対するデータの探索、挿入、削除について基本的な操作を学ぶ。
* 連結リストと配列との操作効率の違いについても学習する

配列の特徴をまとめるとこんな感じ

* 配列は添字を使ってデータを挿入・探索する。
* 探索においては添字でデータへアクセスできるので、最初のデータと途中のデータと最後のデータとで同じ時間でアクセスできる。
* 配列のサイズは生成時に定義するので、後からサイズを変更するのは難しい。可変長配列はサイズ変更を意識しないでコーディングできるが、コードの書き方によっては参照先が変わることに注意が必要だったりする。
* 配列の途中にデータを挿入・削除した場合にできる空きを埋める処理ではデータ数に応じて相応の負荷がかかる。

対して連結リストの特徴は次のようになる

* 利用したいデータの個数だけメモリを都度確保したり、開放したりできるのでメモリを効率的に使用できる。
* 探索は最初のノードから行うのでデータ数に応じた計算量が必要になる。
* データ間の相対的な位置の変更が容易なためデータの挿入や削除時に配列のようなデータ移動は発生しない。

連結リストは構造体の中にデータと次のデータへの参照を持つデータ構造で、Goで書くと連結リストとノードは次のようになる。

```Go
// List 連結リスト
type List struct {
	head *Node
}

// Node 連結リストのノード
type Node struct {
	value int
	next  *Node
}
```

## 連結リストへのデータ挿入

説明はコードを見たほうがわかりやすいと思うので、愚直にデータを入れていく例を書きます。

```Go
func main() {
	// 先頭の Node を作る
	head := new(Node)
	head.value = 100
	// 連結する Node を生成して next に入れる
	head.next = new(Node)
	head.next.value = 200
	// 更に繰り返す
	head.next.next = new(Node)
	head.next.next.value = 300
	head.next.next.next = new(Node)
	head.next.next.next.value = 400
	linkedListPrint(*head)
}


func linkedListPrint(n Node) {
	for {
		fmt.Printf("%d ", n.value)
		if n.next == nil {
			break
		}
		n = *n.next
	}
	fmt.Printf("\n")
}
```

出力結果

```
100 200 300 400 
```

`.next` に生成した `Node` を代入して `Node` を連結していくのが連結リストの作り方です。

連結リストに関連した操作には挿入と削除があるので関数として実装していく。（探索は一旦置いとく）  
連結リストへのノード挿入ではリストのどこへノードを挿入するかによって処理が異なるので、次の順に実装する。

1. 先頭への挿入
2. 末尾への挿入
3. 中間部への挿入（インデックスで指定）

### 先頭への挿入例

```Go
// linkedListInsertHead 連結リストの先頭へのノード挿入
func linkedListInsertHead(head *Node, v int) *Node {
	// 挿入ノードの生成
	n := new(Node)
	n.value = v

	if head == nil {
		n.next = nil
	} else {
		// 既存の先頭ノードを後ろにつなげる
		n.next = head
	}
	head = n
	return head
}

func main() {
	// 先頭の Node を作る
	head := new(Node)
	head.value = 100
	linkedListPrint(*head)
	head = linkedListInsertHead(head, 200)
	linkedListPrint(*head)
}
```

出力結果

```
100 
200 100 
```

### 末尾への挿入例

```Go
// linkedListInsertTail 連結リストの末尾へのノード挿入
func linkedListInsertTail(head *Node, v int) *Node {
	// 挿入ノードの生成
	node := new(Node)
	node.value = v

	if head == nil {
		// 先頭が空の場合
		head = node
		return head
	}

	if head.next == nil {
		// 要素が1つの場合
		head.next = node
		return head
	}

	// 要素が2つ以上ある場合
	n := head.next
	for n.next != nil {
		n = n.next
	}
	n.next = node
	return head
}

func main() {
	// 空の連結リストで試す
	l := new(List)
	head := l.head
	head = linkedListInsertTail(head, 100)
	linkedListPrint(*head)
	head = linkedListInsertTail(head, 200)
	linkedListPrint(*head)
	head = linkedListInsertTail(head, 300)
	linkedListPrint(*head)
}
```

出力結果

```
100 
100 200 
100 200 300 
```

### 中間部への挿入例（インデックスで指定）

```Go
// linkedListInsertNth 連結リストの中間へノードを挿入（インデックスで指定）
func linkedListInsertNth(head *Node, v, nth int) *Node {
	if nth == 0 || head == nil {
		return linkedListInsertHead(head, v)
	}

	// n番目のノードを取得
	node := head
	for i := 1; i <= nth; i++ {
		if i == nth {
			break
		}
		node = node.next
	}

	// 間に挿入する
	n := new(Node)
	n.value = v
	n.next = node.next
	node.next = n

	return head
}

func main() {
	// 空の連結リストで試す
	l := new(List)
	head := l.head
	// 中間部分への挿入（インデックスで指定する）
	head = linkedListInsertNth(head, 456, 0)
	linkedListPrint(*head)
	head = linkedListInsertNth(head, 123, 0)
	linkedListPrint(*head)
	head = linkedListInsertNth(head, 789, 2)
	linkedListPrint(*head)
	head = linkedListInsertNth(head, 123, 1)
	linkedListPrint(*head)
}
```

出力結果

```
456 
123 456 
123 456 789 
123 123 456 789 
```

## 連結リストからのデータ削除

Node が 3 つの連結リストを作って中間ノードを削除する例です

```Go
func main() {
	// 先頭 Node の生成
	head := new(Node)
	head.value = 100
	// 中間 Node の生成
	head.next = new(Node)
	head.next.value = 200
	// 末尾 Node の生成
	head.next.next = new(Node)
	head.next.next.value = 300
	linkedListPrint(*head)
	// 連結リストの中間 Node を削除する
	head.next.value = head.next.next.value
	head.next = head.next.next
	linkedListPrint(*head)
}
```

出力結果

```
100 200 300 
100 300 
```

やることは値とポインタのつげ替えです。
中間 Node の value に末尾 Node の value を代入して、先頭の Node のポインタに末尾 Node のポインタを代入して中間 Node を削除しています。

続けて次の機能を関数で実装していきます

1. 先頭から削除
2. 末尾から削除
3. 中間部から削除（インデックスで指定）
4. 削除ノードを指定

### 先頭からの削除例

```Go
// linkedListDeleteHead 先頭位置からのノードの削除
func linkedListDeleteHead(head *Node) *Node {
	if head == nil {
		// 先頭ノードがない場合
		return head
	}

	if head.next == nil {
		// 先頭ノードだけある場合
		l := new(List)
		return l.head
	}

	// 複数ノードがある場合
	// head ポインタが連結リストの二番目のノードを指すように変更
	n := head.next
	head.value = n.value
	head.next = n.next
	// 先頭ノードの使われていたメモリを開放する（これは GC に任せる）
	return head
}

func main() {
	l := new(List)
	head := l.head
	head = linkedListInsertTail(head, 123)
	head = linkedListInsertTail(head, 456)
	head = linkedListInsertTail(head, 789)
	linkedListPrint(*head)
	// 先頭位置からの削除
	head = linkedListDeleteHead(head)
	linkedListPrint(*head)
}
```

出力結果

```
123 456 789 
456 789 
```

### 末尾からの削除例

```Go
// linkedListDeleteTail 末尾位置のノードの削除
func linkedListDeleteTail(head *Node) *Node {
	/*
	連結リストの先頭からノードを横断していく
	末尾ノードの一つ手前のノードを探し
	このノードが末尾となるようにポインタには nil を設定する
	 */
	if head == nil {
		return head
	}

	if head.next == nil {
		// 先頭ノードだけある場合
		l := new(List)
		return l.head
	}

	// ノードが2つの場合
	if head.next.next == nil {
		head.next = nil
		return head
	}

	// 要素が3つ以上ある場合
	n := head.next
	// ノードを横断
	for n.next != nil {
		if n.next.next == nil {
			// 末尾ノードの一つ手前のノードで横断を止める
			break
		}
		n = n.next
	}
	// 末尾ノードの一つ手前のノードを末尾ノードにする
	n.next = nil

	return head
}

func main() {
	l := new(List)
	head := l.head
	head = linkedListInsertTail(head, 123)
	head = linkedListInsertTail(head, 456)
	head = linkedListInsertTail(head, 789)
	linkedListPrint(*head)
	// 末尾位置からの削除
	head = linkedListDeleteTail(head)
	linkedListPrint(*head)
	head = linkedListDeleteTail(head)
	linkedListPrint(*head)
}
```

出力結果

```
123 456 789 
123 456 
123 
```

### 中間部からの削除例（インデックスで指定）

```Go

// linkedListDeleteNth 連結リストの中間のノードを削除（インデックスで指定）
func linkedListDeleteNth(head *Node, nth int) *Node {
	if nth == 0 || head == nil {
		return linkedListDeleteHead(head)
	}

	// n番目のノードを取得
	node := head
	for i := 1; i <= nth; i++ {
		if i == nth {
			// 削除対象ノードの一つ手前のノードで止める
			break
		}
		node = node.next
	}

	if node.next == nil {
		// 指定インデックスが要素数を超えている場合は末尾を削除する
		return linkedListDeleteTail(head)
	}

	// 削除対象ノードの一つ手前のノード、削除ノードの一つあとのノードを連結するようにポインタ情報を書き換える
	node.next = node.next.next

	return head
}

func main() {
	l := new(List)
	head := l.head
	head = linkedListInsertTail(head, 123)
	head = linkedListInsertTail(head, 456)
	head = linkedListInsertTail(head, 789)
	head = linkedListInsertTail(head, 999)
	head = linkedListInsertTail(head, 789)
	head = linkedListInsertTail(head, 456)
	head = linkedListInsertTail(head, 123)
	linkedListPrint(*head)
	// 中間位置からの削除
	head = linkedListDeleteNth(head, 3)
	linkedListPrint(*head)
}
```

出力結果

```
123 456 789 999 789 456 123 
123 456 789 789 456 123 
```

### 削除ノードを指定する例

削除するノードを直接指定する方法だと検索不要なので計算量は配列と同じ(?)になる。この実装では指定するノードが末尾ノード以外という条件付きだけど。

```Go
func linkedListDelete(n *Node) {
	n.value, n.next = n.next.value, n.next.next
}

func main() {
	// 先頭 Node の生成
	head := new(Node)
	head.value = 100
	// 中間 Node の生成
	head.next = new(Node)
	head.next.value = 200
	// 末尾 Node の生成
	head.next.next = new(Node)
	head.next.next.value = 300
	linkedListPrint(*head)
	// 連結リストの先頭 Node を直接指定して削除
	linkedListDelete(head)
	linkedListPrint(*head)
}
```

出力内容

```
100 200 300 
200 300 
```

長くなったので双方向連結リストは別のエントリで書こうと思います。

### 補足

このエントリは放送大学の『データ構造とプログラミング』のプログラム部分を Go で書こうと思って始めたものです。



---

1. 『データ構造とプログラミング』鈴木一史 2018年3月

