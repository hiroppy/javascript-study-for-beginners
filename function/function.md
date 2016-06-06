# function

## 目次
- [関数](#function-1)
  - [宣言](#function-1-1)
  - [即時関数](#function-1-2)
  - [変数のスコープ](#function-1-3)
  - [レキシカルスコープ](#function-1-4)
  - [第一級オブジェクト](#function-1-5)
- [高階関数](#function-2)
- [スコープチェーン](#function-3)
  - [スコープチェーンとは](#function-3-1)
  - [スコープチェーンの仕組み](#function-3-2)
- [closure](#function-4)
  - [closureとは](#function-4-1)
  - [メリット](#function-4-2)
  - [使用例](#function-4-3)

---
## {#function-1} 関数
### {#function-1-1} 宣言
#### 関数リテラルとして宣言する
```javascript
const hello = function() {};
```
#### 関数として宣言する
```javascript
function hello() {};
```
2つの違いとしてホイストができるかどうかがあります。  
```javascript
smile();
cry();

const smile = function() {};

function cry() {};
```
を実行した場合、`smile()`の実行はできないが、`cry()`の実行はできる。  
関数リテラルの場合はコード解析時ではなく実行時に関数の登録がされるためです。  

### {#function-1-2} 即時関数
```javascript
(function() {})();
void function() {}();
+function() {}();
!function() {}();
```
このように書くと無名関数を即実行することが可能です。  
基本的に一番最初の `(function() {})();` を使えばいいと思います。  
即時関数はJavaScriptを使うにあたってとても重要な構文になります。  
JavaScriptにはブロックスコープは存在しないので、関数でスコープを制御する必要があります。  
即時関数を利用する大きなメリットとしてグローバルスコープ汚染を防ぐことです。  

```javascript
(function(face) {
  console.log(face);
})(':)');

const hello = (function(face) {
  return face;
})(':)');

console.log(hello); // ":)"
```

### {#function-1-3} 変数のスコープ
JavaScriptにおいて変数のスコープは関数ごとに定義されます。  
```javascript
var hello = ':)';

function say() {
  console.log(hello);
}

function say2() {
  var yay = 'XD';

  console.log(yay);
}

say(); // ":)"
say2(); // "XD"
console.log(yay); // VM1254:1 Uncaught ReferenceError: yay is not define
```
のように`yay`は定義されていないのでエラーとなります。  

JavaScriptではブロックスコープは存在しないのでグローバルスコープとローカルスコープ(関数定義内で宣言された場合)のどちらかになります。  
`let`, `const`を使うと回避できますが、ここではその話はしません。  

```javascript
function say() {
  if (true) {
    var hello = ':)';
  }
  console.log(hello);
}

say(); // ":)"
```
このようにブロックスコープは存在しません。

### {#function-1-4} レキシカルスコープ
JavaScriptはレキシカルスコープです。  
レキシカルスコープとは、変数のスコープが評価時ではなく定義した時に決定するというスコープです。  

```javascript
var face = ':)';

function hello() {
  console.log(face);
}

function hello2() {
  var face = 'XD';

  hello(); // hello2のスコープで実行しているが、helloのfaceが参照するスコープはグローバルスコープである
}

hello2(); // ":)"
```

このように関数の定義時のスコープで実行されます。  
定義時の`hello`の`console.log(face)`はグローバルにある`:)`を指すので`hello2`内で`hello`を呼んでもこのようにグローバルの`:)`を出力します。  
つまり、実行タイミング関係なしで定義時の状態で実行されることがわかります。  

## {#function-1-5}  第一級オブジェクト
第一級オブジェクトとは
> 生成、代入、演算、（引数・戻り値としての）受け渡しといったその言語における基本的な操作を制限なしに使用できる対象のことである。  

c.f. [第一級オブジェクト](https://ja.wikipedia.org/wiki/%E7%AC%AC%E4%B8%80%E7%B4%9A%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88)

JavaScriptにおいては以下のようなことが可能なので第一級オブジェクトといえます。  
#### 関数をリテラルとして扱う
```javascript
const hello = function() {};
```
#### 関数をプロパティとしてセットする 
```javascript
const obj = {
  say: function() {}
};
```
#### 即時関数
```javascript
(function() {})();
```
#### 関数を生成する
```javascript
new Function();
```

---
## {#function-2} 高階関数
高階関数とは、引数や戻り値に関数をとる関数のことをいいます。  
親みたいなイメージです。(子供がclosure)  

###  引数にした場合
```javascript
// 高階関数
function high(say, face) {
  say(face);
}

function hello(face) {
  console.log(face);
}

high(hello, ':)');
```

### 戻り値にした場合
```javascript
// 高階関数
function high(face) {
  return function() {
    console.log(`hello ${face}`);
  }
}

high(':)')();
```
他にも配列に関数を格納できたりいろいろあるのですが、省略します。  

---
## {#function-3} スコープチェーン
### {#function-3-1} スコープチェーンとは
スコープチェーンとは、変数の名前解決をする仕組みをいいます。   
内側から外側に対して探索していくイメージです。  

```javascript
var a = 1;

var hoge = function() {
  var b = 2;

  return function() {
    var c = 3;

    return a + b + c; // [1]
  };
};

hoge()(); // 6
```
[1]から上の階層に変数が存在していくかを探索していきます。   
最終的にグローバルオブジェクトへの検索を行い終了します。  
またスコープチェーンでは最初に値が発見されたらその値を返します。  
ただし、`this`はスコープチェーンの対象にはなりません。  
なぜなら`this`は呼び出し方によって中身が変化するからです。  
FYI: [this](../this/this.md)

プロトタイプチェーンにとても似ていますが、違う点として プロトタイプチェーンでは`undefined`を返しますが、スコープチェーンでは`ReferenceError`を返します。  

### {#function-3-2} スコープチェーンの仕組み
実行時に暗黙的にcallオブジェクト(activation object)と呼ばれるものを生成し、ローカル変数を格納します。  
callオブジェクトの中には`this`, `arguments`, ローカル変数、引数が格納されています。  

```javascript
var hoge = function() {
  // hogeのcall objectが生成される

  return function() {
    // 無名関数のcall objectが生成される

    return 'hello';
  };
};

hoge()();
```
callオブジェクト自体への操作、参照はできず、関数の実行が終わればGCの対象になり回収されます。  
つまり、callオブジェクトは実行するたびに生成され終わると破棄される一時的なものです。  
callオブジェクトへのアクセスはcallオブジェクトが行います。  
callオブジェクトには親がいるのでそれを元に探索していきます。 (ここでは省略しますが、実際はstackに入れていきpopしていき名前解決を行います)

#### グローバルオブジェクト
グローバルオブジェクトには実行時(最初)の時にcallオブジェクトを生成するので、最終的にスコープチェーンの終点はグローバルオブジェクトとなります。  
なので言い換えると、グローバルオブジェクトはcallオブジェクトということです。  
また、グローバルオブジェクトが他のcallオブジェクトと違う点が一つあります。  
それはJavaScriptからアクセスすることが可能という点です。(例えば`window`)   
トップレベルの`this`はグローバルオブジェクトを参照するためです。  

#### まとめ
スコープチェーンはcallオブジェクトを使い、なければ親へ探索することで名前解決をしていきます。  

---
## {#function-4} closure
### {#function-4-1} closureとは
closureとはローカル変数の状態を保持する関数のことです。  
JavaScriptではすべての関数がclosureとなります。  
なぜなら、関数オブジェクトで生成され、必ずcallオブジェクトを参照するからです。  

```javascript
const countUp = function() {
  let cnt = 0;

  return function() { // [1] closure
    return cnt++;
  };
};

/**
  countUpはグローバル変数のcountに代入されたためcountUpのcallスタックは保持され続け、
  このように宣言することにより、GCの回収対象として外す
 */
const count = countUp();

count(); // 0
count(); // 1
count(); // 2
```

なぜ`cnt`が初期化されていないかというと、[1]の関数から見た時のグローバルスコープは`countUp`の`cnt`も含むからです。  
また、[1]の実行が終わった後に`cnt`が破棄されていないのは実行タイミングではなく定義時の状態で実行されるいるからです。  

```javascript
count(); // cnt: 0, output: 0
count(); // cnt: 1, output: 1
count(); // cnt: 2, output: 2
```
上記のように以前の状態を保持し、関数のローカル変数をその関数内の関数([1])が参照していることがわかります。  
closureはこのようにcallスタックを開放させずにローカル変数を参照し続けることを利用します。  
closureとは関数オブジェクトとcallスタックを参照するものをいうので、高階関数の引数や戻り値として定義される関数をclosureとなります。   

### {#function-4-2} メリット
- 関数に状態を保持させることができる(privateの振る舞いをすることが可能)
- グローバルへの宣言を抑えることができる

### {#function-4-3} 使用例
#### 初期値を設定させることによりカスタマイズ性を持たせる
```javascript
const repeatFace = function(f) {
  let face = f;

  return function() {
    face = face.repeat(2);
    return face;
  }
}

const smile = repeatFace(':)');
smile(); // ":):)"
smile(); // ":):):):)"

const cry =  repeatFace(';(');
cry(); // ";(;("
cry(); // ";(;(;(;("
```
このように引数として様々なものを渡すことによりカスタマイズ性の高い関数を作ることができます。 

#### フラグ管理をさせる
```javascript
function say() {
  let flag = false;

  setInterval(function() {
    if (!flag) {
      flag = true;
      console.log(':)');
    }
    else {
      console.log(';(');
    }
  }, 1000);
}

say();
/*
  :)
  ;(
  ;(
  ...
*/
このようにローカル変数としてflagを管理し、保持することでグローバルへの汚染を防ぐことができます。  
```
