```shell
crictl images | awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn
crictl rmi $(crictl images | grep 'xxx' | awk '{print $1 ":" $2}')
```