
# Question 2: Continuous Integration/Continuous Deployment with GitHub Actions
Objective: Set up a CI/CD pipeline for a simple web application in any programming language of your choice.


## Membuat Repository di github

Saya membuat 2 repository di github, yaitu fe-dumbmerch dan be-dumbmerch yang mana ini adalah simple web application

![alt text](https://github.com/DitoIhkam/sr-devops/blob/main/Continuous%20Integration%20Continuous%20Deployment%20with%20GitHub%20Actions/img/FE%20APP.png?raw=true)
![alt text](https://github.com/DitoIhkam/sr-devops/blob/main/Continuous%20Integration%20Continuous%20Deployment%20with%20GitHub%20Actions/img/BE%20APP.png?raw=true)
# Deployment

Pada tahap ini saya akan membuat docker image untuk frontend dan baceknd. Berikut adalah beberapa file2 Dockerfile serta docker compose yang akan saya jalankan di vm 1 bernama appserver untuk deployment web sederhana
## Install Docker

Sebelum saya bisa membuat docker container di vm appserver yang akan diberikan aplikasi sederhana , saya perlu menginstall docker dan akan saya install di semua server untuk next juga nantinya. disini penginstallan saya gunakan ansible agar otomatis.

script ansible untuk install docker
```
- hosts: all
  become: true
  vars:
    user: ditoihkam
  tasks:
    - name: Install Docker Dependencies
      apt:
        update_cache: yes
        name:
          - lsb-release
          - ca-certificates
          - curl
          - gnupg

    - name: Install GPG Key for Docker
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg

    - name: Add Docker APT Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu focal stable

    - name: Install Docker Engine
      apt:
        update_cache: yes
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io

    - name: Install Docker Compose
      apt:
        name: docker-compose
        state: latest
        update_cache: yes

    - name: Install Python Dependencies
      apt:
        name: python3-pip
        state: latest
        update_cache: yes

    - name: Install Docker SDK for Python
      pip:
        name: docker
        state: latest
        executable: pip3

    - name: Add user to the docker group
      user:
        name: "{{ user }}"
        groups: docker
        append: yes
        state: present

    - name: Start Docker service
      service:
        name: docker
        state: started

    - name: Enable Docker on boot
      service:
        name: docker
        enabled: yes
```

![alt text](https://github.com/DitoIhkam/sr-devops/blob/main/Continuous%20Integration%20Continuous%20Deployment%20with%20GitHub%20Actions/img/docker%20install.png?raw=true)

### Front End

`Dockerfile`

```
FROM node:16-alpine
WORKDIR /app
COPY . .
RUN yarn install
EXPOSE 3000
CMD [ "yarn", "start" ]
```

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.1%20fe%20staging%20dockerfile.png?raw=true)



### Back End

`Dockerfile`

```
FROM golang:1.16-alpine
RUN mkdir /app
COPY . /app
WORKDIR /app
RUN go get ./ && go build && go mod download
EXPOSE 5000
CMD ["go", "run", "main.go"]
```

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.2%20be%20staging%20dockerfile.png?raw=true)


### Docker Compose

file `docker-compose.yml` berfungsi untuk membuild dan menjalankan frontend, backend serta database secara bersamaan

```
version: '3.8'
services:
  frontend:
    build: ./fe-dumbmerch2
    container_name: fe-container-staging
    image: kelompok2/fe-staging
    ports:
      - "3000:3000"
    depends_on:
      - postgres
    networks:
      - network
    restart: unless-stopped

  backend:
    build: ./be-dumbmerch2
    container_name: be-container-staging
    image: kelompok2/be-staging
    ports:
      - "5000:5000"
    depends_on:
      - postgres
    networks:
      - network
    restart: unless-stopped

  postgres:
    image: "postgres"
    container_name: db-container-production
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
      POSTGRES_DB: postgres
    volumes:
      - "./data/postgresql:/var/lib/postgresql/data"
    networks:
      - network
    ports:
      - "5432:5432"
    restart: unless-stopped

networks:
  network:
```

lalu saya jalankan docker compose dengan perintah berikut 

`docker compose up -d`

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.3%20docker%20compose%20up%20staging.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.41%20docker%20ps%20dan%20images.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.42%20hasil%201.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.43%20hasil%202.png?raw=true)




## CI/CD

Script for front end

```
name: CI/CD Pipeline

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

#    - name: Git Pull
#      run: |
#        cd /home/ditoihkam/fe-dumbmerch2
#        git pull

    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}


    - name: SSH Login
      uses: appleboy/ssh-action@master
      with:
        host: 103.193.178.179
        username: ditoihkams
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: 'echo "Logged in to SSH"'


    - name: Build and Push Docker Image
      run: |
        docker build -t kelompok2/fe-dumbmerchsr .
        docker push kelompok2/fe-dumbmerchsr
    - name: SSH Deploy
      uses: appleboy/ssh-action@master
      with:
        host: 103.193.178.179
        username: ditoihkams
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /home/ditoihkams/fe-dumbmerchsr
          docker compose up -d
```

Scritp for backend

```
name: CI/CD Pipeline

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

#    - name: Git Pull
#      run: |
#        cd /home/ditoihkam/fe-dumbmerch2
#        git pull

    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}


    - name: SSH Login
      uses: appleboy/ssh-action@master
      with:
        host: 103.193.178.179
        username: ditoihkams
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: 'echo "Logged in to SSH"'


    - name: Build and Push Docker Image
      run: |
        docker build -t kelompok2/be-dumbmerchsr .
        docker push kelompok2/be-dumbmerchsr
    - name: SSH Deploy
      uses: appleboy/ssh-action@master
      with:
        host: 103.193.178.179
        username: ditoihkams
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /home/ditoihkam/be-dumbmerchsr
          docker compose up -d
```



## Github Action Setup

Untuk mengatur agar github action bisa berjalan, kita perlu mengatur secrets didalam github repo nya. Pergi ke bagian setting, di bagian Secrets and Variable klik Action, lalu klik new repository secrets. Disini saya mengatur 3 secrets yang nantinya akan terhubung ke github action untuk cicd seperti di gambar

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.91%20setting%20action.png?raw=true)
untuk token docker hub bisa diambil dari gambar ini
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/last.png?raw=true)


Masih didalam repository nya, kita pergi ke bagian tab action, lalu pilih `set up a workflow yourself`, lalu copy paste script cicd nya. setelah di commit, github action akan memproses cicd nya


![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.92%20setting%20action.png?raw=true)
![alt text](https://github.com/DitoIhkam/sr-devops/blob/main/Continuous%20Integration%20Continuous%20Deployment%20with%20GitHub%20Actions/img/action%20script.png?raw=true)
![alt text](https://github.com/DitoIhkam/sr-devops/blob/main/Continuous%20Integration%20Continuous%20Deployment%20with%20GitHub%20Actions/img/action.png?raw=true)

### Hasil Github Action ke docker hub

![alt text](https://github.com/DitoIhkam/sr-devops/blob/main/Continuous%20Integration%20Continuous%20Deployment%20with%20GitHub%20Actions/img/hasil%20action.png?raw=true)
