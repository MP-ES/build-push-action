# build-push-action

Composite action to build and push docker image into docker registry in the expected format of [on-premises k8s-deploy](https://github.com/MP-ES/k8s-deploy).

[![License](https://img.shields.io/github/license/MP-ES/build-push-action.svg)](LICENSE)
[![Integration](https://github.com/MP-ES/build-push-action/workflows/Integration/badge.svg)](https://github.com/MP-ES/build-push-action/actions?query=workflow%3AIntegration)

## Usage

```yaml
- name: Build and push docker image
  uses: MP-ES/build-push-action@v1
  with:
    # Server address of Docker registry. If not set then will default to Docker Hub
    # DEFAULT: hub.docker.com
    registry: registry.domain.com

    # Username used to log against the Docker registry
    username: ${{ secrets.USERNAME }}
    
    # Password or personal access token used to log against the Docker registry
    password: ${{ secrets.PASSWORD }}

    # List of build-time variables
    build-args: |
      arg1=value1
      arg2=value2

    # List of secrets to expose to the build, consumed in the Dockerfile
    # with RUN --mount=type=secret,id=NPM_TOKEN
    secrets: |
      NPM_TOKEN=${{ secrets.NPM_TOKEN }}

    # List of external cache sources and export destinations for buildx
    # (keeps the layer cache in the registry, shared between runners).
    # When cache-to is set, the action automatically sets up a buildx
    # builder that supports cache export. With push: "true" (the default),
    # the image is pushed to the registry, not loaded into the local engine.
    cache-from: type=registry,ref=registry.example.com/project/service:cache
    cache-to: type=registry,ref=registry.example.com/project/service:cache,mode=max

    # Relative path under $GITHUB_WORKSPACE to place the repository checkout code
    checkout-path: "."

    # Build's context is the set of files located in the specified PATH or URL
    context: "."

    # Path to the Dockerfile
    file: "Dockerfile"

    # The name of docker image, generally the same as the project name
    image: "service name"

    # Control if the image have to be pushed to the registry
    # DEFAULT: true
    push: "true"
```

### Cache

`cache-from` tells Buildx where to look for layers from earlier builds.
`cache-to` tells it where to save layers from the current build. In this example,
the registry reference ending in `:cache` is a separate cache artifact, not the
image tag produced by this action.

On the first build, there may be no cache to import, so the build runs normally
and exports a cache. Later builds can reuse matching layers from that cache.
For example, with multiple self-hosted runners, each run can import and update
the same registry cache, even when a different runner handles the job. This is
useful for expensive, unchanged Dockerfile steps such as installing
dependencies. A cache hit depends on the step's inputs being unchanged; the
cache does not skip steps whose inputs have changed.

Use the same cache reference for `cache-from` and `cache-to` when you want each
build to use and refresh one shared cache. Give each image or service its own
cache reference so unrelated builds do not overwrite or compete for the same
cache. The registry credentials supplied to this action must allow reading and
writing that reference.

`mode=max` exports cache for intermediate build stages as well as the final
image, which can improve reuse in multi-stage Dockerfiles. With `push: "true"`
(the default), the final image is pushed to the registry; the Buildx builder
does not load it into the local Docker engine.

#### Example

Suppose your registry is `registry.example.com` and the image is for the
`gampes/frontend` service. Configure the cache to use a separate `buildcache`
tag in that registry:

```yaml
registry: registry.example.com
image: frontend
cache-from: type=registry,ref=registry.example.com/gampes/frontend:buildcache
cache-to: type=registry,ref=registry.example.com/gampes/frontend:buildcache,mode=max
```

`registry.example.com` is fictional; replace it with the same registry
hostname you provide to `registry`. The path `gampes/frontend` identifies the
cache location, and `:buildcache` is its tag. This cache reference is separate
from the image tag that the action creates and pushes. The registry credentials
must have permission to read and write the cache reference.

### Simplified use

If you have a Dockerfile in the root path with no build args and want to build and push the docker image to a private registry you can write something such as:

```yaml
- name: Build and push docker image
  uses: MP-ES/build-push-action@v1
  with:
    registry: ${{ secrets.REGISTRY_URL }}
    username: ${{ secrets.REGISTRY_USERNAME }}
    password: ${{ secrets.REGISTRY_PASSWORD }}
    image: "service name"
```

## Outputs

Following outputs are available:

| Name       | Type   | Description                                               |
| ---------- | ------ | --------------------------------------------------------- |
| `tag`      | String | Tag assigned to docker image                              |
| `digest`   | String | Image content-addressable identifier also called a digest |
| `metadata` | JSON   | Build result metadata                                     |
