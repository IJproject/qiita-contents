---
title: Laravelのテストコードを書いてみた
tags:
  - Laravel
  - PHPUnit
  - テスト
private: true
updated_at: null
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# 目次

[1. 前書き](#1-前書き)
[2. 知識の下準備](#2-知識の下準備)
[3. 実装](#3-実装)

# 1. 前書き

# 2. 知識の下準備

https://laravel.com/docs/10.x/testing#main-content

https://qiita.com/shindex/items/4f28f8e06ef2d10e8d2b

# 2-1. ディレクトリ構成



# 2-2. テストファイルの作成と実行

以下のコマンドを叩いて、テスト用のファイルを生成します。
**Feature**に作成する場合は１行目、**Unit**に作成する場合は２行目のコマンドを実行します。

```bash
php artisan make:test ファイル名
php artisan make:test ファイル名 --unit
```

次に以下のコマンドを叩くことで、テストを実行することができます。

```bash
php artisan test
```

Paratestというライブラリを使用すると、テストを並列実行させ、テスト時間を短くすることができます。

```bash
composer require brianium/paratest --dev
php artisan test --parallel  # --parallelオプションを付与すると並列実行
```

# 2-3. テストコードの書き方

# 3. 実装

# 3-1. サンプル
