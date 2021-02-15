---
title: Surge API

language_tabs: # must be one of https://git.io/vQNgJ
  - ruby  
  - python  
  - shell

toc_footers:
  - <a href='https://www.app.surgehq.ai/poster_sign_up'>Sign up for an account</a>


includes:
  - errors

search: true

code_clipboard: true
---

# Surge API

Welcome to the Surge API! You can use the API to programmatically create projects and run tasks. All you need is your API key, and funds in your account if you want to have tasks completed by our workforce.


# Authentication

> To authorize, use this code.

```ruby
require 'httparty'

# For POST requests
HTTParty.post("{{API_ENDPOINT}}", 
  basic_auth: { username: "{{YOUR_API_KEY}}" },
  body: {
    ... # Parameters in the POST request
  }
)

# For GET requests
HTTParty.get("{{API_ENDPOINT}}", 
  basic_auth: { username: "{{YOUR_API_KEY}}" }
)
```

```python
import requests

# For POST requests
requests.post("{{API_ENDPOINT}}", 
  auth = ("{{YOUR_API_KEY}}", ""),
  data = {
    ... # Parameters in the POST request
  }
)

# For GET requests
requests.get("{{API_ENDPOINT}}", 
  auth = ("{{YOUR_API_KEY}}", "")
)
```

```shell
# Note that the colon follows the API key and is not part of it.
curl {{API_ENDPOINT}} \
  -u {{YOUR_API_KEY}}:
```

