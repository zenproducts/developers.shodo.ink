# API・システム連携

🚀ようこそ🚀

Shodoの公開APIの利用方法を説明します。

## コマンド

ShodoのAPIをコマンドから便利に使えます。
たとえばMarkdownと画像を一括でダウンロードできる機能を使えます。

```bash
$ pip install shodo
$ shodo login
$ shodo download --target=docs
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

> **Warning**
> 校正APIは現在、一般公開されていません。

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
  "lint_id": "6d639e5f-8bfe-43d7-ac24-8bb6b97ba936"
}
```

Shodoの校正は非同期的に実行されます。
`lint_id` としてShodoが校正した結果を受け取るIDが返されます。

たとえば以下のようなコマンドでご利用いただけます。

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/lint/ text="校正する本文"
```

## 校正結果API

投稿した本文の校正結果を取得するAPIです。

APIのURL：`https://api.shodo.ink/@{organization}/{project}/lint/{lint_id}/`

### レスポンスの例

```json
{
  "messages": [
    {
      "after": "トル",
      "before": "が",
      "from": {
        "ch": 21,
        "line": 669
      },
      "index": 17619,
      "index_to": 17620,
      "message": "もしかしてAI",
      "severity": "warning",
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
    * message: 指摘の内容
    * from: 指摘の位置（{ line: 行, ch: 列 }）。0から始まる順番。
    * to: 指摘の終わり位置（{ line: 行, ch: 列 }）。0から始まる順番。
    * index: 指摘のインデックス番号。0から始まる順番。
    * index_to: 指摘の終わり位置のインデックス番号。
    * before: 指摘対象のテキスト
    * after: 推奨される置き換えテキスト
    * severity: error、warningによる重要度
* status: done（完了）、processing（校正中）、failed（失敗）の3つの状態
* updated: 最後に情報が更新された日時（UNIXタイムスタンプ）

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

一覧の内容は `results` にあります。データがまだ続いている場合は `next` に次のページのURLが格納されています。

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

一覧の内容は `results` にあります。データがまだ続いている場合は `next` に次のページのURLが格納されています。

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

