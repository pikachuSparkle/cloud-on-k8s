# Build the operator binary
FROM docker.elastic.co/wolfi/go:1.24.4@sha256:80347923994bc348fbbcf805a3dfe25e1d9651a11ed6f99164b77eaf8400def2 as builder

ARG VERSION
ARG SHA1
ARG SNAPSHOT
ARG GO_TAGS
ARG LICENSE_PUBKEY

WORKDIR /go/src/github.com/elastic/cloud-on-k8s

# Cache go dependencies
COPY Makefile go.mod go.sum ./
RUN --mount=type=cache,mode=0755,target=/go/pkg/mod go mod download

# Copy default eck config
COPY config/eck.yaml .

# Copy the sources
COPY pkg/ pkg/
COPY cmd/ cmd/
COPY build/$LICENSE_PUBKEY /$LICENSE_PUBKEY

# Build
RUN --mount=type=cache,mode=0755,target=/go/pkg/mod \
      CGO_ENABLED=0 GOOS=linux LICENSE_PUBKEY=/$LICENSE_PUBKEY make go-build

# ---------------------------------------------
# Copy the operator binary into a lighter image
FROM docker.elastic.co/wolfi/static:latest@sha256:797e62f43d04d792e9f930913e7d9f5a63e92bd19ca5e7e5139a692decee2dbc

ARG VERSION

# Add common ECK labels and OCI annotations to image
LABEL io.k8s.description="Elastic Cloud on Kubernetes automates the deployment, provisioning, management, and orchestration of Elasticsearch, Kibana, APM Server, Beats, and Enterprise Search on Kubernetes" \
      io.k8s.display-name="Elastic Cloud on Kubernetes" \
      org.opencontainers.image.authors="eck@elastic.co" \
      org.opencontainers.image.base.name="gcr.io/distroless/static:nonroot" \
      org.opencontainers.image.description="Elastic Cloud on Kubernetes automates the deployment, provisioning, management, and orchestration of Elasticsearch, Kibana, APM Server, Beats, and Enterprise Search on Kubernetes" \
      org.opencontainers.image.documentation="https://www.elastic.co/guide/en/cloud-on-k8s/" \
      org.opencontainers.image.licenses="Elastic-License-2.0" \
      org.opencontainers.image.source="https://github.com/elastic/cloud-on-k8s/" \
      org.opencontainers.image.title="Elastic Cloud on Kubernetes" \
      org.opencontainers.image.vendor="Elastic" \
      org.opencontainers.image.version="$VERSION" \
      org.opencontainers.image.url="https://github.com/elastic/cloud-on-k8s/"

COPY --from=builder /go/src/github.com/elastic/cloud-on-k8s/elastic-operator /elastic-operator
COPY --from=builder /go/src/github.com/elastic/cloud-on-k8s/eck.yaml         /conf/eck.yaml

# Copy NOTICE.txt and LICENSE.txt into the image
COPY *.txt /licenses/

ENTRYPOINT ["/elastic-operator"]
CMD ["manager"]
