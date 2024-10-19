FROM alpine

COPY entrypoint.sh /entrypoint.sh

# The version of `jq` in `ubuntu-latest` is 1.6, which is too old
# for --nul-output.

# I don't know whether `ash` supports the `$'\0'` syntax, and can't easily
# find out.

RUN apk add --no-cache jq bash curl

ENTRYPOINT ["/entrypoint.sh"]
