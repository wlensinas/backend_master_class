# backend_master_class

## Create diagram scheme on https://dbdiagram.io/

```
Table accounts as A {
  id bigserial [pk]
  owner varchar [not null]
  balance bigint [not null]
  currency varchar [not null]
  created_at timestamptz [not null, default: `now()`]
  
  Indexes { 
    owner
  }
}

Table entries {
  id bigserial [pk]
  account_id bigint [ref: > A.id, not null]
  amount bigint [not null, note: 'can be negative or positive']
  created_at timestamptz [not null, default: `now()`]
  
  Indexes { 
    account_id
  }
}

Table transfers {
  id bigserial [pk]
  from_account_id bigint [ref: > A.id, not null]
  to_account_id bigint [ref: > A.id, not null]
  amount bigint [not null, note: 'must be positive']
  created_at timestamptz [not null, default: `now()`]
  
  Indexes { 
    from_account_id
    to_account_id
    (from_account_id, to_account_id)
  }
}
```

Then save the export to postgres

```sql
CREATE TABLE "accounts" (
  "id" bigserial PRIMARY KEY,
  "owner" varchar NOT NULL,
  "balance" bigint NOT NULL,
  "currency" varchar NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE TABLE "entries" (
  "id" bigserial PRIMARY KEY,
  "account_id" bigint NOT NULL,
  "amount" bigint NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE TABLE "transfers" (
  "id" bigserial PRIMARY KEY,
  "from_account_id" bigint NOT NULL,
  "to_account_id" bigint NOT NULL,
  "amount" bigint NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE INDEX ON "accounts" ("owner");

CREATE INDEX ON "entries" ("account_id");

CREATE INDEX ON "transfers" ("from_account_id");

CREATE INDEX ON "transfers" ("to_account_id");

CREATE INDEX ON "transfers" ("from_account_id", "to_account_id");

COMMENT ON COLUMN "entries"."amount" IS 'can be negative or positive';

COMMENT ON COLUMN "transfers"."amount" IS 'must be positive';

ALTER TABLE "entries" ADD FOREIGN KEY ("account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("from_account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("to_account_id") REFERENCES "accounts" ("id");

```

## Golang migration

In this part we watch how to create a migration file and run it.

CLI: https://github.com/golang-migrate/migrate

1. Install with brew on macos: `brew install golang-migrate`
2. create migrations: `migrate create -ext sql -dir db/migrations -seq init_schema`

## sqlc

This a CLI for make easy creation of models and manipulate with simple SQL for data manipulation.

url: https://docs.sqlc.dev/en/stable/tutorials/getting-started-postgresql.html

1. Install the CLI: `brew install sqlc`
2. Test if the cli was installed: `sqlc version`
3. Inits the sqlc configuration file: `sqlc init` this generate a empty configuration file. Then copy and paste this config:

```yml
version: 1
packages:
  - path: "./db/sqlc"
    name: "db"
    engine: "postgresql"
    schema: "./db/migrations/"
    queries: "./db/query/"
    emit_json_tags: true
    emit_prepared_queries: false
    emit_interface: false
    emit_exact_table_names: false
```

* path: indicate that where the cli generate the code
* name: package name
* engine: the database engine, in this case `postgresql`
* schema: the folder where is the schema of the database for generate the `models.go`
* queries: the folder where is at minimum one query. we have to specified the action and query like this:

```sql
-- name: GetAuthor :one // many or exec
SELECT * FROM authors
WHERE id = $1 LIMIT 1;
```

4. Generate code with the CLI: `sqlc generate` this create the followings files:
* `db.go` 
* `<model>.sql.go` for example `account.sql.go`
* `models.go` from the schema create structs.


# Test

use this lib: https://github.com/stretchr/testify

# Popular web frameworks

- Gin https://gin-gonic.com/ ✅
- Beego https://beego.vip/ ✅
- Echo https://echo.labstack.com/ ✅
- Revel https://revel.github.io/ ✅
- Martini https://github.com/go-martini/martini ⚠️ (No longer maintained)
- Fiber https://docs.gofiber.io/ ✅
- Buffalo https://gobuffalo.io/ ✅

