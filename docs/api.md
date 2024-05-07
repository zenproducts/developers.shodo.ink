# API・システム連携

🚀ようこそ🚀

Shodoの公開APIの利用方法を説明します。

## コマンド

ShodoのAPIをコマンドから便利に使えます。
たとえばMarkdownファイルを校正するコマンドは以下のようになります。

```bash
$ pip install shodo
$ shodo login
$ shodo lint README.md
```

詳しくは [コマンドのREADME](https://github.com/zenproducts/shodo-python/) をご確認ください。

## APIルート

組織の英数名（スラッグ）と、プロジェクトの英数名を指定します。

```
https://api.shodo.ink/@{organization}/{project}/
```

## 認証

Shodoのプロジェクトより、「API連携」設定画面 で、トークンを発行してください。

以下のようにHTTPヘッダーからトークンを指定します。

```
Authorization: Bearer {token}
```

## 校正API

記事や文章を校正するAPIです。

APIのURL：`https://api.shodo.ink/@{organization}/{project}/lint/`

校正APIに文章をPOSTします。
最大で4万文字まで対応しています。

```json
{
  "body": "校正する本文"
}
```

### レスポンスの例

```json
{
  "lint_id": "6d639e5f-8bfe-43d7-ac24-8bb6b97ba936",
  "monthly_amount": 1000000,
  "current_usage": 116049,
  "len_body": 8429,
  "len_used": 8112
}
```

Shodoの校正は非同期的に実行されます。
`lint_id` としてShodoが校正した結果を受け取るIDが返されます。

たとえば以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/lint/ body="校正する本文"
```

### 文字数とキャッシュ

Shodoは投稿されたテキストのうち、日本語を含む文のみを利用文字数とカウントします。
投稿されたテキストすべての長さより少ない文字数になります。

投稿された文はキャッシュされるので、同じ文が投稿された場合には利用文字数は減りません。たとえば1万文字の文章があるときに、1文字だけを変更して再度校正した場合、変更した文字を含む文だけが再度カウントの対象となります。

校正APIのレスポンスはそれぞれ以下のような意味を持ちます。

* `lint_id`：校正APIの結果を取得するID
* `monthly_amount`：APIの利用可能文字数（月ごと）
* `current_usage`：今月利用した文字数
* `len_body`：投稿したテキストの日本語文の長さ
* `len_used`：利用された校正文字数（キャッシュ分は差し引き）

たとえばすべてキャッシュでまかなわれた場合、 `len_used` は `0` となります。キャッシュは30分程度有効ですが、キャッシュ時間が保証されるわけではありません。また、本文のキャッシュは他の利用者に漏えい・影響しない形式で管理されておりますので、安心してご利用ください。

## 校正結果API

投稿した本文の校正結果を取得するAPIです。

APIのURL：`https://api.shodo.ink/@{organization}/{project}/lint/{lint_id}/`

### レスポンスの例

```json
{
  "messages": [
    {
      "after": "",
      "before": "が",
      "code": "ai_recommend",
      "from": {
        "ch": 21,
        "line": 669
      },
      "index": 17619,
      "index_to": 17620,
      "message": "もしかしてAI",
      "severity": "error",
      "operation": "delete",
      "score": 0.9886295795440674,
      "to": {
        "ch": 22,
        "line": 669
      }
    }
  ],
  "status": "done",
  "updated": 1658379913
}
```
たとえば以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/lint/6d639e5f-8bfe-43d7-ac24-8bb6b97ba936/
```

### レスポンスの意味

* messages:
    * code: 指摘のコード
    * message: 指摘のメッセージ
    * from: 指摘の位置（{ line: 行, ch: 列 }）。0から始まる順番。
    * to: 指摘の終わり位置（{ line: 行, ch: 列 }）。0から始まる順番。
    * index: 指摘のインデックス番号。0から始まる順番。
    * index_to: 指摘の終わり位置のインデックス番号。
    * before: 指摘対象のテキスト
    * after: 推奨される置き換えテキスト
    * operation: 指摘対象のテキストを置き換える（`replace`）か削除するか（`delete`）、保持するか（`keep`）。不明な場合 `null`。
    * score: 校正の指摘がどれだけ確からしいかのスコア
    * severity: error、warning、infoによる重要度（infoは変更を求めない付帯情報）
* status: done（完了）、processing（校正中）、failed（失敗）の3つの状態
* updated: 最後に情報が更新された日時（UNIXタイムスタンプ）

### エンタープライズプランの動作速度

エンタープライズプランの場合、システム構成により文章校正が完了するまでの時間が変わります。
コストを大幅に抑えた構成の場合、コールドスタンバイ状態では10秒ほど余計に時間がかかります。

ホスティングのShodo（api.shodo.ink）や通常のエンタープライズプランのシステム構成の場合は高速に動作します。

## 複数文章の校正

ShodoのAPIは複数の文章をまとめて投稿できます。
`body` の代わりに `bulk_body` というパラメータで文章の一覧を指定してください。

```json
{
  "bulk_body": ["校正する本文", "校正する本文2"]
}
```

校正結果の `messages` は各本文ごとの結果になります。
たとえば `bulk_body` に2つの本文を指定すると、 `messages` も2要素のリストとなります。

```json
{
  "messages": [
    [
      {
        "after": "",
        "before": "が",
        "from": {
          "ch": 0,
          "line": 3
        }
      }
    ],
    []
  ],
  "status": "done",
  "updated": 1658379913
}
```

## 利用状況API

利用状況APIは月次での利用文字数を取得できます。
利用文字数は組織ごとにカウントされ、過去12ヶ月の利用状況を取得できます。

API URL：`https://api.shodo.ink/@{organization}/{project}/usage/`

### Response

```json
{
  "usage": [
      {"year":  2023, "month":  6, "amount":  24000},
      {"year":  2023, "month":  5, "amount":  28000},
      {"year":  2023, "month":  4, "amount":  32000},
      {"year":  2023, "month":  3, "amount":  150129},
      {"year":  2023, "month":  2, "amount":  4003},
      {"year":  2023, "month":  1, "amount":  44000},
      {"year":  2022, "month":  12, "amount":  48081},
      {"year":  2022, "month":  11, "amount":  52000},
      {"year":  2022, "month":  10, "amount":  56000},
      {"year":  2022, "month":  9, "amount":  60022},
      {"year":  2022, "month":  8, "amount":  64330},
      {"year":  2022, "month":  7, "amount":  68830},
      {"year":  2022, "month":  6, "amount":  72120}
  ],
  "monthly_mount":  160000 
}
```

たとえば、以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/usage/
```

## 用語・表記ゆれAPI

校正APIで使われる用語と表記ゆれを設定するAPIです。
このAPIで設定する表記ゆれは、Shodoの画面から設定する表記ゆれと同じものです。

APIのURL：`https://api.shodo.ink/@{organization}/{project}/terms/`

リクエストはプロジェクトごとに、1分間に60回まで許可されています。
それ以上の回数をご要望の場合は[お問い合わせ](https://shodo.ink/contact/)ください。

### Response

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": 100000,
            "description": "",
            "text": "Shodo",
            "variants": [{"text":  "ShoDo"}],
            "severity": "error",
            "fuzzy_complement": false
        }
    ]
}
```

たとえば、以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/terms/
```

