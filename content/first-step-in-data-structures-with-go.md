+++
title="Goによるデータ構造 はじめの一歩"
date=2021-01-07
category="Go"
+++

このエントリは放送大学の『データ構造とプログラミング』のプログラム部分を Go で書こうと思って始めました。完走しないかもしれないので徐々に追記するスタイルでやっていきます。

## データ構造

### 配列

なにはともあれ配列の定義方法です

```go
var a [10]int
a[0] = 1
...
a[9] = 9
```

逐一書くのは面倒なのでループで入れる

```go
const arraySize = 10
var a [arraySize]int
for i := 0; i < arraySize; i++ {
    a[i] = i
}
```

定義と一緒にデータも入れる

```go
a := [10]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
```

要素数の省略も可能

```go
a := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
```

要素数を省略した場合の型は初期値の数が要素数になる。  
この場合は `[10]int` になる。


要素は構造体でもいい

```go
type Student struct {
    id      int
    grade   string
    average float64
}

var a [3]Student
a[0] = Student{id: 100, grade: "A", average: 510.20}
a[1] = Student{id: 101, grade: "C", average: 350.00}
a[2] = Student{id: 105, grade: "B", average: 450.50}

fmt.Printf("%#v", a)
```


二次元配列を使って九九を出力する

```go
var a [9][9]int
for i := 0; i < 9; i++ {
    for j := 0; j < 9; j++ {
        a[i][j] = (i + 1) * (j + 1)
    }
}

for i := 0; i < 9; i++ {
    for j := 0; j < 9; j++ {
        fmt.Printf("%02d ", a[i][j])
    }
    fmt.Printf("\n")
}
```

配列は最初にサイズを指定するので、ゆるっとコード書くときは次に出てくるスライスが可変長配列のように使えるので取り回しがいいです。


### スライス

定義とデータ挿入。配列のリテラルと同じような書き方が可能

```go
var s []int
for i := 0; i < 10; i++ {
    s = append(s, i)
}
```

Go のスライスは参照型なので `make()` でも作れる

```go
s := make([]int, 0)
for i := 0; i < 10; i++ {
    s = append(s, i)
}
```

定義してからデータを挿入する場合は `append()` を使うところが配列と異なります。  
添字を使ってアクセスするところは配列と同じですが、スライスの実装は配列とは異なるので後述したいと思ってます。

---

## データの探索

配列内にデータがあるかを確認するときは探索を行います

### 線形探索

配列の先頭からデータを探索していく方法。先頭から順に探索を行うので、最悪のケースの計算量はデータの個数に比例する。ちなみに計算量の指標には最悪実行時間、最悪コストというパンチの強い表現があるけど、気分的なものではないので気にしないでください。

#### 線形探索を行ってステップ数と結果を出力する例

線形探索の関数

```go
func linerSearch(arr []int, key int) int {
    max := len(arr)
    for i := 0; i < max; i++ {
        fmt.Printf("step %d\n", i+1)
        if arr[i] == key {
            return i
        }
    }
    return -1
}
```

呼び出し側

```go
// データは順序配列（順序スライス？）
s := []int{
    50, 80, 150, 210, 250, 280, 330, 470, 510, 530, 800, 900, 990,
    1000, 1011, 1209, 1244, 1291, 1299, 1333, 1354, 1359, 1389, 1390,
    1400, 1402, 1490, 1499, 1509, 1573, 1588, 1591, 1605, 1654, 1662,
}
key := 1509
index := linerSearch(s, key)
if index < len(s) && s[index] == key {
    fmt.Printf("found %d at index %d\n", key, index)
} else {
    fmt.Printf("%d not found\n", key)
}
```

出力結果

```
step 1
step 2
step 3
...
step 27
step 28
step 29
found 1509 at index 28
```

### 二分探索（バイナリサーチ）

順序配列（ソート済みの配列）に対して探索を行う。データ数が多い場合に、線形探索に比べて計算量が非常に少なく済む探索方法。  
検索したい値とソート済み配列の中央値とを比較して、一致すれば探索終了、小さければ中央値よりも前の値を対象に再帰的に探索を続け、大きければ中央値よりも後ろの値を対象に再帰的に探索を続ける。


