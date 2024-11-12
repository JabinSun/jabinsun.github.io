```shell
log::err() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%2N%z')] - [${RED}ERROR${NC}] %b\n" "$@"
}

log::info() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%2N%z')] - [INFO] %b\n" "$@"
}

log::warning() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%2N%z')] - [${YELLOW}WARNING${NC}] \033[0m%b\n" "$@"
}

log::info "This is an informational message."
log::warning "This is a warning message."
log::err "This is an error message."
```