# Particle41 DevOps Challenge

You are the cloud engineer who has just been handed Snowman. The developers
have moved on. Your job is to make it run reproducibly, deploy safely, and be
observable. Use whatever AI tooling you use on a real project, and show us how
you work with it.

## Effort

Expect 2-3 hours of focused work.

> Note on the "zero effort" route:
> if you take the zero-effort route of trying to one-shot this by giving this CHALLENGE.md
> straight to an agent as-is and give us back the result, EXPECT IT TO FAIL.
> We've already done this, and our grading pipeline will reject the attempt
> as lower than the baseline.

## What to do

### Part 0. Configure your agent

Commit the agent configuration you would use on day one of this project:
`AGENTS.md`, `CLAUDE.md`, Cursor rules, skills, MCP configuration, hooks, or
whatever your tooling uses. It must be specific to this repository, not generic
advice.

`harness-example/AGENTS.md` in this repository is a deliberately generic
sample.

### Part 1. Make it run reproducibly

- Multi-stage Dockerfile, non-root runtime user, pinned base images.
- `docker compose` stack with MySQL and ActiveMQ, healthchecks, and Liquibase
  migrations run as an init step rather than on every boot.
- All configuration and secrets externalized to environment variables or
  mounted files. No credentials in the repository.

### Part 2. Make it operable

- `/health` wired into container and orchestrator healthchecks; add readiness
  distinct from liveness if the platform supports it.
- Structured JSON logging to stdout, SQL statement logging off by default.
- Graceful shutdown on SIGTERM.
- Resource requests and limits wherever workloads are declared.

### Part 3. Replace CI

- GitHub Actions workflow replacing Travis: build, unit tests, image build,
  `hadolint`, `trivy` image scan, push to GHCR on the default branch.
- The workflow must call the same targets a developer runs locally (a
  `Makefile` or `justfile` with `lint`, `test`, `build`, `check`).

### Part 4. Infrastructure as code

Kubernetes only: manifests (Helm or Kustomize or both) for the workload, plus
Terraform or OpenTofu for the network and cluster on any major cloud provider
(AWS, GCP or Azure). Other deployment targets are accepted, if well executed.

Constraints: compute in private subnets, ingress through a public load
balancer, variables with sensible defaults, remote state configuration
described.

If you deploy through a Helm chart, commit the rendered manifests as well.
They are validated against the Kubernetes API schema, and a raw template is
not valid YAML until it has been rendered.

### Part 5. Your AI notes

Write `AI-NOTES.md` at the repository root: which tools and models you used
and why, what the model got wrong, cost and time observations, what you would
automate next time.

## Deliverables

Your submitted repository must contain at minimum:

- Agent harness files (Part 0)
- Containerised and composed application (Part 1)
- CI workflow (Part 3)
- Kubernetes manifests with network infrastructure code for one cloud
  provider (Part 4)
- `AI-NOTES.md` (Part 5)

## What we look for

- An agent file that is specific to this repository rather than
  generic boilerplate.
- Honest, reflective AI notes: a justified model choice, cost and latency
  observed, and a frank account of what the model got wrong.
- Session samples, with corrections and one overrules if any, with
  reasoning, showing selected prompts.
- A README a colleague could deploy from.

## AI use

AI use is expected. Work may be done anywhere, with any tooling, on any
machine. Coderbyte is only where you accept the terms and where you submit;
there is no work to do inside it and no environment there to use.

## Fixtures

The `fixtures/` directory in this repository contains any seeded data provided
with the challenge. See its README for details.

## Version

See `CHALLENGE-VERSION` for the version of this brief. Cite it if you need to
reference the exact revision you received.
