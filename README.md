### sup
---
https://github.com/pressly/sup

```
go get 0u github.com/pressly/sup/cmd/sup
sup
sup stg ping
sup stg health
sup prod health
sup prod deploy
sup prod health
sup prod tail-logs
sup prod shell
```

```yml
networks:
  production: 
  hosts:
    - api1.example.com
    - api2.example.com
    - api3.example.com
  staging:
    inventory: curl http://example.com/latest/meta-data/hostname

commands:
  restart:
    desc: Restart example Docker container
    run: sudo docker restart example
  tail-logs:
    desc: Watch tail of Docker logs from all hosts
    run: sudo docker logs --tail=20 -f example
    
commands:
  restart:
    desc: Restart example Docker container
    run: sudo docker restart example
    serial: 2
    
commands:
  build:
    desc: Build Docker image and push to registry
    run: sudo docker build -t image:latest . && sudo docker push image:latest
    once: true
  pull:
    desc: Pull latest Docker image from registry
    run: sudo docker pull image:latest

commands:
  prepare:
    desc: Prepare to upload
    local: npm run build

commands:
  upload:
    desc: Upload dist files to all hosts
    upload:
      - src: ./dist
        dst: /tmp/
        
commands:
  bash:
    desc: Interactive Bash on all hosts
    stdin: true
    run: bash

commands:
  exec:
    desc: Exec into Docker containre on all hosts
    stdin: true
    run: sudo docker exec -i $CONTAINER bash

targets:
  deploy:
    - build
    - pull
    - migrate-db-up
    - stop-rm-run
    - health
    -slack-notify
    -sirbrake-notify
# Supfile
---
version: 0.4

env:
  NAME: api
  IMAGE: example/api
  
networks:
  local:
    hosts:
      - localhost
  staging:
    hosts:
      - stg1.example.com
  production:
    hosts:
      - api1.example.com
      - api2.example.com
      
commands:
  echo:
    desc: Print some env vars
    run: echo $NAME $IMAGE $SUP_NETWORK
  date:
    desc: PrintOS name and current date/time
    run: uname -a; date
    
targets:
  all:
    - echo
    - date

restart-scheduler:
  desc: Restart scheduler
  local: >
    sup -f ./services/scheduler/Supfile $SUP_ENV $SUP_NETWORK restart
db-up:
  desc: Migrate datebase
  local: >
    sup -f ./database/Supfile $SUP_ENV $SUP_NETWORK up
```

```
sup production bash

echo 'sudo apt-get update -y' | sup production bash
sup production bash <<< 'sudo apt-get update -y'
cat <<EOF | sup production bash

sup production exec

dup produciton deploy
sup produciton build pull mirate-db-up stop-rm-run health slack-notify airbrake-notify

./Supfile
./datebase/Supfile
./services/scheduler/Supfile

ssh-add -l
ssh-add ~/.ssh/id_rsa
make build
```


