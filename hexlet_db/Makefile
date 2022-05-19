load-all: load-schema load-data

load-schema:
	psql hexlet < schema.sql

load-data:
	psql hexlet < users.sql
	psql hexlet < topics.sql

db-reset:
	dropdb hexlet || true
	createdb hexlet

db-create:
	createdb hexlet

hexlet-setup: db-reset load-schema load-data
