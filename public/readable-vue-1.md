---
title: 「あなたのVueファイルは読みづらい」と文句を言われたくない①
tags:
  - Vue
  - JavaScript
  - TypeScript
  - 可読性
  - コード品質
private: true
updated_at: ''
id: ReadableVue1
organization_url_name: null
slide: false
ignorePublish: false
---



# 目次
[1. 前書き](#1-前書き)
[2. サンプルコード概要](#2-サンプルコード概要)
[3. 実装ポイント](#3-実装ポイント)
[4. 後書き](#4-後書き)

# 1. 前書き

この記事のテーマは、「Vueファイルの可読性を可能な限り上昇させる」です。

## 自己紹介

自称フルスタックエンジニアの岩田と申します。業務の経験は 2023/6~ なので、まだ１年にも満たないミーハーな人間です。自分の経験が他のエンジニアさんの参考になれば良いなと思いながら記事を書いていますので、もし良ければ他の記事もご覧ください！！

## 概要

コーディングスタイルの統一は、チーム開発において１つの大事な要素になっていると思います。

各々の書き方で進めてしまうと、ある部分では「A」の書き方で書いているのに、他の部分では「B」の書き方で、、、ということが起こりえます。というより起こります。さらに、適度にファイルの分割をしなかったり、分割法が人によって違ったり、そもそも全く分割しないために行数が増え過ぎたり。そういったプロダクトに対してフロントエンドエンジニアが感じることはカオス過ぎて目も当てられないです。(実体験に基づく)

少しでもその状態になる可能性を減らすためにも、コーディングスタイルを統一することは必須だと思っています。

この記事ではVueファイルの書き方に焦点を当て、僕が個人的に構想した可読性の高いファイルの書き方について紹介していこうと思います。
ちなみにこの記事は①となっています。②ではファイルの分割について紹介しようと思っていますので、作成が完了し次第、この下にリンクを貼り付けます。

## この記事の対象者

・Vue.jsのコーディングに慣れてきて、次のステップに進みたい方
・ある程度の規模感になってくると、途端にコードが読みづらくなってしまうと感じている方

あくまで一個人のコーディングスタイルなので参考程度にしてくださればと思っています。

# 2. サンプルコード
サンプルコードについて、事前に少しだけ解説を入れておきます。

```vue:App.vue (ts)
<script setup lang="ts">
import { ref } from 'vue';

interface ShoppingItem {
    id: number;
    name: string;
    price: number;
    count: number;
}

const shoppingList = ref<ShoppingItem[]>([
    { id: 1, name: '', price: 0, count: 0 },
]);

const deleteShoppingItem = (id: number) => {
    shoppingList.value = shoppingList.value.filter((item) => item.id !== id);
};

const addShoppingItem = () => {
    shoppingList.value.push({
        id: getNewShoppingItemId(),
        name: '',
        price: 0,
        count: 0,
    });
};

const totalPrice = ref(0);
const isCalcFinished = ref(false);

const calcTotalPrice = () => {
    totalPrice.value = getTotalPrice(shoppingList.value);
    isCalcFinished.value = true;
};

const resetAll = () => {
    shoppingList.value = [{ id: 1, name: '', price: 0, count: 0 }];
    totalPrice.value = 0;
    isCalcFinished.value = false;
};

function getNewShoppingItemId() {
    const lastIndex = shoppingList.value.length - 1;
    return lastIndex < 0 ? 1 : shoppingList.value[lastIndex].id + 1;
}

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

const shoppingList = ref([
    { id: 1, name: '', price: 0, count: 0 },
]);

const deleteShoppingItem = (id) => {
    shoppingList.value = shoppingList.value.filter((item) => item.id !== id);
};

const addShoppingItem = () => {
    shoppingList.value.push({
        id: getNewShoppingItemId(),
        name: '',
        price: 0,
        count: 0,
    });
};

const totalPrice = ref(0);
const isCalcFinished = ref(false);

const calcTotalPrice = () => {
    totalPrice.value = getTotalPrice(shoppingList.value);
    isCalcFinished.value = true;
};

const resetAll = () => {
    shoppingList.value = [{ id: 1, name: '', price: 0, count: 0 }];
    totalPrice.value = 0;
    isCalcFinished.value = false;
};

function getNewShoppingItemId() {
    const lastIndex = shoppingList.value.length - 1;
    return lastIndex < 0 ? 1 : shoppingList.value[lastIndex].id + 1;
}

function getTotalPrice(list){
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
```vue:App.vue (良い例)
<script setup>
import { ref, onMounted } from 'vue'; 
import { SmapleComponent1 } from './components/SampleComponent1'
import { SmapleComponent2 } from './components/SampleComponent2'
import { getSampleUsers } from './hooks/getSampleUser'
</script>
```
```vue:App.vue (悪い例)
<script setup>
import { SmapleComponent2 } from './components/SampleComponent2'
import { getSampleUsers } from './hooks/getSampleUser'
import { SmapleComponent1 } from './components/SampleComponent1'
import { ref, onMounted } from 'vue'; 
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
```js:App.vue (良い例)
const ShoppingItem = {}
const ShoppingItemDetail = {}
```
```js:App.vue (悪い例)
// 購入する商品情報
const item = {}
// 購入する商品の詳細情報
const detail = {}
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
見比べてみてどうでしょう？全く同じ結果が得られていますが、個人的に ref の方がメリットが多いように思えます。
* オブジェクト型だけでなく、プリミティブ型も扱えるため汎用性が高い
* scriptタグ内で値にアクセスする際、`.value`を付けないといけないため ref で定義された変数なのだとひと目で分かる

同じ参照を扱う変数なのに、アクセスの仕方が違うというのは統一感がなくコードの品質を落としかねないので、ref に統一する方が良いのかなと思っています。refやreactiveで定義された変数の値が変更された場合、画面の際レンダリングがされますから、ref や reactive の値が変更されるタイミングはパッと見で分かる方が良いですよね。

## 3-7. ref と not ref の使い分け
僕のイメージですが、ref についての理解が浅い段階だと、僕もそうでしたが、とりあえず変数を定義するときは ref を使っておけばいいという風潮があるような気がします。しかし、ref の場合は値の変更をvueが検知し、再レンダリングさせるので少々挙動が違います。値の監視が必要ない変数を ref で定義してしまうと不要なメモリの消費になってしまうので避けたいですね。
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
この場合、`test`の値が変更されたら再レンダリングして表示を更新して欲しいので ref で定義しています。ただ、`testContentList`は値が変更されて、再レンダリングされたとしても表示画面は何も変わりません。つまり値の監視が不要なので ref を使っていません。
## 3-8. Scriptファイル内の記述順序

基本的には、templateタグ内の順番に合わせて記述していくようにしましょう。

## 3-9. 属性の書く順番





## 3-10. ２つの関数
今回のサンプルコードでは２種類の関数を使用しています。
* アロー関数 
* 返り値を持つ関数

それぞれの使い分けは以下のように行っています。
### アロー関数
```js:アロー関数
const name = () => {}
```
* 一連の処理をまとめるために使う
* 適切な場所に記述する
* 主にtemplateタグ内の`@click`などから呼び出す

### 返り値を持つ関数
```js:返り値を持つ関数
function name() {
    return value
}
```
* 値の変更や加工に関する処理をまとめる
* scriptタグの一番下の`function`郡の中に記述する
* 主にscriptタグ内から呼び出して使う

アロー関数だけで十分なのでは？と思われる方が多いと思うのですが、わざわざ使用している理由は以下の通りです。
* 定義場所について、使用したい箇所よりも後(下)に関数の定義をすることができる
* 値の変更や加工はデータフローを理解するのにそれほど重要ではない

今回のサンプルコードの場合、どのように値が変更・加工されるのかに関する構文を`function`で分離し、データフローの理解の手助けをするために、処理内容が分かるような関数の名前を付けています。
おそらくですが、処理内容を把握しようと思った場合、最悪`function`の部分が全て欠如していても、なんとなく何をしようとしているのか分かりますよね？(圧)
つまり、理解するという面で言えば、その分のコードを省略したのとほぼ同じですよね？(圧)

これぐらい簡易的なコードであれば恩恵はあまり感じないと思いますが、コードが肥大化していくほど恩恵を感じていくのではないかと思っています。

## 3-11. 変数、関数の命名規則

# 4. 後書き

## コードの品質の重要性

## CSSについて