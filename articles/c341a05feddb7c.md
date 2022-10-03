---
title: "Moodle Data manipulation API 覚書"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Moodle", "SQL"]
published: true
---

# Moodle Data manipulation API 覚書

# あらまし

以前 Moodle プラグイン開発に関わることがあったため、公式ドキュメントを読みながら自分なりに試したメモを残しておく。
あとで楽に見返せるように日本語翻訳も残しておきます。

公式ドキュメントはこちら

https://docs.moodle.org/dev/Data_manipulation_API

## データ操作 API

> This page describes the functions available to access data in the Moodle database. You should exclusively use these functions in order to retrieve or modify database content because these functions provide a high level of abstraction and guarantee that your database manipulation will work against different RDBMSes.

このページでは、Moodle の データベース内のデータにアクセスするための可用性のある関数について述べる。
データベースコンテンツの検索や変更のために、排他的にこれらの関数を用いるべきである。
理由としては、これらの関数は高度な抽象化を提供し、異なる RDBMS に対してデータ操作を保証するため。

> Where possible, tricks and examples will be documented here in order to make developers' lives a bit easier. Of course, feel free to clarify, complete and add more information to this documentation. It will be welcome, absolutely!

可能であれば、手法や例を開発者の敷居を下げるためにドキュメント化する。
このドキュメントでは、自由に情報を明確にし、完成させ、追加してください。もちろん大歓迎である!

メモ
ここで言う exclusively は単に「独自関数書くな && 別の OR マッパとか使うな」と言うこと。

## 目次

- 1 General concepts (一般的なコンセプト)

  - 1.1 DB object (DB オブジェクト)
  - 1.2 Table prefix (テーブルの接頭辞)
  - 1.3 Conditions (コンディション)
  - 1.4 Placeholders (プレースホルダー)
  - 1.5 Strictness (厳密性)

- 2 Getting a single record (シングルレコードの取得)

  - 2.1 get_record
  - 2.2 get_record_select
  - 2.3 get_record_sql

- 3 Getting a hashed array of records (ハッシュ化されたレコード配列の取得)

  - 3.1 get_records
  - 3.2 get_records_select
  - 3.3 get_records_sql
  - 3.4 get_records_list

- 4 Getting data as key/value pairs in an associative array (連想配列のキー/値ペアとしてデータを取得)

  - 4.1 get_records_menu
  - 4.2 get_records_select_menu
  - 4.3 get_records_sql_menu

- 5 Counting records that match the given criteria (与えられたクライテリアにマッチしたレコードの数え上げ)

  - 5.1 count_records
  - 5.2 count_records_select
  - 5.3 count_records_sql

- 6 Checking if a given record exists (レコードの存在チェック)

  - 6.1 record_exists
  - 6.2 record_exists_select
  - 6.3 record_exists_sql

- 7 Getting a particular field value from one record (1 レコードから特定のフィールドの値取得)

  - 7.1 get_field
  - 7.2 get_field_select
  - 7.3 get_field_sql

- 8 Getting field values from multiple records (複数レコードからフィールドの値取得)

  - 8.1 get_fieldset_select
  - 8.2 get_fieldset_sql

- 9 Setting a field value (フィールドの値設定)

  - 9.1 set_field
  - 9.2 set_field_select

- 10 Deleting records (レコードの削除)

  - 10.1 delete_records
  - 10.2 delete_records_select

- 11 Inserting records (レコードの挿入)
  - 11.1 insert_record
  - 11.2 insert_records
  - 11.3 insert_record_raw
- 12 Updating records (レコードの更新)

  - 12.1 update_record

- 13 Executing a custom query (カスタムクエリの実行)

  - 13.1 execute

- 14 Using recordsets (レコードセットを使用する)

  - 14.1 get_recordset
  - 14.2 get_recordset_select
  - 14.3 get_recordset_sql
  - 14.4 get_recordset_list

- 15 Delegated transactions (デリゲートされた(委任) トランザクション)

  - 15.1 Example

- 16 Cross-DB compatibility (DB 間の互換性)

  - 16.1 sql_bitand
  - 16.2 sql_bitnot
  - 16.3 sql_bitor
  - 16.4 sql_bitxor
  - 16.5 sql_null_from_clause
  - 16.6 sql_ceil
  - 16.7 sql_equal
  - 16.8 sql_like
  - 16.9 sql_like_escape
  - 16.10 sql_length
  - 16.11 sql_modulo
  - 16.12 sql_position
  - 16.13 sql_substr
  - 16.14 sql_cast_char2int
  - 16.15 sql_cast_char2real
  - 16.16 sql_compare_text
  - 16.17 sql_order_by_text
  - 16.18 sql_concat
  - 16.19 sql_group_concat
  - 16.20 sql_concat_join
  - 16.21 sql_fullname
  - 16.22 sql_isempty
  - 16.23 sql_isnotempty
  - 16.24 get_in_or_equal
  - 16.25 sql_regex_supported
  - 16.26 sql_regex
  - 16.27 sql_intersect

- 17 Debugging (デバッグ)

  - 17.1 set_debug

- 18 Special cases (特殊ケース)

  - 18.1 get_course
  - 18.2 get_courses

- 19 See also (関連項目)

## 1 General concepts (一般的なコンセプト)

### 1.1 DB object (DB オブジェクト)

> The data manipulation API is exposed via public methods of the $DB object.

データ操作 API は Public メソッドのオブジェクト $DB を通してあらわになる。

> Moodle core takes care of setting up the connection to the database according to values specified in the main config.php file.

Moodle コアは、 main の config.php ファイルで指定された値に従って、データベースへの接続の設定を行う。

> The $DB global object is an instance of the moodle_database class. It is instantiated automatically during the bootstrap setup, i.e. as a part of including the main config.php file.

このグローバルオブジェクトの $DB は　 moodle_databese class のインスタンスである。
これは起動、セットアップ時に自動的にインスタンティネイトされる。
つまり、 main の config.php ファイルをインクルードする際にインスタンス化される。

> The DB object is available in the global scope right after including the config.php file:

この DB オブジェクトは以下の config.php file のインクルード後にグローバルスコープ内で適用される。

```php
<?php

require(__DIR__.'/../../../config.php');
// You can access the database via the $DB method calls here.

```

> To make the DB object available in your local scope, such as within a function:

ローカルスコープ内に適用される DB オブジェクトを作るためには、以下のように関数内部に宣言する。

```php
<?php

defined('MOODLE_INTERNAL') || die();

function my_function_making_use_of_database() {
    global $DB;

    // You can access the database via the $DB method calls here.
}
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

if (isset($_GET['f']) && isset($_GET['t'])) {
    $f = $_GET['f'];
    $t = $_GET['t'];
    echo $do = "SELECT ".$f." FROM {".$t."};";
    $user = $DB->get_records_sql($do);
    echo "<p>".json_encode($user)."</p>";
    consoleLog($user);
} else {
    echo "none";
    consoleLog("none");
}

```

ガバガバな書き方 ↑
つまり、データ操作 API は Public メソッドのオブジェクト $DB を通して全てにアクセスできるので注意が必要。

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    $user = $DB->get_record('user', ['firstname' => 'hoge', 'lastname' => 'fuga']);
    output($user->{firstname}.':'.$user->{email});
    consoleLog($user->{firstname}.':'.$user->{email});
};

getdata();
```

![](https://storage.googleapis.com/zenn-user-upload/e5a35a5169e2-20221003.png)

こうなるので、次のように宣言してあげる。

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $user = $DB->get_record('user', ['firstname' => 'hoge', 'lastname' => 'fuga']);
    output($user->{firstname}.':'.$user->{email});
    consoleLog($user->{firstname}.':'.$user->{email});
};

getdata();
```

