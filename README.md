# Kubernetes

This is a repo to help reproduce https://github.com/kubernetes/kubernetes/issues/97445.

For the original README of the forked project see https://github.com/vikyd/go-cpu-load.

Steps to reproduce:

1. build and push the image (set `IMAGE_NAME` to a image repo that you have push access to)

```
docker build -t $IMAGE_NAME .
docker push $IMAGE_NAME
```

2. run a pod with limit 400m and uses 6 cores with 5 percent (50m) usage each

```
kubectl run cpu-load --restart=Never \
                     --requests="cpu=400m" \
                     --limits="cpu=400m"  \
                     --image $IMAGE_NAME \
                     -- /app/cpu_load -c 6 -p 5
```

3. observe the usage is around 300m (6 x 50m)
```
➜ kubectl top pods
NAME       CPU(cores)   MEMORY(bytes)
cpu-load   303m         2Mi
```

4. log into the machine the pod is running on and inspect the container
```
# first get the container ID
# for containerd
crictl ps --name cpu-load
crictl inspect $CONTAINER_ID

# for docker
docker ps --filter "name=cpu-load"
docker inspect $CONTAINER_ID
```

5. search for the cgroups path of the container

```
# this should have throttled zero
cat /sys/fs/cgroup/cpu,cpuacct/$CGROUPS_PATH/cpu.stat
```
