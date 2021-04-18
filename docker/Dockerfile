FROM ruby:2.7.3-alpine3.13

RUN \
    apk add gcc g++ make \
    && gem install minima jekyll bundler

WORKDIR /app/docs
COPY entrypoint /entrypoint
ENTRYPOINT ["/entrypoint"]