### 1.2 Table prefix (テーブルの接頭辞)

> Most Moodle installations use a prefix for all the database tables, such as mdl\_. This prefix is NOT to be used in the code in the code itself.
> All the $table parameters in the functions are meant to be the table name without prefixes:

ほとんどの 「 Moodle インストレーション 」 では 全てのデータベーステーブルのためのプレフィックスを使うが、`mdl_.` のようなプレフィックスは　コード内、コード自身が使うことは無い。

```php
$user = $DB->get_record('user', ['id' => '1']);
```

↑ つまり、　 user テーブルにアクセスするときは 'user' と書けば良いと言うこと。

> In custom SQL queries, table names must be enclosed between curly braces. They will be then automatically converted to the real prefixed table name. There is no need to access $CFG->prefix

カスタム SQL クエリでは、テーブル名は必ず　`{}` で囲まなければならない。
これらは実際のプレフィックスドネームに自動コンバートされるため。
また、　`$CFG->prefix` のようにアクセスする必要もない。

```php
$user = $DB->get_record_sql('SELECT COUNT(*) FROM {user} WHERE deleted = 1 OR suspended = 1;');
```

### 1.3 Conditions (コンディション)

> All the `$conditions` parameters in the functions are arrays of ` fieldname=>fieldvalue` elements.

関数内の全ての $conditions パラメータは `fieldname=>fieldvalue`　(フィールド/バリュー)要素の配列。

> They all must be fulfilled - i.e. logical AND is used to populate the actual WHERE statement.

これら全てを満たさなければならない。
つまり、論理和を使用して実際の WHERE 句 が作成される。

```php
$user = $DB->get_record('user', ['firstname' => 'Martin', 'lastname' => 'Dougiamas']);
```

### 1.4 Placeholders (プレースホルダー)

> All the $params parameters in the functions are arrays of values used to fill placeholders in SQL statements.

関数内のすべての$params パラメータは、SQL 文のプレースホルダに値を代入するための配列。

> Placeholders help to avoid problems with SQL-injection and/or invalid quotes in SQL queries. They facilitate secure and cross-db compatible code.

プレースホルダを使用すると、SQL インジェクションや SQL クエリ内の無効な クウォート(引用符) に関する問題を回避する助けになる。これにより、安全でデータベース間に互換性のあるコードが容易になる。

> Two types of placeholders are supported - question marks (SQL_PARAMS_QM) and named placeholders (SQL_PARAMS_NAMED).

クエスチョンマーク (SQL_PARAMS_QM) と 名前付きプレースホルダー (SQL_PARAMS_NAMED) の 2 種類のプレースホルダーがサポートされてる。

> Named params must be unique even if the value passed is the same. If you need to pass the same value multiple times, you need to have multiple distinct named parameters.

名前付きパラメーターは、渡された値が同じ場合でもユニークである必要がある。同じ値を複数回渡す必要がある場合は、複数の個別の名前付きパラメーターが必要。
※メモ これは割と独特なので注意しよう。

```php
// Example of using question mark placeholders.
$DB->get_record_sql('SELECT * FROM {user} WHERE firstname = ? AND lastname = ?',
    ['Martin', 'Dougiamas']);

// Example of using named placeholders.
$DB->get_record_sql('SELECT * FROM {user} WHERE firstname = :firstname AND lastname = :lastname',
    ['firstname' => 'Martin', 'lastname' => 'Dougiamas']);
```

### 1.5 Strictness (厳密性)

> Some methods accept the $strictness parameter affecting the method behaviour. Supported modes are specified using the constants:

いくつかのメソッドは、メソッドの動作に影響する `$strictness` パラメータを受け付ける。
サポートされているモードは、次の `constrants` (定数) を使用して指定する。

- MUST_EXIST

  - In this mode, the requested record must exist and must be unique. An exception will be thrown if no record is found or multiple matching records are found.
  - このモードは、リクエストされたレコードが必ず存在してユニークであること。レコードが見つからない場合、または複数の一致するレコードが見つかった場合は、例外が投げられる。

- IGNORE_MISSING

  - In this mode, a missing record is not an error. False boolean is returned if the requested record is not found. If more records are found, a debugging message is displayed.
  - このモードでは、欠損レコードはエラーにならない。要求されたレコードが `not found` の場合に、False のブール値が返される。さらにレコードが見つかった場合は、デバッグメッセージが表示される。

- IGNORE_MULTIPLE
  - This is not a recommended mode. The function will silently ignore multiple records found and will return just the first one of them.
  - これは推奨されないモード。この関数は警告なく複数のレコードを無視し、最初のレコードだけを返すヤバいやつ。

## 2 Getting a single record (シングルレコードの取得)

### 2.1 get_record

> Return a single database record as an object where all the given conditions are met.

指定されたすべての条件を満たす一つのデータベースレコードをオブジェクトとして返す。

```php
$DB->get_record($table, array $conditions, $fields='*', $strictness=IGNORE_MISSING)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $conditions=array('firstname' => 'hoge');
    $fields='*';
    $strictness=IGNORE_MISSING;
    $example = $DB->get_record($table,$conditions, $fields, $strictness);
    output($example);
    consoleLog($example);
};

getdata();
```

### 2.2 get_record_select

> Return a single database record as an object where the given conditions are used in the WHERE clause.

WHERE 句で与えられたコンディションを使用して、一つのデータベースレコードをオブジェクトとして返す。

```php
$DB->get_record_select($table, $select, array $params=null, $fields='*', $strictness=IGNORE_MISSING)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata1() {
    global $DB;
    $table='user';
    $select='firstname=? and lastname=?';
    $params=array('hoge','fuga');
    $fields='*';
    $strictness=IGNORE_MISSING;
    $example = $DB->get_record_select($table, $select, $params, $fields,$strictness);
    output($example);
    consoleLog($example);
};

function getdata2() {
    global $DB;
    $table='user';
    $select='firstname=:firstname1 and lastname=:lastname1';
    $params=array('firstname1'=>'hoge','lastname1'=>'fuga');
    $fields='*';
    $strictness=IGNORE_MISSING;
    $example = $DB->get_record_select($table, $select, $params, $fields,$strictness);
    output($example);
    consoleLog($example);
};

getdata1();
getdata2();

```

### 2.3 get_record_sql

> Return a single database record as an object using a custom SELECT query.

カスタム SELECT クエリを使用して、一つのデータベースレコードをオブジェクトとして返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $sql='SELECT * FROM {user} WHERE firstname=:firstname AND lastname=:lastname;';
    $params=array('firstname'=>'hoge', 'lastname'=>'fuga');
    $strictness=IGNORE_MISSING;
    $example = $DB->get_record_sql($sql, $params, $strictness);
    output($example);
    consoleLog($example);
};