### レスポンスの意味

一覧の内容は `results` にあります。データがまだ続いている場合は `next` に次のページ番号（`?page=2` など）が格納されています。

* `id`：用語のID
* `description`：用語の説明（画面上でのみ利用されます）
* `text`：用語の表記
* `variants`：表記ゆれの一覧（`{"text": "表記ゆれ文字"}` として指定）
* `severity`: 表記ゆれの重大度（`error`、`warning`、`info` の3種類）
* `fuzzy_complement`：表記ゆれの自動判定を行うかどうか（`true` の場合 `variants` は無効）

### クエリーパラメーター

* `page`：ページネーションの番号を数値で指定

### 用語・表記ゆれの追加

用語・表記ゆれの追加は `POST` メソッドで行います。

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/terms/ text="Shodo"
```

`text` 、 `description` 、 `variants` 、 `severity`、 `fuzzy_complement` のパラメーターを指定できます。

## 用語・表記ゆれ詳細API

表記ゆれの詳細を取得します。

APIのURL：`https://api.shodo.ink/@{organization}/{project}/terms/{id}/`

たとえば、以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/terms/1/
```

### レスポンスの意味

レスポンスは一覧と同じ内容です。

```json
{
    "id": 100000,
    "description": "",
    "text": "Shodo",
    "variants": [{"text":  "ShoDo"}],
    "severity": "error",
    "fuzzy_complement": false
}
```

### 用語・表記ゆれの更新

用語・表記ゆれの更新は `PUT` メソッドで行います。

たとえば、以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 PUT https://api.shodo.ink/@org/project/terms/1/ text="SHODO"
```

