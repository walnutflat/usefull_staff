### Action для создания docker образа, получения списка ip адресов из БД postgresql и обновления образов на данных хостах
На примере получения IP-адресов

main.yml:
```
# YML файл для GitActions
# Собираем образ, кладем его на ДокерХаб, получаем список IP виртуалок из основной базы,
# проходии по ним и выполняем нужные действия
# SSH работает по ключу, без пароля
# Хосты берутся из БД

name: CI_CD
on:
  push:
    branches: [ main ]
env:
# ---------- CHANGE ME ----------
  IMAGE_NAME: image_name
# ---------- CHANGE ME ----------
jobs:
  build:
    name: Build&push image do DockerHub
    runs-on: ubuntu-latest
    steps:
      - name: Check Out Repo
        uses: actions/checkout@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

      - name: Set up Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v1

      - name: Create private key file
        run: |
          echo "${{ secrets.SSH_KEY }}" > id_rsa

      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          context: ./
          file: ./Dockerfile
          push: true
          tags: |
            ${{ secrets.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ secrets.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:latest
          ssh: |
            github_ssh_key=id_rsa

      - name: Image digest
        run: echo ${{ steps.docker_build.outputs.digest }}

  get_ip_addresses:
    name: Get IP-addresses from PSQL
    runs-on: ubuntu-18.04
    outputs:
      ip_addresses: ${{ steps.correct_addresses.outputs.ip_addresses }}
    env:
      QUERY: "select ip_address from public.ip_addresses"
    steps:
      - name: Install psql client
        run: |
          sudo apt-get install ca-certificates gnupg
          curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
          sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
          sudo apt-get update
          sudo apt -y install postgresql-client-11

      - name: Get data from PSQL
        run: |
          psql postgres://postgres:${{ secrets.POSTGRES_PASSWORD }}@1.2.3.4:6432/dbname -a -b -t \
          -c "$QUERY" > ip_addresses.txt

      - name: Show addresses
        run: |
          cat ip_addresses.txt

      - id: correct_addresses
        name: Correct addresses as a string
        run: |
          str=""
          mapfile -t ips < ip_addresses.txt
          for ip in ${ips[@]}; do str="${str}${ip},"; done
          echo ::set-output name=ip_addresses::${str:0:(( ${#str} - 1 ))}

  deploy:
    name: Deploy ALL on ALL the servers
    runs-on: ubuntu-latest
    needs: [build, get_ip_addresses]
    steps:
      - name: Show ip addresses
        run: |
          echo ${{ needs.get_ip_addresses.outputs.ip_addresses }}

      - name: Create private key file
        run: |
          echo "${{ secrets.SSH_KEY }}" > id_rsa

      - name: Execute remote ssh commands
        uses: appleboy/ssh-action@master
        with:
          host: ${{ needs.get_ip_addresses.outputs.ip_addresses }}
          username: host_username
          key_path: id_rsa
          script: |
            cd /home/host_username/docker/
            docker-compose pull image_name
            docker-compose up -d
```
