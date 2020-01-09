# How do you like PostgreSQL12

## 2020-01-25 <br> 第28回 中国地方DB勉強会 in 岡山

#### 日本PostgreSQLユーザー会 中国地方支部長 <br> 高橋　一騎

---

# 注意事項

- スライドは公開しています。
- 質問とかあれば、セッション中に遠慮なく聞いてください！
- 聞くのはちょっと・・って人がいれば #ChugokuDB へお願い致します！

---

# おしながき

1. 自己紹介
2. PostgreSQLとは
3. PostgreSQL12の新機能
4. PostgreSQL13に期待される機能

---

# 1. 自己紹介

- 高橋　一騎
- 岡山在住
- 株式会社オミカレ
Webアプリケーションエンジニア
- 日本PostgreSQLユーザー会
中国支部長

![right fit](twitter.png)

---

# 株式会社オミカレ

- 全国の婚活パーティー
約30,000件を掲載してる
ポータルサイト.
- 出会いが0をZeroにする
をVisionに日々活動してます
- エンジニア（インフラの人）を
募集してます！

![right fit](app_omicale.png)

---

# 2. PostgreSQLとは

---

# 2. PostgreSQLとは

- `PostgreSQL（ぽすとぐれすきゅーえる）`

代表的なオープンソースのRDBMSの1つ
元々、大学の研究用に開発された研究用のRDBMSの
Ingresが元となっている。
研究用に利用されていた事もあり、
理論に忠実に開発されている。

---

# 2. PostgreSQLとは

- SQL標準に準拠した構文のサポート
- 複数種類のIndex種別のサポート
- 豊富なデータ型のサポート

---

## SQL標準に準拠した構文のサポート

- SQL標準
メーカーやシステムによって仕様が大きく異なってた。
標準化を求める声が多く集まった。
- ISO/IEC 9075 として国際標準化された。
- Check制約とかWindow関数とか。
（8.0までMySQLにはなかったんや・・）

---

## 複数種類のIndex種別のサポート

- B-Tree Index
- Hash Index
- GiST Index, SP-GiST Index , GIN Index (全文検索)
- BRIN Index

---

## 豊富なデータ型のサポート

- 列列挙（Enum）
- 幾何データ型
  - 座標点、直線、円
- IPアドレス型 (IPv4, IPv6)
  - 192.168.10.5, 192.168.11.1 どっちが大きいか比較出来る.
- JSON型 / 配列型 / 範囲型

---

# バージョンアップ履歴

- 約1年サイクルでメジャーバージョンアップが行われている。
- 9.6 までは x.y.z というバージョン形式だった
  - x.y : メジャーバージョン
  - z   : マイナーバージョン
- 10より x.z というバージョン形式になった。
  - x : メジャーバージョン
  - z : マイナーバージョン

---

# [fit]という所で・・・ <br>PostgreSQL12 今日のお話です！！！

---

# 3. PostgreSQL12の新機能

---

# 3. PostgreSQL12の新機能

- 生成列のサポート
- JSON PATH のサポート
- pg_dumpの強化
- テーブル・パーティショニングが更に強化
- Access Method

---

# 生成列(GENERATED列)のサポート

---

# 生成列のサポート

- 生成列は、テーブルに対して計算結果を元にした列を定義する.

```sql
CREATE TABLE member
(
 id SERIAL NOT NULL CONSTRAINT member_pkey PRIMARY KEY,
 first_name VARCHAR(50) NOT NULL,
 last_name VARCHAR(50) NOT NULL,
 full_name VARCHAR(50) NOT NULL GENERATED ALWAYS AS ( first_name || ' ' || last_name) STORED
);
```

- `full_name`は `first_name` と `空白` と `last_name` を
文字列結合したものになる。

---

# 生成列

実際に動かしてみるとこんな感じ

```sql
chugokudb=# INSERT INTO member (first_name, last_name) VALUES ('高橋', '一騎');
INSERT 0 1
chugokudb=#
chugokudb=# SELECT * FROM member;
 id | first_name | last_name | full_name 
----+------------+-----------+-----------
  1 | 高橋       | 一騎      | 高橋 一騎
(1 row)

```

---

# [fit] 「お、やった！ これで誕生日から年齢を常に計算出来るやん！」

---
[.autoscale: true]

# 生成列

**生成列の制約**

- 不変な値を返す関数しか使えない
- その他の生成列の値を参照する事は出来ない
- 生成列はパーティションキーに指定する事は出来ない
- など

... age(timestamp) はimmutableな関数じゃない.
（そもそも、INSERT時に値を生成して実体としてデータを保持する仕組み.）

---

## つまり、ずっと ハタチ

---

[.autoscale: true]
# 生成列

- 書き換わるタイミングは INSERT/UPDATE の時。
- ちなみに豆知識として...
SELECTの度に計算してくれる仕組みも他のRDBMSにある
  - `仮想列` と呼ばれる
  - `GENERATED ALWAYS AS ( ) STORED` を
   `GENERATED ALWAYS AS ( ) VIRTUAL` と書けば仮想列（他RDBMSの場合）
  - 構文も似てるのでいつかは実装されそう（願望

---

