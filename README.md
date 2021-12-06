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

### Simplified use

If you have a Dockerfile in the root path with no build args and want to build and push the docker image to a private registry you can just write:

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

| Name  | Type   | Description                  |
| ----- | ------ | ---------------------------- |
| `tag` | String | Tag assigned to docker image |
