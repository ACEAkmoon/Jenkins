#take a basic image of the alpine
FROM alpine

RUN apk update
RUN apk add nextcloud-sqlite
RUN apk add nextcloud-pgsql postgresql postgresql-client
RUN /etc/init.d/postgresql setup && \
	/etc/init.d/postgresql start
RUN psql -U postgres && \
	CREATE USER mycloud WITH PASSWORD 'test123';
	ALTER ROLE mycloud CREATEDB;
	\q

#blocks the death of the container
CMD tail -f /dev/null

