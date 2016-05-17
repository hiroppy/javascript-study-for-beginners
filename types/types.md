# 型・値

## 目次
- [型の種類](#types-1)
  - [プリミティブ型](#types-1-1)
  - [オブジェクト型](#types-1-2)
- [プリミティブ型](#types-2)
  - [undefined](#types-2-1)
  - [null](#types-2-2)
  - [undefinedとnullの比較方法](#types-2-3)
  - [undefinedとnullのまとめ](#types-2-4)
- [オブジェクト型](#types-3)
  - [オブジェクト型の注意点](#types-3-1)
  - [配列かどうかの判定](#types-3-2)
- [Wrapper Objects](#types-4)
- [各型の比較](#types-5)
- [falseになる値](#types-6)

---
## {#types-1} 型の種類

### {#types-1-1} プリミティブ型
- 数値(number)
- 文字列(string)
- 論理型(boolean)
- undefined
- null

### {#types-1-2} オブジェクト型
- object
  - object
  - function
  - array

---
## {#types-2} プリミティブ型

```javascript
> typeof 1
  'number'
> typeof 'javascript'
  'string'
> typeof true
  'boolean'
> typeof undefined
  'undefined'
> typeof null
  'object'
```

nullとundefinedは特殊な型

### {#types-2-1} undefined

定義されたグローバルな変数で読み込み専用(ES5から読み込み専用となった)  

#### undefinedになる例
- 初期化してない変数の値
- 存在しないobjectへのアクセス
- 存在しない配列要素へのアクセス
- 戻り値のない関数が返す値
- 関数の呼び出し時の引数値

```javascript
> let b
  undefined
> obj = {}
  {}
> obj.a
  undefined
> arr = []
  []
> arr[0]
  undefined
> fn = (a) => {console.log(`argument: ${a}`)}
  [Function]
> fn()
  argument: undefined
  undefined
```

### {#types-2-2} null

他の言語のnullやnilと等価の意味を持ちます  
該当した値がない場合にnullを使います  
つまり、返すものがない場合に意図的にnullを用います  

### {#types-2-3} undefinedとnullの比較方法
```javascript
> a = undefined
  undefined
> b = null
  null
> a == b
  true
> a === b
  false
```
厳密等価演算子で比較しなければ両者ともtrueを返す  
区別せずに判断したい場合は以下のように書く
```javascript
> a = undefined
  undefined
> if (a == null) console.log('ok')
  ok
```

### {#types-2-4} undefined と null のまとめ
#### undefinedはシステムが予期してない場合に使われる  
#### nullはプログラムレベルで予定していた場合に使われる  

なので関数に値を渡したり、値を設定したい場合は*null*を使うのが適切である

---
## {#types-3} オブジェクト型
- object
- function
- array

```javascript
> obj = {}
  {}
> fn = () => {}
  [Function]
> arr = []
  []
> typeof obj
  'object'
> typeof fn
  'function'
> typeof arr
  'object'
```

functionはtypeofでfunctionとなるがobjectである

以下の通りfunctionもobjectであることがわかる
```javascript
> fn.test = 'test'
  'test'
> fn
  { [Function] test: 'test' }
```

### {#types-3-1} オブジェクト型の注意点
変数のコピー時には*アドレス*がコピーされる  
なのでオブジェクト型は*参照型*とも呼ばれる

```javascript
> obj1 = {test: 'test1'}
  { test: 'test1' }
> obj2 = obj1
  { test: 'test1' }
> obj2
  { test: 'test1' }
> obj2.test = 'test2'
  'test2'
> obj1.test
  'test2'
```

コピーをする方法

- `JSON.parse(JSON.stringify(obj))`
- `Object.assign`
  - ES2015から入った
- jQueryの`extend`
- lodashの`clone`
`Object.assign`が一番いいと思う(polyfillは必要である)

### {#types-3-2} 配列かどうかの判定

```javascript
> arr = []
  []
> typeof arr
  'object'
> arr instanceof Array
  true
> Array.isArray(arr)
  true
> Object.prototype.toString.call(arr)
  '[object Array]'
  ```

`Array.isArray()` を使うのが一番ベストだと思います(IE8以下をサポートしないなら)

---
## {#types-4} Wrapper Objects
プリミティブ型でもオブジェクトのようにプロパティへ参照できる仕組み  
プリミティブ型がプロパティへ参照した際に生成される一時的なオブジェクト

nullとundefinedには適用されません

```javascript
> str = 'string'
  'string'
> str.length
  6
```

あくまで読み出し専用なため独自定義はできません  
明示的に生成する場合(基本的に使いません)
```javascript
> str = new String('hoge')
  [String: 'hoge']
> num = new Number(1)
  [Number: 1]
> bool = new Boolean(true)
  [Boolean: true]
> typeof str
  'object'
```
もちろんtypeofで見るとobjectになっている

---
## {#types-5} 各型の比較

```javascript
> num = 1
  1
> str = '1'
  '1'
> num == str
  true
> num === str
  false
> obj1 = {test: 'hoge'};
  { test: 'hoge' }
> obj2 = {test: 'hoge'};
  { test: 'hoge' }
> obj1 == obj2
  false
> obj1 === obj2
  false
```

理由がない限り型チェックをする厳密等価演算子(`===`)を使いましょう  
オブジェクト型は値では比較しません
```javascript
> obj1 = {}
  {}
> obj2 = obj1
  {}
> obj1 === obj2
  true
```

上記のように両者とも同じオブジェクトを参照している時に同一と判断されます

---
## {#types-6} falseになる値
```javascript
> !!0
false
> !!-0
false
> !!NaN <-- NaNはfalseだがInfinityはtrue
false
> !!'' <-- 空文字の時はfalse
false
> !!undefined
false
> !!null
false
```
`0, -0, NaN, '', undefined, null` の6つがfalseになる値となる
