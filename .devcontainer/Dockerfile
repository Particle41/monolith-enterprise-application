# The one environment. Candidates work in it, and the grading pipeline runs
# the same published digest, so "it worked in the environment you gave me" is
# a claim either side can check.
#
# Every ARG default below is asserted equal to devops-challenge-ops/pins.env by
# tests/test_pins.py. Change a version there and here, or the test fails.
ARG DEVCONTAINER_BASE_IMAGE=mcr.microsoft.com/devcontainers/base:bookworm
ARG DEVCONTAINER_BASE_DIGEST=sha256:86165cfc170e9b2aa8df90b847127eea97b08eb9987021e6e6ec6c3a96545d7c
FROM ${DEVCONTAINER_BASE_IMAGE}@${DEVCONTAINER_BASE_DIGEST}

# Declared without a default on purpose: BuildKit fills it in from the target
# platform, and a default would silently win over it, installing x86 binaries
# on an arm64 base.
ARG TARGETARCH

ARG JDK_BUILD=21.0.5+11
ARG MAVEN_VERSION=3.9.9
ARG KUBECTL_VERSION=1.31.2
ARG HELM_VERSION=3.16.3
ARG KUSTOMIZE_VERSION=5.5.0
ARG TOFU_VERSION=1.8.6
ARG KIND_VERSION=0.24.0
ARG DOCKER_VERSION=26.1.4
ARG DOCKER_COMPOSE_VERSION=2.27.1
ARG GITLEAKS_VERSION=8.21.2
ARG HADOLINT_VERSION=2.12.0
ARG TFLINT_VERSION=0.53.0
ARG CHECKOV_VERSION=3.3.16
ARG CONFTEST_VERSION=0.56.0
ARG HCL2JSON_VERSION=0.6.4
ARG KUBECONFORM_VERSION=0.6.7

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Every release below names the same architecture four different ways. Resolve
# it once; each later step sources this file rather than repeating the case.
RUN set -eux; \
    case "${TARGETARCH:-}" in \
        amd64) go=amd64; alt=x86_64; jdk=x64; mixed=x86_64; gitleaks=x64 ;; \
        arm64) go=arm64; alt=aarch64; jdk=aarch64; mixed=arm64; gitleaks=arm64 ;; \
        "") echo "TARGETARCH is empty: build with BuildKit" >&2; exit 1 ;; \
        *) echo "unsupported TARGETARCH=${TARGETARCH}" >&2; exit 1 ;; \
    esac; \
    printf 'GOARCH=%s\nALTARCH=%s\nJDKARCH=%s\nMIXEDARCH=%s\nGITLEAKSARCH=%s\n' \
        "$go" "$alt" "$jdk" "$mixed" "$gitleaks" > /usr/local/share/arch.env

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get install -y --no-install-recommends \
        unzip \
        jq \
        make \
        python3 \
        python3-venv \
    && rm -rf /var/lib/apt/lists/*

# Eclipse Temurin JDK 21: the build JDK this codebase is known to compile on
# once its compiler release level is raised. The upstream tree targets Java 1.7
# and no supported JDK will build it as it stands.
RUN set -eux; . /usr/local/share/arch.env; \
    url_build="$(echo "${JDK_BUILD}" | sed 's/+/%2B/')"; \
    file_build="$(echo "${JDK_BUILD}" | tr '+' '_')"; \
    curl -fsSL -o /tmp/jdk.tar.gz \
        "https://github.com/adoptium/temurin21-binaries/releases/download/jdk-${url_build}/OpenJDK21U-jdk_${JDKARCH}_linux_hotspot_${file_build}.tar.gz"; \
    mkdir -p /opt/java; \
    tar -xzf /tmp/jdk.tar.gz -C /opt/java --strip-components=1; \
    rm /tmp/jdk.tar.gz
ENV JAVA_HOME=/opt/java
ENV PATH="${JAVA_HOME}/bin:${PATH}"

# archive.apache.org rather than the mirror: dlcdn carries only the current
# release, so a pinned older Maven 404s there the day it is superseded.
RUN set -eux; \
    curl -fsSL -o /tmp/maven.tar.gz \
        "https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz"; \
    mkdir -p /opt/maven; \
    tar -xzf /tmp/maven.tar.gz -C /opt/maven --strip-components=1; \
    rm /tmp/maven.tar.gz
ENV PATH="/opt/maven/bin:${PATH}"

RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL -o /usr/local/bin/kubectl \
        "https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/linux/${GOARCH}/kubectl"; \
    chmod +x /usr/local/bin/kubectl

RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL "https://get.helm.sh/helm-v${HELM_VERSION}-linux-${GOARCH}.tar.gz" \
        -o /tmp/helm.tar.gz; \
    tar -xzf /tmp/helm.tar.gz -C /tmp; \
    mv "/tmp/linux-${GOARCH}/helm" /usr/local/bin/helm; \
    rm -rf /tmp/helm.tar.gz "/tmp/linux-${GOARCH}"

RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv${KUSTOMIZE_VERSION}/kustomize_v${KUSTOMIZE_VERSION}_linux_${GOARCH}.tar.gz" \
        -o /tmp/kustomize.tar.gz; \
    tar -xzf /tmp/kustomize.tar.gz -C /usr/local/bin; \
    rm /tmp/kustomize.tar.gz

RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/opentofu/opentofu/releases/download/v${TOFU_VERSION}/tofu_${TOFU_VERSION}_linux_${GOARCH}.zip" \
        -o /tmp/tofu.zip; \
    unzip -q /tmp/tofu.zip -d /usr/local/bin tofu; \
    chmod +x /usr/local/bin/tofu; \
    rm /tmp/tofu.zip

RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL -o /usr/local/bin/kind \
        "https://kind.sigs.k8s.io/dl/v${KIND_VERSION}/kind-linux-${GOARCH}"; \
    chmod +x /usr/local/bin/kind

# The Docker client only. Candidates get a daemon from the devcontainer
# docker-in-docker feature; the grader mounts one in. Neither supplies the
# client itself, and every Tier 0 and Tier 2 check shells out to it.
RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL "https://download.docker.com/linux/static/stable/${ALTARCH}/docker-${DOCKER_VERSION}.tgz" \
        -o /tmp/docker.tgz; \
    tar -xzf /tmp/docker.tgz -C /tmp docker/docker; \
    mv /tmp/docker/docker /usr/local/bin/docker; \
    rm -rf /tmp/docker.tgz /tmp/docker; \
    mkdir -p /usr/local/lib/docker/cli-plugins; \
    curl -fsSL -o /usr/local/lib/docker/cli-plugins/docker-compose \
        "https://github.com/docker/compose/releases/download/v${DOCKER_COMPOSE_VERSION}/docker-compose-linux-${ALTARCH}"; \
    chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

# The grading toolchain. It ships in the candidate's image on purpose: every
# check a submission faces can be run by the candidate first.
RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/gitleaks/gitleaks/releases/download/v${GITLEAKS_VERSION}/gitleaks_${GITLEAKS_VERSION}_linux_${GITLEAKSARCH}.tar.gz" \
        -o /tmp/gitleaks.tar.gz; \
    tar -xzf /tmp/gitleaks.tar.gz -C /usr/local/bin gitleaks; \
    rm /tmp/gitleaks.tar.gz

RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL -o /usr/local/bin/hadolint \
        "https://github.com/hadolint/hadolint/releases/download/v${HADOLINT_VERSION}/hadolint-Linux-${MIXEDARCH}"; \
    chmod +x /usr/local/bin/hadolint

RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/terraform-linters/tflint/releases/download/v${TFLINT_VERSION}/tflint_linux_${GOARCH}.zip" \
        -o /tmp/tflint.zip; \
    unzip -q /tmp/tflint.zip -d /usr/local/bin tflint; \
    chmod +x /usr/local/bin/tflint; \
    rm /tmp/tflint.zip

RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/open-policy-agent/conftest/releases/download/v${CONFTEST_VERSION}/conftest_${CONFTEST_VERSION}_Linux_${MIXEDARCH}.tar.gz" \
        -o /tmp/conftest.tar.gz; \
    tar -xzf /tmp/conftest.tar.gz -C /usr/local/bin conftest; \
    rm /tmp/conftest.tar.gz

RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL -o /usr/local/bin/hcl2json \
        "https://github.com/tmccombs/hcl2json/releases/download/v${HCL2JSON_VERSION}/hcl2json_linux_${GOARCH}"; \
    chmod +x /usr/local/bin/hcl2json

RUN set -eux; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/yannh/kubeconform/releases/download/v${KUBECONFORM_VERSION}/kubeconform-linux-${GOARCH}.tar.gz" \
        -o /tmp/kubeconform.tar.gz; \
    tar -xzf /tmp/kubeconform.tar.gz -C /usr/local/bin kubeconform; \
    rm /tmp/kubeconform.tar.gz

# Its own virtualenv: checkov pulls a large dependency tree and must not be
# able to disturb whatever Python the candidate installs later.
RUN set -eux; \
    python3 -m venv /opt/checkov; \
    /opt/checkov/bin/pip install --no-cache-dir "checkov==${CHECKOV_VERSION}"; \
    ln -s /opt/checkov/bin/checkov /usr/local/bin/checkov

# A stable, writable Maven repository path. The grader mounts a cache volume
# here for the offline `make check` run, and a volume inherits the ownership
# of the directory it covers, so the directory has to exist and belong to the
# unprivileged user before it is mounted over.
RUN set -eux; \
    mkdir -p /opt/m2; \
    chown vscode:vscode /opt/m2

RUN set -eux; \
    java -version; \
    mvn -v; \
    docker --version; \
    docker compose version; \
    for t in kubectl helm kustomize tofu kind gitleaks hadolint tflint \
             checkov conftest hcl2json kubeconform jq make python3; do \
        command -v "$t" >/dev/null || { echo "missing: $t" >&2; exit 1; }; \
    done

USER vscode
