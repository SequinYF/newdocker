FROM armhf/python:2.7-alpine

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
RUN echo 'http://mirrors.aliyun.com/alpine/v3.9/community/'>/etc/apk/repositories && \
echo 'http://mirrors.aliyun.com/alpine/v3.9/main/'>>/etc/apk/repositories
RUN apk update

ARG ADDITIONAL_PACKAGE
# Alternatively use ADD https:// (which will not be cached by Docker builder)
RUN apk --no-cache add curl ${ADDITIONAL_PACKAGE} \
    && echo "Pulling watchdog binary from Github." \
    && curl -sSL https://github.com/openfaas/faas/releases/download/0.9.14/fwatchdog-armhf > /usr/bin/fwatchdog \
    && chmod +x /usr/bin/fwatchdog \
    && apk del curl --no-cache

# Add non root user
RUN addgroup -S app && adduser app -S -G app
RUN chown app /home/app

USER app
ENV PATH=$PATH:/home/app/.local/bin

WORKDIR /home/app/

COPY index.py           .
COPY requirements.txt   .
RUN pip install --user -r requirements.txt

RUN mkdir -p function
RUN touch ./function/__init__.py

WORKDIR /home/app/function/
COPY function/requirements.txt	.
RUN pip install --user -r requirements.txt

WORKDIR /home/app/
COPY function           function


ENV fprocess="python index.py"
EXPOSE 8080

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
