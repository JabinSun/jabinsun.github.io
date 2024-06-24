```shell
#!/bin/bash
# set -euxo pipefail

SHELL_STEP=$1
IMAGE_NAME_ITEM=$2
IMAGE_DELETE_NUM=$3


function echo_message_red() {
	  echo -e "\033[31m ${ECHO_MESSAGE}\033[0m"
}

function echo_message_green() {
	  echo -e "\033[32m ${ECHO_MESSAGE}\033[0m"
}

function echo_message_yellow() {
	  echo -e "\033[33m ${ECHO_MESSAGE}\033[0m"
}


function usage() {
	  ECHO_MESSAGE="Usage: sh $0 [all_status|status|help|remove|all_remove] \n eg: \n [sh $0 all_status] \n [sh $0 status {IMAGE_NAME}] \n [sh $0 remove {IMAGE_NAME} {IMAGE_DELETE_NUM}] \n [sh $0 all_remove]"
	  echo_message_yellow
	  exit 1
}

function status_all() {
	  docker images|grep -v REPOSITORY|awk '{a[$1]++} END{for(i in a){print i,a[i]}}'|column -t|sort -t " " -rn -k 2
}

function status_item() {
	  ECHO_MESSAGE="Usage: [sh $0 status {IMAGE_NAME}]"
	  echo_message_yellow
	  if [[ -n ${IMAGE_NAME_ITEM} ]];then
	  	docker images|grep -v REPOSITORY|grep -e ${IMAGE_NAME_ITEM}
	  fi
}

function remove_item() {
	  ECHO_MESSAGE="Usage: [sh $0 remove {IMAGE_NAME} {IMAGE_NAME_DELETE_NUM}]"
	  echo_message_yellow
	  if [[ -n ${IMAGE_DELETE_NUM} ]]; then
	  	IMAGE_DELETE=$(docker images|grep -v REPOSITORY|grep -e ${IMAGE_NAME_ITEM}|awk '{print $1,$2,$3}'|column -t|tail -n ${IMAGE_DELETE_NUM}|awk '{print $3}')
	  	echo ${IMAGE_DELETE}
	  	docker rmi ${IMAGE_DELETE}
	  fi
}

function remove_all() {
	  echo -e "\033[33m Please use \033[32m <docker image prune/docker image prune -a> \033[0m \033[0m"
}


case "${SHELL_STEP}" in
    "all_status")
      status_all
      ;;
    "status")
      status_item
      ;;
    "remove")
      remove_item
      ;;
    "all_remove")
      remove_all
      ;;
    *)
      usage
      ;;
esac
```