パラメーターは用語・表記ゆれの追加と同じです。

### 用語・表記ゆれの削除

用語・表記ゆれの更新は `DELETE` メソッドで行います。

たとえば、以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 DELETE https://api.shodo.ink/@org/project/terms/1/
```

## 用語・表記ゆれ一括削除API

プロジェクトに設定されたすべての用語・表記ゆれを削除するAPIです。

APIのURL：`https://api.shodo.ink/@{organization}/{project}/terms/deleteall/`

URLに対して `POST` メソッドを送信してください。

たとえば、以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 POST https://api.shodo.ink/@org/project/terms/deleteall/
```

## 人物・著名人API

**エンタープライズプランでオプション対応時に有効になります**

校正APIで使われる人物情報を設定するAPIです。

APIのURL：`https://api.shodo.ink/@{organization}/{project}/bio/people/`

リクエストはプロジェクトごとに、1分間に60回まで許可されています。
それ以上の回数をご要望の場合は[お問い合わせ](https://shodo.ink/contact/)ください。

### Response

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
      {
          "id": "dec203d4-995e-4f59-9a7c-f057e1527006",
          "belonging": "自民",
          "last_name": "河野",
          "first_name": "太郎",
          "last_name_kana": "こうの",
          "first_name_kana": "たろう",
          "name": null,
          "birth_date": null,
          "prefixes": [],
          "suffixes": [
              "議員",
              "衆議院議員",
              "衆院議員",
              "大臣",
              "デジタル大臣",
              "デジタル行財政改革担当",
              "デジタル田園都市国家構想担当",
              "行政改革担当",
              "国家公務員制度担当",
              "内閣府特命担当大臣（規制改革）",
              "デジタル相",
              "デジ相"
          ],
          "ref_urls": [
              "https://www.shugiin.go.jp/internet/itdb_giinprof.nsf/html/profile/182.html",
              "https://www.kantei.go.jp/jp/101_kishida/meibo/daijin/kono_taro.html"
          ],
          "is_english_name": false,
          "is_representing_last_name": true,
          "updated_by": "Shodo",
          "updated_at": "2024-01-24T17:00:00+09:00"
      }
    ]
}
```

たとえば、以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/bio/people/
```

### レスポンスの意味

一覧の内容は `results` にあります。データがまだ続いている場合は `next` に次のページ番号（`?page=2` など）が格納されています。

* `id`：人物のID
* `belonging`：所属の名前（事前に設定した下記「所属」の `name`）
* `last_name`：人物の姓
* `first_name`：人物の名前
* `last_name_kana`：人物の姓（かな）
* `first_name_kana`：人物の名前（かな）
* `name`：名称（姓名で入力しない場合の名前）
* `birth_date`：生年月日（設定した場合、「河野太郎大臣（61）」のような年齢が正しいかを判定）
* `prefixes`：人物の先頭につく名前で所属以外のもの（「大リーグ・大谷翔平」など）
* `suffixes`：人物の末尾につく名前（「大谷翔平選手」など）
* `ref_urls`：その情報のもとになった情報ソース
* `is_english_name`：英語名の場合 `true`
* `is_representing_last_name`：姓を代表する名前の場合 `true` 。「河野大臣」のように姓だけでその人物を特定する場合
* `updated_by`：情報を設定した人物（文字列で設定）
* `updated_at`：情報を設定した日時

### クエリーパラメーター

* `page`：ページネーションの番号を数値で指定
* `search`: 姓、名もしくは名称での検索。複数項目（姓と名など）を指定する場合スペース区切りで入力

### 追加、詳細、更新API

追加、詳細、更新APIについては表記ゆれ・用語APIと同様の仕様（POST内容等は人物の情報）で行います。

## 所属API

**エンタープライズプランでオプション対応時に有効になります**

人物に指定する「所属」を扱うAPIです。

APIのURL：`https://api.shodo.ink/@{organization}/{project}/bio/belongings/`

### Response

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
      {
        "id": "21a960ba-368d-4872-90a2-92089036007c",
        "name": "自民",
        "aliases": [
            "自民党",
            "自"
        ],
        "updated_at": "2024-02-01T15:56:52.140786+09:00"
      }
    ]
}
```

### レスポンスの意味

一覧の内容は `results` にあります。データがまだ続いている場合は `next` に次のページ番号（`?page=2` など）が格納されています。

* `id`：所属のID
* `name`：所属の名前（プロジェクトごとに一意な値）
* `alises`：所属の別名（「自民党・河野大臣」、「自・河野大臣」など複数のパターンで判定可能）
* `updated_at`：情報を設定した日時

### クエリーパラメーター

* `page`：ページネーションの番号を数値で指定

### 追加、詳細、更新API

追加、詳細、更新APIについては表記ゆれ・用語APIと同様の仕様（POST内容等は人物の情報）で行います。

## 記事ファイルAPI

作成されたMarkdownの記事を取得するAPIです。
ファイルシステムへ保存するためや、ヘッドレスCMSとしてご利用ください。

APIのURL：`https://api.shodo.ink/@{organization}/{project}/files/`

