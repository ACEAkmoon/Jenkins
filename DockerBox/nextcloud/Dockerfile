#take a basic image of the alpine
FROM alpine

RUN apk update
RUN apk add nextcloud-sqlite

#blocks the death of the container
CMD tail -f /dev/null