#### 二分探索を行ってステップ数と結果を出力する例

二分探索の関数を定義

```go
func binarySearch(s []int, key int) int {
    low := 0
    high := len(s) - 1
    for i := 0; low <= high; i++ {
        fmt.Printf("step %d ", i+1)
        middle := (low + high) / 2
        fmt.Printf("low: %d, mid: %d, high: %d ", low, middle, high)
        if key == s[middle] {
            fmt.Println("一致")
            return middle
        } else if key < s[middle] {
            fmt.Println("前半分を再探索")
            high = middle - 1
        } else {
            fmt.Println("後半分を再探索")
            low = middle + 1
        }
    }
    return -1
}
```

呼び出し側

```go
// 比較のため線形探索のときと同じデータを使う
s := []int{
    50, 80, 150, 210, 250, 280, 330, 470, 510, 530, 800, 900, 990,
    1000, 1011, 1209, 1244, 1291, 1299, 1333, 1354, 1359, 1389, 1390,
    1400, 1402, 1490, 1499, 1509, 1573, 1588, 1591, 1605, 1654, 1662,
}
key := 1509
index := binarySearch(s, key)
if index < len(s) && s[index] == key {
    fmt.Printf("found %d at index %d\n", key, index)
} else {
    fmt.Printf("%d not found\n", key)
}
```

出力結果

```
step 1 low: 0, mid: 17, high: 34 後半分を再探索
step 2 low: 18, mid: 26, high: 34 後半分を再探索
step 3 low: 27, mid: 30, high: 34 前半分を再探索
step 4 low: 27, mid: 28, high: 29 一致
found 1509 at index 28
```

最初のステップでは探索するデータの 1509 が中央値よりも後ろなので、次のステップでは後ろ半分を対象に再探索し、一致するまで続ける。  
このケースでは線形探索に比べてステップ数が少なく済んでいることがわかる。

こちらは Go の sort パッケージを使用した例

```go
s := []int{
    50, 80, 150, 210, 250, 280, 330, 470, 510, 530, 800, 900, 990,
    1000, 1011, 1209, 1244, 1291, 1299, 1333, 1354, 1359, 1389, 1390,
    1400, 1402, 1490, 1499, 1509, 1573, 1588, 1591, 1605, 1654, 1662,
}
key := 1509
index := sort.Search(len(s), func(i int) bool { return s[i] >= key })
if index < len(s) && s[index] == key {
    fmt.Printf("found %d at index %d\n", key, index)
} else {
    fmt.Printf("%d not found\n", key)
}
```

出力結果

```
found 1509 at index 28
```

以下、参考資料より二分探索について抜粋

> データ数を 100,000,000個（1億個）とした時、線形探索では最悪の場合、1億回の比較、平均では5千万回の比較が必要である。  
> しかし、二分探索なら約log<sub>2</sub>100,000,000回、つまり、約27回の比較で済む。

---

二分探索のほうが線形探索よりも優れているかという話ではなく、データの内容や、データ数に応じた探索方法を選べるようになればいいんじゃないかなと。  
データ数が少なければ線形探索でもパフォーマンスは十分出るし、順序配列（ソート済み配列）でなくても探索できるので、データ構造に応じて探索方法は自明になるみたいな感じでしょうか。  
データ数が 1億個ある場合は線形探索だと最悪の場合の計算量が 1億回になるから、ソートしてから二分探索したほうが速いんじゃないかなという当たりをつけられるようにしとくと、きっと良いです。

---


## データ構造2

### スタック

スタックは後入れ先出しのデータ構造で、FILO で処理を行います。
  
基本操作は次の2つ

- データを頂上に積む `Push`
- データを頂上から取り出す `Pop`

補助的に、頂上のデータを参照する `Peek` や、頂上のデータとそのひとつ下のデータを交換する `Swap` などがある。

#### スライスを使って int 型のデータをスタックする例

int 型のスタックを定義

