FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY init/01-CreateScheme.sql /docker-entrypoint-initdb.d
COPY init/02-InsertData.sql /docker-entrypoint-initdb.d

EXPOSE 5432
