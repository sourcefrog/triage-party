# Triage Party 🎉
`NOTE: This is not an officially supported Google product`

Triage Party is a tool for triaging incoming GitHub issues for large open-source projects, built with the GitHub API. 

![screenshot](screenshot.png)

Triage focuses on reducing response latency for incoming GitHub issues and PR's, and ensure that conversations are not lost in the ether. It was built from the [Google Container DevEx team](http://github.com/GoogleContainerTools)'s experience contributing to open-source projects, such as minikube, kaniko, and skaffold. 

Triage Party is a stateless Go web application, configured via YAML. While it has been optimized for Google Cloud Run deployments, it's deployable anywhere due to it's low memory footprint: even on a Raspberry Pi.

Novel features:
* Queries across multiple repositories
* Queries that are not possible on GitHub:
  * conversation direction (`tag: recv`)
  * duration (`updated: +30d`)
  * regexp (`label: priority/.*`)
  * reactions (`reactions: >=5`)
  * comment popularity (`comments-per-month: >0.9`)
  * ... and more!
* Multi-player mode: for simultaneous group triage of a pool of issues
* Button to open issue groups as browser tabs (pop-ups must be disabled)
* "Shift-Reload" for live data pull

## See it in production!

Here is how Triage Party is used for [kubernetes/minikube](https://github.com/kubernetes/minikube)

* [Triage Party dashboard](http://tinyurl.com/mk-tparty) - if you see a rate error, refresh
* [configuration](examples/minikube.yaml)
* [deployment script](examples/minikube-deploy.sh)

## Requirements

- [GitHub API token](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line)
- Go v1.12 or higher

## Configuration

Each page is configured with a `strategy` that references multiple queries (`tactics`). These tactics can be shared across pages:

```yaml
strategies:
  - id: soup
    name: I like soup!
    tactics:
      - discuss
      - many-reactions

tactics:
  discuss:
    name: "Items for discussion"
    resolution: "Discuss and remove label"
    filters:
      - label: triage/discuss
      - state: "all"

  many-reactions:
    name: "many reactions, low priority, no recent comment"
    resolution: "Bump the priority, add a comment"
    filters:
      - reactions: ">3"
      - reactions-per-month: ">1"
      - label: "!priority/p0"
      - label: "!priority/p1"
      - responded: +60d
```

## Filter language

```yaml
# issue state (default is "open")
- state:(open|closed|all)

- label: [!]regex
- tag: [!]regex

- milestone: string

- created: [-+]duration
- updated: [-+]duration
- responded: [-+]duration

- reactions: [><=]int
- reactions-per-month: [><=]float

- comments: [><=]int
- comments-per-month: [><=]int
- comments-while-closed: [><=]int

- commenters: [><=]int
- commenters-while-closed: [><=]int
- commenters-per-month: [><=]float
```

## Running locally

Start the webserver:

```
export GO111MODULE=on
cd cmd/server
go run main.go \
  --token $GITHUB_TOKEN \
  --config ../../examples/minikube.yaml
```

This will use minikube's configuration as a starting point. The first time you run Triage Party against a new repository, there will be a long delay as it will download and cache every issue and PR. This data will be cached for subsequent runs.

## Running in Docker

```
docker build --tag=tp --build-arg CFG=examples/minikube.yaml --build-arg TOKEN=$GITHUB_TOKEN .
docker run -p 8080:8080 tp
```

## Cloud Run Build & Deploy

See `examples/gcloud-deploy.sh`

Or use the easy button:

[![Run on Google Cloud](https://storage.googleapis.com/cloudrun/button.svg)](https://console.cloud.google.com/cloudshell/editor?shellonly=true&cloudshell_image=gcr.io/cloudrun/button&cloudshell_git_repo=https://github.com/google/triage-party)
