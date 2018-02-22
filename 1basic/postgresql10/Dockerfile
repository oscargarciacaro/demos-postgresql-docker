FROM postgres:10.2-alpine

COPY sql/* /docker-entrypoint-initdb.d

COPY docker-healthcheck /usr/local/bin/

HEALTHCHECK --interval=5s CMD docker-healthcheck || exit 1
