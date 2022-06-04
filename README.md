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
  created_at timestamptz [default: `now()`]
  
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
  "created_at" timestamptz DEFAULT (now())
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


## Docker

1. Pull postgresql image `docker pull postgres:12-alpine`
2. List images: `docker images`
3. Run docker image: `docker run --name postgres12 -p 5432:5432 -e POSTGRES_USER=root -e POSTGRES_PASSWORD=secret -d postgres:12-alpine`
4. Connect to the container: `docker exec -it postgres12 psql -U root`
5. Execute some select for testing porpouse: `select now();` and then quit `\q`

Tips:
* delete images with ´none´ in the name: `docker rmi $(docker images | tail -n +2 | awk '$1 == "<none>" {print $'3'}')`

Operation with containers dockers:
* stop container: `docker stop <name or hash>` in this case `docker stop postgres12`
* list containers actives and stopped: `docker ps -a`
* start the container: `docker start <name or hash>`
* enter to the container terminal: `docker exec -it <name or hash> /bin/sh`
* create db without enter to the container: `docker exec -it <name or hash> createdb --username=root --owner=root name_db`
For example: `docker exec -it postgres12 createdb --username=root --owner=root simple_bank`

## Golang migrate

In this part we watch how to create a migration file and run it.

CLI: https://github.com/golang-migrate/migrate

1. Install with brew on macos: `brew install golang-migrate`
2. create migrations: `migrate create -ext sql -dir db/migrations -seq init_schema`

## sqlc

url: https://docs.sqlc.dev/en/stable/tutorials/getting-started-postgresql.html

# Test

use this lib: https://github.com/stretchr/testify