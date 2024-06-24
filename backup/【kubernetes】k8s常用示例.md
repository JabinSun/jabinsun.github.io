```shell
# kubectl jsonpath usage
kubectl get pods -n ${NAMESPACE} -o=jsonpath='{range .items[*]}[ {.metadata.name}, {.status.phase}, {.status.containerStatuses[0].state.running.startedAt} {.status.containerStatuses[0].image} ]{"\n"}{end}'

kubectl get pods -n ${NAMESPACE} -o jsonpath="{.items[*].spec.containers[*].image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c

kubectl get pods -n ${NAMESPACE} -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' | sort

# kubectl pod age usage
kubectl get pods -A --no-headers | awk '$NF~/^[0-9]+[m|s]$/ {print $1, $2, $NF}'

# kubectl cp usage
kubectl cp -n ${NAMESPACE} ${POD}:/PATH/TO/REMOTE/FILE /PATH/TO/LOCAL/FILE
tar cf - ${file} | kubectl exec -it -n ${NAMESPACE} ${POD} -- tar xf - -C ${path}
```