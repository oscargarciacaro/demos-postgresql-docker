FROM postgres:10.2-alpine
COPY docker-entrypoint.sh /usr/local/bin/

RUN apk add --no-cache bash
COPY docker-healthcheck /usr/local/bin/
#HEALTHCHECK CMD ["docker-healthcheck"]
HEALTHCHECK --interval=5s CMD docker-healthcheck || exit 1
