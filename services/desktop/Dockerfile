# syntax=docker/dockerfile:1
#
# Alpine Linux 3.24 with KDE Plasma (Wayland), in the browser, for Railway.
#
# Base: linuxserver.io webtop, release alpine-kde-b1bc61ac-ls152 (built 2026-11-03, git
# revision 830ffa18 of alpine-kde of linuxserver/docker-webtop).
# The digest is the multi-arch index, identical on lscr.io, ghcr.io and
# Docker Hub. To upgrade, replace tag and digest together with a newer
# "alpine-kde-<hash>-ls<n>" release and redeploy.
FROM lscr.io/linuxserver/webtop:alpine-kde-b1bc61ac-ls152@sha256:73e40a986740fcf2f920834ad992f2e9a04d88ff69fd4a243139cbf8e93776c7

# Image defaults for Railway. Variables set on the Railway service win.
#  - PUID/PGID 1000: fixed owner of the /config volume. Do not change them
#    after the first boot, because files already on the volume keep the old ids.
#  - START_DOCKER=false: Railway gives no privileged containers, so the
#    bundled Docker daemon can never run there.
#  - SELKIES_ALLOWED_ORIGINS empty: the image ships "*", which lets any
#    website open the desktop's WebSocket with the browser's cached login.
#    Empty is Selkies' own default: same-origin only.
#  - SELKIES_ENABLE_BASIC_AUTH=false: nginx does the login. Selkies listens
#    on loopback only.
ENV PUID=1000 \
    PGID=1000 \
    START_DOCKER=false \
    SELKIES_ALLOWED_ORIGINS="" \
    SELKIES_ENABLE_BASIC_AUTH=false \
    TITLE="Alpine 3.24 KDE Plasma Desktop" \
    CUSTOM_USER=alpine \
    PORT=3000

COPY patch-image.sh /tmp/railway-desktop/patch-image.sh
COPY nginx-origin-map.conf nginx-server-hardening.conf /tmp/railway-desktop/
COPY railway-start.sh /usr/local/bin/railway-start
COPY pkg-restore.sh /usr/local/bin/railway-pkg-restore

# Build-time edits of the image's nginx files. The script checks every
# anchor it edits and fails the build if the upstream files have changed.
RUN bash /tmp/railway-desktop/patch-image.sh \
 && rm -rf /tmp/railway-desktop \
 && chmod 0755 /usr/local/bin/railway-start /usr/local/bin/railway-pkg-restore

EXPOSE 3000

# Railway runs its own init (podman-init) as PID 1, and the image's /init
# (s6-overlay 3.2.1.0) refuses to start unless it is PID 1. railway-start
# brings up the same s6 supervision tree without that check. catatonit
# (part of the linuxserver.io base image) reaps zombies and forwards signals,
# as a child subreaper when it is not PID 1 itself.
ENTRYPOINT ["/usr/bin/catatonit", "--", "/usr/local/bin/railway-start"]