```go
// 型で定義
type Stack []int

const StackMax = 10

func (s *Stack) Size() int {
    return len(*s)
}

func (s *Stack) Push(data int) error {
    // サイズの上限を定義する時の例
    if s.Size() >= StackMax {
        return errors.New("stack overflow")
    }
    fmt.Printf("push: %d\n", data)
    *s = append(*s, data)
    return nil
}

func (s *Stack) Pop() (int, error) {
    if s.Size() < 1 {
        return 0, errors.New("stack empty")
    }
    // スタックの頂上の要素を取り出す
    top := (*s)[s.Size()-1]
    // 頂上の要素を除いてスタックに入れ直す
    *s = (*s)[:s.Size()-1]
    fmt.Printf("pop: %d\n", top)
    return top, nil
}

func (s *Stack) Peek() (int, error) {
    if s.Size() < 1 {
        return 0, fmt.Errorf("stack empty")
    }
    top := (*s)[s.Size()-1]
    fmt.Printf("peek: %d\n", top)
    return top, nil
}

func NewStack() *Stack {
    s := make(Stack, 0)
    return &s
}
```

呼び出し側

```go
s := NewStack()

s.Push(300)
fmt.Printf("Stack (%d) %v\n", s.Size(), *s)

s.Push(401)
fmt.Printf("Stack (%d) %v\n", s.Size(), *s)

s.Pop()
fmt.Printf("Stack (%d) %v\n", s.Size(), *s)

s.Peek()
fmt.Printf("Stack (%d) %v\n", s.Size(), *s)
```

出力結果

```
push: 300
Stack (1) [300]
push: 401
Stack (2) [300 401]
pop: 401
Stack (1) [300]
peek: 300
Stack (1) [300]
```

2回 Push したあと、1回 Pop したので、1回目に Push した値が Peek で確認できる。

#### スタックを使って文字列を反転させる例

文字列のスタックを定義（簡略版）

```go
type StringStack []string

func (s *StringStack) Push(v string) {
	*s = append(*s, v)
}

func (s *StringStack) Pop() string {
	top := (*s)[len(*s)-1]
	*s = (*s)[:len(*s)-1]
	return top
}
```

呼び出し側（一文字ずつ Push するのも何なのでループで入れ込む）

```go
// 文字列をスライスに入れる
str := "なぜベストを尽くさないのか？"
fmt.Printf("%s\n", str)
s := make([]string, 0)
for _, v := range []rune(str) {
    s = append(s, string(v))
}
// スタックに格納
stack := make(StringStack, 0)
for i := 0; i < len(s); i++ {
    stack.Push(s[i])
}
// スタックからデータを取り出すと文字列が反転する
for i, l := 0, len(stack); i < l; i++ {
    fmt.Printf("%s", stack.Pop())
}
fmt.Printf("\n")
```

出力結果

```
なぜベストを尽くさないのか？
？かのいなさく尽をトスベぜな
```

スタックに入れなくても文字列の反転はできると思うんですが、入れたほうがイメージしやすい気がします。

#### 逆ポーランド記法での計算例

`80 * 20 - 5` の計算を行う

```go
rpn := "80 20 * 5 -"
var stack StringStack
for _, v := range strings.Split(rpn, " ") {
    switch v {
    case "+":
        r, _ := strconv.Atoi(stack.Pop())
        l, _ := strconv.Atoi(stack.Pop())
        result := l + r
        stack.Push(strconv.Itoa(result))
    case "-":
        r, _ := strconv.Atoi(stack.Pop())
        l, _ := strconv.Atoi(stack.Pop())
        result := l - r
        stack.Push(strconv.Itoa(result))
    case "*":
        r, _ := strconv.Atoi(stack.Pop())
        l, _ := strconv.Atoi(stack.Pop())
        result := l * r
        stack.Push(strconv.Itoa(result))
    case "/":
        r, _ := strconv.Atoi(stack.Pop())
        l, _ := strconv.Atoi(stack.Pop())
        result := l / r
        stack.Push(strconv.Itoa(result))
    default:
        stack.Push(v)
    }
}

fmt.Println(stack.Pop())
```

出力結果

```
1595
```

### キュー

キュー（Queue）は待ち行列のデータ構造です。FIFO で処理を行います。

基本操作は次の2つ