Surge uses API keys to allow access to the API. You can find your API key in [your profile](https://www.app.surgehq.ai/me).

Authentication is performed via [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication). Your API key is your basic auth username, and you do not need to provide a password.

If you need to authenticate via bearer auth (e.g., for a cross-origin request), use `-H "Authorization: {{YOUR_API_KEY}}:"` instead of `-u {{YOUR_API_KEY}}:`.

# Example

Let's walk through an example using Hybrid to categorize companies. If you'd like to run the example yourself, find your API key in [your profile](https://app.surgehq.ai/me).

```ruby
require 'httparty'

# First, we create a "categorize company" project.
response = HTTParty.post("https://app.surgehq.ai/api/projects", 
  basic_auth: { username: "DJ34KFX24M2G3JRHGXNZVCPE" },
  body: {
    name: "Categorize this company",
    payment_per_response: 0.1,
    questions: [
      {
        "text": "What is the name of the company at this website?"
      },
      {
        "text": "What category does this company belong to?",
        "options": ["Tech", "Sports", "Gaming"]
      }
    ]
  }
)

# Extract the project ID.
project_id = JSON.parse(response.body)["id"]

# Post a task to the API.
HTTParty.post("https://app.surgehq.ai/api/projects/#{project_id}/tasks",
  basic_auth: { username: "DJ34KFX24M2G3JRHGXNZVCPE" },
  body: {
    "fields": {
      "website": "google.com"
    }    
  }
)

# Retrieve the tasks and any responses.
response = HTTParty.get("https://app.surgehq.ai/api/projects/#{project_id}/tasks",
  basic_auth: { username: "DJ34KFX24M2G3JRHGXNZVCPE" }
)

JSON.parse(response.body).each do |task|
  puts task["responses"][0]["response_data"]
end
```

```python
import requests

requests.post("https://app.surgehq.ai/api/projects", 
  auth = ("{{YOUR_API_KEY}}", ""),
  data = {
    "name": "Categorize this website",
    "payment_per_response": 0.1,
    "questions": [
      {
        "text": "What is the name of the company at this website?"
      },
      {
        "text": "What category does this company belong to?",
        "options": ["Tech", "Sports", "Gaming"]
      }
    ]
  }
)
```

# Projects

## The project object

> Example Response

```
{
  "id": "17e323f1-f7e4-427c-a2d5-456743aba8",
  "callback_url": "https://www.mywebsite.com/hybrid_callback",
  "name": "Categorize this website",
  "num_tasks": 1000,
  "num_tasks_completed": 239,
  "num_workers_per_task": 1,
  "payment_per_response": 0.1,
  "status": "unlaunched"
}
```

Parameter | Type | Description
--------- | ---- | -----------
id | string |
callback_url | string | Once a task is completed, it is POSTed to this url.
created_at | string |
fields_template | string | A template describing how fields are shown to workers working on the task. For example, if `fields_template` is "<a href="{{url}}">{{company_name}}</a>", then workers will be shown a link to the company.
instructions | string | Instructions shown to workers describing how they should complete the task.
name | string | Name of the project.
num_tasks | integer | Number of tasks in the project.
num_tasks_completed | integer | Number of completed tasks.
num_workers_per_task | integer | How many workers work on each task (i.e., how many responses per task).
payment_per_response | float | How much a worker is paid (in US dollars) for an individual response.
questions | array | An array of question objects describing the questions to be answered. Each question object contains a `text` field (the text of the question), an optional `options` array (for multiple choice and checkbox questions, listing the answer options), a `type` field (one of `free_response`, `multiple_choice`, or `checkbox`), and a `required` boolean field (describing whether or not workers must answer the question)
status | string | One of `in_progress`, `completed`, `canceled`, or `paused`.

## Create a project

> Definition

```
POST https://app.surgehq.ai/api/projects
```

> Example Request

```shell
curl https://app.surgehq.ai/api/projects \
  -u {{YOUR_API_KEY}}: \
  -d name="Categorize this company" \
  -d payment_per_response=0.1 \
  -d questions=... (can't do this in curl)
```

```ruby
require 'httparty'

HTTParty.post("https://app.surgehq.ai/api/projects", 
  basic_auth: { username: "{{YOUR_API_KEY}}" },
  body: {
    name: "Categorize this company",
    payment_per_response: 0.1,
    instructions: "You will be asked to categorize a company.",
    questions: [
      {
        "type": "free_response",
        "value": "What is the name of the company at this website?"
      },
      {
        "type": "multiple_choice",
        "value": "What category does this company belong to?",
        "options": ["Tech", "Sports", "Gaming"]
      },
      {
        "type": "checkbox",
        "value": "Check all the social media accounts this company has",
        "options": ["Facebook", "Twitter", "Pinterest", "Google+"]
      }
    ]
  }
)
```

```python
import requests

requests.post("https://app.surgehq.ai/api/projects", 
  auth = ("{{YOUR_API_KEY}}", ""),
  data = {
    "name": "Categorize this website",
    "payment_per_response": 0.1,
    "instructions": "You will be asked to categorize a company.",
    "questions": [
      {
        "type": "free_response",
        "value": "What is the name of the company at this website?"
      },
      {
        "type": "multiple_choice",
        "value": "What category does this company belong to?",
        "options": ["Tech", "Sports", "Gaming"]
      },
      {
        "type": "checkbox",
        "value": "Check all the social media accounts this company has",
        "options": ["Facebook", "Twitter", "Pinterest", "Google+"]
      }
    ]
  }
)
```

> Example Response

```json
{
  "id": "17e323f1-f7e4-427c-a2d5-456743aba8",
  "callback_url": "https://www.mywebsite.com/hybrid_callback",
  "name": "Categorize this website",
  "num_tasks": 0,
  "num_tasks_completed": 0,
  "num_workers_per_task": 1,
  "payment_per_response": 0.1,
  "status": "unlaunched"
}
```

Creates a project. It is also possible to create a project through the web UI (instead of the API), and you will still be able to use the API to post tasks to the project.

### Parameters

Parameter | Type | Description
--------- | ---- | -----------
name | string | Name to give your project. (required)
payment_per_response | float | Amount in dollars to pay workers for each response. (required)
question | array | An array of JSON objects that define the questions that are shown to workers. Each question object takes a `text` field (the question that gets shown to workers), an optional `options` array (for `multiple_choice` and `checkbox`, which define the answer options that workers see), an optional `type` field (one of `free_response`, `multiple_choice`, or `checkbox`; by default, if the field is not provided, then question objects without `options` are assumed to be `free_response`, and question objects with `options` are assumed to be `multiple_choice`), and an optional `required` field (a boolean that defines whether or not the question must be answered; default true). (required)
callback_url | string | URL where completed tasks will be POSTed. (optional, default null)
num_workers_per_task | integer | Number of workers who work on each task. (optional, default 1)

## List all projects

> Definition

```
GET https://app.surgehq.ai/api/projects
GET https://app.surgehq.ai/api/projects?page=n

```

> Example Request

```ruby
require 'httparty'

HTTParty.get("https://app.surgehq.ai/api/projects?page=2", 
  basic_auth: { username: "{{YOUR_API_KEY}}" }
)
```

```python
import requests

requests.get("https://app.surgehq.ai/api/projects", 
  auth = ("{{YOUR_API_KEY}}", "")
)
```

```shell
curl https://app.surgehq.ai/api/projects \
  -u {{YOUR_API_KEY}}:
```

> Example Response

```json
[
  {
    "id": "17e323f1-f7e4-427c-a2d5-456743aba8",
    "callback_url": "https://www.mywebsite.com/hybrid_callback",
    "name": "Categorize customer websites",
    "num_tasks": 1000,
    "num_tasks_completed": 239,
    "num_workers_per_task": 1,
    "payment_per_response": 0.1,
    "status": "unlaunched"
  },
  {
    "id": "81533541-0359-4d4b-a545-af38a2cb3e8c",
    "callback_url": null,
    "name": "Label product images",
    "num_tasks": 5000,
    "num_tasks_completed": 4315,
    "num_workers_per_task": 1,
    "payment_per_response": 0.25,
    "status": "in_progress"
  }
]
```

Lists all projects you have created.

### Parameters

Parameter | Type | Description
--------- | ------- | -----------
page | integer | Projects are returned in descending `created_at` order. Each page contains a maximum of 25 projects. Pages start at 1. (optional, default 1)

## Retrieve a project

> Definition

```
GET https://app.surgehq.ai/api/projects/{{PROJECT_ID}}
```

> Example Request

```ruby
require 'httparty'

HTTParty.get("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8", 
  basic_auth: { username: "{{YOUR_API_KEY}}" }
)
```

```python
import requests

requests.get("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8", 
  auth = ("{{YOUR_API_KEY}}", "")
)
```

```shell
curl https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8 \
  -u {{YOUR_API_KEY}}:
```

> Example Response

```json
{
  "id": "17e323f1-f7e4-427c-a2d5-456743aba8",
  "callback_url": "https://www.mywebsite.com/hybrid_callback",
  "name": "Categorize this website",
  "num_tasks": 1000,
  "num_tasks_completed": 239,
  "num_workers_per_task": 1,
  "payment_per_response": 0.1,
  "status": "unlaunched"
}
```

Retrieves a specific project you have created.

## Pause a project

> Definition

```
POST https://app.surgehq.ai/api/projects/{{PROJECT_ID}}/pause
```

> Example Request

```ruby
require 'httparty'

HTTParty.post("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/pause", 
  basic_auth: { username: "{{YOUR_API_KEY}}" }
)
```

```python
import requests

requests.post("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/pause", 
  auth = ("{{YOUR_API_KEY}}", "")
)
```

```shell
curl https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/pause \
  -u {{YOUR_API_KEY}}: \
  -X POST
```

> Example Response

```json
{
  "id": "17e323f1-f7e4-427c-a2d5-456743aba8",
  "callback_url": "https://www.mywebsite.com/hybrid_callback",
  "name": "Categorize this website",
  "num_tasks": 1000,
  "num_tasks_completed": 239,
  "num_workers_per_task": 1,
  "payment_per_response": 0.1,
  "status": "paused"
}
```

Pauses a project. Tasks added to the project will not be worked on until you resume the project.

## Resume a project

> Definition

```
POST https://app.surgehq.ai/api/projects/{{PROJECT_ID}}/resume
```

> Example Request

```ruby
require 'httparty'

HTTParty.post("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/resume", 
  basic_auth: { username: "{{YOUR_API_KEY}}" }
)
```

```python
import requests

requests.post("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/resume", 
  auth = ("{{YOUR_API_KEY}}", "")
)
```

```shell
curl https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/resume \
  -u {{YOUR_API_KEY}}: \
  -X POST
```

> Example Response

```json
{
  "id": "17e323f1-f7e4-427c-a2d5-456743aba8",
  "callback_url": "https://www.mywebsite.com/hybrid_callback",
  "name": "Categorize this website",
  "num_tasks": 1000,
  "num_tasks_completed": 239,
  "num_workers_per_task": 1,
  "payment_per_response": 0.1,
  "status": "in_progress"
}
```

Resumes a paused project.

## Cancel a project

> Definition

```
POST https://app.surgehq.ai/api/projects/{{PROJECT_ID}}/cancel
```

> Example Request

```ruby
require 'httparty'

HTTParty.post("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/cancel", 
  basic_auth: { username: "{{YOUR_API_KEY}}" }
)
```

```python
import requests

requests.post("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/cancel", 
  auth = ("{{YOUR_API_KEY}}", "")
)
```

```shell
curl https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/cancel \ 
  -u {{YOUR_API_KEY}}: \
  -X POST
```

> Example Response

```json
{
  "id": "17e323f1-f7e4-427c-a2d5-456743aba8",
  "callback_url": "https://www.mywebsite.com/hybrid_callback",
  "name": "Categorize this website",
  "num_tasks": 1000,
  "num_tasks_completed": 239,
  "num_workers_per_task": 1,
  "payment_per_response": 0.1,
  "status": "canceled"
}
```

Cancels a project.

# Tasks

## The task object

> Example Response

```
{
  "id": "38da6bc5-a644-41b9-a964-4678bc7375c6",
  "project_id": "b31ede78-afdf-4938-b775-8813453c7fc5",
  "created_at": "2016-11-01T18:56:56.000Z",
  "is_complete": true,
    
  "fields": {
    "company": "Hybrid",
    "city": "San Francisco",
    "state": "CA"
  },

  "responses": [
    {
      "created_at": "2016-11-01T23:29:17.971Z",
      "id": "1befb37b-8e4f-42b6-8a61-2c946ad4b5ce",
      "response_data": {
        "website": "https://app.surgehq.ai",
        "category": "Technology"
      }
    }
  ]
}
```

Parameter | Type | Description
--------- | ---- | -----------
id | string |
created_at | string |
project_id | string | ID of the project that this task belongs to.
is_complete | boolean | Whether or not this task has received the desired number of responses (equal to the project's `num_workers_per_task` field).
fields | dictionary | A dictionary of named fields that get shown (according to the project template) to workers when working on the task.
responses | array | An array of responses to the task. Each response contains an `id`, a `created_at` field, and a `response_data` dictionary that maps questions to their answers.

## Create a task

> Definition

```
POST https://app.surgehq.ai/api/projects/{{PROJECT_ID}}/tasks
```

> Example Request

```ruby
require 'httparty'

HTTParty.post("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/cancel", 
  basic_auth: { username: "{{YOUR_API_KEY}}" },
  body: {
    "fields": {
      "company": "Hybrid",
      "city": "San Francisco",
      "state": "CA"
    }    
  }
)
```

```python
import requests

requests.post("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/cancel", 
  auth = ("{{YOUR_API_KEY}}", ""),
  data = {
    "fields": {
      "company": "Hybrid",
      "city": "San Francisco",
      "state": "CA"
    }
  }
)
```

```shell
curl https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/tasks \
  -u {{YOUR_API_KEY}}: \
  -d data=... (can't do this in curl)
  
```

> Example Response

```json
{
  "id": "38da6bc5-a644-41b9-a964-4678bc7375c6",
  "project_id": "b31ede78-afdf-4938-b775-8813453c7fc5",
  "created_at": "2016-11-01T18:56:56.000Z",
  "is_complete": false,
    
  "fields": {
    "company": "Hybrid",
    "city": "San Francisco",
    "state": "CA"
  },

  "responses": []
}
```

Creates a task.

### Parameters

Parameter | Type | Description
--------- | ---- | -----------
data | dictionary | A dictionary of named fields that get shown (according to the project template) to workers when working on the task. (required)

## List all tasks

> Definition

```
GET https://app.surgehq.ai/api/projects/{{PROJECT_ID}}/tasks
GET https://app.surgehq.ai/api/projects/{{PROJECT_ID}}/tasks?page=n
```

> Example Request

```ruby
require 'httparty'

HTTParty.get("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/tasks", 
  basic_auth: { username: "{{YOUR_API_KEY}}" }
)
```

```python
import requests

requests.post("https://app.surgehq.ai/api/projects/17e323f1-f7e4-427c-a2d5-456743aba8/tasks", 
  auth = ("{{YOUR_API_KEY}}", "")
)
```

```shell
curl https://app.surgehq.ai/api/projects/b31ede78-afdf-4938-b775-8813453c7fc5/tasks \
  -u {{YOUR_API_KEY}}: \
  ...
```

> Example Response

```json
[
  {
    "id": "38da6bc5-a644-41b9-a964-4678bc7375c6",
    "project_id": "b31ede78-afdf-4938-b775-8813453c7fc5",
    "created_at": "2016-11-01T18:56:56.000Z",
    "is_complete": true,
    
    "fields": {
      "company": "Hybrid",
      "city": "San Francisco",
      "state": "CA"
    },

    "responses": [
      {
        "created_at": "2016-11-01T23:29:17.971Z",
        "id": "1befb37b-8e4f-42b6-8a61-2c946ad4b5ce",
        "response_data": {
          "website": "https://app.surgehq.ai",
          "category": "Technology"
        }
      }
    ]
  },
  {
    "id": "dd66e80a-88b3-4c4f-9efc-2aca8eed73db",
    "project_id": "b31ede78-afdf-4938-b775-8813453c7fc5",
    "created_at": "2016-11-01T23:23:26.016Z",
    "is_complete": true,

    "fields": {
      "company": "Google",
      "city": "Mountain View",
      "state": "CA"
    },

    "responses": [
      {
        "created_at": "2016-11-01T23:22:19.124Z",
        "id": "cc9f4263-1d0f-4730-a97e-199851b6ade5",
        "response_data": {
          "website": "https://www.google.com",
          "category": "Technology"
        }
      }
    ]
  }
]
```

Lists all tasks belonging to a project.

### Query Parameters

### Parameters

Parameter | Type | Description
--------- | ------- | -----------
page | integer | Tasks are returned in ascending `created_at` order. Each page contains a maximum of 25 tasks. Pages start at 1. (optional, default 1)

## Retrieve a task

> Definition

```
GET https://app.surgehq.ai/api/tasks/{{TASK_ID}}
```

> Example Request

```ruby
require 'httparty'

HTTParty.get("https://app.surgehq.ai/api/tasks/38da6bc5-a644-41b9-a964-4678bc7375c6", 
  basic_auth: { username: "{{YOUR_API_KEY}}" }
)
```

```python
import requests

requests.post("https://app.surgehq.ai/api/tasks/38da6bc5-a644-41b9-a964-4678bc7375c6", 
  auth = ("{{YOUR_API_KEY}}", "")
)
```

```shell
curl https://app.surgehq.ai/api/tasks/38da6bc5-a644-41b9-a964-4678bc7375c6 \
  -u {{YOUR_API_KEY}}:
```

> Example Response

```json
{
  "id": "38da6bc5-a644-41b9-a964-4678bc7375c6",
  "project_id": "b31ede78-afdf-4938-b775-8813453c7fc5",
  "created_at": "2016-11-01T18:56:56.000Z",
  "is_complete": true,
  
  "fields": {
    "company": "Hybrid",
    "city": "San Francisco",
    "state": "CA"
  },

  "responses": [
    {
      "created_at": "2016-11-01T23:29:17.971Z",
      "id": "1befb37b-8e4f-42b6-8a61-2c946ad4b5ce",
      "response_data": {
        "website": "https://app.surgehq.ai",
        "category": "Technology"
      }
    }
  ]
}
```

Retrieves a specific task you have created.

