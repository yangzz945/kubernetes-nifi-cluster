# kubernetes-nifi-cluster

A nifi cluster running in kubernetes

## Major update March2019

_Secure SSL Cluster guide coming soon_

### Prerequisites

- This example is using Google Cloud Platform persistent volumes for its backing store (easy to convert to AWS).
- Requires [vortex](https://github.com/AlexsJones/vortex)
  - Requires golang installed and on the path
  - Can be installed with `go get github.com/AlexsJones/vortex`

- Requires [zookeeper](https://github.com/AlexsJones/kubernetes-zookeeper-cluster)

from the zookeeper repo repo run:
```
./build_environment.sh small
kubectl create -f deployment/
```

Now zookeeper is setup with three nodes on the zk namespace you are ready!

## Deploy Nifi

_Make sure zk is running to avoid headaches_

```
./build_environment.sh default
# Please create the nifi namespace first

kubectl create ns nifi

kubectl create -f deployment/
```

Once running you should see...
```
kubectl get pods -n nifi 
NAME      READY     STATUS    RESTARTS   AGE
nifi-0    1/1       Running   0          25m
nifi-1    1/1       Running   0          25m
nifi-2    1/1       Running   0          25m

kubectl get svc -n nifi                
NAME        TYPE           CLUSTER-IP     EXTERNAL-IP                          PORT(S)                      AGE
nifi        ClusterIP      None           <none>                               8081/TCP,2881/TCP,2882/TCP   1d
nifi-0      ExternalName   <none>         nifi-0.nifi.nifi.svc.cluster.local   <none>                       27m
nifi-1      ExternalName   <none>         nifi-1.nifi.nifi.svc.cluster.local   <none>                       27m
nifi-2      ExternalName   <none>         nifi-2.nifi.nifi.svc.cluster.local   <none>                       27m
nifi-3      ExternalName   <none>         nifi-3.nifi.nifi.svc.cluster.local   <none>                       26m
nifi-4      ExternalName   <none>         nifi-4.nifi.nifi.svc.cluster.local   <none>                       27m
nifi-http   LoadBalancer   10.59.252.26   35.193.200.235                       8080:31995/TCP               1d

kubectl get pvc -n nifi
NAME                          STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
contentrepository-nifi-0      Bound     pvc-c00b39d5-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
contentrepository-nifi-1      Bound     pvc-c0116c25-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
contentrepository-nifi-2      Bound     pvc-c019d7ee-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
databaserepository-nifi-0     Bound     pvc-c00a3682-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
databaserepository-nifi-1     Bound     pvc-c00f87a8-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
databaserepository-nifi-2     Bound     pvc-c017dbe4-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
flowfilerepository-nifi-0     Bound     pvc-c0096aac-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
flowfilerepository-nifi-1     Bound     pvc-c00df6bb-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
flowfilerepository-nifi-2     Bound     pvc-c016020d-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
provenancerepository-nifi-0   Bound     pvc-c008b6bd-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
provenancerepository-nifi-1   Bound     pvc-c0132c86-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
provenancerepository-nifi-2   Bound     pvc-c01aec6b-4710-11e9-b1b0-42010a800055   5Gi        RWO            standard       1d
```

---

## UI

`http://<LB_IP>:8080/nifi/`


## Configuration

The `environments/` folder can have new files added and used with `build_environment.sh <envname>` and those values interpolated
For more informatino on this please see [here](https://github.com/AlexsJones/vortex/blob/master/README.md)

### Production readiness

I will eventually write a guide for SSL but its a real pain in the ass (keystore/truststore dynamic generation and sharing.)

Apart from the scaling issue I'd recommend upping the persistent volume claim size.
Ideally I'd find a solution for the dynamic node situation, but I haven't found any other Nifi K8s manifests let alone covering this issue just yet.
