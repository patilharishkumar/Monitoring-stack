# Monitoring-stack
```console
$ kubectl get nodes -owide
NAME                                           STATUS   ROLES    AGE     VERSION              INTERNAL-IP      EXTERNAL-IP      OS-IMAGE         KERNEL-VERSION                CONTAINER-RUNTIME
ip-192-168-27-232.eu-west-1.compute.internal   Ready    <none>   3h47m   v1.20.7-eks-135321   192.168.27.232   34.248.175.100   Amazon Linux 2   5.4.141-67.229.amzn2.x86_64   docker://19.3.13
ip-192-168-32-254.eu-west-1.compute.internal   Ready    <none>   3h47m   v1.20.7-eks-135321   192.168.32.254   34.255.5.1       Amazon Linux 2   5.4.141-67.229.amzn2.x86_64   docker://19.3.13


bash install-helm.sh

kubectl get deploy -n kube-system | grep -i tiller 

charts=(
"elasticsearch:cpaas-elasticsearch-cluster"
"kibana:kube-logging"
)

bash deploy-all-charts.sh "${charts[@]}"
```

It will deploy the Helm charts for the following applications/tools creates the following namespaces;

- cpaas-elasticsearch-cluster
- kube-logging

Verify the installation by executing the following commands:

```console
helm ls
NAME              	REVISION            STATUS
elasticsearch     	1         	    DEPLOYED            ....
kibana            	1       	    DEPLOYED            ....


kubectl get pods -n cpaas-elasticsearch-cluster

NAME                 READY   STATUS
cpaas-elasticsearch-0   1/1     Running
cpaas-elasticsearch-1   1/1     Running
cpaasa-elasticsearch-2   1/1     Running

kubectl get pods -n kube-logging

NAME                      READY   STATUS
kibana-7bc7984bf5-pmfql   1/1     Running


```

The Elasticsearch cluster can be accessed by using kubectl port-forward. To forward Elasticsearch service port to your local environment, execute the following command;
```console
kubectl port-forward cpaas-elasticsearch-0 9200:9200 -n cpaas-elasticsearch-cluster
```

Then test it with the following command;
```console
curl -XGET http://localhost:9200/_cluster/stats?pretty
```

Kibana
```console
$ kubectl get pods -n kube-logging
NAME                    READY   STATUS    RESTARTS   AGE
kibana-c775c948-j5sz7   1/1     Running   1          19m

kubectl port-forward kibana-c775c948-j5sz7  5601:5601 -n kube-logging
```

Access Kibana through browser
http://localhost:5601


## Tear Down the Elasticsearch Cluster

### Delete the Helm Charts
```console
# Set an array in the terminal that contains chart directories
charts=(
  "elasticsearch"
  "kibana"
)

# Then execute the script to delete the charts
bash delete-all-charts.sh "${charts[@]}"
```

### Delete the Persistent Volume Claims

```console
kubectl delete persistentvolumeclaims \
            --namespace cpaas-elasticsearch-cluster \
            --selector app=elasticsearch
```

### Delete the Namespaces

```console
for namespace in "cpaas-elasticsearch-cluster" "kube-logging"; do
    kubectl delete ns $namespace;
done
```