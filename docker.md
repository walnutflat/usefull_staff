### Установка в образ из приватного репозитория GitHub по SSH (на примере библиотек Python)
Dockerfile:
```
RUN mkdir ~/.ssh/
RUN ssh-keyscan -t rsa github.com > ~/.ssh/known_hosts
RUN --mount=type=ssh,id=github_ssh_key pip install \
    --no-cache-dir \
    --requirement requirements.txt
```
Создание образа:
```
docker build --ssh github_ssh_key=C:\Users\UserName\.ssh\id_rsa -t image_name .
```
### Менеджер для запуска нескольких процессов в одном контейнере
На примере модулей python приложения

manager.sh:
```
#!/bin/bash
# Processes manager
#
# Env templates:
# MODULES=module1 module2 module3
# CHECK_IN_SECONDS=5

check_in_seconds=$CHECK_IN_SECONDS
modules=$MODULES

if [[ $modules == "" ]]
then
  echo "No modules loaded with MODULES env, exiting"
  exit 0
fi

if [[ $check_in_seconds == "" ]]
then
  check_in_seconds=5
  echo "Standard CHECK_IN_SECONDS set"
fi


start_service() {
  python3 -m "$1" &
  echo "$func start command executed"
}

echo "Check in seconds = $check_in_seconds sec"
echo "Modules = $modules"

mapfile -t functions < <(echo "$modules" | tr " " "\n")

for func in "${functions[@]}";do
  start_service "$func"
done

while sleep "$check_in_seconds";do
  for func in "${functions[@]}";do
    result=$(pgrep -fc "$func")
    if [[ $result -eq 0 ]]
    then
      echo "$func exited. Let's reload!"
      start_service "$func"
    fi
  done
done
```
Dockerfile:
```
CMD /bin/bash ./manager.sh
```
### Ожидание старта сервисов в других контейнерах
Проверяет порты на доступность

wait_for.sh:
```
#!/bin/bash
#
# Wait for other containers
#
# environment:
#   - CONTAINER_NAME=container_name
#   - WAIT_FOR=container1:45000 container2:50006
#

hosts=$WAIT_FOR
counter=0
max_wait_sec=30

print()  {
  echo "[$CONTAINER_NAME]: ********** $1 **********"
}

print "WAIT_FOR script started"

if [[ $hosts == "" ]]
then
  print "No hosts to check"
  exit
fi

mapfile -t hosts_array < <(echo "$hosts" | tr " " "\n")

for host_raw in "${hosts_array[@]}"
do
  print "Waiting for ${host_raw}"
  host=$(echo "$host_raw" | tr ":" "/")

  # shellcheck disable=SC2188
  until < /dev/tcp/"$host"; do
    (( counter++ ))
    if [[ $counter -gt $max_wait_sec ]]
    then
        echo "DO SOMETHING HERE!"
    fi
    print "${host_raw} is unavailable - sleeping ${counter} sec totally"
    sleep 1
    done
done
print "All containers started-up!"
```

Dockerfile:
```
CMD /bin/bash ./wait_for.sh && /bin/bash something_useful.sh
```
