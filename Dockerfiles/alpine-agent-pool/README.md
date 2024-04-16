# ubuntu-azure-agent

## Introduce

The image uses to build Azure agent as kubernetes pod. We treat it as StatefulSet strategy. The main goal is keep data cache of software such as Maven, Nuget, Conda,.. after they get completed regardless the data cannot be same in all pod that does not matter. For instance, if we have 3 pods, the first pipeline will run in the pod-0 that the cache data as Maven dependencies will be save in pod-0 that does not matter when the other two do not. The next turn the Maven dependencies will be filled on. The main goal is still good for us purpose. Remember you need mount PV with SC is type of disk to speed up the storage here (If you treat it as FileShare, the bad will happen).
With my current Dockerfile the /data directory will by persistent on kubernetes.

I used the podman to easily run rootless podman in rootful podman without set privileged flag.

## Treat with it
### Build image

```bash
    podman build -t bigcat3997/azp-agent-fedora:1.0.0 .
```

### Push image

```bash
    podman push bigcat3997/azp-agent-fedora:1.0.0
```

### Run image
```bash
        podman run -it \
            --user podman \
            --security-opt label=disable \
            --security-opt unmask=ALL \
            --device /dev/fuse \
            -e AZP_URL=https://dev.azure.com/<ORG_NAME> \
            -e AZP_TOKEN=<PAT> \
            -e AZP_POOL=<POOL_NAME> \
            -e AZP_AGENT_NAME=<AGENT_NAME> \
            --name azp-agent-fedora \
            bigcat3997/azp-agent-fedora:1.0.0
```
