---
title: 「あなたのVueファイルは読みづらい」と文句を言われたくない① -- 記述方法編 --
tags:
  - JavaScript
  - TypeScript
  - Vue.js
  - 可読性
  - コード品質
private: false
updated_at: '2024-02-05T14:59:17+09:00'
id: 46342f529320652a8fad
organization_url_name: null
slide: false
ignorePublish: false
---



# 目次
[1. 前書き](#1-前書き)
[2. サンプルコード概要](#2-サンプルコード概要)
[3. 実装ポイント](#3-実装ポイント)
[4. 後書き](#4-後書き)

現在の記事は①になります。②はこちらのリンクからどうぞ

https://qiita.com/IJproject/items/eece70d9c75558a621db

# 1. 前書き

この記事のテーマは、「Vueファイルの可読性を可能な限り上昇させる」です。ファイル内の記述方法を工夫し、**優しいコーディング**をすることを目的としています。
**もし、この記事を参考にしたコーディングをして同僚に文句を言われたら、是非その旨をこの記事のコメントに記入してください。** ひとつひとつのコメントに対して涙目になりながら中身を改善していきます。

## 自己紹介

自称フルスタックエンジニアの岩田と申します。業務の経験は 2023/6~ なので、まだ１年にも満たないミーハーな人間です。自分の経験が他のエンジニアさんの参考になれば良いなと思いながら記事を書いていますので、もし良ければ他の記事もご覧ください！！

## 概要

**コーディングスタイルの統一**は、チーム開発において１つの大事な要素になっていると思います。

各々の書き方で進めてしまうと、ある部分では「A」の書き方で書いているのに、他の部分では「B」の書き方で、、、ということが起こりえます。というより起こります。さらに、適度にファイルの分割をしなかったり、分割法が人によって違ったり、そもそも全く分割しないために行数が増え過ぎたり。そういったプロダクトに対してフロントエンドエンジニアが感じることはカオス過ぎて目も当てられないです。(実体験に基づく)

少しでもその状態になる可能性を減らすためにも、コーディングスタイルを統一することは必須だと思っています。

この記事ではVueファイルの書き方に焦点を当て、僕が個人的に考案した可読性の高いファイルの書き方について紹介していこうと思います。

## この記事の対象者

・Vue.jsのコーディングに慣れてきて、次のステップに進みたい方
・ある程度の規模感になってくると、途端にコードが読みづらくなってしまうと感じている方

**あくまで一個人のコーディングスタイルなので参考程度にしてくださればと思っています。**

# 2. サンプルコード
サンプルコードについて、事前に少しだけ解説を入れておきます。
**記述する際に意識していること**ですが、
* 順序を分かりやすく = 乱雑に並べない
    * import順序
    * 関数の並べる順序 など
* 初見さんが理解しやすく = アロー関数内の処理を簡潔にする
    * 別関数に分離する など
* 処理内容を理解しやすく = 処理ごとに記述をまとめる
    * 機能ごとにコメントを書く
    * 改行を上手く使う

の３つです。
この辺りを認識していただいた上で、サンプルコードや実装ポイントをご覧になってくださればと思っています。

(注) 説明の関係上、無理矢理な部分(別関数への分離の部分など)があります。その点ご理解いただければと思います。
(注) CSS無しなので、デザイン関しては許してください。あ、これは敢えて付けてないだけですからね、別にめんどくさかった訳では（滝汗）

![スクリーンショット 2024-02-01 11.37.53.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687984/9ff04ea5-3523-eb5d-b32e-cef19e558f1a.png)

```vue:App.vue (ts)
<script setup lang="ts">
import { ref } from 'vue';

/**
 * HOME画面('/')
 */

interface ShoppingItem {
    id: number;
    name: string;
    price: number;
    count: number;
}

const shoppingList = ref<ShoppingItem[]>([
    { id: 1, name: '', price: 0, count: 0 },
]);

// 登録されている商品を削除する
const deleteShoppingItem = (id: number) => {
    shoppingList.value = shoppingList.value.filter((item) => item.id !== id);
};

// 新規商品を追加する
const addShoppingItem = () => {
    shoppingList.value.push({
        id: getNewShoppingItemId(),
        name: '',
        price: 0,
        count: 0,
    });
};

// 合計金額を計算する
const totalPrice = ref(0);
const isCalcFinished = ref(false);
const calcTotalPrice = () => {
    totalPrice.value = getTotalPrice(shoppingList.value);
    isCalcFinished.value = true;
};

// 画面上の入力を全てリセットする
const resetAll = () => {
    shoppingList.value = [{ id: 1, name: '', price: 0, count: 0 }];
    totalPrice.value = 0;
    isCalcFinished.value = false;
};

// 新規商品を追加する際、IDの重複を防ぐための関数
function getNewShoppingItemId() {
    const lastIndex = shoppingList.value.length - 1;
    return lastIndex < 0 ? 1 : shoppingList.value[lastIndex].id + 1;
}

// 合計金額を計算する関数
function getTotalPrice(list: ShoppingItem[]): number {
    let totalPrice = 0;
    list.forEach((item) => {
        totalPrice += item.price * item.count;
    });
    return totalPrice;
}
</script>

<template>
    <div>
        <h1>Qiita金額計算</h1>
        <div>
            <h3>買い物リスト</h3>
            <form @submit.prevent="calcTotalPrice">
                <ol>
                    <li v-for="(item, index) in shoppingList" :key="item.id">
                        <label>
                            商品名
                            <input type="text" v-model="shoppingList[index].name" />
                        </label>
                        <label>
                            値段
                            <input type="number" v-model="shoppingList[index].price" />
                            円
                        </label>
                        <label>
                            購入数
                            <input type="number" v-model="shoppingList[index].count" />
                            個
                        </label>
                        <button type="button" @click="deleteShoppingItem(item.id)">
                            削除
                        </button>
                    </li>
                </ol>
                <button type="button" @click="addShoppingItem">追加</button>
                <button type="submit">計算</button>
            </form>
        </div>
        <div v-if="isCalcFinished">
            <h3>合計金額</h3>
            <div>
                <h5>合計金額：{{ totalPrice }}円</h5>
                <button type="button" @click="resetAll">リセット</button>
            </div>
        </div>
    </div>
</template>
```
```vue:App.vue (js)
<script setup>
import { ref } from 'vue';

/**
 * HOME画面('/')
 */

const shoppingList = ref([
    { id: 1, name: '', price: 0, count: 0 },
]);

// 登録されている商品を削除する
const deleteShoppingItem = () => {
    shoppingList.value = shoppingList.value.filter((item) => item.id !== id);
};

// 新規商品を追加する
const addShoppingItem = () => {
    shoppingList.value.push({
        id: getNewShoppingItemId(),
        name: '',
        price: 0,
        count: 0,
    });
};

// 合計金額を計算する
const totalPrice = ref(0);
const isCalcFinished = ref(false);
const calcTotalPrice = () => {
    totalPrice.value = getTotalPrice(shoppingList.value);
    isCalcFinished.value = true;
};

// 画面上の入力を全てリセットする
const resetAll = () => {
    shoppingList.value = [{ id: 1, name: '', price: 0, count: 0 }];
    totalPrice.value = 0;
    isCalcFinished.value = false;
};

// 新規商品を追加する際、IDの重複を防ぐための関数
function getNewShoppingItemId() {
    const lastIndex = shoppingList.value.length - 1;
    return lastIndex < 0 ? 1 : shoppingList.value[lastIndex].id + 1;
}

// 合計金額を計算する関数
function getTotalPrice() {
    let totalPrice = 0;
    list.forEach((item) => {
        totalPrice += item.price * item.count;
    });
    return totalPrice;
}
</script>

<template>
    <div>
        <h1>Qiita金額計算</h1>
        <div>
            <h3>買い物リスト</h3>
            <form @submit.prevent="calcTotalPrice">
                <ol>
                    <li v-for="(item, index) in shoppingList" :key="item.id">
                        <label>
                            商品名
                            <input type="text" v-model="shoppingList[index].name" />
                        </label>
                        <label>
                            値段
                            <input type="number" v-model="shoppingList[index].price" />
                            円
                        </label>
                        <label>
                            購入数
                            <input type="number" v-model="shoppingList[index].count" />
                            個
                        </label>
                        <button type="button" @click="deleteShoppingItem(item.id)">
                            削除
                        </button>
                    </li>
                </ol>
                <button type="button" @click="addShoppingItem">追加</button>
                <button type="submit">計算</button>
            </form>
        </div>
        <div v-if="isCalcFinished">
            <h3>合計金額</h3>
            <div>
                <h5>合計金額：{{ totalPrice }}円</h5>
                <button type="button" @click="resetAll">リセット</button>
            </div>
        </div>
    </div>
</template>
```

# 3. 実装ポイント

ではここから、一つずつポイントを紹介していきます。
サンプルコードを参照しながら追っていくと分かりやすいと思います。

## 3-1. ファイルの説明をjsdoc記法で書いておく

記入が不要なファイルもあると思いますが、できるだけ記入しておいた方が良いかなと思っています。
詳しい書き方は、以下のQiitaの記事を参照してみてください。

https://qiita.com/zaburo/items/c90ab1a3d7751f610d27

```vue:jsdoc
<script setup>
/**
 * ここに説明
 */
</script>
```

## 3-2. `<script setup>`を使用する

```vue:App.vue (js)
<script setup>

</script>
```
```vue:App.vue (ts)
<script setup lang="ts">

</script>
```
まずscriptタグについては`<script setup>`で統一しましょう。setupを付けない理由があるのであれば敢えて付けないという選択もアリだとは思いますが、基本的にそのようなケースはないと思っています。

## 3-3. importの順序
正直なところ、これに関しては好みの問題になるとは思いますが、最低でもimport元のジャンルだけは合わせておいた方が良いと思います。
今回は`ref`しかimportしていないので、参考程度に架空のものを置いておきます。
```vue:App.vue (悪い例)
<script setup>
import { SmapleComponent2 } from './components/SampleComponent2'
import { getSampleUsers } from './hooks/getSampleUser'
import { SmapleComponent1 } from './components/SampleComponent1'
import { ref, onMounted } from 'vue'; 
</script>
```
```vue:App.vue (良い例)
<script setup>
import { ref, onMounted } from 'vue'; 
import { SmapleComponent1 } from './components/SampleComponent1'
import { SmapleComponent2 } from './components/SampleComponent2'
import { getSampleUsers } from './hooks/getSampleUser'
</script>
```
例えばですけど、僕の場合は
1. node_modules
2. コンポーネント(templateタグ内で使用される順番)
3. ファイルから切り離した変数や関数、型定義
4. その他

といった順番でimportするようにしています。(良い例のとおり)
## 3-4. define系やinterface系は最初に
```vue:App.vue (ts)
<script setup lang="ts">
import { ref } from 'vue';

interface ShoppingItem {
    id: number;
    name: string;
    price: number;
    count: number;
}

// この下に処理を実装していく
</script>
```
コンポーネントを分離させた時に使用する`defineProps`のようなdefine系や、TypeScriptで型定義をするときに使用するinterface系は、importの下に書いてしまいましょう。理由は特にないですが、ファイル内で埋もれることだけは避けたいので。
## 3-5. コメントを入れないとわからないような変数名はつけない
```js:App.vue (悪い例)
// 購入する商品情報
const item = {}
// 購入する商品の詳細情報
const detail = {}
```
```js:App.vue (良い例)
const ShoppingItem = {}
const ShoppingItemDetail = {}
```
コメントを書かないといけなくなるような変数名にするのはやめましょう。例えば detail や item などの命名がそれに当たります。長すぎる変数名もどうかと思いますが、初見さんがパッと見でどんな変数なのか理解できるような名前にしてあげましょう。
## 3-6. reactive ではなく ref を使う
```js:ref
const test = ref('test')
const testObj = ref({id: 1, content: "testObj"})
console.log(test.value, testObj.value.content);
// 'test' 'testObj'
```
```js:reactive
const test = reactive({content: 'test'})
const testObj = reactive({id: 1, content: "testObj"})
console.log(test.content, testObj.content)
// 'test' 'testObj'
```
見比べてみてどうでしょう？全く同じ結果が得られていますが、個人的に **ref** の方がメリットが多いように思えます。
* オブジェクト型だけでなく、プリミティブ型も扱えるため汎用性が高い
* scriptタグ内で値にアクセスする際、`.value`を付けないといけないため ref で定義された変数なのだとひと目で分かる

同じ参照を扱う変数なのに、アクセスの仕方が違うというのは統一感がなくコードの品質を落としかねないので、**ref に統一する**方が良いのかなと思っています。refやreactiveで定義された変数の値が変更された場合、画面の際レンダリングがされますから、ref や reactive の値が変更されるタイミングはパッと見で分かる方が良いですよね。

## 3-7. ref と not ref の使い分け
僕のイメージですが、ref についての理解が浅い段階だと、僕もそうでしたが、とりあえず変数を定義するときは ref を使っておけばいいという風潮があるような気がします。しかし、ref の場合は値の変更をvueが検知し、再レンダリングさせるので少々挙動が違います。値の監視が必要ない変数を ref で定義してしまうと不要なメモリの消費になってしまうので避けたいですね。
```vue:悪い例
<script setup>
import { ref } from 'vue';

const testContentList = ref(['test0', 'test1', 'test2'])
const test = ref(testContentList.value[0]);
</script>
<template>
    <div>{{ test }}</div>
</template>
```
```vue:良い例
<script setup>
import { ref } from 'vue';

const testContentList = ['test0', 'test1', 'test2']
const test = ref(testContentList.value[0]);
</script>
<template>
    <div>{{ test }}</div>
</template>
```
この場合、`test`の値が変更されたら再レンダリングして表示を更新して欲しいので ref で定義しています。ただ、`testContentList`は値が変更されて、再レンダリングされたとしても表示画面は何も変わりません。つまり値の監視が不要なので ref を使っていません。
## 3-8. Scriptファイル内の記述順序

基本的には、templateタグ内の順番に合わせて記述していくようにしましょう。

### 1. 関数
サンプルコードを見てもらうと分かる通り、scriptタグ内でのアロー関数の定義の順番は、
1. 商品の削除ボタン
2. 新規商品の追加ボタン
3. 合計金額計算ボタン
4. リセットボタン

となっています。これはtemplateタグ内で出てくる順番に記述しています。

### 2. ref 定義の位置
配置位置については２種類あります。
* ページ全体で使用する変数
    * サンプルコードでいう`shoppingList`
    * 定義位置は`define系`や`interface系`の直下
* 一部の処理に紐づいて使用する変数
    * サンプルコードでいう`totalPrice, isCalcFinished`
    * 配置位置は各処理をする関数の直前

## 3-9. 属性の書く順番
今回は煩雑さを省くために使用していませんが、`Tailwind`のようなCSSフレームワークを使用すると、以下のようにタグの属性が長くなってしまうことがあります。
```vue:属性の書き方(悪い例)
<button type="button" class="rounded-md bg-indigo-600 px-2.5 py-1.5 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600" @click="clickedUpdateButton">
    更新
</button>
```
```vue:属性の書き方(良い例)
<button type="button" @click="clickedUpdateButton" class="rounded-md bg-indigo-600 px-2.5 py-1.5 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600">
    更新
</button>
```
見比べてみてもらうと分かる通り、圧倒的に後者の方がコードが見やすいですよね。
このボタンが押されるとどんな処理が走るのだろうかと見てみようとしても、わざわざ右にスクロールしていかないと関数名等を見ることはできません。

書く順番は人それぞれですが、例えばですけど僕の場合は
1. タグの性質を決める属性
2. タグの動作を決める属性
3. その他

とにかく、動作面に関する属性を先に書こうというスタンスです。
## 3-10. ２つの関数
今回のサンプルコードでは２種類の関数を使用しています。
* アロー関数 
* 返り値を持つ関数

それぞれの使い分けは以下のように行っています。
### 1. アロー関数
```js:アロー関数
const name = () => {}
```
* 一連の処理をまとめるために使う
* 適切な場所に記述する
* 主にtemplateタグ内の`@click`などから呼び出す

### 2. 返り値を持つ関数
```js:返り値を持つ関数
function name() {
    return value
}
```
* 値の変更や加工に関する処理をまとめる
* scriptタグの一番下の`function`郡の中に記述する
* 主にscriptタグ内で呼び出して使う
* **関数外の変数の値を変更しない**

**アロー関数だけで十分なのでは？と思われる方が多いと思うのですが、わざわざ使用している理由は以下の通りです。**
* 定義場所について、使用したい箇所よりも後(下)に関数の定義をすることができるため、scriptタグ内の最下部に配置することができ、コードの見通しが良くなる
* 値の変更や加工はデータフローを理解するのにそれほど重要ではない

今回のサンプルコードの場合、どのように値が変更・加工されるのかに関する構文を`function`で分離し、データフローの理解の手助けをするために、処理内容が分かるような関数の名前を付けています。
おそらくですが、処理内容を把握しようと思った場合、最悪`function`の部分が全て欠如していても、なんとなく何をしようとしているのか分かりますよね？(圧)
つまり、理解するという面で言えば、その分のコードを省略したのとほぼ同じですよね？(圧)

これぐらい簡易的なコードであれば恩恵はあまり感じないと思いますが、コードが肥大化していくほど恩恵を感じていくのではないかと思っています。

# 4. 後書き

## コードの品質の重要性
チーム開発においてコードの品質を高く保つためには、
* コーディングルールを徹底する
* 可読性を上げるような工夫をする

この２点が最重要だと思います。正直この辺りが疎かになると、たとえ２人の開発でもごちゃごちゃになります。さらに人が増えるとなると尚更です。こういったルールを身体に染み込ませることにもある程度の工数は掛かってしまいますが、その後の保守のことを考えるとプラスの要素しかないですよね。
少しだけコーディングルールを見直してみてはいかがでしょうか。

コーディングを快適にするためのルール作りにこの記事の内容が参考になっていることを切に願っています。
