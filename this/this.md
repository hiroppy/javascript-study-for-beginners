# this

# [めも] autobindとFunction Bind Syntaxの話いれたい

## 目次
- [thisとは](#this-1)
- [thisの種類](#this-2)
  - [グローバルスコープの参照](#this-2-1)
  - [関数呼び出し](#this-2-2)
  - [メソッド呼び出し](#this-2-3)
  - [コンストラクタ呼び出し](#this-2-4)
  - [call/apply](#this-2-5)
  - [bind](#this-2-6)
- [アロー関数](#this-3)

---
## {#this-1} thisとは
### 関数実行時に決定される値
#### つまり *呼び出し時に値が決まるため呼び出し元から判断する(結論!)*

## {#this-2} thisの種類
- グローバルスコープの参照
- 関数呼び出し
- メソッド呼び出し
- コンストラクタ呼び出し
- `Function.prototype.apply()`, `Function.prototype.call()` での呼び出し
- `Function.prototype.bind()` での呼び出し

---
### {#this-2-1} グローバルスコープの参照

ブラウザではグローバルスコープはwindowオブジェクトである  
グローバルな値を設定するとwindowの下につく  

```javascript
console.log(this);
// window

var global1 = '1';
window.global2 = '2';

console.log(this.global1, this.global2);
// 1 2
```

---
## {#this-2-2} 関数呼び出し

```javascript
function say() {
  console.log(this);
  console.log(this.global1);
}

var global1 = 'hoge';
say();
// window
// hoge
```

つまり `say()` の呼び出し元がグローバルであるため、`say()` 内のthisはグローバルオブジェクトを示す
```javascript
(function() {
  console.log(this);
  // window
})();

```

即時関数(来週説明する)も同様に呼び出し元は上記のコードはグローバルなのでthisはグローバルオブジェクトを示す

```javascript
function say() {
  check();

  function check() {
    console.log(this);
  }
}

say();
```

これはどうなるでしょうか？

答えは `window` が出力されます。
呼び出し元を見るとわかります。

### JavaScriptにおける関数とメソッドの違い
#### 結論から言うと、*thisが指しているポイントによって呼び方が変わります*
_後ほど話すのでコードの違いだけ覚えておいてください_

```javascript
// メソッド呼び出し
obj.say();

// 関数呼び出し
say();
```

---
## {#this-2-3} メソッド呼び出し

```javascript
const obj = {};

obj.say = function() {
  console.log(this);
};

obj.say();

/**
  Object {}
say: function()
__proto__: Object
 */
```

所属先が帰ってきます

つまり、インスタンスの内部にあるのがメソッドです

### 問題
```javascript
const obj =  {
  hoge: 100,
  say: function() {
    console.log('say', this.hoge); // [1]
    test();

    function test() {
      console.log('test', this.hoge); // [2]
    }
  }
};

obj.say(); // [1], [2]

const say = function() {
  console.log(this); // [3], [4]
}

say(); // [3]
obj.say = say;
obj.say(); // [4]

function test() {
  console.log(this); // [5]
}

obj.say = test;
obj.say(); // [5]
```

答え
```
[1] 100
[2] undefined(windowを指すが、windowにhogeはないため)
[3] window
[4] obj {hoge:100, say: function()}
[5] obj {hoge:100, say: function()}
```

---
## {#this-2-4} コンストラクタ呼び出し
```javascript
function Hello(w) {
  console.log(this); // Hello {}

  this.word = w;
  this.combine = function() {
    this.w += 'piyo';
  }
}

const hello = new Hello('hoge'); // Hello {word: "hoge"}
hello.combine();

console.log(hello.word); // hogepiyo
```

インスタンス化した時に、自身にthisをセットします

```javascript
(new Hello('hoge')) // Hello {word: "hoge"}
Hello('hoge') // windowに挿入されたので関数内のwordとcombineはグローバルになった
```
結局ただの関数であるので、関数内のthisの意味を捉えるようにしてnewをするべきか判断する必要がある

---
## {#this-2-5} call / apply
強制的にthisを束縛するものです

```javascript
function say() {
  console.log(this);
}

say(); // window
say.call({}); // {}
say.apply({}) // {}
  ```

このように関数が呼ばれた時に決まるthisの値をコントロールすることが可能です  
第一引数にthisの値を渡します  

callとapplyの違いは第二引数にあります  
- call  引数の数は決まっていません
- apply 配列の一つのみとなる(受け取り先で分解されます)

```javascript
const obj = {};

function say() {
  console.log(arguments); // [1]
  this.hoge = arguments[0];
  this.fuga = arguments[1];
  console.log(this); // [2]
}

say.call(obj, 'hoge', 'fuga');
// [1] ["hoge", "fuga"]
// [2] Object {hoge: "hoge", fuga: "fuga"}

console.log(obj);
// Object {hoge: "hoge", fuga: "fuga"}

say.apply(obj, ['piyo', 'hogera']);
// [1] ["piyo", "hogera"]
// [2] Object {hoge: "piyo", fuga: "hogera"}

console.log(obj);
// Object {hoge: "piyo", fuga: "hogera"}
```
callやapplyが使われている場合、必ず意図があるはずなのでしっかりと見極める必要があります  

---
## {#this-2-6} bind

bindもcall/apply同様にthisを束縛します  

違う点は、*関数実行をしない*点です
```javascript
function say() {
  console.log(this); // [1]
}

say.call({}); // [1] Object {}
say.bind({}); // 実行されているわけではないのでconsoleの出力はない

say.bind({})(); // [1] Object {}
```

つまり、bindは新しい関数を作って返します

第二引数では引数を予めセットしておくこともできるので引数の部分適用も可能です(今回はthisの話なので省きます)

---
## {#this-3} アロー関数
ES2015から入った無名関数の糖衣構文です  
しかし全くの糖衣構文ではないため、使いどころは考える必要があります

thisでの違いはアロー関数は関数呼び出し時によってthisの値は*決まらず*に、定義時にthisの値が決まります  
#### つまり、アロー関数を使うことにより定義時のthisを束縛することが可能になります
なので呼び出し元のthisであることが保証されたことになります

```javascript
const obj = {};

obj.say = function() {
  console.log(this);
  // { say: [Function], test: [Function] }
}

obj.test = () => {
  console.log(this);
  // {}
}
```

_今回はthisの話なので省略しますが、違いとして arguments objectを持たなかったり、コンストラクタにならないと違いはあります_

今の時代、基本的にアロー演算子を使って書きますが、thisを使う場合とarguments objectを使う場合は注意して使うべきでです

---
## まとめ

以下のパターンを覚えておけばjavascriptのthisはどうにかなります
- グローバルスコープの参照
- 関数呼び出し
- メソッド呼び出し
- コンストラクタ呼び出し
- `Function.prototype.apply()`, `Function.prototype.call()` での呼び出し
- `Function.prototype.bind()` での呼び出し
