```shell
#!/bin/bash

log::error() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%3N%z')] - [\033[0;31mERROR\033[0m] - %s\n" "$@"  
}

log::info() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%3N%z')] - [\033[0;32mINFO\033[0m] - %s\n" "$@"   
}

log::warning() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%3N%z')] - [\033[1;33mWARNING\033[0m] - %s\n" "$@"
}

log::info "This is an informational message."
log::warning "This is a warning message."
log::error "This is an error message."
```