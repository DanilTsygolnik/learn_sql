northwind-setup: db-reset load-data

load-data:
	psql northwind < northwind.sql

db-reset:
	dropdb northwind || true
	createdb northwind
