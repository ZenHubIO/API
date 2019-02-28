# Getting support
If you have any questions or feedback, **contact [support](mailto:support@zenhub.com) for support**. The ZenHub Board for this repo has been sunset - all support inquiries should be submitted directly through the email above.

You can also submit a feature request [here](https://portal.productboard.com/zenhub).

# Overview

[Overview](#overview)

- [Root Endpoint](#root-endpoint)
- [Authentication](#authentication)
- [API Rate Limit](#api-rate-limit)
- [Errors](#errors)

[Endpoint Reference](#endpoint-reference)

- [Issues](#issues)
  - [Get Issue Data](#get-issue-data)
  - [Get Issue Events](#get-issue-events)
  - [Move an Issue Between Pipelines](#move-an-issue-between-pipelines)
  - [Set Issue Estimate](#set-issue-estimate)
- [Epics](#epics)
  - [Get Epics for a Repository](#get-epics-for-a-repository)
  - [Get Epic Data](#get-epic-data)
  - [Convert an Epic to an Issue](#convert-an-epic-to-an-issue)
  - [Convert an Issue to Epic](#convert-issue-to-epic)
  - [Add or Remove Issues from an Epic](#add-or-remove-issues-to-epic)
- [Board](#board)
  - [Get Board Data for a Repository](#get-the-zenhub-board-data-for-a-repository)
- [Milestones](#milestones)
  - [Set the Milestone Start Date](#set-milestone-start-date)
  - [Get the Milestone Start Date](#get-milestone-start-date)
- [Dependencies](#dependencies)
  - [Get Dependencies for a Repository](#get-dependencies-for-a-repository)
  - [Create a Dependency](#create-a-dependency)
  - [Remove a Dependency](#remove-a-dependency)
- [Release Reports](#release-reports)
  - [Create a Release Report](#create-a-release-report)
  - [Get a Release Report](#get-a-release-report)
  - [Get Release Reports for a Repository](#get-release-reports-for-a-repository)
  - [Edit a Release Report](#edit-a-release-report)
  - [Add a Repository to a Release Report](#add-a-repository-to-a-release-report)
  - [Remove a Repository from a Release Report](#remove-a-repository-from-a-release-report)
- [Release Report Issues](#release-report-issues)
  - [Get all the Issues in a Release Report](#get-all-the-issues-for-a-release-report)
  - [Add or Remove Issues from a Release Report](#add-or-remove-issues-to-or-from-a-release-report)

[Webhooks](#webhooks)

- [Custom Webhooks](#custom-webhooks)

[Contact Us](#contact-us)

## Root Endpoint

On Cloud, the root endpoint for the public API is `https://api.zenhub.io/`.
For ZenHub Enterprise, the root endpoint is `https://<zenhub_enterprise_host>/`.

## Authentication

All requests to the API need an API token. Generate a token in the **API Tokens** section of your ZenHub [Dashboard](https://app.zenhub.com/dashboard/tokens) (or `https://<zenhub_enterprise_host>/app/dashboard/tokens` for ZenHub Enterprise). The token is sent in the `X-Authentication-Token` header. For example, using `curl` it’d be:

```sh
curl -H 'X-Authentication-Token: TOKEN' URL
```

Alternatively, you can choose to send the token in the URL using the `access_token` query string attribute. To do so, add `?access_token=TOKEN` to any URL.

#### Notes

- Each user may only have one token, so generating a new token will invalidate previously created tokens.
- For ZenHub Enterprise users, please follow the instructions in `https://<zenhub_enterprise_host>/setup/howto/api`

## API Rate Limit

We allow a maximum of 100 requests per minute to our API. All requests responses include the following headers related to this limitation.

Header | Description
------ | -------
`X-RateLimit-Limit` | Total number of requests allowed before the reset time
`X-RateLimit-Used` | Number of requests sent in the current cycle. Will be set to 0 at the reset time.
`X-RateLimit-Reset` | Time in UTC epoch seconds when the usage gets reset.

To avoid time differences between your computer and our servers, we suggest to use the `Date` header in the response to know exactly when the limit is reset.

## Errors

The ZenHub API can return the following errors:

Status Code | Description
----------- | -------
`401` | The token is not valid. See [Authentication](#authentication).
`403` | Reached request limit to the API. See [API Limits](#api-rate-limits).
`404` | Not found.

# Endpoint Reference

#### Notes

- `repo_id` is the ID of the repository, not its full name. For example, the ID of the `ZenHubIO/API` repository is `47655910`. To find out the ID of your repository, use [GitHub’s API](https://developer.github.com/v3/repos/#get), or copy it from the URL of the Board (for this repo, the Board URL is https://github.com/ZenHubIO/API#boards?repos=47655910).

## Issues

- [Get Issue Data](#get-issue-data)
- [Get Issue Events](#get-issue-events)
- [Move an Issue Between Pipelines](#move-an-issue-between-pipelines)
- [Set Issue Estimate](#set-issue-estimate)

### Get Issue Data

Get the data for a specific issue.

#### Endpoint

`GET /p1/repositories/:repo_id/issues/:issue_number`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required
|`issue_number`|`Number`|Required

#### Example Response

```json
{
  "estimate": {
    "value": 8
  },
  "plus_ones": [
    {
      "created_at": "2015-12-11T18:43:22.296Z"
    }
  ],
  "pipeline": {
    "name": "In Progress"
  },
  "is_epic": true
}
```

#### Notes

- `plus_ones[].user_id` was removed from the response.
- Closed issues might take up to one minute to show up in the Closed Pipeline. Similarly, reopened issues might take up to one minute to show in the correct Pipeline.

### Get Issue Events

Get the events for an issue.

#### Endpoint

`GET /p1/repositories/:repo_id/issues/:issue_number/events`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required
|`issue_number`|`Number`|Required


#### Example Response

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

#### Notes

- Returns issue events, sorted by creation time, most recent first.
- Each event contains the _User ID_ of the user who performed the change, the _Creation Date_ of the event, and the event _Type_.
- Type can be either `estimateIssue` or `transferIssue`. The values before and after the event are included in the event data.

### Move an Issue Between Pipelines

Moves an issue between the Pipelines in your repository.

#### Endpoint

`POST /p1/repositories/:repo_id/issues/:issue_number/moves`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required
|`issue_number`|`Number`|Required

#### Body Parameters

|Name|Type|Comments
------------ | ------ | -------
|`pipeline_id`|`String`|Required
|`position`|`String` or `Number`| Required

#### Notes

- `pipeline_id` is the ID for one of the Pipelines in your repository (i.e: In Progress, Done, QA). In order to obtain this ID, you can use the [_Get Board Data for a Repository_](#get-the-zenhub-board-data-for-a-repository) endpoint.
- `position` can be specified as `top` or `bottom`, or a `0`-based position in the Pipeline such as `1`, which would be the second position in the Pipeline.

#### Example Request Body

```json
{
  "pipeline_id": "58bf13aba426771426665e60",
  "position": "top"
}
```

#### Example Response

Status `200` for a successful move. No response body.

### Set Issue Estimate

#### Endpoint

`PUT /p1/repositories/:repo_id/issues/:issue_number/estimate`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required
|`issue_number`|`Number`|Required

#### Body Parameters

|Name|Type|Comments
------------ | ------ | -------
|`estimate`|`Number`|Required, number representing estimate value

#### Example Request

```json
{ "estimate": 15 }
```

#### Example Response

```json
{ "estimate": 15 }
```

## Epics

- [Get Epics for a Repository](#get-epics-for-a-repository)
- [Get Epic Data](#get-epic-data)
- [Convert an Epic to an Issue](#convert-an-epic-to-an-issue)
- [Convert an Issue to Epic](#convert-issue-to-epic)
- [Add or Remove Issues from an Epic](#add-or-remove-issues-to-epic)

### Get Epics for a repository

Get all Epics for a repository

#### Endpoint

`GET /p1/repositories/:repo_id/epics`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required

#### Example Response

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

#### Notes

- The endpoint returns an array of the repository’s Epics. The issue number, repository ID,
and GitHub issue URL is provided for each Epic.
- If an issue is only an issue belonging to an Epic (and not a parent Epic), it is not considered an Epic and won’t be included in the return array.

### Get Epic Data

Get the data for an Epic issue.

#### Endpoint

`GET /p1/repositories/:repo_id/epics/:epic_id`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required
|`epic_id`|`Number`|Required, Github issue number

#### Notes

- `epic_id` is the GitHub issue number. You may fetch the list of Epics using `Get Epics for a repository` endpoint.

#### Example Response
```json
{
  "total_epic_estimates": { "value": 60 },
  "estimate": { "value": 10 },
  "pipeline": { "name": "Backlog" },
  "issues": [
    {
      "issue_number": 3161,
      "is_epic": true,
      "repo_id": 1099029,
      "estimate": { "value": 40 },
      "pipeline": { "name": "New Issues" }
    },
    {
      "issue_number": 2,
      "is_epic": false,
      "repo_id": 1234567,
      "estimate": { "value": 10 },
      "pipeline": { "name": "New Issues" }
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
  ]
}
```

#### Notes

The endpoint returns:

- the total Epic Estimate value (the sum of all the Estimates of Issues contained within
  the Epic, as well as the Estimate of the Epic itself)
- the Estimate of the Epic
- the name of the Pipeline the Epic is in
- issues belonging to the Epic

For each issue belonging to the Epic:

- issue number
- repo ID
- Estimate value
- `is_epic` flag (`true` or `false`)
- if the issue is from the same repository as the Epic, the ZenHub Board’s Pipeline name (from the repo the Epic is in) is attached.

### Convert an Epic to an Issue

Converts an Epic back to a regular issue.

#### Endpoint

`POST /p1/repositories/:repo_id/epics/:issue_number/convert_to_issue`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required
|`issue_number`|`Number`|Required, the number of the issue to be converted

#### Example Response

- `200` if the issue was converted to Epic successfully

Does not return any body in the response.

### Convert Issue to Epic

Converts an issue to an Epic, along with any issues that should be part of it.

#### Endpoint

`POST /p1/repositories/:repo_id/issues/:issue_number/convert_to_epic`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required
|`issue_number`|`Number`|Required

#### Body Parameters

|Name|Type|Comments
------------ | ------ | -------
|`issues`|`[{repo_id: Number, issue_number: Number}]`|Required, array of Objects with `repo_id` and `issue_number`

#### Example Request Body

```json
{
  "issues": [
    { "repo_id": 13550592, "issue_number": 3 },
    { "repo_id": 13550592, "issue_number": 1 }
  ]
}
```

#### Response

Does not return any body in the response.

- `200` if the issue was converted to Epic successfully
- `400` if the supplied issue is already an Epic

### Add or remove issues to Epic

Bulk add or remove issues to an Epic. The result returns which issue was added or removed from the Epic.

#### Endpoint

`POST /p1/repositories/:repo_id/epics/:issue_number/update_issues`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required
|`issue_number`|`Number`|Required

#### Body Parameters

|Name|Type|Comments
------------ | ------ | -------
|`remove_issues`|[`{repo_id: Number, issue_number: Number}`]|Required, array of Objects with `repo_id` and `issue_number`
|`add_issues`|[`{repo_id: Number, issue_number: Number}`]|Required, array of Objects with `repo_id` and `issue_number`

#### Example Request Body

```json
{
  "remove_issues": [
    { "repo_id": 13550592, "issue_number": 3 }
  ],
  "add_issues": [
    { "repo_id": 13550592, "issue_number": 2 },
    { "repo_id": 13550592, "issue_number": 1 }
  ]
}
```

#### Notes

- `remove_issues` is an array that indicates with issues we want to remove from the specified Epic. They should be specified as an array containing objects with the issue’s `repo_id` and `issue_number`.
- `add_issues` is an array that indicates with issues we want to add to the specified Epic. They should be specified as an array containing objects with the issue’s `repo_id` and `issue_number`.

#### Example Response

```json
{
  "removed_issues":[
    { "repo_id": 3887883, "issue_number": 3 }
  ],
  "added_issues":[
    { "repo_id": 3887883, "issue_number": 2 },
    { "repo_id": 3887883, "issue_number": 1 }
  ]
}
```

#### Notes

- `removed_issues` shows which issues were removed in this operation.
- `add_issues` shows which issues were added in this operation.
- Returns a `404` if the Epic doesn’t exist

## Board

- [Get Board Data for a Repository](#get-the-zenhub-board-data-for-a-repository)

### Get the ZenHub Board data for a repository

#### Endpoint

`GET /p1/repositories/:repo_id/board`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required

#### Example Response

```json
{
  "pipelines": [
    {
      "id": "595d430add03f01d32460080",
      "name": "New Issues",
      "issues": [
        {
          "issue_number": 279,
          "estimate": { "value": 40 },
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
      "id": "595d430add03f01d32460081",
      "name": "Backlog",
      "issues": [
        {
          "issue_number": 303,
          "estimate": { "value": 40 },
          "position": 3,
          "is_epic": false
        }
      ]
    },
    {
      "id": "595d430add03f01d32460082",
      "name": "To Do",
      "issues": [
        {
          "issue_number": 380,
          "estimate": { "value": 1 },
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
          "estimate": { "value": 8 },
          "position": 7,
          "is_epic": false
        }
      ]
    }
  ]
}
```

#### Notes

- The endpoint returns the Board’s pipelines, plus the issues contained within each Pipeline. It returns the issue number of each issue, their position in the Board, the `is_epic` flag (`true` or `false`), and its Estimate (if set).
- Even if the issues are returned in the right order, the position can’t be guessed from its index. Note that some issues won’t have position – this is because they have not been prioritized on your Board.
- The Board returned by the endpoint doesn’t include closed issues. To get closed issues for a repository, you can use the GitHub API. Reopened issues might take up to one minute to appear in the correct Pipeline.

## Milestones

- [Set the Milestone Start Date](#set-milestone-start-date)
- [Get the Milestone Start Date](#get-milestone-start-date)

### Set milestone start date

#### Endpoint

`POST /p1/repositories/:repo_id/milestones/:milestone_number/start_date`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required
|`milestone_number`|`Number`|Required

#### Body Parameters

|Name|Type|Comments
------------ | ------ | -------
|`start_date`| ISO8601 date string|Required

#### Example Request Body

```json
{ "start_date": "2010-11-13T01:38:56.842Z" }
```

#### Example Response

```json
{ "start_date": "2010-11-13T01:38:56.842Z" }
```

### Get milestone start date

#### Endpoint

`GET /p1/repositories/:repo_id/milestones/:milestone_number/start_date`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required
|`milestone_number`|`Number`|Required

#### Example Response

```json
{ "start_date": "2010-11-13T01:38:56.842Z" }
```

## Dependencies

- [Get Dependencies for a Repository](#get-dependencies-for-a-repository)
- [Create a Dependency](#create-a-dependency)
- [Remove a Dependency](#remove-a-dependency)

### Get Dependencies for a Repository

#### Endpoint

`GET /p1/repositories/:repo_id/dependencies`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required

#### Example Response

```json
{
  "dependencies": [
    {
      "blocking": {
        "issue_number": 3953,
        "repo_id": 1234567
      },
      "blocked": {
        "issue_number": 1342,
        "repo_id": 1234567
      }
    },
    {
      "blocking": {
        "issue_number": 5,
        "repo_id":  987
      },
      "blocked": {
        "issue_number": 1342,
        "repo_id": 1234567
      }
    },
  ]
}
```

#### Notes

- This endpoint fetches all dependencies associated to the given repository that the user has read permission to
- The endpoint takes a `repo_id` param in the URL.
- Only dependencies where the user has read permissions to both sides will be returned

## Create a Dependency

#### Endpoint

`POST /p1/dependencies`

#### Body Parameters

|Name|Type|Comments
------------ | ------ | -------
|`blocking`|`Object`| Required
|`blocking.repo_id`| `Number`| Required
|`blocking.issue_number`| `Number`| Required
|`blocked`|`Object`| Required
|`blocked.repo_id`| `Number`| Required
|`blocked.issue_number`| `Number`| Required

#### Example Request Body
```json
{
  "blocking": {
    "repo_id": 92563409,
    "issue_number": 14
  },
  "blocked": {
    "repo_id": 92563409,
    "issue_number": 13
  }
}
```

#### Example Response Body
```json
{
  "blocking": {
    "repo_id": 92563409,
    "issue_number": 14
  },
  "blocked": {
    "repo_id": 92563409,
    "issue_number": 13
  }
}
```

#### Notes

- This endpoint creates one dependency
- The endpoint takes a `dependency` in the Body (see description above).
- User needs write permission on both repositories
- Cannot create dependency that will cause cycle, or between repositories not in the same workspace
- On success: returns HTTP 200 and returns the created object

## Remove a Dependency

#### Endpoint

`DELETE /p1/dependencies`

#### Body Parameters

|Name|Type|Comments
------------ | ------ | -------
|`blocking`|`Object`| Required
|`blocking.repo_id`| `Number`| Required
|`blocking.issue_number`| `Number`| Required
|`blocked`|`Object`| Required
|`blocked.repo_id`| `Number`| Required
|`blocked.issue_number`| `Number`| Required

#### Example Request Body
```json
{
  "blocking": {
    "repo_id": 92563409,
    "issue_number": 14
  },
  "blocked": {
    "repo_id": 92563409,
    "issue_number": 13
  }
}
```

#### Notes

- This endpoint removes one dependency
- The endpoint takes a `dependency` in the Body (see description above).
- User needs write permission on both repositories
- On success: returns HTTP 204 No Content and empty body

## Release Reports

- [Create a Release Report](#create-a-release-report)
- [Get a Release Report](#get-a-release-report)
- [Get Release Reports for a Repository](#get-release-reports-for-a-repository)
- [Edit a Release Report](#edit-a-release-report)
- [Add a Repository to a Release Report](#add-a-repository-to-a-release-report)
- [Remove a Repository from a Release Report](#remove-a-repository-from-a-release-report)

### Create a Release Report

#### Endpoint

`POST /p1/repositories/:repo_id/reports/release`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required

#### Body Parameters

|Name|Type|Comments
------------ | ------ | -------
|`title`|`String`|Required
|`description`| `String`| Optional
|`start_date`| ISO8601 date string| Required
|`desired_end_date`| ISO8601 date string| Required
|`repositories`| `[Number]`| Optional

#### Example Request Body
```json
{
  "title": "Great title",
  "description": "Amazing description",
  "start_date": "2007-01-01T00:00:00Z",
  "desired_end_date": "2007-01-01T00:00:00Z",
  "repositories": [
    103707262
  ]
}
```

#### Example Response

```json
{
  "release_id": "59dff4f508399a35a276a1ea",
  "title": "Great title",
  "description": "Amazing description",
  "start_date": "2007-01-01T00:00:00.000Z",
  "desired_end_date": "2007-01-01T00:00:00.000Z",
  "created_at": "2017-10-12T23:04:21.795Z",
  "closed_at": null,
  "state": "open",
  "repositories": [
    103707262
  ]
}
```

#### Notes

- **CHANGE NOTICE:** Only the repositories provided in the param and the body will be added to the Release Report - change in effect mid February 2019.
- The endpoint takes a `repo_id` param in the URL.
- Additional repository IDs can be passed in the body `repositories` parameter
- Any Boards not associated with the URL `repo_id` parameter, but associated with repositories in the request body `repositories` parameter will also be associated to the Release Report.
- The user creating the release requires push permission to the repositories in the request.

### Get a Release Report

#### Endpoint

`GET /p1/reports/release/:release_id`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`release_id`|`String`|Required

#### Example Response

```json
{
  "release_id": "59d3cd520a430a6344fd3bdb",
  "title": "Test release",
  "description": "",
  "start_date": "2017-10-01T19:00:00.000Z",
  "desired_end_date": "2017-10-03T19:00:00.000Z",
  "created_at": "2017-10-03T17:48:02.701Z",
  "closed_at": null,
  "state": "open",
  "repositories": [
    105683718
  ]
}
```

### Get Release Reports for a Repository

#### Endpoint

`GET /p1/repositories/:repo_id/reports/releases`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`repo_id`|`Number`|Required

#### Example Response

```json
[
  {
    "release_id": "59cbf2fde010f7a5207406e8",
    "title": "Great title for release 1",
    "description": "Great description for release",
    "start_date": "2000-10-10T00:00:00.000Z",
    "desired_end_date": "2010-10-10T00:00:00.000Z",
    "created_at": "2017-09-27T18:50:37.418Z",
    "closed_at": null,
    "state": "open"
  },
  {
    "release_id": "59cbf2fde010f7a5207406e8",
    "title": "Great title for release 2",
    "description": "Great description for release",
    "start_date": "2000-10-10T00:00:00.000Z",
    "desired_end_date": "2010-10-10T00:00:00.000Z",
    "created_at": "2017-09-27T18:50:37.418Z",
    "closed_at": null,
    "state": "open"
  }
]
```

### Edit a Release Report

#### Endpoint

`PATCH /p1/reports/release/:release_id`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`release_id`|`String`|Required

#### Body Parameters

|Name|Type|Comments
------------ | ------ | -------
|`title`|`String`|Required
|`description`|`String`|Optional
|`start_date`|ISO8601 date string|Optional
|`desired_end_date`| ISO8601 date string| Optional
|`state`| `String`|Optional, `open` or `closed`

#### Example Request Body

```json
{
  "title": "Amazing title",
  "description": "Amazing description",
  "start_date": "2007-01-01T00:00:00Z",
  "desired_end_date": "2007-01-01T00:00:00Z",
  "state": "closed"
}
```

#### Example Response

```json
{
  "release_id": "59d3d6438b3f16667f9e7174",
  "title": "Amazing title",
  "description": "Amazing description",
  "start_date": "2007-01-01T00:00:00.000Z",
  "desired_end_date": "2007-01-01T00:00:00.000Z",
  "created_at": "2017-10-03T18:26:11.700Z",
  "closed_at": "2017-10-03T18:26:11.700Z",
  "state": "closed",
  "repositories": [
    105683567,
    105683718
  ]
}
```

### Add a Repository to a Release Report
#### Endpoint

`POST /p1/reports/release/:release_id/repository/:repo_id`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`release_id`|`String`|Required
|`repo_id`|`Number`|Required


#### Notes
- On success, returns HTTP 200 OK and empty body


### Remove a Repository from a Release Report
#### Endpoint

`DELETE /p1/reports/release/:release_id/repository/:repo_id`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`release_id`|`String`|Required
|`repo_id`|`Number`|Required


#### Notes
- On success, returns HTTP 204 OK and empty body


### Add Workspaces to a Release Report
#### **DEPRECATED**
#### Endpoint
`PATCH /p1/reports/release/:release_id/workspaces/add`


### Remove Workspaces from Release Report
#### **DEPRECATED**
#### Endpoint
`PATCH /p1/reports/release/:release_id/workspaces/remove`

## Release Report Issues

- [Get all the Issues in a Release Report](#get-all-the-issues-for-a-release-report)
- [Add or Remove Issues from a Release Report](#add-or-remove-issues-to-or-from-a-release-report)

### Get all the Issues for a Release Report

#### Endpoint

`GET /p1/reports/release/:release_id/issues`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`release_id`|`String`|Required

#### Example Response

```json
[
  { "repo_id": 103707262, "issue_number": 2 },
  { "repo_id": 103707262, "issue_number": 3 }
]
```

### Add or Remove Issues to or from a Release Report

#### Endpoint

`PATCH /p1/reports/release/:release_id/issues`

#### URL Parameters

|Name|Type|Comments
------------ | ------ | -------
|`release_id`|`String`|Required

#### Body Parameters

|Name|Type|Comments
------------ | ------ | -------
|`add_issues`|`[{repo_id: Number, issue_number: Number}]`|Required, array of Objects with `repo_id` and `issue_number`
|`remove_issues`|`[{repo_id: Number, issue_number: Number}]`|Required, array of Objects with `repo_id` and `issue_number`

#### Note

- Both the `add_issues` and `remove_issues` keys are required, but can be an empty array when not used

#### Example Body Request

```json
{
  "add_issues": [
    { "repo_id": 103707262, "issue_number": 3 }
  ],
  "remove_issues": []
}
```

#### Example Response

```json
{
  "added": [
    { "repo_id": 103707262, "issue_number": 3 }
  ],
  "removed": []
}
```

#### Note

- Adding and removing issues can be done in the same request by populating with the `add_issues` and `remove_issues` keys.

### Webhooks

You can use our webhooks to fetch or store your ZenHub data, in real time, across services like Slack, Gitter, Spark, HipChat, or something custom!

To set up an integration, head on over to our [Dashboard](https://app.zenhub.com/dashboard), navigate to your organization, and select the **Slack & Integrations** tab. From there, you may choose one of the 5 services (Slack, HipChat, Gitter, Spark, or Custom).

For instructions, you'll notice the `How to create a webhook` link changes dynamically based on the service you select. Simply choose a repository with which to connect, add an optional description, paste your webhook, and click "Add" to save your new integration.

<img src="https://cloud.githubusercontent.com/assets/8771909/18925366/937133e2-8568-11e6-9f6a-da09edd63d16.jpg" alt="ZenHub integrations">

### Custom webhooks

Our custom webhook sends a POST request to your webhook for multiple events that occur on your ZenHub board:

#### Issue transfer

```json
{
  "type": "issue_transfer",
  "github_url": "https://github.com/ZenHubIO/support/issues/618",
  "organization": "ZenHubHQ",
  "repo": "support",
  "user_name": "ZenHubIO",
  "issue_number": "618",
  "issue_title": "ZenHub Change Log",
  "to_pipeline_name": "New Issues",
  "from_pipeline_name": "Discussion"
}
```

#### Estimate Set

```json
{
  "type": "estimate_set",
  "github_url": "https://github.com/ZenHubIO/support/issues/618",
  "organization": "ZenHubHQ",
  "repo": "support",
  "user_name": "ZenHubIO",
  "issue_number": "618",
  "issue_title": "ZenHub Change Log",
  "estimate": "8"
}
```

#### Estimate Cleared

```json
{
  "type": "estimate_cleared",
  "github_url": "https://github.com/ZenHubIO/support/issues/618",
  "organization": "ZenHubHQ",
  "repo": "support",
  "user_name": "ZenHubIO",
  "issue_number": "618",
  "issue_title": "ZenHub Change Log",
}
```

#### Issue Reprioritized

```json
{
  "type": "issue_reprioritized",
  "github_url": "https://github.com/ZenHubIO/support/issues/618",
  "organization": "ZenHubHQ",
  "repo": "support",
  "user_name": "ZenHubIO",
  "issue_number": "618",
  "issue_title": "ZenHub Change Log",
  "to_pipeline_name": "Backlog",
  "from_position": "4",
  "to_position": "0"
}
```

As an example, here's a simple Node/Express app that would be able receive the webhooks (using ngrok):

```javascript
var express = require('express');
var http = require('http');
var bodyParser = require('body-parser');
var app = express();

http.createServer(app).listen('6000', function() {
  console.log('Listening on 6000');
});

app.use(bodyParser());

app.post('*', function(req, res) {
  console.dir(req.body);
});
```

# Contact us

We’d love to hear from you. If you have any questions, concerns, or ideas related to the ZenHub API, open an issue in our [Support repo](https://github.com/ZenHubIO/support/issues#boards) or find us on [Twitter](http://www.twitter.com/ZenHubHQ).
