# Github Repos Stats

This task consists from 3 parts: 2 required and 1 bonus.

1. Import job
2. REST API
3. Front-end (UI) (this is the bonus task)

Import job imports repos data from Github REST API v3 and stores them in the database. REST API provides their data in a simplified JSON output format. The front-end is a couple of charts to display the data provided by REST API in a catchy UI.

You have to develop at least import job and REST API. If you have time/energy/inspiration, you are welcome to develop the front-end as well. All of these parts are supposed to live in a single Github repository (this one), be a monolith Rails application and be deployed as a single application to the Heroku under a [free](https://www.heroku.com/pricing) pricing plan.

As for the front-end, is should live in a `frontend` top-level dir and be a SPA (single page application) written using one of the following JS frameworks: React, Vue, Angular 1.x (most recent versions).

## Import job

This supposed to be a job (i.e. Heroku worker process) that runs in a cron-based manner and imports top 100 Github repositories (ranked by stars count) and stores it into a Postgres database. We are interested in the following attributes of the repos:

Dimensions

* name
* full_name
* description
* private
* fork
* default_branch

Metrics

* forks_count
* stargazers_count
* watchers_count
* size
* open_issues_count
* subscribers_count
  
Let this job run once an hour and use any Github credentials (maybe yours?). If such a repository already exists in our DB, overwrite its data. If not, append it.

You are required to cover all your code with tests (RSpec) and use Rubocop. You are required to use the CI service, which should run a pipeline `"Rubocop check" && "Run tests" && "Deploy to Heroku"`

## REST API

You are required to create an API.

* provides email/password authentication of users (you are free to pre-populate or even hardcode some users in the DB, the actual admin interface is not required); after login, API responds with `{ token: "20 hex chars" }`.
* for authenticated users, the API must provide a single endpoint which returns aggregated stats over our repos and can take the following params
* * filters (Hash): `filters[name]=...&filters[full_name]=...`, i.e. can filter over any **dimension**; for Boolean dimensions, valid values for filters are `true` or `false`, for String dimensions, it can be any substring of the expected value (think in terms of SQL `LIKE` operator)
  * group_by (Array): `group_by[]=fork&group_by[]=private&...`, i.e. can group by over **Boolean dimensions only**
  * metrics (Array): `metrics[]=forks_count&metrics[]=`, i.e. what metrics to include in response
  * page (Integer): self-descriptive, for pagination of the results
  * per_page (Integer): self-descriptive, for pagination of the results; if omitted, the default value is 5.
  * a client should use `Bearer: <token>` HTTP header for authentication, let's assume that tokens don't expire ever

Metrics are calculated just as `SUM()` over all repos that are filtered and grouped by according to the query criteria.

This API should be done in Rails 5, API-only mode.

On any invalid params, respond with 400 error. Let your response have the structure of `{ data: Array[Hash], meta: { page: Integer, per_page: Integer, total_pages: Integer, total_entries: Integer }`.

An example of the request-response:

Request: `GET /api/v1/repo_stats?group_by[]=private&filters[name]=Foo&metrics[]=forks_count&metrics[]=size&page=1&per_page=1`
Response:

```json
{
  "data": [ { "size": 123, "forks_count": 456, "private": true } ],
  "meta": { "page": 1, "per_page": 1, "total_pages": 2, "total_entries": 2 }
}
```

Request: `GET /api/v1/repo_stats?group_by[]=private&filters[name]=Foo&metrics[]=forks_count&metrics[]=size&page=1`
Response:

```json
{
  "data": [ { "size": 123, "forks_count": 456, "private": true }, { "size": 789, "forks_count": 42, "private": false } ],
  "meta": { "page": 1, "per_page": 2, "total_pages": 1, "total_entries": 2 }
}
```

You are required to cover all your code with tests (RSpec) and use Rubocop. You are required to use the CI service, which should run a pipeline `"Rubocop check" && "Run tests" && "Deploy to Heroku"`

## Front-end

* Authentication form (email/password)
* For autenticated users
  * Provide an ability to build a query in a form of a graphical form-based UI (it's OK to hardcode possible values for group_by, filters, metrics and other stuff)
  * Show the data returned from the API in a form of a bar chart; you can use any charts library

## General considerations

* Adhere to best practices (Ruby version management tools, REST, 12-factor apps etc.)
* Adhere to modern approaches (Rails best practices: choose services over model callbacks etc.)
* Use gems if applicable, not reinvent the wheel
* Do not overengineer but don't be quick and dirty
* For questions, use Github issues
* For contributions, use Github pull requests, prefer making them small; we use [Github flow](https://guides.github.com/introduction/flow/), and rebase feature branches against master before making a PR
* Smile :)

