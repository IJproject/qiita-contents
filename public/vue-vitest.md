---
title: Vue+Vitestでのフロントテスト ~Laravel,Vuetifyを添えて~
tags:
  - Vue
  - Vitest
  - テスト
  - Laravel
  - Vuetify
private: true
updated_at: null
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# 目次

[1. 前書き](#1-前書き)
[2. セットアップ](#2-セットアップ)
[3. テストの実装](#3-テストの実装)

# 1. 前書き



# 2. セットアップ

**今回は TypeScript で実装していますが、JavaScript でも全く同じ方法でセットアップすることができます。** JavaScript をお使いの方は、拡張子が`ts`になっているところを`js`に適宜変更してください。

## 2-1. Vitest のインストール

まず始めに、テストランナーライブラリの [Vitest](https://vitest.dev/guide/) のインストールをします。

```bash
npm install -D vitest
```

## 2-2. 設定

インストールが完了したら、テストが実行できるような環境を整えていきます。
まずは `package.json` に記述を加えて、テストコマンドを実行できるようにします。

```json:package.json
"scripts": {
    "dev": "vite",
    "build": "vite build",
    "test": "vitest"
},
```

`vite` を採用している場合、既に `vite.config.ts` があると思いますので、そちらにテスト用の記述を追加します。`vite.config.js` の場合も同様の記述です。
`include` の部分には読み込みたいテストファイルをワイルドカードで指定しておきます。テストファイルには `*.test.ts` という命名をするのですが、今回の設定では、`resources/js/Tests` 配下のテストファイルが読み込まれることになります。

```ts:vite.config.ts
/// <reference types="vitest" />
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.ts',
            refresh: true,
        }),
        vue({
            template: {
                transformAssetUrls: {
                    base: null,
                    includeAbsolute: false,
                },
            },
        }),
    ],
    test: {
        globals: true,
        include: ['resources/js/Tests/**/*.test.ts'],
    },
} as any);
```

## 検証

ここで一旦、テスト環境の構築ができているのか試してみましょう。テストファイルを作成して、以下のコードをそのままコピペしてください。
※ 現在の状態ではまだ Vue コンポーネントのテストはできません。

```ts:resources/js/Tests/sample.test.ts
import { describe, it, expect } from 'vitest';

// テスト対象の関数
function add(a: number, b: number): number {
    return a + b;
}

// テストケース
describe('add function', () => {
    it('adds two numbers', () => {
        const result = add(2, 3);
        expect(result).toBe(5);
    });

    it('adds negative numbers', () => {
        const result = add(-1, -1);
        expect(result).toBe(-2);
    });
});
```

コマンドを入力してテストを実行してみます。

```bash
npm run test
```

エラーが出ていなければ成功です。

## Vue-Test-Utils のインストール

次に Vue コンポーネントのテストをするために [Vue-Test-Utils](https://test-utils.vuejs.org/guide/) のインストールをします。

```bash
npm install @vue/test-utils jsdom --save-dev
```

`jsdom` は DOM エミュレーションライブラリで、テストの実行環境(DOM 環境)を構築するためのものです。
`jsdom` の代替として、`happy-dom` を使用することもできます。

これだけで Vue コンポーネントのテストをすることができるようになります。具体的なテストコードについては [テストの実装](#3-テストの実装) にお進みください。

## Laravel を使っている場合

Laravelを使用している場合、Ziggyに関する設定をしなければなりません。
詳しくは以下のリンクを参照してください。

https://laracasts.com/discuss/channels/inertia/vitest-inertiajs-errors-when-running-tests-on-pagecomponents

まずは

* `Ziggy` に関するモジュールのインストール
* 設定ファイルの生成

をします。

```bash
npm install ziggy-js
php artisan ziggy:generate  # 自動でZiggy.jsというファイルが生成されます
```

生成された `Ziggy.js` について、拡張子の変更はプロジェクトに合わせて行ってください。**ファイルの中身を変更することはしません。**Prettierなどでフォーマットを整えると見やすくなります。

お好みの位置に、セットアップ用のファイルを新規で作成して、`setupFiles` で読み込むようにしてください。今回は`/resources/js/Tests` 配下に `setup.ts` ファイルを生成しました。

```ts:vite.config.ts
/// <reference types="vitest" />
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';
import path from 'path';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.ts',
            refresh: true,
        }),
        vue({
            template: {
                transformAssetUrls: {
                    base: null,
                    includeAbsolute: false,
                },
            },
        })
    ],
    test: {
        globals: true,
        setupFiles: path.resolve(__dirname, 'resources/js/Tests/setup.ts'), // セットアップファイルの読み込み
        environment: 'jsdom', // happy-domを使ってもいい
        include: ['resources/js/Tests/**/*.test.ts'], // テスト対象のファイル群
    },
});
```

`setup.ts` ファイルには以下の記述をコピペします。
このコードでは、vueファイル内でルーティングの実装等で使用する `route` 関数を使用できるようにしています。

```ts:setup.ts
import { config } from '@vue/test-utils';
import route from 'ziggy-js';
import { Config, RouteParams } from 'ziggy-js';
import { Ziggy } from '../ziggy';

config.global.mocks.route = (name: string, params?: RouteParams<any>) => route(name, params, undefined, Ziggy as Config);
```

```js:setup.js
import { config } from '@vue/test-utils';
import route from 'ziggy-js';
import { Ziggy } from '../ziggy';

config.global.mocks.route = (name, params) => route(name, params, undefined, Ziggy);
```

## vuetify を使っている場合

`Vuetify` を使用している場合、テスト環境でDOMを構成する際に、Vuetifyコンポーネントが正しくレンダリングされるように設定しなければなりません。
詳しくは以下のリンクを参照してください。

https://ma-vericks.com/blog/nuxt3-vuetify-vitest/

```ts:vite.config.ts
/// <reference types="vitest" />
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import vuetify from 'vite-plugin-vuetify';
import path from 'path';

export default defineConfig({
    plugins: [
        vue({
            template: {
                transformAssetUrls: {
                    base: null,
                    includeAbsolute: false,
                },
            },
        }),
        vuetify()
    ],
    test: {
        globals: true,
        setupFiles: path.resolve(__dirname, '/resources/js/Tests/setup.ts'), // セットアップファイルの読み込み
        environment: 'jsdom',
        include: ['resources/js/Tests/**/*.test.ts'],　// テスト対象のファイル群
        server: {
            deps: {
                inline: ['vuetify'], // Vuetifyをインライン化して処理
            },
        }

    },
});
```

```ts:setup.ts
import { config } from '@vue/test-utils';
import { createVuetify } from 'vuetify'

const vuetify = createVuetify()

config.global.plugins = [vuetify];
```

# 3. テストの実装

ここからは実際にテストコードを記述していきます。

# 3-1. テストコードの基本構造

```ts
// テストするコンポーネントの
import CommoditiesList from '@/Components/Domains/Shopping/Index/CommoditiesList.vue';
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';

describe('Shopping/Index/CommoditiesList', () => {
    // コンポーネントをマウントし、propsに商品データを渡す
    const wrapper = mount(CommoditiesList, {
        props: { commodities: commodities },
    });

    it('テストケース１', () => {
        // テスト１をここに記述
    });

    it('テストケース２', () => {
        // テスト２をここに記述
    });
});
```

# 3-2. dsjak