# Popular HTTP routers

- FastHttp https://github.com/valyala/fasthttp ✅
- Gorilla Mux https://github.com/gorilla/mux ⚠️ [(Looking for a New Maintainer)](https://github.com/gorilla/mux/issues/659)
- HttpRouter https://github.com/julienschmidt/httprouter ✅
- Chi https://github.com/go-chi/chi ✅

# Gin

1. Install `go get -u github.com/gin-gonic/gin``

# GOMOCK

## Install

We use the CLI por generate the mock file

1. Go to https://github.com/golang/mock for review the new form of install, at this moment is executing this: `go install github.com/golang/mock/mockgen@v1.6.0`
2. Generate the mock: `mockgen -package mockdb -destination db/mock/store.go github.com/wlensinas/backend_master_class/db/sqlc Store`

If you execute one test function and need to watch the verbouse output:
1. Edit Preferences: command+p
2. Enter `> Preferences: Open Settings (JSON)`
3. Add this line `"go.testFlags": ["-v"]`
4. Save it

Executes again and them you have this kind of output:

```bash
Running tool: /usr/local/bin/go test -timeout 30s -run ^TestGetAccountAPI$ github.com/wlensinas/backend_master_class/api -v

=== RUN   TestGetAccountAPI
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /accounts                 --> github.com/wlensinas/backend_master_class/api.(*Server).createAccount-fm (3 handlers)
[GIN-debug] GET    /accounts/:id             --> github.com/wlensinas/backend_master_class/api.(*Server).getAccount-fm (3 handlers)
[GIN-debug] GET    /accounts                 --> github.com/wlensinas/backend_master_class/api.(*Server).ListAccounts-fm (3 handlers)
[GIN] 2022/06/13 - 15:17:00 | 200 |     250.307µs |                 | GET      "/accounts/27"
--- PASS: TestGetAccountAPI (0.00s)
PASS
ok  	github.com/wlensinas/backend_master_class/api	0.651s
```

# Docker

## Commands

1. Pull postgresql image `docker pull postgres:12-alpine`
2. List images: `docker images`
3. Run docker image: `docker run --name postgres12 -p 5432:5432 -e POSTGRES_USER=root -e POSTGRES_PASSWORD=secret -d postgres:12-alpine`
4. Connect to the container: `docker exec -it postgres12 psql -U root`
5. Execute some select for testing porpouse: `select now();` and then quit `\q`

Tips:
* delete images with ´none´ in the name: `docker rmi $(docker images | tail -n +2 | awk '$1 == "<none>" {print $'3'}')`
* Inspect container postgres12: `docker container inspect postgres12`
* Inspect container app: `docker container inspect simplebank`

Operation with containers dockers:
* stop container: `docker stop <name or hash>` in this case `docker stop postgres12`
* list containers actives and stopped: `docker ps -a`
* start the container: `docker start <name or hash>`
* enter to the container terminal: `docker exec -it <name or hash> /bin/sh`
* create db without enter to the container: `docker exec -it <name or hash> createdb --username=root --owner=root name_db`
For example: `docker exec -it postgres12 createdb --username=root --owner=root simple_bank`

## For development in docker

1. Create network: `docker network create bank-network`
2. Connect postgres12 container to the network: `docker network connect bank-network postgres12`
3. Build the image for the app: `docker build -t simplebank:latest .`

* Execute the app for development: `docker run --name simplebank --network bank-network -p 8080:8080 -e DB_SOURCE="postgresql://root:secret@postgres12:5432/simple_bank?sslmode=disable" simplebank:latest`
* Execute the app for test production: `docker run --name simplebank --network bank-network -p 8080:8080 -e GIN_MODE=release -e DB_SOURCE="postgresql://root:secret@postgres12:5432/simple_bank?sslmode=disable" simplebank:latest`