getdata();
```

## 3 Getting a hashed array of records (ハッシュ化されたレコード配列の取得)

> Each of the following methods return an array of objects. The array is indexed by the first column of the fields returned by the query. To assure consistency, it is a good practice to ensure that your query include an "id column" as the first field. When designing custom tables, make id their first column and primary key.

次の各メソッドは、オブジェクトの配列を返す。
配列のインデックスは、クエリによって返されるフィールドの最初の列。
一貫性を確保するには、クエリの最初のフィールドに "id column" を含める。
カスタムテーブルをデザインする場合は、id を最初の列および主キーにする。

### 3.1 get_records

> Return a list of records as an array of objects where all the given conditions are met.

指定されたすべての条件を満たす、`list` のデータベースレコードをオブジェクトの `array` として返す。

`$limitfrom`

- 指定した数値から、レコードのサブセットを開始する。

`$limitnum`

- サブセットの数を指定。（ `$limitform` と一緒に設定する必要あり ）

## サブセット数指定の例

```sql
mysql> SELECT id, email FROM mdl_user ORDER BY id LIMIT 1,2;
```

```php
function getdata() {
    global $DB;
    $sql='select id, email from {user} order by id';
    $params=null;
    $limitfrom=1;
    $limitnum=2;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $conditions=array();
    $sort='username';
    $fields='*';
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records($table, $conditions, $sort, $fields, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();

```

### 3.2 get_records_select

> Return a list of records as an array of objects where the given conditions are used in the WHERE clause.

WHERE 句で与えられたコンディションを使用して、`list` のデータベースレコードをオブジェクトの `array` として返す。

`$fields` はコンマで区切られた、返されるフィールドの `list` (省略可能、デフォルトではすべてのフィールドが返される。)。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $select='firstname=:firstname and lastname=:lastname';
    $params=array('firstname'=>'hoge','lastname'=>'fuga');
    $sort='username';
    $fields='id, email';
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_select($table,$select,$params, $sort, $fields, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

### 3.3 get_records_sql

> Return a list of records as an array of objects using a custom SELECT query.

カスタム SELECT クエリを使用して、`list` のデータベースレコードをオブジェクトの `array` として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $sql='SELECT id, email FROM {user}';
    $params=null;
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

### 3.4 get_records_list

> Return a list of records as an array of objects where the given field matches one of the possible values.

指定されたフィールドが取り得る値の 1 つと一致するレコードの `list` をオブジェクトの `array` として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $field='firstname';
    $values=array('hoge');
    $sort='';
    $fields='id, username, email';
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_list($table, $field, $values, $sort, $fields, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

## 4 Getting data as key/value pairs in an associative array (連想配列のキー/値ペアとしてデータを取得)

### 4.1 get_records_menu

> Return the first two columns from a list of records as an associative array where all the given conditions are met.

指定した条件がすべて満たされる際に、レコードのリストから最初の 2 列を連想配列として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $conditions=array('firstname'=>'hoge');
    $sort='';
    $fields='*';
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_menu($table, $conditions, $sort, $fields, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

### 4.2 get_records_select_menu

> Return the first two columns from a list of records as an associative array where the given conditions are used in the WHERE clause.

WHERE 句で指定した条件を使用して、レコードのリストから最初の 2 つのカラムを連想配列として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $select='firstname=:firstname';
    $params=array('firstname'=>'hoge');
    $sort='';
    $fields='*';
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_select_menu($table, $select, $params, $sort, $fields, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

### 4.3 get_records_sql_menu

> Return the first two columns from a number of records as an associative array using a custom SELECT query.

カスタム SELECT クエリを使用して、レコードのリストから最初の 2 つのカラムを連想配列として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $sql='SELECT * FROM {user} WHERE firstname=:firstname';
    $params=array('firstname'=>'hoge');
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql_menu($sql, $params, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

## 5 Counting records that match the given criteria (与えられたクライテリアにマッチしたレコードの数え上げ)

### 5.1 count_records

> Count the records in a table where all the given conditions are met.

指定した条件がすべて満たされる際に、テーブル内のレコードを数え上げる。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $conditions=array('firstname'=>'hoge');
    $example = $DB->count_records($table, $conditions);
    output($example);
    consoleLog($example);
};

getdata();
```

### 5.2 count_records_select

> Count the records in a table where the given conditions are used in the WHERE clause.

WHERE 句で指定した条件を使用して、テーブル内のレコードを数え上げる。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $select='firstname=:firstname';
    $params=array('firstname'=>'hoge');
    $countitem="COUNT('id')";
    $example = $DB->count_records_select($table, $select, $params, $countitem);
    output($example);
    consoleLog($example);
};

getdata();
```

### 5.3 count_records_sql

> Counting the records using a custom SELECT COUNT(...) query.

カスタム SELECT COUNT (...) クエリを使用してレコードを数え上げる。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $sql='SELECT COUNT(id) FROM {user} WHERE firstname=:firstname';
    $params=array('firstname'=>'hoge');
    $example = $DB->count_records_sql($sql, $params);
    output($example);
    consoleLog($example);
};

getdata();
```

## 6 Checking if a given record exists (レコードの存在チェック)

### 6.1 record_exists

> Test whether a record exists in a table where all the given conditions are met.

指定されたすべての条件を満たすテーブルにレコードが存在するかどうかをテスト。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $conditions=array('firstname'=>'hoge');
    $example = $DB->record_exists($table, $conditions);
    output($example);
    consoleLog($example);
};

getdata();
```

### 6.2 record_exists_select

> Test whether any records exists in a table where the given conditions are used in the WHERE clause.

指定された条件が WHERE 句で使用されているテーブルにレコードが存在するかどうかをテスト。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $select='firstname=:firstname';
    $params=array('firstname'=>'hoge');
    $example = $DB->record_exists_select($table, $select, $params);
    output($example);
    consoleLog($example);
};

getdata();
```

### 6.3 record_exists_sql

> Test whether the given SELECT query would return any record.

与えられた SELECT クエリがレコードを返すかどうかをテスト。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $sql='SELECT * FROM {user} WHERE firstname=:firstname';
    $params=array('firstname'=>'hoge');
    $example = $DB->record_exists_sql($sql, $params);
    output($example);
    consoleLog($example);
};

getdata();
```

## 7 Getting a particular field value from one record (1 レコードから特定のフィールドの値取得)

### 7.1 get_field

> Get a single field value from a table record where all the given conditions are met.

指定されたすべての条件を満たすテーブルレコードから一つのフィールド値を取得する。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $return='email';
    $conditions=array('firstname'=>'hoge');
    $strictness=IGNORE_MISSING;
    $example = $DB->get_field($table, $return, $conditions, $strictness);
    output($example);
    consoleLog($example);
};

getdata();
```

### 7.2 get_field_select

> Get a single field value from a table record where the given conditions are used in the WHERE clause.

WHERE 句で指定された条件が使用されているテーブルレコードから、一つのフィールド値を取得。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $return='email';
    $select='firstname=:firstname';
    $params=array('firstname'=>'hoge');
    $strictness=IGNORE_MISSING;
    $example = $DB->get_field_select($table, $return, $select, $params, $strictness);
    output($example);
    consoleLog($example);
};

getdata();
```

### 7.3 get_field_sql

> Get a single field value (first field) using a custom SELECT query.

カスタム SELECT クエリを使用して、一つのフィールド値(最初のフィールド)を取得。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $sql='SELECT * FROM {user} WHERE firstname=:firstname';
    $params=array('firstname'=>'admin');
    $strictness=IGNORE_MISSING;
    $example = $DB->get_field_sql($sql, $params, $strictness);
    output($example);
    consoleLog($example);
};

getdata();
```

## 8 Getting field values from multiple records (複数レコードからフィールドの値取得)

### 8.1 get_fieldset_select

> Return values of the given field as an array where the given conditions are used in the WHERE clause.

WHERE 句で指定された条件を使用して、与えられたフィールドの配列として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $return='username';
    $select='firstname=:firstname';
    $params=array('firstname'=>'admin');
    $example = $DB->get_fieldset_select($table, $return, $select, $params);
    output($example);
    consoleLog($example);
};

getdata();
```

### 8.2 get_fieldset_sql

> Return values of the first column as an array using a custom SELECT field FROM ... query.

カスタム 「SELECT field FROM ...」 クエリ を使用して、最初の列の値を配列として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $sql='SELECT * FROM {user} WHERE firstname=:firstname';
    $params=array('firstname'=>'admin');
    $example = $DB->get_fieldset_sql($sql, $params);
    output($example);
    consoleLog($example);
};

getdata();
```