- データを末尾に追加する `EnQueue`
- データを先頭から取り出す `DeQueue`

#### スライスを使った int 型のキューの例

定義

```go
type intQueue []int

func (q *intQueue) EnQueue(v int) {
    // Queue はデータを最後尾に追加する
    *q = append(*q, v)
}

func (q *intQueue) DeQueue() int {
    // Queue はデータを最前列から取得する
    v := (*q)[0]
    // 二番目以降のデータを Queue に入れ直す
    *q = (*q)[1:]
    return v
}
```

利用側

```go
queue := make(intQueue, 0)
queue.EnQueue(1)
fmt.Printf("Queue (%d) %v\n", len(queue), queue)
queue.EnQueue(2)
fmt.Printf("Queue (%d) %v\n", len(queue), queue)
queue.DeQueue()
fmt.Printf("Queue (%d) %v\n", len(queue), queue)
queue.EnQueue(3)
fmt.Printf("Queue (%d) %v\n", len(queue), queue)
queue.DeQueue()
fmt.Printf("Queue (%d) %v\n", len(queue), queue)
queue.DeQueue()
fmt.Printf("Queue (%d) %v\n", len(queue), queue)
```

出力結果

```
Queue (1) [1]
Queue (2) [1 2]
Queue (1) [2]
Queue (2) [2 3]
Queue (1) [3]
Queue (0) []
```

#### スライスを使った優先度付きキューの例

int 型の順序配列での優先度付きキューの定義

```go
type priorityQueue []int

func (pq *priorityQueue) EnQueue(v int) {
    if len(*pq) == 0 {
        *pq = append(*pq, v)
    } else {
        // 順序配列にする
        i := sort.Search(len(*pq), func(i int) bool {
            return (*pq)[i] >= v
        })
        // 既存データは index の前後に入れる
        t := make([]int, len((*pq)[0:i]))
        copy(t, (*pq)[0:i])
        // index の位置にデータを入れる
        t = append(t, v)
        t = append(t, (*pq)[i:]...)
        *pq = t
    }
}

func (pq *priorityQueue) DeQueue() int {
    v := (*pq)[0]
    *pq = (*pq)[1:]
    return v
}
```


利用側

```go
pq := make(priorityQueue, 0)
pq.EnQueue(31)
pq.EnQueue(2)
pq.EnQueue(200)
pq.EnQueue(0)
pq.EnQueue(-1)
pq.EnQueue(7)
for i, max := 0, len(pq); i < max; i++ {
    fmt.Printf("%d ", pq.DeQueue())
}
```

出力結果

```
-1 0 2 7 31 200 
```

EnQueue のタイミングで順序配列にしてデータを保持する構造を採用しました。  
順序配列ではなく、配列を使う場合は DeQueue のタイミングで優先度を調べることになります。  
一般に優先度付きキューはデータ構造にヒープを使うらしい（まだよくわかってない）


### スライス補足

len() や cap() をもとにスライスの挙動

```go
// int 型で要素数 1、容量 2のスライスを定義
s := make([]int, 1, 2)
fmt.Printf("p: %p len: %d capa: %d values: %v\n", s, len(s), cap(s), s)
// 要素数が定義の範囲内であれば添字でアクセス可能
s[0] = 1
fmt.Printf("p: %p len: %d capa: %d values: %v\n", s, len(s), cap(s), s)
// 要素数が定義の範囲を超える場合は append でデータを追加する
s = append(s, 2)
fmt.Printf("p: %p len: %d capa: %d values: %v\n", s, len(s), cap(s), s)
// 定義時の容量を超える際に新たなメモリ領域を確保するのでポインタの値が変わっている事がわかる
s = append(s, 3)
fmt.Printf("p: %p len: %d capa: %d values: %v\n", s, len(s), cap(s), s)
```

```
p: 0xc000016070 len: 1 capa: 2 values: [0]
p: 0xc000016070 len: 1 capa: 2 values: [1]
p: 0xc000016070 len: 2 capa: 2 values: [1 2]
p: 0xc0000140e0 len: 3 capa: 4 values: [1 2 3]
```

---

1. 『データ構造とプログラミング』鈴木一史 2018年3月

