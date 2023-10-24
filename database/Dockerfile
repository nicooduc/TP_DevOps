FROM postgres:14.1-alpine

COPY /SQL/*.sql /docker-entrypoint-initdb.d

ENV POSTGRES_DB=${{secrets.POSTGRES_DB}} \
   POSTGRES_USER=${{secrets.POSTGRES_USER}} \
   POSTGRES_PASSWORD=${{secrets.POSTGRES_PASSWORD}}