## 9 Setting a field value (フィールドの値設定)

### 9.1 set_field

> Set a single field in every record where all the given conditions are met.

指定されたすべての条件を満たす各レコードに 1 つのフィールドの値をセットする。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;

    $table='user';
    $newfield='lastname';
    $newvalue='test';
    $conditions=array('firstname'=>'hoge');
    $example=$DB->set_field($table, $newfield, $newvalue, $conditions);
    output($example);
    consoleLog($example);

    $conditions=array('firstname'=>'hoge');
    $fields='id,username,firstname,lastname,email';
    $strictness=IGNORE_MISSING;
    $valid=$DB->get_record($table, $conditions, $fields, $strictness);
    output($valid);
    consoleLog($valid);
};

getdata();
```

### 9.2 set_field_select

> Set a single field in every table record where the given conditions are used in the WHERE clause.

WHERE 句で指定された条件を使用して、各テーブルレコードに 1 つのフィールドの値をセットする。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;

    $table='user';
    $newfield='lastname';
    $newvalue='test';
    $select='firstname=:firstname';
    $params=array('firstname'=>'hoge');

    $example=$DB->set_field_select($table, $newfield, $newvalue, $select, $params);
    output($example);
    consoleLog($example);

    $conditions=array('firstname'=>'hoge');
    $fields='id,username,firstname,lastname,email';
    $strictness=IGNORE_MISSING;
    $valid=$DB->get_record($table, $conditions, $fields, $strictness);
    output($valid);
    consoleLog($valid);
};

getdata();
```

## 10 Deleting records (レコードの削除)

### 10.1 delete_records

> Delete records from the table where all the given conditions are met.

指定したすべての条件を満たすテーブルからレコードを削除。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;

    $table='dummy';
    $conditions=array('name'=>'dummy0');
    $example=$DB->delete_records($table, $conditions);
    output($example);
    consoleLog($example);
};

getdata();

```

### 10.2 delete_records_select

> Delete records from the table where the given conditions are used in the WHERE clause.

WHERE 句で指定した条件を使用して、テーブルからレコードを削除。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;

    $table='dummy';
    $select='name=:name';
    $params=array('name'=>'dummy0');
    $example=$DB->delete_records_select($table, $select,  $params);
    output($example);
    consoleLog($example);
};

getdata();
```

## 11 Inserting records (レコードの挿入)

### 11.1 insert_record

> Insert the given data object into the table and return the "id" of the newly created record.

与えられたテーブル内のデータオブジェクトを挿入し、新たに作成されたレコードの "id" を返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='dummy';

    $dataobject = new stdClass();
    $dataobject->name='dummy0';

    $returnid=true;
    $bulk=false;
    $example=$DB->insert_record($table, $dataobject, $returnid, $bulk);
    output($example);
    consoleLog($example);
};

getdata();
```

### 11.2 insert_records

> Insert multiple records into the table as fast as possible. Records are inserted in the given order, but the operation is not atomic. Use transactions if necessary.

可能な限り早く複数のレコードをテーブルに挿入する。
レコードは指定された順序で挿入されるが、操作はアトミックではない。
必要に応じてトランザクションを使用すること。

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='dummy';

    $obj0 = new stdClass();
    $obj0->name='dummy0';
    $obj1 = new stdClass();
    $obj1->name='dummy1';
    $obj2 = new stdClass();
    $obj2->name='dummy2';

    $dataobjects = array($obj0, $obj1, $obj2);

    $returnid=true;
    $bulk=false;
    $example=$DB->insert_records($table, $dataobjects);
    output($example);
    consoleLog($example);
};

getdata();

```

### 11.3 insert_record_raw

> For rare cases when you also need to specify the ID of the record to be inserted.

レアなケースとして、挿入するレコードの ID を指定する必要がある場合があるがその際に使用する。

## 12 Updating records

### 12.1 update_record

> Update a record in the table. The data object must have the property "id" set.

テーブル内のレコードを更新する。
データオブジェクトには、"id" プロパティが設定されている必要がある。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $table='user';
    $dataobject = new stdClass();
    $dataobject->id=1; //guest account
    $dataobject->lastname='updatedDummy';
    $bulk=false;
    $example=$DB->update_record($table, $dataobject, $bulk);

    output($dataobject);
    output($example);
    consoleLog($example);
};

getdata();
```

## 13 Executing a custom query (カスタムクエリの実行)

### 13.1 execute

> If you need to perform a complex update using arbitrary SQL, you can use the low level "execute" method. Only use this when no specialised method exists.

任意の SQL を使用して、複雑な更新を実行する必要がある場合は、ローレベルの "execute" メソッドを使用できる。

特定のメソッドが存在しない場合にのみ使用する。

> Do NOT use this to make changes in database structure, use database_manager methods instead!

これを使用してデータベース構造を変更しないで、代わりに database_manager メソッドを使用すること!

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);

    $sql='SELECT * FROM {user} WHERE firstname=:firstname';
    $params=array('firstname' => 'hoge');
    $example=$DB->execute($sql, $params);
    output($example);
    consoleLog($example);
};

getdata();

```

## 14 Using recordsets (レコードセットを使用する)

> If the number of records to be retrieved from DB is high, the 'get_records_xxx() functions above are far from optimal, because they load all the records into the memory via the returned array.

もし DB から回収されたレコード数が大量にある場合は、'get_records_xxx()' 関数は最適とは言い難いものである。
なぜなら、返された配列を介してすべてのレコードをメモリにロードするため。

> Under those circumstances, it is highly recommended to use these get_recordset_xxx() functions instead. They return an iterator to iterate over all the found records and save a lot of memory.

これらの状況下において、特に推奨されるのは、'get_recordset_xxx()' を代わりに使用すること。
これらは、見つかったすべてのレコードを反復処理する反復子を返し、メモリをとても節約できる。

> It is absolutely important to not forget to close the returned recordset iterator after using it. This is to free up a lot of resources in the RDBMS.

返された recordset イテレータを使用した後は、それを閉じること、これは本当に重要なので忘れてはならない。
これは RDBMS の大量のリソースを解放するためである。

> A general way to iterate over records using the get_recordset_xxx() functions:

一般的な方法として、レコードを反復処理するために 'get_recordset_xxx()' 関数を使用する。

```php
$rs = $DB->get_recordset(....);
foreach ($rs as $record) {
    // Do whatever you want with this record
}
$rs->close();
```

> Unlike get_record functions, you cannot check if `$rs` == true or !empty(`$rs`) to determine if any records were found. Instead, if you need to, you can use:

`get_record` 関数とは異なり、レコードが見つかったかどうかを判断するために `if $rs==true` または `if !empty($rs)` をチェックすることはできない。

代わりに、必要に応じて以下を使用できる。

```php
if ($rs->valid()) {
    // The recordset contains some records.
}
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;

    $table='user';
    $conditions=array();
    $sort='';
    $fields='*';
    $limitfrom=0;
    $limitnum=0;
    $rs = $DB->get_recordset($table, $conditions, $sort, $fields, $limitfrom, $limitnum);
    if ($rs->valid()) {
        output("first OK");
    }
    foreach ($rs as $record) {
        if ($rs->valid()) {
            output("each OK");
            output($record);
            consoleLog($record);
        }
    }
    $rs->close();
};

getdata();

```

### 14.1 get_recordset

> Return a list of records as a moodle_recordset where all the given conditions are met.

指定されたすべての条件を満たす、レコードのリストを moodle_recordset として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;

    $table='user';
    $conditions=array();
    $sort='';
    $fields='*';
    $limitfrom=0;
    $limitnum=0;
    $rs = $DB->get_recordset($table, $conditions, $sort, $fields, $limitfrom, $limitnum);
    if ($rs->valid()) {
        output($record);
        consoleLog($record);
    }
    $rs->close();
};

