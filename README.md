
# Zenhub API

This document outlines the setup and usage of ZenHub's public API: a RESTful API with JSON responses.

- [Authentication](#authentication)
- [Endpoints](#endpoints)
  - [Get issue data](#get-issue-data)
  - [Get issue events](#get-issue-events)
  - [Get Epics for a repository](#get-repo-epics)
  - [Get Epic data](#get-epic-data)
  - [Get the ZenHub Board data for a
   repository](#get-the-zenhub-board-data-for-a-repository)
- [API limits](#api-limits)
- [Errors](#errors)
- [Contact us](#contact-us)

## Authentication

### For ZenHub IO users
All requests to the API need an API token. Generate a token in the [Settings](https://dashboard.zenhub.io/#/settings) section of your ZenHub Dashboard.
Note: Each user may only have one token, so generating a new token will make any previous tokens invalid.

The token is sent in the `X-Authentication-Token` header. For example, using `curl` it'd be `curl -H 'X-Authentication-Token: TOKEN' URL`. Alternatively, you can send the token in the URL using the `access_token` query string attribute. To do so, add ```?access_token=TOKEN``` to any url.

### For ZenHub Enterprise users
Please follow the instruction in https://{{zenhub_enterprise_host}}/setup/howto/api

## Endpoints

### Get issue data

```
GET https://api.zenhub.io/p1/repositories/:repo_id/issues/:issue_number
```

Please note: `repo_id` is the ID of the repository, not its full name. For example, the ID of the `ZenHubIO/support` repository is `13550592`.
To find out the ID of your repository, use [GitHub's API](https://developer.github.com/v3/repos/#get).

Issue number is the same as displayed in your GitHub Issues page. For example, to fetch the [Zenhub Public API](https://github.com/ZenHubIO/support/issues/172) issue information, the URL would be `https://api.zenhub.io/p1/repositories/13550592/issues/172`.

The endpoint returns that issue's assigned _Time Estimate_ (if applicable), its _Pipeline_ in the Board, an _is epic_ flag (true/false), as well as any _+1s_.

NOTE: Closed issues might take up to 1 minute to show up in the Closed pipeline. Similarly reopened issues might take up to 1 minute to show in the right pipeline.

Here is an example of returned JSON data:
```json
{
  "estimate": {
    "value": 8
  },
  "plus_ones": [
    {
      "user_id": 16717,
      "created_at": "2015-12-11T18:43:22.296Z"
    }
  ],
  "pipeline": {
    "name": "In Progress"
  },
  "is_epic": true
}
```

### Get issue events

```
GET https://api.zenhub.io/p1/repositories/:repo_id/issues/:issue_number/events
```

Please note: `repo_id` is the ID of the repository, not its full name. For example, the ID of the `ZenHubIO/support` repository is `13550592`.
To find out the ID of your repository, use [GitHub's API](https://developer.github.com/v3/repos/#get).

Issue number is the same as displayed in your GitHub Issues page. For example, to fetch the [Zenhub Public API](https://github.com/ZenHubIO/support/issues/172) issue information, the URL would be `https://api.zenhub.io/p1/repositories/13550592/issues/172`.

The endpoint returns that issue's events, sorted by most recent. Each event contains the _User ID_ of the person who performed the change, the _Creation Date_ of the event, and _Type_. Type can be either `estimateIssue` or `transferIssue`. Old and new values are included for both event types.

Here is an example of returned JSON data:
```json
[
  {
    "user_id": 16717,
    "type": "estimateIssue",
    "created_at": "2015-12-11T19:43:22.296Z",
    "from_estimate": {
      "value": 8
    }
  },
  {
    "user_id": 16717,
    "type": "estimateIssue",
    "created_at": "2015-12-11T18:43:22.296Z",
    "from_estimate": {
      "value": 4
    },
    "to_estimate": {
      "value": 8
    }
  },
  {
    "user_id": 16717,
    "type": "estimateIssue",
    "created_at": "2015-12-11T13:43:22.296Z",
    "to_estimate": {
      "value": 4
    }
  },
  {
    "user_id": 16717,
    "type": "transferIssue",
    "created_at": "2015-12-11T12:43:22.296Z",
    "from_pipeline": {
      "name": "Backlog"
    },
    "to_pipeline": {
      "name": "In progress"
    }
  },
  {
    "user_id": 16717,
    "type": "transferIssue",
    "created_at": "2015-12-11T11:43:22.296Z",
    "to_pipeline": {
      "name": "Backlog"
    }
  }
]
```

### Get the ZenHub Board data for a repository

```
GET https://api.zenhub.io/p1/repositories/:repo_id/board
```

Note: The `repo_id` is the ID of the repository, not the full name. For example, the ID of the `ZenHubIO/support` repository is `13550592`. Use [GitHub's API](https://developer.github.com/v3/repos/#get) to find out your repository's ID.

For example, the URL to fetch the [ZenHubIO/support](https://github.com/ZenHubIO/support#boards) board would be `https://api.zenhub.io/p1/repositories/13550592/board`.

The endpoint returns the Board's pipelines, plus the issues contained within each pipeline. For each issue it returns the _issue number_, its _position_ in the board, an _is epic_ flag(true/false), and its _Time Estimate_ (if one is assigned).

Even if the issues are returned in the right order, the _position_ can't be guessed from its index. Notice some issues won't have _position_ – this is because they have not been prioritized in the Board.

Note: Closed issues might take up to 1 minute to show up in the Closed pipeline. Similarly reopened issues might take up to 1 minute to show in the right pipeline.

This is an example of returned JSON data:
```json
{
  "pipelines": [
    {
      "name": "New Issues",
      "issues": [
        {
          "issue_number": 279,
          "estimate": {
            "value": 40
          },
          "position": 0,
          "is_epic": true
        },
        {
          "issue_number": 142,
          "is_epic": false
        }
      ]
    },
    {
      "name": "Backlog",
      "issues": [
        {
          "issue_number": 303,
          "estimate": {
            "value": 40
          },
          "position": 3,
          "is_epic": false
        }
      ]
    },
    {
      "name": "To Do",
      "issues": [
        {
          "issue_number": 380,
          "estimate": {
            "value": 1
          },
          "position": 0,
          "is_epic": true
        },
        {
          "issue_number": 284,
          "position": 2,
          "is_epic": false
        },
        {
          "issue_number": 329,
          "estimate": {
            "value": 8
          },
          "position": 7,
          "is_epic": false
        }
      ]
    }
  ]
}
```

### Get Epics for a repository

```
GET https://api.zenhub.io/p1/repositories/:repo_id/epics
```

Please note: `repo_id` is the ID of the repository, not its full name. For example, the ID of the `ZenHubIO/support` repository is `13550592`.
To find out the ID of your repository, use [GitHub's API](https://developer.github.com/v3/repos/#get).

For example, the URL to fetch the [ZenHubIO/support](https://github.com/ZenHubIO/issues) epics would be `https://api.zenhub.io/p1/repositories/13550592/epics`.

The endpoint returns an array of the repository's epics. For each epic, _issue number_, _repository ID_, and the _GitHub issue URL_ is provided.

Note: If an issue is only an issue belonging to an epic (and not an epic itself), it is not an epic, and will not be included in the return array.

Here is an example of returned JSON data:

```json
{
  "epic_issues": [
    {
      "issue_number": 3953,
      "repo_id": 1234567,
      "issue_url": "https://github.com/RepoOwner/RepoName/issues/3953"
    },
    {
      "issue_number": 1342,
      "repo_id": 1234567,
      "issue_url": "https://github.com/RepoOwner/RepoName/issues/1342"
    },
  ]
}
```

### Get Epic data

```
GET https://api.zenhub.io/p1/repositories/:repo_id/epics/:epic_id
```
Please note: `repo_id` is the ID of the repository, not its full name. For example, the ID of the `ZenHubIO/support` repository is `13550592`.
To find out the ID of your repository, use [GitHub's API](https://developer.github.com/v3/repos/#get).

`epic_id` is the GitHub issue number (you may fetch the list of epics using **Get Epics for a repository** endpoint).

The endpoint returns the _total estimate epic value_ (estimate of epic + all issue estimates belonging to it summed), the _estimate of the epic_, _pipeline name_ the epic issue is in, and the _issues_ belonging to it. For each issue belonging to the epic, its _issue number_, _repo id_, _estimate value_, _is epic_ flag (true/false) are provided; in addition, if the issue is from the same repository as the epic, the ZenHub Board's _pipeline name_ (from the repo the epic is in) is attached.

NOTE: if an issue belonging to the epic is from a different repository than the epic, the pipeline name is not attached.

```json
{
  "total_epic_estimates": {
    "value": 60
  },
  "issues": [
    {
      "issue_number": 3161,
      "is_epic": true,
      "repo_id": 1099029,
      "estimate": {
        "value": 40
      },
      "pipeline": {
        "name": "New Issues"
      }
    },
    {
      "issue_number": 2,
      "is_epic": false,
      "repo_id": 1234567,
      "estimate": {
        "value": 10
      },
      "pipeline": {
        "name": "New Issues"
      }
    },
    {
      "issue_number": 1,
      "is_epic": false,
      "repo_id": 1234567
    },
    {
      "issue_number": 6,
      "is_epic": false,
      "repo_id": 1234567
    },
    {
      "issue_number": 7,
      "is_epic": true,
      "repo_id": 9876543
    }
  ],
  "estimate": {
    "value": 10
  },
  "pipeline": {
    "name": "Backlog"
  }
}
```



## API limits

We allow 100 requests per minute to our API. All requests responses include some headers related to this limitation.

Header | Meaning
------ | -------
X-RateLimit-Limit | Total number of requests allowed before the reset time
X-RateLimit-Used | Number of requests sent in the current cycle. Will be set to 0 at the reset time.
X-RateLimit-Reset | Time in UTC epoch seconds when the usage gets reset.

To avoid time differences between your computer and our servers, we suggest to use the `Date` header in the response to know exactly when the limit is reset.

## Errors

The ZenHub API can return the following errors:

Status Code | Meaning
----------- | -------
401 | The token is not valid. See [Authentication](#authentication).
403 | Reached request limit to the API. See [API Limits](#api-limits).
404 | Not found.

## Contact us

We'd love to hear from you. If you have any questions, concerns, or ideas related to the ZenHub API, open an issue in our  [Support](https://github.com/ZenHubIO/support/issues#boards) repo or find us on [Twitter](http://www.twitter.com/zenhubio).