リクエストはプロジェクトごとに、1分間に60回まで許可されています。
それ以上の回数をご要望の場合は[お問い合わせ](https://shodo.ink/contact/)ください。

### 基本情報

執筆したMarkdownを「記事」、執筆の管理をしているものを「タスク」といいます。
記事は執筆タスクごとに最新のファイルのみ取得できます。

執筆のステータスは以下の数値で表現されます：

* アイディア：5
* 執筆中：30
* レビュー：50
* レビューOK：60
* 完了：70
* 却下：90

### レスポンスの例

```json
{
  "count": 30,
  "next": "?page=2",
  "previous": null,
  "results": [
    {
      "number": 16,
      "version": 3,
      "directory_path": "ブログ/新着ニュース",
      "filename": "執筆の「いいね」を使って良い感じのネタを決めよう.md",
      "body": "# タイトル\n\n記事の本文",
      "committed_at": "2022-04-04T00:00:00.976185+09:00",
      "task": {
        "status": 70,
        "status_display": "完了",
        "title": "執筆の「いいね」を使って良い感じのネタを決めよう",
        "due_datetime": "2021-06-18T12:00:00+09:00",
        "assign_data": {
          "id": 2,
          "icon_url": "https://media.shodousercontents.com/...jpg",
          "fullname": "Kiyohara Hiroki",
          "username": "hirokiky"
        },
        "reviewer_data": {
          "id": 10,
          "icon_url": "https://media.shodousercontents.com/...jpg",
          "fullname": "shodo.ink",
          "username": "shodo.ink"
        },
        "directory_data": {
          "name": "新着ニュース",
          "uuid": "75e8f0b9-...-165d63069919"
        }
      },
      "images": [
        {
          "created_at": "2022-04-04T00:00:00.976185+09:00",
          "url": "https://media.shodousercontents.com/...jpg"
        }
      ]
    }
  ]
}
```

たとえば以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/files/
```

### レスポンスの意味

一覧の内容は `results` にあります。データがまだ続いている場合は `next` に次のページ番号（`?page=2` など）が格納されています。

* `number`：執筆タスクの番号
* `version`：記事のバージョン番号
* `directory_path`：タスクの「フォルダー」に対応した、ファイルを保存するディレクトリーのパス（未指定で `null`）
* `filename`：執筆タスクのタイトルを元にした、ファイル名として保存できる記事の名前
* `body`：Markdownファイルの中身（全文）
* `committed_at`：記事が作成された日時
* `task`
    * `status`：執筆タスクのステータス（番号）
    * `status_display`：執筆タスクのステータス（表示名）
    * `title`：執筆タスクのタイトル
    * `due_datetime`：タスクの期限
    * `assign_data`：アサインされたメンバーの情報（未指定で `null`）
    * `reviewer_data`：レビュアーのメンバーの情報（未指定で `null`）
    * `directory_data`：タスクのフォルダーの情報（未指定で `null`）
* `images`：記事中のShodoにアップロードされた画像（`body` 内の順序）

### クエリーパラメーター

* `page`：ページネーションの番号を数値で指定
* `assign__username`：指定したユーザーがアサインされた記事だけ取得
* `directory__name`：指定したフォルダーの記事だけ取得（当該フォルダーの直下にある記事のみ取得）
* `status_ready`：`1`を指定すると「レビューOK」、「完了」になった執筆タスクの記事だけ取得
* `in_tree`：`1`を指定すると「フォルダー」内の記事のみ取得

### 記事ファイル詳細API

APIのURL：`https://api.shodo.ink/@{organization}/{project}/files/{number}/`

執筆タスクの `number` を指定して、1つの記事を取得できます。
内容は記事の一覧APIと同じです。

## タスクAPI

執筆タスクの一覧を取得するAPIです。新しく作成された記事の順に取得できます。

APIのURL：`https://api.shodo.ink/@{organization}/{project}/tasks/`

### レスポンスの例

```json
{
  "count": 50,
  "next": "?page=2",
  "previous": null,
  "results": [
    {
      "project": 1,
      "number": 100,
      "title": "無題",
      "status": 30,
      "status_display": "執筆",
      "body": "読者の方は〇〇を想定しています。\nShodoで執筆する概要がつかめる記事にします。\n",
      "body_rendered": "<p>読者の方は〇〇を想定しています。<br>\nShodoで執筆する概要がつかめる記事にします。</p>\n",
      "due_datetime": "2022-04-04T00:00:00.976185+09:00",
      "assign_data": {
        "id": 2,
        "icon_url": "https://media.shodousercontents.com/...jpg",
        "fullname": "Kiyohara Hiroki",
        "username": "hirokiky"
      },
      "reviewer_data": {
        "id": 10,
        "icon_url": "https://media.shodousercontents.com/...jpg",
        "fullname": "shodo.ink",
        "username": "shodo.ink"
      },
      "created_by": {
        "id": 2,
        "icon_url": "https://media.shodousercontents.com/...jpg",
        "fullname": "Kiyohara Hiroki",
        "username": "hirokiky"
      },
      "directory": "75e8f0b9-...-165d63069919",
      "created_at": "2022-04-04T00:00:00.976185+09:00",
      "like_count": 0,
      "review_count": 0,
      "review_solved_count": 0
    }
  ]
}
```

### レスポンスの意味

一覧の内容は `results` にあります。データがまだ続いている場合は `next` に次のページ番号（`?page=2` など）が格納されています。

* `project`：プロジェクトのID
* `number`：執筆タスクの番号
* `title`：執筆タスクのタイトル
* `status`：執筆タスクのステータス（番号）
* `status_display`：執筆タスクのステータス（表示名）
* `due_datetime`：タスクの期限
* `assign_data`：アサインされたメンバーの情報（未指定で `null`）
* `reviewer_data`：レビュアーのメンバーの情報（未指定で `null`）
* `created_by`：タスクを作成したメンバーの情報
* `directory`：タスクのフォルダーごとの一意な値
* `created_at`：タスクが作成された日時
* `like_count`：「いいね」された数
* `review_count`：レビューされたコメントの数
* `review_solved_count`：解決されたレビューコメントの数

### クエリーパラメーター

* `page`：ページネーションの番号を数値で指定
* `ordering`：一覧で取得する順序
    * `number`：執筆タスクの番号の昇順
    * `-number`：執筆タスクの番号の降順
    * `due_datetime`：タスクの期限の昇順（期限なしは末尾）
    * `-due_datetime`：タスクの期限の降順（期限なしは末尾）
    * `title`：タイトルの昇順
    * `-like_count`：「いいね」数の降順
* `status`：執筆タスクのステータス（ボン号）を指定
* `status-open`：`1`を指定して、アイディア、執筆中、レビューとレビューOKのタスクのみに絞り込み
* `assign__username`：アサインされたメンバーの `username` を指定

### タスク詳細API

APIのURL：`https://api.shodo.ink/@{organization}/{project}/tasks/{number}/`

執筆タスクの `number` を指定して、1つの記事を取得できます。
内容は記事の一覧APIと同じです。
