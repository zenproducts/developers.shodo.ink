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
  "lint_id": "6d639e5f-8bfe-43d7-ac24-8bb6b97ba936",
  "monthly_amount": 1000000,
  "current_usage": 116049,
  "len_body": 8429,
  "len_used": 8112
}
```

Shodo runs proofreading asynchronously.
The `lint_id` is used to get results of proofreading.

For example, you can use httpie to call this API.

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/lint/ text="校正する本文"
```

### Counting and caching

Shodo counts only Japanese sentences in the submitted body as the number of characters used.
The number of characters will be less than the length of all submitted body.

Submitted sentences will be cached, so if the same sentence is submitted again,
Shodo won't count them as monthly usage.

Each response of the proofreading API has the following meaning.

* `lint_id`: ID to get the result of the proofreading.
* `monthly_amount`: Number of characters available in the API (per month).
* `current_usage`: Number of characters used this month.
* `len_body`: Length of Japanese sentences of the submitted body.
* `len_used`: Used number of character (subtracting the cache).

For example, if all the cache is used, `len_used` will be `0`.
The cache is valid for about 30 minutes, but the cache time is not guaranteed.
Please be assured that the cache of the text is managed in a way that it is not leaked or affected by other users.

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

## Usage API

Usage API will return monthly usage of the organization.
The response will contain last 12 months of usage aggregation.

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

For example, you can use httpie to call this API.

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/usage/
```

## Terms API

You can set terms and variants for proofreading API.
The terms and variants set by this API are the same as those set by the Shodo screen.

API URL：`https://api.shodo.ink/@{organization}/{project}/terms/`

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
            "fuzzy_complement": false
        }
    ]
}
```

Command example:

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/terms/
```

### Response detail

`results` contains the list of terms. If there are more data, `next` will contain the next page number (`?page=2`, etc.).

* `id`：Term ID
* `description`：Description for the term (won't be used for AI proofreading)
* `text`：Term text
* `variants`：List of variants（each variants will be like `{"text": "VariantText"}`）
* `fuzzy_complement`： Whether to automatically detect fuzzy complements (if `true`, `variants` will be ignored)

### Add terms and variants

You can add terms and variants by `POST` method.

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/terms/ text="Shodo"
```

You can specify `text` , `description` , `variants` , `fuzzy_complement` params.

## Term detail API

You can get the detail of the term by the following API.

API URL：`https://api.shodo.ink/@{organization}/{project}/terms/{id}/`

Command example:

```bash
$ http -A bearer -a d8eb...3359 https://api.shodo.ink/@org/project/terms/1/
```

### Response detail

```json
{
    "id": 100000,
    "description": "",
    "text": "Shodo",
    "variants": [{"text":  "ShoDo"}],
    "fuzzy_complement": false
}
```

### Update terms and variants

You can update terms and variants by `PUT` method.

```bash
$ http -A bearer -a d8eb...3359 PUT https://api.shodo.ink/@org/project/terms/1/ text="SHODO"
```

You can specify `text` , `description` , `variants` , `fuzzy_complement` params.

### Delete terms and variants

You can delete terms and variants by `DELETE` method.

```bash
$ http -A bearer -a d8eb...3359 DELETE https://api.shodo.ink/@org/project/terms/1/
```

## Delete all terms

You can delete all terms and variants set in the project by the following API.

API URL：`https://api.shodo.ink/@{organization}/{project}/terms/deleteall/`

Send `POST` method to the API URL.

Command example:

```bash
$ http -A bearer -a d8eb...3359 POST https://api.shodo.ink/@org/project/terms/deleteall/
```

## Post file API

Please refer [APIドキュメント](./api.md) file (not translated yet).
