# ubuntu-azure-agent

## Introduce

The image uses to build Azure self-hosted agent as Docker container.

## Treat with it

### Build image

```bash
    docker build -t bigcat3997/azp-agent-ubuntu:1.0.0 .
```

### Run image

```bash
    docker run -it \
        -v <WHERE_YOU_WANT_ON_HOST>/data:/data \
        -e AZP_URL="https://dev.azure.com/<ORG_NAME>" \
        -e AZP_TOKEN=<PAT> \
        -e AZP_POOL="azp-agent-ubuntu-docker" \
        -e AZP_AGENT_NAME="Docker Agent 1 - Ubuntu" \
        --name azp-agent-ubuntu \
        azp-agent-ubuntu:1.0.0
```

### Compose images

```bash
    docker compose up -d
```

## Preference

https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/linux-agent?view=azure-devops
