```shell
# 查看k8s node节点有哪些pod
kubectl get pods --all-namespaces -o wide --field-selector spec.nodeName=${NODENAME}

# 查看k8s pod上次重启日志
kubectl logs -f ${POD} --tail 100 --previous

# 查看k8s某个节点requests和limits
kubectl get pods --all-namespaces -o json | jq -c '.items[] | select(.spec.nodeName=="${NODENAME}") | {name: .metadata.name, namespace: .metadata.namespace, containers: .spec.containers[].resources}'

# 通过/var/lib/docker/overlay2/*查找哪个pod占用磁盘较大
docker ps -aq | xargs -n 1 docker inspect --format '{{.Id}}, {{.Name}}, {{.GraphDriver.Data.WorkDir}}' | grep "6570df2379ab10da3c4a2984212ed8fa5841dc194fc3adf964f0f5c4dec7520b"

# kubectl jsonpath usage
kubectl get pods -n ${NAMESPACE} -o=jsonpath='{range .items[*]}[ {.metadata.name}, {.status.phase}, {.status.containerStatuses[0].state.running.startedAt} {.status.containerStatuses[0].image} ]{"\n"}{end}'

kubectl get pods -n ${NAMESPACE} -o jsonpath="{.items[*].spec.containers[*].image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c

kubectl get pods -n ${NAMESPACE} -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' | sort

# kubectl pod age usage
kubectl get pods -A --no-headers | awk '$NF~/^[0-9]+[m|s]$/ {print $1, $2, $NF}' | column -t

# kubectl cp usage
kubectl cp -n ${NAMESPACE} ${POD}:/PATH/TO/REMOTE/FILE /PATH/TO/LOCAL/FILE
tar cf - ${file} | kubectl exec -it -n ${NAMESPACE} ${POD} -- tar xf - -C ${path}
```