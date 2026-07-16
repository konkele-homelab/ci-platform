FROM alpine:3.23

LABEL org.opencontainers.image.title="CI Test Image" \
      org.opencontainers.image.description="Minimal runtime validation image"

RUN printf '%s\n' "Hello from the CI validation container" > /message.txt

CMD ["sh", "-c", "while true; do sleep 3600; done"]