# docker-resources

## Introduce

The image uses to build Azure agent as kubernetes pod. We treat it as StatefulSet strategy. The main goal is keep data cache of software such as Maven, Nuget, Conda,.. after they get completed regardless the data cannot be same in all pod that does not matter. For instance, if we have 3 pods, the first pipeline will run in the pod-0 that the cache data as Maven dependencies will be save in pod-0 that does not matter when the other two do not. The next turn the Maven dependencies will be filled on. The main goal is still good for us purpose. Remember you need mount PV with SC is type of disk to speed up the storage here (If you treat it as FileShare, the bad will happen).
With my current Dockerfile the /data directory will by persistent on kubernetes.

## Treat with it

Now I just accomplished with ARM platform. The AMD will coming soon.

### Build image

```bash
    docker build -t bigcat3997/ubuntu-azure-agent:1.0.6-arm64 --build-arg PLATFORM=arm .
```

### Push image

```bash
    docker push bigcat3997/ubuntu-azure-agent:1.0.6-arm64
```