getdata();
```

### 14.2 get_recordset_select

> Return a list of records as a moodle_recordset where the given conditions are used in the WHERE clause.

WHERE 句で指定された条件を使用して、レコードのリストを moodle_recordset として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;

    $table='user';
    $select='firstname=:firstname';
    $params=array('firstname' => 'hoge');
    $sort='';
    $fields='*';
    $limitfrom=0;
    $limitnum=0;

    $rs = $DB->get_recordset_select($table, $select, $params, $sort, $fields, $limitfrom, $limitnum);
    if ($rs->valid()) {
        output("first OK");
    }
    foreach ($rs as $record) {
        if ($rs->valid()) {
            output("each OK");
            output($record);
            consoleLog($record);
        }
    }
    $rs->close();
};

getdata();
```

### 14.3 get_recordset_sql

> Return a list of records as a moodle_recordset using a custom SELECT query.

カスタム SELECT クエリを使用して、レコードのリストを moodle_recordset として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;

    $sql='SELECT * FROM {user} WHERE firstname=:firstname';
    $params=array('firstname' => 'hoge');
    $limitfrom=0;
    $limitnum=0;

    $rs = $DB->get_recordset_sql($sql, $params, $limitfrom, $limitnum);
    if ($rs->valid()) {
        output("first OK");
    }
    foreach ($rs as $record) {
        if ($rs->valid()) {
            output("each OK");
            output($record);
            consoleLog($record);
        }
    }
    $rs->close();
};

getdata();
```

### 14.4 get_recordset_list

> Return a list of records as a moodle_recordset where the given field matches one of the possible values.

指定したフィールドが、指定可能な値のいずれかと一致するレコードのリストを moodle_recordset として返す。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;

    $table='user';
    $field='firstname';
    $values=array('hoge');
    $sort='';
    $fields='id, username, email';
    $limitfrom=0;
    $limitnum=0;

    $rs = $DB->get_recordset_list($table, $field, $values, $sort, $fields, $limitfrom, $limitnum);
    if ($rs->valid()) {
        output("first OK");
    }
    foreach ($rs as $record) {
        if ($rs->valid()) {
            output("each OK");
            output($record);
            consoleLog($record);
        }
    }
    $rs->close();
};

getdata();

```

## 15 Delegated transactions (デリゲートされた(委任) トランザクション)

> Please note some databases do not support transactions (such as the MyISAM MySQL database engine), however all server administrators are strongly encouraged to migrate to databases that support transactions (such as the InnoDB MySQL database engine).

一部のデータベース (MyISAM MySQL データベースエンジンなど) はトランザクションをサポートしていないが、すべてのサーバー管理者はトランザクションをサポートするデータベース (InnoDB MySQL データベースエンジンなど) に移行することを強くお勧めする。

> Previous versions supported only one level of transaction. Since Moodle 2.0, the DML layer emulates delegated transactions that allow nesting of transactions.

以前のバージョンでは 1 つのレベルのトランザクションのみをサポートしていたが、Moodle 2.0 以来、この　 DML レイヤーはデリゲートされたトランザクションをエミュレートしている。つまり、ネストされたトランザクションが許されている。

> Some subsystems (such as messaging) do not support transactions because it is not possible to rollback in external systems.

いくつかのサブシステム(メッセージングなど) はトランザクションをサポートしていない、理由としては、外部システム内のロールバックが不可能であるから。

> A transaction is started by:

トランザクションを以下のように開始できる。

```php
$transaction = $DB->start_delegated_transaction();
```

> and finished by:

また、以下のように終了できる。

```php
$transaction->allow_commit();
```

> Usually a transaction is rolled back when an exception is thrown:

通常、トランザクションは例外が投げられたときにロールバックされる。

```php
$transaction->rollback($ex);
```

> which must be used very carefully because it might break compatibility with databases that do not support transactions. Transactions cannot be used as part of expected code flow; they can be used only as an emergency protection of data consistency.

これらは、とても慎重に使わなければならない、なぜなら、これはトランザクションがサポートされていないデータベースの互換性を破壊してしまうかもしれないので。
トランザクションは、期待されるコードフローの一部としては使用できない;
これらは、データ整合性のための緊急保護としてのみ使用できる。

> See more details in DB layer 2.0 delegated transactions or MDL-20625.

詳細は DB layer 2.0 delegated transactions もしくは　 MDL-20625 を見よ。

### 15.1 Example

```php
global $DB;
try {
     $transaction = $DB->start_delegated_transaction();
     $DB->insert_record('foo', $object);
     $DB->insert_record('bar', $otherobject);

     // Assuming the both inserts work, we get to the following line.
     $transaction->allow_commit();

} catch(Exception $e) {
     $transaction->rollback($e);
}
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    try {
        $transaction = $DB->start_delegated_transaction();
        $table='user';

        $dataobject = new stdClass();
        $dataobject->username='test';
        $returnid=true;
        $bulk=false;
        // increment id
        $example=$DB->insert_record($table, $dataobject, $returnid, $bulk);
        output($example);
        consoleLog($example);
    } catch(Exception $e) {
        // rollback
        $transaction->rollback($e);
    }
};

getdata();

```

この例では id はインクリメントされるが、実際には rollback が走っているので、データは作成されず、ユーザーを作成してみると、その分の id が飛ばされているのを確認できる。

- 1 画面から id 14 の test 　ユーザーを作成

- 2 検証用の上記スクリプトを 4 回ほど読み込んでみる

- 3 画面から id 19 の test 　ユーザーが作成されていて、id 15、16、17、18 のレコードがロールバックされたことを確認

![](https://storage.googleapis.com/zenn-user-upload/ff4bc469d990-20221003.png)

## 16 Cross-DB compatibility (DB 間の互換性)

> Moodle supports several SQL servers (MySQL, PostgreSQL, MS-SQL and Oracle). Each of them have some specific syntax in certain cases.

Moodle はいくつかの SQL サーバ(MySQL、 PostgreSQL、 MS-SQL、 および Oracle)をサポートしている。
特定の場合において、それぞれ固有の構文がある。

> In order to achieve cross-db compatibility of the code, following functions must be used to generate the fragments of the query valid for the actual SQL server.

コードのデータベース互換性を実現するには、次の関数を使用して、実際の SQL サーバーに対して有効なクエリの断片(一部分)を生成する必要がある。

### 16.1 sql_bitand

> Return the SQL text to be used in order to perform a bitwise AND operation between 2 integers.

2 つの整数間のビット単位の AND 演算を実行するために使用する SQL 文を返す。

```php
$DB->sql_bitand($int1, $int2)
```

## ちょっとやってみよう

### 例題

両方のビットが 1 である場合に 1 を埋める。

- 0011 and 0101 => 0001

十進数 １〜３ を 二進数に基数変換する。

- 1 -> 0001
- 2 -> 0010
- 3 -> 0011

以下のレコードを事前に準備し、 AND 演算を用いて SELECT する。

### クエリ

```sql
CREATE TABLE mdl_dummy (flag int DEFAULT 0, name VARCHAR(50));
INSERT INTO mdl_dummy VALUES (1,'group1-1');
INSERT INTO mdl_dummy VALUES (2,'group2-1');
INSERT INTO mdl_dummy VALUES (3,'group1@2-1');
INSERT INTO mdl_dummy VALUES (1,'group1-2');
INSERT INTO mdl_dummy VALUES (2,'group2-2');
INSERT INTO mdl_dummy VALUES (3,'group1@2-2');

SELECT flag, name FROM mdl_dummy WHERE flag=(flag & 1);
```

### コード

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $int1='flag';
    $int2=1;
    $params=array();
    $sql='SELECT name FROM {dummy} WHERE flag='.$DB->sql_bitand($int1, $int2);
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/d9ad714b230e-20221003.png)

