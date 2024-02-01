---
title: 「あなたのVueファイルは読みづらい」と文句を言われたくない② -- コンポーネント＆ディレクトリ設計編 --
tags:
  - Vue
  - JavaScript
  - TypeScript
  - 可読性
  - コード品質
private: true
updated_at: ''
id: ReadableVue2
organization_url_name: null
slide: false
ignorePublish: false
---



# 目次
[1. 前書き](#1-前書き)
[2. ディレクトリ構成](#2-ディレクトリ構成)
[3. 共通のデータや処理を分離して管理するディレクトリ](#3-共通のデータや処理を分離して管理するディレクトリ)
[4. 画面表示に関するディレクトリ](#4-画面表示に関するディレクトリ)
[5. 後書き](#5-後書き)

# 1. 前書き

この記事のテーマは、「Vueファイルの可読性を可能な限り上昇させる」です。

## 自己紹介

自称フルスタックエンジニアの岩田と申します。業務の経験は 2023/6~ なので、まだ１年にも満たないミーハーな人間です。自分の経験が他のエンジニアさんの参考になれば良いなと思いながら記事を書いていますので、もし良ければ他の記事もご覧ください！！

## 概要

## この記事の対象者

・Vue.jsのコーディングに慣れてきて、次のステップに進みたい方
・ある程度の規模感になってくると、途端にコードが読みづらくなってしまうと感じている方

あくまで一個人のコーディングスタイルなので参考程度にしてくださればと思っています。

# 2. ディレクトリ構成
簡単なページ構成をここで提示しておきます。

| パス | 画面の説明                                         |
|------------|----------------------------------------------|
|`*/users`|ユーザーの一覧画面 (検索機能付き)|
|`*/users/:id`|ユーザーの詳細 & 編集画面 (`:id`には任意のユーザーIDが入ります)|
|`*/blogs`|ブログ一覧画面 (検索機能付き)|

その上で、以下のようなディレクトリ構造にしました。
僕がディレクトリ構成を考える際、`パスとディレクトリ構成を合わせる`ということは強く意識をして作成しています。

```dir:ディレクトリ構成
src/
├── components/
│   ├── common/
│   │   ├── feedbacks/
│   │   ├── inputs/
│   │   └── wrappers/
│   └── domains/
│       ├── users/
│       │   ├── index/
│       │   │   ├── UsersSearchWindow.vue
│       │   │   └── UsersListItem.vue
│       │   └── show/
│       │       └── UserInfo.vue
│       └── blogs/
│           └── index/
│               ├── BlogsSearchWindow.vue
│               └── BlogsList.vue
├── constants/
│   ├── commonData.ts (jsも可)
│   ├── usersData.ts (jsも可)
│   └── blogsData.ts (jsも可)
├── hooks/
│   ├── users/
│   │   ├── getUsers.ts (jsも可)
│   │   └── getUserDetail.ts (jsも可)
│   └── blogs/
│       └── getBlogs.ts (jsも可)
├── pages/
│   ├── users/
│   │   ├── Index.vue
│   │   └── Show.vue
│   └── blogs/
│       └── Index.vue
└── types/  #TypeScriptを使用する場合
    ├── users.d.ts
    └── blogs.d.ts
```

次に各ディレクトリについて説明していきます。
# 3. 共通のデータや処理を分離して管理するディレクトリ
ここでは３つのディレクトリに焦点を当てて説明していこうと思います。簡単な説明を以下に載せておきます。
| ディレクトリ | 説明                                         |
|------------|----------------------------------------------|
|constants|プロジェクト全体で共通の定数をまとめておく|
|hooks|一定の処理(scriptファイルに書く)を分離しておく|
|types|プロジェクト全体で使い回したい型を定義しておく|

基本的にこれらのファイルは、.vueファイルのscriptタグ内に書いて同様の結果を得ることができるため**無くても問題ない**です。ただ、なにもかも.vueファイルに書いてしまうとファイルが肥大化して可読性が下がってしまうので、**敢えて分離**しています。

それでは、各ディレクトリについて詳しく説明していこうと思います。
## constants
全体で共通のデータをここで定義します。
今回のようなユーザーとブログのアプリケーションだと必要なさそうなので、フランチャイズ展開している飲食店を例にしてみると、

* 店舗一覧
* メニュー一覧

といったデータは基本的には不変ですし、全体で使用したくなりそうなものではあります。もちろんプロダクトの性質によって差はあります。
```js:*Data.ts
export const stores = [
    {
        id: 1,
        name: '東京本店',
            :
            :
    },
        :
        :
]
```
```vue:*.vue
<script setup>
import { stores } from '*Data';
</script>
```

## hooks
以下のようなケースで、このファイル内に分離してあげます
* API通信 (外部のDBとの通信など)
* カスタムイベントの実装
* フォームの状態管理 (Reactの場合)

例えば、FireBaseからデータを引っ張ってくる処理の場合、

```ts:hooks/users/getUsers.ts
import { ref, onMounted } from 'vue';
import axios from 'axios';

export const getUsers = () => {
    const users = ref([]);
    const isLoading = ref(false);
    const error = ref(null);

    const fetchUsers = async () => {
        isLoading.value = true;
        try {
            const response = await axios.get('ここにURL');
            users.value = response.data;
        } catch (err) {
            error.value = err;
        } finally {
            isLoading.value = false;
        }
    };

    onMounted(fetchUsers);

    return { users, isLoading, error };
};
```
```vue:pages/users/Index.vue
<script setup>
import { getUsers } from '/hooks/users/getUsers';

const { users, isLoading, error } = getUsers();
</script>
```
こんな形で記述することができ、ご覧の通り.vueファイル内の記述が非常にスッキリしました。何度もしつこいですが全てを`pages/users/Index.vue`に書いても同様の機能を実装できますが、コードの可読性という点においてどちらが良いのかは一目瞭然ですよね。

## types
```ts:types/users.d.ts
export interface Users {
    id: number;
    name: string;
    email: string;
        :
        :
}
        :
        :
```
```vue:pages/users/Index.vue
<script setup>
import { Users } from '/types/users';
</script>
```
# 4. 画面表示に関するディレクトリ
次に残りの２つのディレクトリに焦点を当てて説明していこうと思います。簡単な説明を以下に載せておきます。
| ディレクトリ | 説明                                         |
|------------|----------------------------------------------|
|pages|データの取り込みとルーティング先のプラットフォーム|
|components|画面に表示する要素の部品を作る|

それでは、各ディレクトリについて詳しく説明していこうと思います。
## pages
このディレクトリ内のファイルの役割は
* ルーティングの指定先になる
* データを集積する
* 子コンポーネントにデータを分配する

つまり、ルールとしては単純で、
* ルーティング先として指定できるファイルは pages ディレクトリにあるファイルのみ
* pages ディレクトリ配下の.vueファイルについて、templateタグ内では**templateタグ**と**子コンポーネント**のみを並べ、その他のHTMLは書かない。
```vue:pages/users/Index.vue (良い例)
<script setup>
import { UsersSearchWindow } from '/components/domains/users/UsersSearchWindow.vue'
import { UsersListItem } from '/components/domains/users/UsersListItem.vue'

const users = ~
</script>
<template>
    <UsersSearchWindow />
    <template v-for="user in users" :key="user.id">
        <UsersListItem :user="user" />
    </template>
</template>
```
```vue:pages/users/Index.vue (悪い例)
<script setup>
import { UsersSearchWindow } from '/components/domains/users/UsersSearchWindow.vue'
import { UsersListItem } from '/components/domains/users/UsersListItem.vue'

const users = ~
</script>
<template>
    <h1>タイトル</h1>
    <UsersSearchWindow />
    <div v-for="user in users" :key="user.id">
        <h3>ユーザー一覧</h3>
        <UsersListItem :user="user" />
    </div>
</template>
```
## components
ここには pages ディレクトリ内のファイルから呼び出されるUIの部品を作っていきます。２つのディレクトリがあるので、それぞれについて説明します。
| ディレクトリ | 説明                                         |
|------------|----------------------------------------------|
|common|全ページ共通で使うコンポーネントを置いておく|
|domains|ページごとのUIの部品を置いておく(domain配下は他のディレクトリと同じ設計)|

先ほどの`UsersListItem`を例にして作ってみると、
```vue:components/domains/users/UsersListItem.vue
<script setup>
const props = defineProps({
    user: Object
})
    :
    :
</script>
```
のような感じになります。

# 5. 後書き

