FROM erlang:21.2.1-alpine
RUN apk add git
ENV DOCKER_URL=https://download.docker.com/linux/static/stable/x86_64/docker-18.06.1-ce.tgz
RUN wget -qO - ${DOCKER_URL} | tar xzvf - docker/docker && \
    mv docker/docker /usr/local/bin && \
    rmdir docker && \
    mkdir -p /project
WORKDIR /project
ENTRYPOINT ["/bin/sh"]
