# La Roqueria

After much deliberation you've decided to open _La Roqueria_, a climbing gym. To start simple you'll be offering your customers three subscription plans to choose from (A customer can be subscribed to oa single at a time):
 - **Casual**: 1 hour/week, 10 EUR
 - **Regular**: 2 hours/week, 15 EUR
 - **Pro**: Unlimited hours/week, 50 EUR

You don't want to pay big bucks for some user management software so you've decided to write your own and see how far it can take you.

## Data
You want to capture the following information from your users:
 - Full name (name + surname)
 - Date of birth
 - Plan

## Functionality
To manage your customers the new system needs to support:
 - Subscribe a user to any of the plans
 - Update a customer's plan
 - Unsubscribe a user.
 - Get a specific customer's plan & details
 - Search users by:
   - name
   - plan

You also want analytics to be a first-class citizen of your application so you want to be able to generate a basic report including:
 - Total subscribed users
 - Total subscribed users per plan
 - Unsubscribed users
 - Total monthly earnings (see prices above)
 - Total monthly earnings per plan (see prices above)

## Implementation considerations
 - Upon registration each user will be assigned an unique ID.
 - Your language of choice is python

## Stage 1: CLI + file storage
You want to start small so a simple Command Line Interface (CLI) + storing data into a text file will suffice. It will also keep things simple and you'll be able to look at the users data directly for debugging.
Your command line interface `rq` will have the following API:
 * `create [FULL_NAME] [BIRTH_DATE(YYYY-MM-DD)] [PLAN(CASUAL|REGULAR|PRO)]`: Create a new subscription
 * `update [USER_ID] [PLAN(CASUAL|REGULAR|PRO)]`: Update an existing subscription with the specified plan
 * `delete [USER_ID]`: Delete an user's subscription
 * `get [USER_ID]`: Prints the specified user's details:
    * ID
    * Name
    * Age
    * Plan
 * `search [--name=<name>] [--plan=<plan>]`: Prints a list of all the users with the given name or plan. Only one of them can be used at the same time. If both are specified you need to desambiguate somehow.
 * `report [--json]`: Prints analytics report as specified above to the screen. If the `json` flag is provided then it prints it in json format

An example, Running the following command:
```shell
rq create "Bill Clinton" "1946/08/19" CASUAL
```
Will create a new user with name "Bill Clinton", birthdate "1946/08/19" & plan "Casual"

Because you don't want to lose your customers' details once the command exits you'll be saving the customer data into a file. Format & location are up to you.

## Stage 1.a: Shell - Optional
You are not fond of storing data into plaintext files and you happen to have a magical computer that never crashes nor shutdowns so you've decided that as an alternative to the CLI you could implement a [shell](https://en.wikipedia.org/wiki/Shell_(computing)) for your application & keep the data in memory.

The spec is the same as the CLI but now `rq` will start a shell process wher you can input the commands and data will be stored in memory for as long as the shell process is running instead of being persisted to disk.

Example:
```
$ rq
rq> create "Bill Clinton" "1946/08/19" CASUAL
User "Bill Clinton" created
rq> exit
$
```

## Stage 2: DB storage
Persisting data into a file has served you well so far but after a couple of nasty accidents caused by accidental modifications you've decided to use something a bit more reliable for your storage, so you'll be updating your application to use [sqlite](https://www.sqlite.org/index.html) as your backend storage.
[Python already includes a lot of batteries for sqlite](https://docs.python.org/3/library/sqlite3.html) pre-packaged with Python and scales well way over the number of customers you can fit in your current facilities so you suspect it'll do just fine.

**Extra**: If you look it up on the internet you'll most likely see people recommending you to use [SQLAlchemy](https://www.sqlalchemy.org/) to interact with the db. Try to do it without it first so you get to see what it looks like.

## Stage 2.a: Alternative DB storage
You've read on the internet that sqlite is not a "real database" and you're easily influenciable so you decide to use [Postgres](https://www.postgresql.org/) instead. The simplest way is to use [Docker](https://www.docker.com/) to run a [local postgres instance in a container](https://hub.docker.com/_/postgres).

Your storage layer will most likely need some tweaking to connect & interact to postgres instead of sqlite. [psycopg](https://pypi.org/project/psycopg/) is the typical postgres connector, and what SQLAlchemy & other ORMs will user under the hood

## Stage3: HTTP API
Your CLI implementation has served you well so far, but you've recently have heard about this "internet" thing and you think it might allow for customers to register remotely instead of having to walk to the gym and have you type into your computer.

You'll be implementing a simple HTTP service that will act as the backend of a future webpage you'll definitely write later. The service will have the following API:

**TODO**: OpenAPI spec for this

 - `POST /subscriptions`: Create a new subscription.  
   Body: `{full_name: str, date_of_birth: str, plan: str}`  
   Response:
    - 201 CREATED - Body: `{id: str|int}` whatever you use for the id
    - 400 BAD REQUEST - Body: `{details: str}` If the user sends some parameter that doesn't exist in the body
 - `[PUT|PATCH] /subscriptions/<id>`: Update a subscription.  
   Body: `{full_name: str|None, date_of_birth: str|None, plan: str|None}`   
   Response:
    - 200 OK - Body: `{id: str|int}` whatever you use for the id
    - 404 NOT FOUND - Body: `{details: str}` Include some error message
    - 400 BAD REQUEST - Body: `{details: str}` If the user sends some parameter that doesn't exist in the body
 - `DELETE /subscriptions/<id>`: Delete a subscription.  
   Response:
    - 204 NO CONTENT - Deleted successfully
    - 404 NOT FOUND - Body: `{details: str}` Include some error message
 - `GET /subscriptions/<id>`: Get subscription details.    
   Response:
    - 200 OK - Body: `{id: str|int, full_name: str, age: int, plan: str}` whatever you use for the id
    - 404 NOT FOUND - Body: `{details: str}` Include some error message
 - `GET /subscriptions/search?name:<name>&plan:<plan>`: Find all users with a given plan or name.    
   Response:
    - 200 OK - Body: `{users: List[{id: str|int, full_name: str, age: int, plan: str}]}` whatever you use for the id
    - 400 BAD REQUEST - Body: `{details: str}` If an invalid query parameter is passed

To implement the service you can use whatever framework although I'd advice to keep it simple (read: not Django). Some popular, small ones are:
 - [Flask](https://flask.palletsprojects.com/en/2.3.x/)
 - [FastAPI](https://fastapi.tiangolo.com/)
 - [Tornado](https://www.tornadoweb.org/en/stable/)

**Important**: You're not meant to implement a completely separate codebase for the server, but rather reuse part of the code you wrote for the CLI to get the data from the database & prepare it for the output, the server is only a frontend. If you find yourself having to rewrite most of the code you might want to think a bit how to organize it in modules & functions so you can use the same logic for both the CLI aand the server.