![](https://storage.googleapis.com/zenn-user-upload/ba6b3edb043c-20221003.png)

### 16.2 sql_bitnot

> Return the SQL text to be used in order to perform a bitwise NOT operation on the given integer.

与えられた整数に対してビット単位の NOT 演算を行うために使用される SQL 文を返す。

```php
$DB->sql_bitnot($int1)
```

## ちょっとやってみよう

NOT を用いて SELECT する。

### クエリ

```sql
mysql> SELECT ~(1);
```

### コード

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $int1=1;
    $sql='SELECT '.$DB->sql_bitnot($int1);
    $params=array();
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/821b7a1040a4-20221003.png)
![](https://storage.googleapis.com/zenn-user-upload/637d5a48dd89-20221003.png)

- メモ

```sql
0000000000000000000000000000000000000000000000000000000000000001

SELECT BIN(18446744073709551614);
1111111111111111111111111111111111111111111111111111111111111110
```

### 16.3 sql_bitor

> Return the SQL text to be used in order to perform a bitwise OR operation between 2 integers.

2 つの整数間のビット単位の OR 演算に使用する SQL 文を返す。

```php
$DB->sql_bitor($int1, $int2)
```

## ちょっとやってみよう

### 例題

片方のビットが 1 である場合に 1 を埋める。

- 0011 or 0101 => 0111

十進数 １〜３ を 二進数に基数変換する。

- 1 -> 0001
- 2 -> 0010
- 3 -> 0011

以下のレコードを事前に準備し、 OR 演算を用いて SELECT する。

### クエリ

```sql
CREATE TABLE mdl_dummy (flag int DEFAULT 0, name VARCHAR(50));
INSERT INTO mdl_dummy VALUES (1,'group1-1');
INSERT INTO mdl_dummy VALUES (2,'group2-1');
INSERT INTO mdl_dummy VALUES (3,'group1@2-1');
INSERT INTO mdl_dummy VALUES (1,'group1-2');
INSERT INTO mdl_dummy VALUES (2,'group2-2');
INSERT INTO mdl_dummy VALUES (3,'group1@2-2');

SELECT flag, name FROM mdl_dummy WHERE flag=(flag | 1);
```

### コード

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $int1='flag';
    $int2=1;
    $sql='SELECT name FROM {dummy} WHERE flag='.$DB->sql_bitor($int1, $int2);
    $params=null;
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/76441108fd00-20221003.png)

![](https://storage.googleapis.com/zenn-user-upload/9fbda0bd4d53-20221003.png)

### 16.4 sql_bitxor

> Return the SQL text to be used in order to perform a bitwise XOR operation between 2 integers.

2 つの整数間のビット単位の XOR 演算に使用する SQL 文を返す。

```php
$DB->sql_bitxor($int1, $int2)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $int1='flag';
    $int2=1;
    $sql='SELECT name FROM {dummy} WHERE'.$DB->sql_bitxor($int1, $int2);
    $params=array();
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/07f4f777ad1f-20221003.png)

![](https://storage.googleapis.com/zenn-user-upload/6a213b72c278-20221003.png)

### 16.5 sql_null_from_clause

> Return an empty FROM clause required by some DBs in all SELECT statements.

すべての SELECT 文の一部の DB で必要な空の FROM 句を返す。

```php
$DB->sql_null_from_clause()
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $sql='SELECT 1 AS id '.$DB->sql_null_from_clause();
    $params=null;
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/f54b68e65a3f-20221003.png)

### 16.6 sql_ceil

> Return the correct CEIL expression applied to the given fieldname.

与えられた fieldname に適用された正しい CEIL 式を返す。

```php
$DB->sql_ceil($fieldname)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $fieldname=':flag';
    $sql='SELECT flag, name FROM {dummy} WHERE flag ='.$DB->sql_ceil($fieldname);
    $params=array('flag'=>'1.2');
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/40bd560e5778-20221003.png)

### 16.7 sql_equal

> Return the query fragment to perform cross-db varchar comparisons when case-sensitiveness is important.

大文字小文字を区別することが重要な場合に、DB 間の varchar 比較を実行するクエリフラグメントを返す。

```php
$DB->sql_equal($fieldname, $param, $casesensitive = true, $accentsensitive = true, $notequal = false)
```

## ちょっとやってみよう

- 小文字

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $fieldname='name';
    $param=':name';
    $params=array('name'=>'group1-1');
    $casesensitive = true;
    $accentsensitive = true;
    $notequal = false;
    $sql='SELECT flag, name FROM {dummy} WHERE '.$DB->sql_equal($fieldname, $param, $casesensitive, $accentsensitive, $notequal);
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();

```

![](https://storage.googleapis.com/zenn-user-upload/d3d009b15d6e-20221003.png)

- 大文字

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $fieldname='name';
    $param=':name';
    $params=array('name'=>'GROUP1-1');
    $casesensitive = true;
    $accentsensitive = true;
    $notequal = false;
    $sql='SELECT flag, name FROM {dummy} WHERE '.$DB->sql_equal($fieldname, $param, $casesensitive, $accentsensitive, $notequal);
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/76e4c40706b5-20221003.png)

### 16.8 sql_like

> Return the query fragment to perform the LIKE comparison.

LIKE 比較を実行するクエリーフラグメントを返す。

```php
$DB->sql_like($fieldname, $param, $casesensitive = true, $accentsensitive = true, $notlike = false, $escapechar = ' \\ ')
```

> Example: Searching for records partially matching the given hard-coded literal.

例:指定されたハードコードされたリテラルに部分的に一致するレコードを検索する。

```php
$DB->get_records_sql('SELECT id,fullname FROM {course} WHERE '.$DB->sql_like('idnumber', ':idnum'), ['idnum' => 'DEMO-%']);
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $sql='SELECT * FROM {dummy} WHERE '.$DB->sql_like('name', ':name');
    $params=array('name'=>'group%');
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/549e1325d8fd-20221003.png)

> See below if you need to compare with a value submitted by the user.

ユーザーが送信した値と比較する必要がある場合は、以下を参照。

### 16.9 sql_like_escape

> Escape the value submitted by the user so that it can be used for partial comparison and the special characters like '\_' or '%' behave as literal characters, not wildcards.

部分的な比較に使用できるように、ユーザーが送信した値をエスケープする。
また、 '\_' や '%' などの特殊文字は、ワイルドカードではなくリテラル文字として動作する。

```php
$DB->sql_like_escape($text, $escapechar = '\\')
```

> Example: If you need to perform a partial comparison with a value that has been submitted by the user.

例:ユーザーによって送信された値と部分比較を実行する必要がある場合。

```php
$search = required_param('search', PARAM_RAW);
$DB->get_records_sql('SELECT id,fullname FROM {course} WHERE '.$DB->sql_like('fullname', ':fullname'), ['fullname' => '%'.$DB->sql_like_escape($search).'%']);
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $search=required_param('search', PARAM_RAW);
    $flagment=$DB->sql_like('name', ':name');
    $sql='SELECT * FROM {dummy} WHERE'.$flagment;
    $params=array('name' => '%'.$DB->sql_like_escape($search).'%');
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/a706b5e99e94-20221003.png)

### 16.10 sql_length

> Return the query fragment to be used to calculate the length of the expression in characters.

長さを文字単位で計算するために使用されるクエリフラグメントを返す。

