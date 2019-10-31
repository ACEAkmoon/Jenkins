#take a basic image of the alpine
FROM alpine

#add packet python-PIP and install console-AWS
RUN apk add --no-cache curl wget busybox-extras netcat-openbsd python py-pip && \
    pip install awscli

#deinstall python-PIP
RUN apk --purge -v del py-pip

#blocks the death of the container
CMD tail -f /dev/null

