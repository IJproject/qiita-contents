---
title: LaravelのDB設計って今どうなってん？
tags:
  - Laravel
  - Database
  - スキーマ
private: false
updated_at: '2024-02-02T15:40:25+09:00'
id: 413e8e87bedb4b75225b
organization_url_name: null
slide: false
ignorePublish: false
---



# 目次
[1. マイグレーションファイル多すぎ](#1-マイグレーションファイル多すぎ)
[2. スキーマファイル作れるらしい](#2-スキーマファイル作れるらしい)
[3. これでやっと](#3-これでやっと)

# 1. マイグレーションファイル多すぎ
Laravelに限らず、バックエンド系フレームワークのマイグレーションファイルって多すぎますよね。僕のようにインフラ面が分からなすぎるために、ソースコードからDB設計を把握するしかない人間もいるんですよね。
Railsであれば`schema.rb`ファイルとかいう**神ファイル**があるので問題ないのですが、Laravelってそういうファイルがないから時々どころか相当な頻度で困ることがあるんです。何のカラムを持っているのか分からないからマイグレーションファイルを見にいってという労力を毎回使ってたんです。

# 2. スキーマファイル作れるらしい

これが当たり前だと思っていたのですが、ふと調べてみたのです。`「Laravel Schema」`でググってみました。すると、**スキーマ系のファイル作れるけど今まで何してたの**と言わんばかりにあるコマンドのことが大量に出てきたんです。それが、

```shell:スキーマを作成
php artisan schema:dump
```

これさえ実行してしまえば、こんなSchemaファイルができてしまうんです。

```sql:database/schema/*-schema.sql
  :
  :
  :
DROP TABLE IF EXISTS `members`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `members` (
    `id` char(26) COLLATE utf8mb4_unicode_ci NOT NULL,
    `image_id` char(26) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
    `name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL,
    `last_name` varchar(25) COLLATE utf8mb4_unicode_ci NOT NULL,
    `first_name` varchar(25) COLLATE utf8mb4_unicode_ci NOT NULL,
    `address` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
    `email` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
    `email_verified_at` timestamp NULL DEFAULT NULL,
    `password` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
    `created_at` timestamp NULL DEFAULT NULL,
    `updated_at` timestamp NULL DEFAULT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `members_email_unique` (`email`),
    KEY `members_image_id_foreign` (`image_id`),
    CONSTRAINT `members_image_id_foreign` FOREIGN KEY (`image_id`) REFERENCES `images` (`id`) ON DELETE SET NULL,
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
  :
  :
  :
```

# 3. これでやっと
長年(数ヶ月)の「Railsは`schema.rb`ファイルあって良いなあ」案件が解消されました。
また一段階、Laravelのことを好きになれた気がします。今日から沢山スキーマファイル作ろうと思います。
