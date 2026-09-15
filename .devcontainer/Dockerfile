# The one environment. Candidates work in it, and the grading pipeline runs
# the same published digest, so "it worked in the environment you gave me" is
# a claim either side can check.
#
# Every version comes from image-pins.env, which is copied in below and sourced
# by each install step. The two exceptions are the base image and its digest:
# a FROM line is resolved before any file can be read, so those are literals
# here and a test asserts they equal the file.
ARG DEVCONTAINER_BASE_IMAGE=mcr.microsoft.com/devcontainers/base:bookworm
ARG DEVCONTAINER_BASE_DIGEST=sha256:86165cfc170e9b2aa8df90b847127eea97b08eb9987021e6e6ec6c3a96545d7c
FROM ${DEVCONTAINER_BASE_IMAGE}@${DEVCONTAINER_BASE_DIGEST}

# Declared without a default on purpose: BuildKit fills it in from the target
# platform, and a default would silently win over it, installing x86 binaries
# on an arm64 base.
ARG TARGETARCH

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Early, and on its own layer: it is small, and every step below depends on it,
# so a version change should invalidate everything that installs a version.
COPY image-pins.env /usr/local/share/image-pins.env

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
RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
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
RUN set -eux; . /usr/local/share/image-pins.env; \
    curl -fsSL -o /tmp/maven.tar.gz \
        "https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz"; \
    mkdir -p /opt/maven; \
    tar -xzf /tmp/maven.tar.gz -C /opt/maven --strip-components=1; \
    rm /tmp/maven.tar.gz
ENV PATH="/opt/maven/bin:${PATH}"

# The ENV lines above are enough for `docker run` and for every check, which
# are not login shells. Debian's /etc/profile sets PATH unconditionally, so a
# login shell throws them away: an interactive terminal in a devcontainer is a
# login shell, and without this a candidate opens one and finds java and mvn
# missing, while `postCreateCommand` and the whole grading run find them fine.
# Only these two matter here; everything else installs into /usr/local/bin,
# which /etc/profile keeps.
RUN set -eux; \
    printf 'export JAVA_HOME=/opt/java\nexport PATH="/opt/maven/bin:/opt/java/bin:${PATH}"\n' \
        > /etc/profile.d/10-challenge-toolchain.sh

RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL -o /usr/local/bin/kubectl \
        "https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/linux/${GOARCH}/kubectl"; \
    chmod +x /usr/local/bin/kubectl

RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL "https://get.helm.sh/helm-v${HELM_VERSION}-linux-${GOARCH}.tar.gz" \
        -o /tmp/helm.tar.gz; \
    tar -xzf /tmp/helm.tar.gz -C /tmp; \
    mv "/tmp/linux-${GOARCH}/helm" /usr/local/bin/helm; \
    rm -rf /tmp/helm.tar.gz "/tmp/linux-${GOARCH}"

RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv${KUSTOMIZE_VERSION}/kustomize_v${KUSTOMIZE_VERSION}_linux_${GOARCH}.tar.gz" \
        -o /tmp/kustomize.tar.gz; \
    tar -xzf /tmp/kustomize.tar.gz -C /usr/local/bin; \
    rm /tmp/kustomize.tar.gz

RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/opentofu/opentofu/releases/download/v${TOFU_VERSION}/tofu_${TOFU_VERSION}_linux_${GOARCH}.zip" \
        -o /tmp/tofu.zip; \
    unzip -q /tmp/tofu.zip -d /usr/local/bin tofu; \
    chmod +x /usr/local/bin/tofu; \
    rm /tmp/tofu.zip

RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL -o /usr/local/bin/kind \
        "https://kind.sigs.k8s.io/dl/v${KIND_VERSION}/kind-linux-${GOARCH}"; \
    chmod +x /usr/local/bin/kind

# The Docker client only. A daemon arrives from elsewhere: the devcontainer
# docker-in-docker feature provides one here, and one is mounted in when this
# image runs unattended. Neither supplies the client itself.
RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
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
RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/gitleaks/gitleaks/releases/download/v${GITLEAKS_VERSION}/gitleaks_${GITLEAKS_VERSION}_linux_${GITLEAKSARCH}.tar.gz" \
        -o /tmp/gitleaks.tar.gz; \
    tar -xzf /tmp/gitleaks.tar.gz -C /usr/local/bin gitleaks; \
    rm /tmp/gitleaks.tar.gz

RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL -o /usr/local/bin/hadolint \
        "https://github.com/hadolint/hadolint/releases/download/v${HADOLINT_VERSION}/hadolint-Linux-${MIXEDARCH}"; \
    chmod +x /usr/local/bin/hadolint

RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/terraform-linters/tflint/releases/download/v${TFLINT_VERSION}/tflint_linux_${GOARCH}.zip" \
        -o /tmp/tflint.zip; \
    unzip -q /tmp/tflint.zip -d /usr/local/bin tflint; \
    chmod +x /usr/local/bin/tflint; \
    rm /tmp/tflint.zip

RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/open-policy-agent/conftest/releases/download/v${CONFTEST_VERSION}/conftest_${CONFTEST_VERSION}_Linux_${MIXEDARCH}.tar.gz" \
        -o /tmp/conftest.tar.gz; \
    tar -xzf /tmp/conftest.tar.gz -C /usr/local/bin conftest; \
    rm /tmp/conftest.tar.gz

RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL -o /usr/local/bin/hcl2json \
        "https://github.com/tmccombs/hcl2json/releases/download/v${HCL2JSON_VERSION}/hcl2json_linux_${GOARCH}"; \
    chmod +x /usr/local/bin/hcl2json

RUN set -eux; . /usr/local/share/image-pins.env; . /usr/local/share/arch.env; \
    curl -fsSL "https://github.com/yannh/kubeconform/releases/download/v${KUBECONFORM_VERSION}/kubeconform-linux-${GOARCH}.tar.gz" \
        -o /tmp/kubeconform.tar.gz; \
    tar -xzf /tmp/kubeconform.tar.gz -C /usr/local/bin kubeconform; \
    rm /tmp/kubeconform.tar.gz

# Its own virtualenv: checkov pulls a large dependency tree and must not be
# able to disturb whatever Python the candidate installs later.
RUN set -eux; . /usr/local/share/image-pins.env; \
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