```php
$DB->sql_length($fieldname)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $field='(select name from {dummy} where name=:name limit 1)';
    $flagment=$DB->sql_length($field);
    $sql='select '.$flagment.$DB->sql_null_from_clause();
    $params=array('name' => 'group1-1');
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/854c207bea10-20221003.png)

### 16.11 sql_modulo

> Return the query fragment to be used to calculate the remainder after division.

除算後の剰余を計算するために使用するクエリフラグメントを返す。

```php
$DB->sql_modulo($int1, $int2)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $flagment=$DB->sql_modulo(11, 3);
    $sql = "SELECT ".$flagment." AS modulo ".$DB->sql_null_from_clause();
    $params=array();
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();

```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/d55a68e6f71d-20221003.png)

### 16.12 sql_position

> Return the query fragment for searching a string for the location of a substring. If both needle and haystack use placeholders, you must use named placeholders.

文字列内で部分文字列の位置を検索するためのクエリフラグメントを返す。
`needle` と `haystack` の両方でプレースホルダを使用する場合は、名前付きプレースホルダを使用する必要がある。

```php
$DB->sql_position($needle, $haystack)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $flagment=$DB->sql_position("'1-1'", '(SELECT name FROM {dummy} WHERE name=:name)');
    $sql ="SELECT ".$flagment.$DB->sql_null_from_clause();
    $params=array('name'=>'group1-1');
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/023a358e4bf4-20221003.png)

### 16.13 sql_substr

> Return the query fragment for extracting a substring from the given expression.

与えられた式から部分文字列を取り出すためのクエリフラグメントを返す。

```php
$DB->sql_substr($expr, $start, $length=false)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $sql='SELECT '.$DB->sql_substr("'hhoge'", 2, 4);
    $params=array();
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/4013ec8c92db-20221003.png)

### 16.14 sql_cast_char2int

> Return the query fragment to cast a CHAR column to INTEGER

CHAR カラムを INTEGER にキャストするためのクエリフラグメントを返す。

```php
$DB->sql_cast_char2int($fieldname, $text=false)
```

## ちょっとやってみよう

ORDER BY で flag をソートする。

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $sql = "SELECT * FROM {dummy} ORDER BY ".$DB->sql_cast_char2int('name');
    $params=array();
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/5ab73d8301cc-20221003.png)

### 16.15 sql_cast_char2real

> Return the query fragment to cast a CHAR column to REAL (float) number

CHAR 列を REAL (float) 数値にキャストするためのクエリフラグメントを返す。

```php
$DB->sql_cast_char2real($fieldname, $text=false)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $sql = "SELECT * FROM {dummy} ORDER BY ".$DB->sql_cast_char2real('name');
    $params=array();
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/55e3c66b7699-20221003.png)

### 16.16 sql_compare_text

> Return the query fragment to be used when comparing a TEXT (clob) column with a given string or a VARCHAR field (some RDBMs do not allow for direct comparison).

TEXT (clob) カラムを指定された文字列または VARCHAR フィールドと比較するときに使用されるクエリフラグメントを返す (一部の RDBMS は直接の比較を許可しない)

```php
$DB->sql_compare_text($fieldname, $numchars=32)
```

> Example:

- 例

```php
$todogroups = $DB->get_records_sql('SELECT id FROM {group} WHERE ' . $DB->sql_compare_text('description') . ' = ' . $DB->sql_compare_text(':description'), ['description' => 'TODO']);
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $sql='SELECT * FROM {dummy} WHERE ' . $DB->sql_compare_text('name') . ' = ' . $DB->sql_compare_text(':name');
    $params=array('name'=>'group1-1');
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();

```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/b39c6386ad7b-20221003.png)

### 16.17 sql_order_by_text

> Return the query fragment to be used to get records ordered by a TEXT (clob) column. Note this affects the performance badly and should be avoided if possible.

TEXT (clob) カラムで順序付けされたレコードを取得するために使用されるクエリフラグメントを返す。
これはパフォーマンスに悪影響を与えるため、可能であれば避けるべき。

```php
$DB->sql_order_by_text($fieldname, $numchars=32)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $sql="SELECT * FROM {dummy} ORDER BY ".$DB->sql_order_by_text('name');
    $params=array();
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/6d8a98e7932e-20221003.png)

### 16.18 sql_concat

> Return the query fragment to concatenate all given paremeters into one string.

与えられたパラメータを 1 つの文字列に連結するためのクエリフラグメントを返す。

```php
$DB->sql_concat(...)
```

> There is a gotcha if you are trying to concat fields which may be null which result in the entire result being null:

null の可能性があるフィールドを連結しようとすると、結果全体が null になるという落とし穴がある。

```php
$DB->sql_concat('requiredfield', 'optionalfield'); // BAD!
```

> You must cast or coalesce every nullable argument eg:

null 許容の引数はすべてキャストまたは結合する必要がある。

```php
$DB->sql_concat('requiredfield', "COALESCE(optionalfield, '')"); // Good.
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $sql="SELECT ".$DB->sql_concat("?", "?", "?")." AS fullname ". $DB->sql_null_from_clause();
    $params=array('Hello','World','!');
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/cb04ffb59620-20221003.png)

### 16.19 sql_group_concat

> Return SQL for performing group concatenation on given field/expression.

指定された `field/expression` (フィールド/式) でグループ連結を実行するための SQL を返す。

```php
$DB->sql_group_concat(string $field, string $separator = ', ', string $sort = '')
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $fieldsql=$DB->sql_group_concat('name', ', ', 'flag');
    $sql = 'SELECT '.$fieldsql.'FROM {dummy}';
    $params=array();
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/051ea9ac5775-20221003.png)

### 16.20 sql_concat_join

> Return the query fragment to concatenate all given elements into one string using the given separator.

指定された区切り文字を使用して、指定されたすべての要素を 1 つの文字列に連結するクエリフラグメントを返す。

```php
$DB->sql_concat_join($separator="' '", $elements=array())
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $separator="' '";
    $elements=array('flag','name');
    $sql = 'SELECT '.$DB->sql_concat_join($separator, $elements).'AS dummylist FROM {dummy} WHERE name=:name';
    $params=array('name'=>'group1-1');
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/918ead8d8c5e-20221003.png)

### 16.21 sql_fullname

> Return the query fragment to concatenate the given `$firstname` and `$lastname`

指定した `$firstname` と `$lastname` を連結したクエリフラグメントを返す。

```php
$DB->sql_fullname($first='firstname', $last='lastname')
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $sql='SELECT '.$DB->sql_fullname(':first', ':last').' AS fullname '.$DB->sql_null_from_clause();
    $params=array(
        'first'=>'hello',
        'last'=>'world'
    );
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/846bc80ac664-20221003.png)

### 16.22 sql_isempty

> Return the query fragment to check if the field is empty

フィールドが空であるかどうかを確認するクエリフラグメントを返す。

```php
$DB->sql_isempty($tablename, $fieldname, $nullablefield, $textfield)
```

### 16.23 sql_isnotempty

> Return the query fragment to check if the field is not empty

フィールドが空でないかどうかをチェックするためのクエリフラグメントを返す。

```php
$DB->sql_isnotempty($tablename, $fieldname, $nullablefield, $textfield)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $table='dummy';
    $field='name';
    $nullablefield=true;
    $textfield=true;
    $sql='SELECT * FROM {dummy} WHERE '.$DB->sql_isempty($table, $field, $nullablefield, $textfield);
    $params=array();
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/60606a3ec0b4-20221003.png)

