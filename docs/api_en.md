# Shodo API

🚀Welcome to Shodo API documentation🚀

## Command

You can use Shodo's API from shodo command.
For example, you can proofread Japanese in a Markdown file.

```bash
$ pip install shodo
$ shodo login
$ shodo lint README.md
```

For more details, refer [shodo command's README](https://github.com/zenproducts/shodo-python/).

## API root

Specify Organization slug and project name.

```
https://api.shodo.ink/@{organization}/{project}/
```

## Authorization

First, issue Bearer tokens in the "API integration" settings page of Shodo's project.

Specify the token on Authorization header.

```
Authorization: Bearer {token}
```

## Lint/Proofreading API

This is API to proofread Japanese texts.

API URL：`https://api.shodo.ink/@{organization}/{project}/lint/`

POST a Japanese body to the proofreading API.
Up to 40,000 characters are allowed.

```json
{
  "body": "校正する本文"
}
```

### Response

```json
{
  "lint_id": "6d639e5f-8bfe-43d7-ac24-8bb6b97ba936"
}
```

Shodo runs proofreading asynchronously.
The `lint_id` is used to get results of proofreading.

For example, you can use httpie to call this API.

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/lint/ text="校正する本文"
```

## Result API

You can get results of proofreading.

API URL：`https://api.shodo.ink/@{organization}/{project}/lint/{lint_id}/`

### Response

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

Command example:

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/lint/6d639e5f-8bfe-43d7-ac24-8bb6b97ba936/
```

### Response detail

* messages:
    * code: Proofreading code
    * message: Proofreading message detail
    * from: Starting position（{ line: line number, ch: column number }）. Starting from 0.
    * to: Ending position（{ line: line number, ch: column number }）. Starting from 0.
    * index: Index number starting from 0.
    * before: Text with Japanese problem.
    * after: Recommended text.
    * operation: Which you should "delete" or "replace" the "before" word. null if it's unpredictable.
    * score: Probability of proofreading changes
    * severity: Severity of the message (error or warning).
* status: The status of proofreading process. done, processing, or failed.
* updated: Last updated datetime (UNIX timestamp)

## Proofreading multiple sentences

Shodo's API allows you to submit multiple texts at once.
Instead of `body`, you can specify a list of body texts with the parameter `bulk_body`.

```json
{
  "bulk_body": ["本文1", "本文2"]
}
```

The `messages` param of the proofreading result will be the result of each body.
For example, if `bulk_body` contains two bodies, the `messages` will be a list of two elements.

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

## Post file API

Please refer [APIドキュメント](./api.md) file (not translated yet).
