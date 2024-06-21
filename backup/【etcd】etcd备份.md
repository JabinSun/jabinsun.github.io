```shell
#!/bin/bash
# etcd backup
# 0 */3 * * * /home/monitor/etcd_backup.sh >> /home/monitor/etcd_backup.log 2>&1
   
set -e 
   
ETCD_CA_CERT="/etc/kubernetes/pki/etcd/ca.crt"  
ETCD_CERT="/etc/kubernetes/pki/etcd/server.crt"  
ETCD_KEY="/etc/kubernetes/pki/etcd/server.key"  
BACKUP_DIR="/var/lib/docker/etcd_backup"  
DT=$(date +%Y%m%d.%H%M%S)  
   
[[ ! -d ${BACKUP_DIR} ]] && mkdir -p ${BACKUP_DIR}  
find ${BACKUP_DIR} -name "*.db" -mtime +7 -exec rm -f {} \;  
   
ETCDCTL_API=3 /usr/local/bin/etcdctl --endpoints=https://127.0.0.1:2379 \  
  --cacert="${ETCD_CA_CERT}" --cert="${ETCD_CERT}" --key="${ETCD_KEY}" \  
  snapshot save "${BACKUP_DIR}/etcd-snapshot-${DT}.db"  
   
echo "Etcd backup success, backup file: ${BACKUP_DIR}/etcd-snapshot-${DT}.db, \  
  file size: $(du -sh ${BACKUP_DIR}/etcd-snapshot-${DT}.db |awk '{print $1}')"  
echo
```