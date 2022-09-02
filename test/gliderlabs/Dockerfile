ARG STACK_VERSION=
FROM gliderlabs/herokuish:latest$STACK_VERSION as builder
COPY . /tmp/app
ARG BUILDPACK_URL
ENV USER=herokuishuser
RUN /bin/herokuish buildpack build

FROM gliderlabs/herokuish:latest$STACK_VERSION as tester
# or if you use cypress
# FROM ghcr.io/acdh-oeaw/herokuish-for-cypress/main:latest$STACK_VERSION as tester
COPY . /tmp/app
ARG BUILDPACK_URL
ENV PORT=8984
ENV USER=herokuishuser
RUN /bin/herokuish buildpack test

FROM gliderlabs/herokuish:latest$STACK_VERSION
COPY --chown=herokuishuser:herokuishuser --from=builder /app /app
COPY --from=tester /etc/lsb-release /etc/lsb-release
ENV PORT=8984
ENV USER=herokuishuser
EXPOSE 8984
CMD ["/bin/herokuish", "procfile", "start", "web"]