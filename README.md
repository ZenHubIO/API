
# Zenhub API

This document outlines the setup and usage of ZenHub's public API: a RESTful API with JSON responses. 

- [Authentication](#authentication)
- [Endpoints](#endpoints)
  - [Get issue data](#get-issue-data)
  - [Get the ZenHub Board data for a repository](#get-the-zenhub-board-data-for-a-repository)
- [API limits](#api-limits)
- [Errors](#errors)
- [Contact us](#contact-us)

## Authentication

All requests to the API need an API token. Generate a token in the [Settings](https://dashboard.zenhub.io/#/settings) section of your ZenHub Dashboard.
Note: Each user may only have one token, so generating a new token will make any previous tokens invalid.

The token is sent in the `X-Authentication-Token` header. For example, using `curl` it'd be `curl -H 'X-Authentication-Token: TOKEN' URL`.

Alternatively, you can send the token in the URL using the `access_token` query string attribute. To do so, add `?access_token=TOKEN` to any url.

## Endpoints

### Get issue data

```
GET https://api.zenhub.io/p1/repositories/:repo_id/issues/:issue_number
```

Remember, the `repo_id` is the ID of the repository, not the full name. For example, the ID of the `ZenHubIO/support` repository is `13550592`.
You can use [GitHub's API](https://developer.github.com/v3/repos/#get) to find out the ID of your repository.

The issue number is the same as displayed in your GitHub Issues page. 

For example, to fetch the [Zenhub Public API](https://github.com/ZenHubIO/support/issues/172) issue information, the URL would be `https://api.zenhub.io/p1/repositories/13550592/issues/172`.

The endpoint returns that issue's assigned _Time Estimate_, the _Pipeline_ where it sits in the Board, as well as any _+1s_.

This is an example of returned JSON data:
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
  }
}
```


### Get the ZenHub Board data for a repository

```
GET https://api.zenhub.io/p1/repositories/:repo_id/board
```

Note: The `repo_id` is the ID of the repository, not the full name. For example, the ID of the `ZenHubIO/support` repository is `13550592`. Use [GitHub's API](https://developer.github.com/v3/repos/#get) to find out your repository's ID.

For example, the URL to fetch the [ZenHubIO/support](https://github.com/ZenHubIO/support#boards) would be `https://api.zenhub.io/p1/repositories/13550592/board`.

The endpoint returns the Board's pipelines, plus the issues contained within each pipeline. For each issue it returns the _issue number_, its _position_ in the board, and its _Time Estimate_ (if one is assigned).

Even if the issues are returned in the right order, the _position_ can't be guessed from its index. Notice some issues won't have _position_ – this is because they have not been prioritized in the Board. 

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
          "position": 0
        },
        {
          "issue_number": 142
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
          "position": 3
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
          "position": 0
        },
        {
          "issue_number": 284,
          "position": 2
        },
        {
          "issue_number": 329,
          "estimate": {
            "value": 8
          },
          "position": 7
        }
      ]
    }
  ]
}
```

## API limits

We allow 10 requests per minute to our API. All requests responses include some headers related to this limitation.

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
