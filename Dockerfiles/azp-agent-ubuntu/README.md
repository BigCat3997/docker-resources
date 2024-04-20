# ubuntu-azure-agent

## Introduce

The image uses to build Azure self-hosted agent as Docker container. The Docker build in this image mount the /var/run/docker.socket that mean I remote to Docker deamon on the host.

## Treat with it

### Build image

```bash
    docker build -t bigcat3997/azp-agent-ubuntu:1.0.0 .
```

### Run image

```bash
    docker run -it \
        -v /data:/data \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -e AZP_URL=<ORG_URL> \
        -e AZP_TOKEN=<PAT> \
        -e AZP_POOL=<POOL_NAME> \
        -e AZP_AGENT_NAME="Docker Agent 1 - Ubuntu" \
        --name azp-agent-ubuntu-1 \
        bigcat3997/azp-agent-ubuntu:1.0.0
```

### Compose images

```bash
    export AZP_URL=<ORG_URL>
    export AZP_TOKEN=<PAT>
    export AZP_POOL=<POOL_NAME>

    docker compose up -d
```

## Preference

https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/linux-agent?view=azure-devops
https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops
