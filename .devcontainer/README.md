# The challenge environment

This directory is the environment the challenge is done in, and the same
image the submission is later run in. Anything you can check here, we see the
same way.

It carries two toolchains. What the work needs: JDK 21 (Temurin, fetched by
exact build), Maven, kubectl, Helm, Kustomize, OpenTofu and `kind`. And what
your submission is checked with, so you can run the checks yourself before
submitting: `gitleaks`, `hadolint`, `tflint`, `checkov`, `conftest`,
`hcl2json` and `kubeconform`, alongside `make`, `python3` and `jq`.

The Docker **client** is in the image; the daemon comes from the
`docker-in-docker` feature in `devcontainer.json`. That is why `docker` works
inside a devcontainer or codespace and not in a bare `docker run` of the image.

You are not required to use any of it. Working locally with your own tools is
fine, and nothing about scoring depends on where the work was done.

## Toolchain pins

Every version is a build argument with a literal default, pinned deliberately
rather than tracking latest. The base image is pinned to a multi-architecture
index digest, and the image is published for `linux/amd64` and `linux/arm64`,
so an Apple Silicon machine runs it natively.

## kind caveats

`kind` is installed but nothing creates a cluster automatically, since a
control plane at rest competes with MySQL, ActiveMQ and the JVM for the
2-core/8GB floor set in `hostRequirements`. Run `kind create cluster`
yourself if you want one for Part 4.

Known, Codespaces-specific failure modes, most recent first:

- **Docker v27's IPv6 default breaks `kind create cluster`.** The
  docker-in-docker feature is pinned to `26.1` specifically to avoid this,
  and the Docker client in the image is pinned to match. The symptom, if it
  ever returns, is an error at the `docker network create -d=bridge` step
  that precedes cluster creation, and it does not reproduce locally with an
  identical `devcontainer.json`.
- **DNS resolution can fail inside the kind control-plane container** even
  when `/etc/resolv.conf` looks correct and other containers on the same
  inner `dockerd` resolve fine.
- **A second `kind create cluster` can fail after a successful `kind delete
  cluster`.** If this happens, rebuild the container rather than debugging
  the daemon state.

Because of this history, no live `kind`-backed deployment is required
anywhere in scoring. Your Kubernetes manifests and your Terraform or OpenTofu
code are checked statically, so a `kind` problem on your machine costs you
nothing.
