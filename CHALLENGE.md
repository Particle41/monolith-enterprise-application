# Particle41 DevOps Challenge

You are the cloud engineer who has just been handed Snowman. The developers
have moved on. Your job is to make it run reproducibly, deploy safely, and be
observable. Use whatever AI tooling you use on a real project, and show us how
you work with it.

## Effort and timeline

Expect 2-4 hours of focused work, can be split into multiple sessions.

Two clocks govern the timeline:

1. **14 days to start.** Open the Coderbyte assessment linked in your invite
   within 14 days of receiving it. After that the link expires and the invite
   lapses automatically.
2. **7 days to submit.** Opening the assessment starts your 7-day submission
   window. Do your work, then come back to the assessment and submit within
   that window.

> Note on the "zero effort" route:
> if you take the zero-effort route of trying to one-shot this by giving this CHALLENGE.md
> straight to an agent as-is and give us back the result, EXPECT IT TO FAIL.
> We've already done this, and our grading pipeline will reject the attempt
> as lower than the baseline.

## Getting started

1. Click **Use this template** on this repository and create a **private**
   repository on your own GitHub account named `snowman-<CANDIDATE_ID>`, where
   `<CANDIDATE_ID>` is the identifier from your invite.
2. Install the **Particle41 Challenge Grader** GitHub App on that repository.
   <!-- screenshot: install-step-1 -->
3. Grant it **read access to repository contents** when prompted.
   <!-- screenshot: install-step-2 -->
4. Push your work to that repository as you go.
   <!-- screenshot: install-step-3 -->

Keep the repository private and keep the App installed until you hear a
decision from us. Removing either before then leaves us unable to read your
work, which counts as no submission.

## Environment

This repository ships a `.devcontainer/` with a real Docker daemon, JDK 21,
Maven, kubectl, Helm, Kustomize, OpenTofu and `kind` preinstalled. It also
carries the tools your submission is checked with: `gitleaks`, `hadolint`,
`tflint`, `checkov`, `conftest`, `hcl2json` and `kubeconform`. We run the same
image you do, so anything you can verify here we see the same way.

- Open your new repository in a **Codespace** (Code > Codespaces > Create
  codespace). First boot pulls a prebuilt image rather than building one.
- A 2-core codespace is enough and is what we size for. Personal GitHub
  accounts include 120 free core-hours a month (60 hours at 2-core), which
  comfortably covers the 2-4 hour effort window even with rebuilds. Stop your
  codespace when you are not using it. There is no default spending cap once
  free usage runs out, so watch your usage if you keep it running.
- You do not have to use Codespaces. The same `.devcontainer/` opens in VS
  Code Dev Containers locally, or on any other devcontainer-compatible host.
  Working entirely outside a devcontainer is also fine; nothing about grading
  depends on it.
- `kind` is installed for your own use in Part 4, but is not required to pass
  grading. It has known, Codespaces-specific rough edges (see
  `.devcontainer/README.md` if `kind create cluster` misbehaves). Your
  Kubernetes manifests and Terraform/OpenTofu code are checked statically
  either way, so a `kind` problem on your end costs you nothing in scoring.

## What to do

### Part 0. Bring your own harness

Commit the agent configuration you would use on day one of this project:
`AGENTS.md`, `CLAUDE.md`, Cursor rules, skills, MCP configuration, hooks, or
whatever your tooling uses. It must be specific to this repository, not generic
advice.

`harness-example/AGENTS.md` in this repository is a deliberately generic
sample, included so you can see the shape of the artefact we mean. Do not
submit it, or anything close to it: a harness file that resembles it is
treated as a missing deliverable.

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

### Part 5. The AI session bundle

Create a directory `ai-session/` curated for review. We do not want your raw
transcript. We want what you think we should see:

- The prompts that mattered and why.
- Corrections: where the model was wrong, how you noticed, what you did.
- Where you overruled the model and why.
- Specs, research notes, and any skills or instruction files used to steer.

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
- `ai-session/` directory (Part 5)
- `AI-NOTES.md` (Part 5)

## How your submission is run

Scoring starts by running your repository, unattended, on a machine identical
to the environment above. That only works if we can find the entry points, so
these few things are fixed. Everything else is yours to name and arrange.

- **`compose.yml` at the repository root.** It is brought up with no
  arguments, so anything it needs must be committed or have a default.
- **A service named `app`**, listening on port `8080` inside the stack and
  answering `GET /health` with 200 once the stack is up. It is reached over
  the compose network by service name, so publishing the port to the host is
  yours to decide.
- **`app` does not run as root.** Its container is inspected for the user it
  runs as.
- **A MySQL service**, configured through the official image's environment
  variables (`MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, or
  `MYSQL_ROOT_PASSWORD`). Name it whatever you like: it is located by those
  variables, not by its service name.
- **Liquibase runs the migrations.** The stack is started twice and the
  changelog table is compared across both starts, which is how "migrations run
  once, as an init step" is confirmed. Seeding the schema from a raw SQL
  script instead leaves no changelog table and reads as no migrations at all.
- **`app` logs one JSON object per line to stdout**, and exits cleanly when
  sent SIGTERM.
- **A `Makefile` with `lint`, `test`, `build` and `check`.** Every `make`
  target your workflow calls must exist in it, and at least one workflow must
  invoke an image scanner (`trivy`, `grype`, `snyk`, `docker scout` or
  `anchore`).
- **No SQL statement logging left on, and no credentials in the tree.** The
  repository is scanned for secrets, including any we issued you.
- **A harness file** at `AGENTS.md`, `CLAUDE.md`, `.claude/AGENTS.md` or
  `.claude/CLAUDE.md`, plus `ai-session/` and `AI-NOTES.md`, each with real
  content in it. See the note in Part 0 about the example harness.

Your Kubernetes manifests and your Terraform or OpenTofu roots are found
wherever you put them, so no directory layout is prescribed for either.

## What we look for

- A harness configuration that names a verification command, constrains scope and destructive
  actions, handles secrets, and is specific to this repository rather than
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

## Submitting

Return to the Coderbyte assessment from your invite and paste the HTTPS URL of
your repository (`https://github.com/<your-account>/snowman-<CANDIDATE_ID>`)
into the answer field, then submit. That is the whole hand-off: nothing is
collected from your repository until you do.

Submit within the 7-day window that started when you opened the assessment.
Keep the repository private and the Grader App installed until you hear from
us.

## Gate recovery

Submissions are checked against a set of mechanical prerequisites before
scoring begins. If a prerequisite fails, you are notified immediately and
given one 48-hour window to fix and resubmit. This window is granted once per
candidate, applies to prerequisite failures only, and is honoured even when it
extends past the 7-day deadline.

## Bundle size

The grader reads text files from your repository up to a capped amount of
text. Files are sorted largest first, and the largest are truncated when the
cap is reached. Keep your deliverables focused.

## Fixtures

The `fixtures/` directory in this repository contains any seeded data provided
with the challenge. See its README for details.

## Version

See `CHALLENGE-VERSION` for the version of this brief. Cite it if you need to
reference the exact revision you received.