![](https://storage.googleapis.com/zenn-user-upload/fb0f2cd32fcd-20221003.png)

### 16.24 get_in_or_equal

> Return the query fragment to check if a value is IN the given list of items (with a fallback to plain equal comparison if there is just one item)

値が指定された項目リストに含まれているかどうかをチェックするクエリフラグメントを返します (項目が 1 つだけの場合、通常の等価比較にフォールバックする)

```php
$DB->get_in_or_equal($items, $type=SQL_PARAMS_QM, $prefix='param', $equal=true, $onemptyitems=false)
```

> Example:

- 例

```php
$statuses = ['todo', 'open', 'inprogress', 'intesting'];
list($insql, $inparams) = $DB->get_in_or_equal($statuses);
$sql = "SELECT * FROM {bugtracker_issues} WHERE status $insql";
$bugs = $DB->get_records_sql($sql, $inparams);
```

> An example using named params:

名前付きパラメータの使用例。

```php
...
list($insql, $params) = $DB->get_in_or_equal($contexts, SQL_PARAMS_NAMED, 'ctx');
$contextsql = "AND rc.contextid $insql";
...
```

## ちょっとやってみよう

- ノーマル

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $statuses=['4.5', '5.5'];
    list($insql, $params)=$DB->get_in_or_equal($statuses);
    $sql="SELECT * FROM {dummy} WHERE name $insql";
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();

```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/34b3082b0566-20221003.png)

- 名前付きパラメータ

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $contexts=array('group1-1','1');
    list($insql, $params)=$DB->get_in_or_equal($contexts, SQL_PARAMS_NAMED, 'ctx');
    $contextsql="name $insql";
    $sql="SELECT * FROM {dummy} WHERE $contextsql";
    $limitfrom=0;
    $limitnum=0;
    $example=$DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($sql);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/5b2651d646f4-20221003.png)

### 16.25 sql_regex_supported

> Does the current database driver support regex syntax when searching?

現在のデータベースドライバは、検索時に正規表現構文をサポートしていますか?

```php
$DB->sql_regex_supported()
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    if($DB->sql_regex_supported()){
        $msg='regex_supported is true';
        output($msg);
        consoleLog($msg);
    }
};

getdata();
```

### 16.26 sql_regex

> Return the query fragment to perform a regex search.

正規表現検索を実行するためのクエリフラグメントを返す。

```php
$DB->sql_regex($positivematch = true, $casesensitive = false)
```

> Example: Searching for Page module instances containing links.

例:リンクを含むページモジュールインスタンスの検索。

```php
if ($DB->sql_regex_supported()) {
    $select = 'content ' . $DB->sql_regex() . ' :pattern';
    $params = ['pattern' => "(src|data)\ *=\ *[\\\"\']https?://"]
} else {
    $select = $DB->sql_like('content', ':pattern', false);
    $params = ['pattern' => '%=%http%://%'];
}

$pages = $DB->get_records_select('page', $select, $params, 'course', 'id, course, name');
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    if($DB->sql_regex_supported()){
        $table='dummy';
        $select='name '.$DB->sql_regex().' :pattern';
        $params=array('pattern' => '........');
        $sort='name';
        $fields='flag, name';
        $limitfrom=0;
        $limitnum=0;
        $example = $DB->get_records_select($table, $select, $params, $sort, $fields, $limitfrom, $limitnum);
        output($example);
        consoleLog($example);
    }else{
        $msg='not support regex';
        output($msg);
        consoleLog($msg);
    }
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/797efd33f998-20221003.png)

### 16.27 sql_intersect

> Return the query fragment that allows to find intersection of two or more queries

2 つ以上のクエリの共通部分を見つけることができるクエリフラグメントを返す。

```php
$DB->sql_intersect($selects, $fields)
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);
    $sql1='select name from mdl_dummy';
    $sql2='select name from mdl_dummy2';
    $sql = $DB->sql_intersect(array($sql1,$sql2), 'name') . ' ORDER BY name';
    $params=array();
    $limitfrom=0;
    $limitnum=0;
    $example = $DB->get_records_sql($sql, $params, $limitfrom, $limitnum);
    output($example);
    consoleLog($example);
};

getdata();
```

- 確認

![](https://storage.googleapis.com/zenn-user-upload/7b5c342f16af-20221003.png)

共通部分を取得できている。

![](https://storage.googleapis.com/zenn-user-upload/e9c3d36a6463-20221003.png)

## 17 Debugging (デバッグ)

### 17.1 set_debug

> You can enable a debugging mode to make $DB output the SQL of every executed query, along with some timing information. This can be useful when debugging your code.

デバッグモードを有効にすることによって、いくつかの時間情報の他に、毎回実行される SQL クエリの $DB の出力できる。
自分のコードをデバッグするときに便利。

> Obviously, all such calls should be removed before code is submitted for integration.

言うまでもなく、コードを統合のために提出する前に、このような呼び出しはすべて削除すべき。

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    global $DB;
    $DB->set_debug(true);

    $sql='SELECT * FROM {user} WHERE firstname=:firstname';
    $params=array('firstname' => 'hoge');
    $example=$DB->execute($sql, $params);
    output($example);
    consoleLog($example);
};

getdata();
```

## 18 Special cases (特殊ケース)

### 18.1 get_course

> From Moodle 2.5.1 onwards, you should use the get_course function instead of using get_record('course', ...) if you want to get a course record based on its ID, especially if there is a significant possibility that the course being retrieved is either the current course for the page, or the site course.

Moodle 2.5.1 以降では、ID に基づいてコースレコードを取得する場合、特に取得するコースがページの現在のコースまたはサイトコースのいずれかである可能性が高い場合は、 get_record ('course', ...) を使用する代わりに get_course 関数を使用する必要がある。

> Those two course records have probably already been loaded, and using this function will save a database query.

これらの 2 つのコースのレコードは既に読み込まれている可能性がある。
この関数を使用すると、データベースクエリが保存される。

> Additionally, the code is shorter and easier to read.

加えて、このコードは読むために最も短く簡単である。

### 18.2 get_courses

> If you want to get all the current courses in your Moodle, use get_courses() without parameter:

Moodle で現在のコースをすべて取得したい場合は、 get_courses () をパラメータなしで使用する。

```php
$courses = get_courses();
```

## ちょっとやってみよう

```php
<?php
require('../../config.php');

function consoleLog($obj) {
    echo "<script>console.log(".json_encode($obj).");</script>";
};

function output($obj) {
    echo "<p>".json_encode($obj)."</p>";
};

function getdata() {
    $courses = get_courses();
    output($courses);
    consoleLog($courses);
};

getdata();
```

※ `$DB` の宣言も必要ない。

## 19 See also (関連項目)

- SQL coding style

https://docs.moodle.org/dev/Data_manipulation_API#get_record:~:text=See%20also-,SQL%20coding%20style,-Core%20APIs

- Core APIs

https://docs.moodle.org/dev/Core_APIs

- DML exceptions: New DML code is throwing exceptions instead of returning false if anything goes wrong

https://docs.moodle.org/dev/DML_exceptions

- DML drivers: Database drivers for new DML layer

https://docs.moodle.org/dev/DML_drivers

- DML functions - pre 2.0: (deprecated!) For information valid before Moodle 2.0.

https://docs.moodle.org/dev/DML_functions_-_pre_2.0

- DDL functions: Where all the functions used to handle DB objects (DDL) are defined.

https://docs.moodle.org/dev/DDL_functions

- DB layer 2.0 examples: To see some code examples using various DML functions.

https://docs.moodle.org/dev/DB_layer_2.0_examples

- DB layer 2.0 migration docs: Information about how to modify your code to work with the new Moodle 2.0 DB layer.

https://docs.moodle.org/dev/DB_layer_2.0_migration_docs

- DTL functions: Exporting, importing and moving of data stored in SQL databases

https://docs.moodle.org/dev/DTL_functions
