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

`cache-from` reads reusable layers from a previous build; `cache-to` saves
layers for later builds. A registry cache lets different runners share those
layers. For details, see Docker's
[registry cache backend documentation](https://docs.docker.com/build/cache/backends/registry/).

#### Example

```yaml
registry: registry.example.com
image: web-app
cache-from: type=registry,ref=registry.example.com/sample-project/web-app:buildcache
cache-to: type=registry,ref=registry.example.com/sample-project/web-app:buildcache,mode=max
```

`type=registry` selects the registry backend, and `ref` is the cache location.
The `:buildcache` tag is separate from the image tag. `mode=max` also caches
intermediate build stages. Replace the fictional hostname with your registry;
its credentials must allow reading and writing this cache.

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
