---
title: 「あなたのVueファイルは読みづらい」と文句を言われたくない② -- コンポーネント＆ディレクトリ設計編 --
tags:
  - JavaScript
  - TypeScript
  - Vue.js
  - 可読性
  - コード品質
private: false
updated_at: '2024-02-05T14:59:12+09:00'
id: eece70d9c75558a621db
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

現在の記事は②になります。①はこちらのリンクからどうぞ

https://qiita.com/IJproject/items/46342f529320652a8fad

# 1. 前書き

この記事のテーマは、「Vue ファイルの可読性を可能な限り上昇させる」です。**コンポーネント設計**や**ファイル分割**の手法を通して、**エンジニアに優しいコーディング**をすることを目的としています。あくまで一意見としてなので、参考程度にお願いします。
もし、**この記事を参考にしたコーディングをして同僚に文句を言われたら、是非その旨をこの記事のコメントに記入してください。** ひとつひとつのコメントに対して涙目になりながら中身を改善していきます。

## 自己紹介

自称フルスタックエンジニアの岩田と申します。業務の経験は 2023/6~ なので、まだ１年にも満たないミーハーな人間です。自分の経験が他のエンジニアさんの参考になれば良いなと思いながら記事を書いていますので、もし良ければ他の記事もご覧ください！！

## 概要

コーディングスタイルの統一は、チーム開発において１つの大事な要素になっていると思います。

例えばですが、一つのページにつきコンポーネントファイルが一つだとすると、コードの行数が少なくても 100 行前後、多い時だと 500 行近くに達することがあります。これに関しては Vue のエンジニアに限らず React のエンジニアでも起こりうることだと思います。そんなコードのファイルに当たった時、我々のエンジニアとしてのエネルギーは即座に失われてしまうことでしょう。そんなことがないように、**適切なファイル分割**をし、可読性の高いファイル構成にする必要があります。

この記事では、僕自身が考案した**コンポーネント設計の考え方**や、**ファイル分割の考え方**について紹介していこうと思います。

## この記事の対象者

・Vue.js のコーディングに慣れてきて、次のステップに進みたい方
・ある程度の規模感になってくると、途端にコードが読みづらくなってしまうと感じている方

あくまで一個人のコーディングスタイルなので参考程度にしてくださればと思っています。

# 2. ディレクトリ構成

今回考えたい簡単なページ構成をここで提示しておきます。

| パス          | 画面の説明                                                        |
| ------------- | ----------------------------------------------------------------- |
| `*/users`     | ユーザーの一覧画面 (検索機能付き)                                 |
| `*/users/:id` | ユーザーの詳細 & 編集画面 (`:id`には任意のユーザー ID が入ります) |
| `*/blogs`     | ブログ一覧画面 (検索機能付き)                                     |

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
| ディレクトリ | 説明 |
|------------|----------------------------------------------|
|constants|**プロジェクト全体で共通**の定数をまとめておく|
|hooks|一定の処理(script ファイルに書く)を**分離**しておく|
|types|プロジェクト全体で使い回したい型を定義しておく|

基本的にこれらのファイルは、.vue ファイルの script タグ内に書いて同様の結果を得ることができるため**別ファイルに分離しなくても問題ない**です。ただ、なにもかも.vue ファイルに書いてしまうとファイルが肥大化して可読性が下がってしまうので、**敢えて分離**しています。

それでは、各ディレクトリについて詳しく説明していこうと思います。

## 3-1. constants

全体で共通のデータをここで定義します。
今回のようなユーザーとブログのアプリケーションだと必要なさそうなので、フランチャイズ展開している飲食店を例にしてみると、

- 店舗一覧
- メニュー一覧

といったデータは基本的には不変ですし、全体で使用したくなりそうなものではあります。
※ もちろんプロダクトの性質によって差はあります。

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

## 3-2. hooks

以下のようなケースで、このファイル内に分離してあげます

- API 通信 (外部の DB との通信など)
- カスタムイベントの実装
- フォームの状態管理 (React の場合)

例えば、FireBase からデータを引っ張ってくる処理の場合、

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

こんな形で記述することができ、ご覧の通り.vue ファイル内の記述が非常にスッキリしました。何度もしつこいですが全てを`pages/users/Index.vue`に書いても同様の機能を実装できますが、コードの可読性という点においてどちらが良いのかは一目瞭然ですよね。

## 3-3. types

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
| ディレクトリ | 説明 |
|------------|----------------------------------------------|
|pages|データの取り込みとルーティング先のプラットフォーム|
|components|画面に表示する要素の部品を作る|

それでは、各ディレクトリについて詳しく説明していこうと思います。

## 4-1. pages

このディレクトリ内のファイルの役割は

- ルーティングの指定先になる
- データを集積する
- 子コンポーネントにデータを分配する

つまり、ルールとしては単純で、

- ルーティング先として指定できるファイルは pages ディレクトリにあるファイルのみ
- pages ディレクトリ配下の.vue ファイルでは、template タグ内で**template タグ**と**子コンポーネント**のみを並べ、その他の HTML は書かない。

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

あとこれは個人的なルールなのですが、pages配下の.vueファイルには、一定の名前しかつけないようにしています。これは大した理由はないのですが、考え方としては、

* ディレクトリ構成でファイルのジャンル
* .vueファイルの名前で役割

を示すようにしています。

| ファイル名 | 説明 |
|------------|----------------------------------------------|
|Index.vue|一覧表示画面|
|Show.vue|詳細表示画面|
|Create.vue|新規作成画面|
|Edit.vue|編集画面|

なので、例えば「ユーザーのブログランキング」についてのページを作るということであれば、`pages/users/Ranking.vue`ではなく、`pages/users/ranking/Index.vue`として作成するということになります。ファイル名やディレクトリ構成になるべく多くの情報を詰め込もうという魂胆です。

## 4-2. components

ここには pages ディレクトリ内のファイルから呼び出される UI の部品を作っていきます。２つのディレクトリがあるので、それぞれについて説明します。
| ディレクトリ | 説明 |
|------------|----------------------------------------------|
|common|全ページ共通で使うコンポーネントを置いておく|
|domains|特定のページ内の UI の部品を置いておく(domain 配下は他のディレクトリと同じ設計)|

### common

ここには、**さまざまなファイルで使用したいコンポーネント**を作成します。

* feedbacks
  * inputsやwrappersに該当しないもの
  * スナックバー(トースター) など
* inputs
  * フォームに関するもの
  * フォームの部品のコンポーネント など
* wrappers
  * 囲いになるもの(`<slot />`を使ってコンポーネントの中身を入れていく)
  * ダイアログ(モーダル)の骨格 など

### domains

ここでは、**特定のページの特定のパーツとして使用したいコンポーネント**を作成します。
基本的に、１ページの中を機能ごとに分割し、ここに書く機能ごとのコンポーネントファイルを作成していく形になります。

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

## 5-1. コードの品質の重要性

チーム開発においてコードの品質を高く保つためには、

コーディングルールを徹底する
可読性を上げるような工夫をする
この２点が最重要だと思います。正直この辺りが疎かになると、たとえ２人の開発でもごちゃごちゃになります。さらに人が増えるとなると尚更です。こういったルールを身体に染み込ませることにもある程度の工数は掛かってしまいますが、その後の保守のことを考えるとプラスの要素しかないですよね。
少しだけコーディングルールを見直してみてはいかがでしょうか。

コーディングを快適にするためのルール作りにこの記事の内容が参考になっていることを切に願っています。
