# based on https://medium.com/@chemidy/create-the-smallest-and-secured-golang-docker-image-based-on-scratch-4752223b7324
############################
# STEP 1 get necessary packages
############################
FROM alpine:3.15 AS builder
RUN apk add --upgrade openldap-clients

# Ca-certificates is required to call HTTPS endpoints.
RUN apk update && apk add --no-cache ca-certificates && update-ca-certificates

# Set the Current Working Directory inside the container
WORKDIR /app

# Create appuser.
ENV USER=appuser
ENV UID=10001
# See https://stackoverflow.com/a/55757473/12429735RUN
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    "${USER}"


############################
# STEP 3 build a small image
############################
FROM scratch
LABEL maintainer="Lukas Paluch <fluktuid@users.noreply.github.com>"

ENV UID=10001
# import certificates from builder
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
# Import the user and group files from the builder.
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/group /etc/group
# copy libraries
COPY --from=builder --chown=$UID:$UID /usr/lib/libldap.so.2 /usr/lib/libldap.so.2
COPY --from=builder --chown=$UID:$UID /usr/lib/liblber.so.2 /usr/lib/liblber.so.2
COPY --from=builder --chown=$UID:$UID /usr/lib/libsasl2.so.3 /usr/lib/libsasl2.so.3
COPY --from=builder --chown=$UID:$UID /lib/libuuid.so.1 /lib/libuuid.so.1
# Copy our static executable.
COPY --from=builder --chown=$UID:$UID /usr/bin/ldapadd /ldapadd
# Use an unprivileged user.
USER $UID:$UID

# Run the binary.
ENTRYPOINT ['/ldapadd']
