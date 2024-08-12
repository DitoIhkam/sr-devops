# Gambar ansible running

# Question 3: Monitoring and Logging with Prometheus and Grafana
Objective: Set up monitoring and logging for a simple Dockerized web application.


![alt text](?raw=true)
Create a simple web application (e.g., a Python Flask app) and containerize it using Docker.
Set up Prometheus to monitor the application’s metrics (CPU, memory usage, request counts).
Set up Grafana to visualize these metrics.
Configure basic alerting rules in Prometheus.

kasihtau kalo ini pake ansible buat docker in nya


# Install Monitoring app

## Server configuration using ansbile

Saya akan melakukan installasi dan menjalankan beberapa kebutuhan software/aplikasi seperti :
1. Disable Password
2. mengganti mirror idch menjadi archive/non mirror
3. Install & Run 
* Docker engine (Semua Server)
* Node exporter (Semua Server)
* Grafana (Server Monitoring)
* Prometheus (Server Monitoring)

saya menginstall docker di semua server karna pada akhirnya nanti aplikasi akan berjalan on top docker

khusus dibagian prometheus harus kita buat konfigurasinya sebelum menjalankan dan menginstall software/aplikasi tersebut dengan file 'prometheus.yml'



berikut scriptnya :
```
scrape_configs:
  - job_name: Grafana
    scrape_interval: 5s
    static_configs:
      - targets:
        - node-monit.ditoihkamp.studentdumbways.my.id
        - node-appserver.ditoihkamp.studentdumbways.my.id
        - node-gateway.ditoihkamp.studentdumbways.my.id
```


Berikut untuk script nya dan ketika menjalankan ansible-playbook dengan nama `play.yml`

```
- hosts: all
  become: true
  tasks:
    - name: Copy SSH public key to remote host
      authorized_key:
        user: ditoihkam
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
        state: present

    - name: Change repository apt
      replace:
        path: /etc/apt/sources.list
        regexp: 'http://mirrors\.idcloudhost\.com/ubuntu'
        replace: 'http://archive.ubuntu.com/ubuntu'

    - name: Change PasswordAuthentication for idch
      replace:
        path: /etc/ssh/sshd_config.d/50-cloud-init.conf
        regexp: 'PasswordAuthentication yes'
        replace: 'PasswordAuthentication no'

    - name: Change PasswordAuthentication
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PasswordAuthentication'
        line: 'PasswordAuthentication no'
        state: present
      notify:
        - Restart SSH

  handlers:
    - name: Restart SSH
      service:
        name: sshd
        state: restarted

- hosts: all
  become: true
  vars:
    user: ditoihkam
  tasks:
    
    - name: Pull the bitnami/node-exporter Docker image
      docker_image:
        name: bitnami/node-exporter
        source: pull

    - name: Run the Node Exporter container
      docker_container:
        name: node-exp
        image: bitnami/node-exporter
        state: started
        restart_policy: unless-stopped
        published_ports:
          - "9100:9100"

- name: Deploy Prometheus and Grafana top on Docker
  hosts: monitoring
  become: yes

  tasks:
    - name: Pull Grafana Docker image
      docker_image:
        name: grafana/grafana
        source: pull

    - name: Run Grafana container
      docker_container:
        name: Grafana-container
        image: grafana/grafana
        state: started
        restart_policy: unless-stopped
        published_ports:
          - "3000:3000"

    - name: Pull Prometheus Docker image
      docker_image:
        name: bitnami/prometheus
        source: pull

    - name: Create a directory if it doesn't exist
      ansible.builtin.file:
        path: ~/prometheus
        state: directory
        mode: '0755'

    - name: Copy prometheus config
      copy:
        src: ~/srdevops/vm3/sprometheus.yml
        dest: ~/prometheus/prometheus.yml

    - name: Run Prometheus container
      docker_container:
        name: Prometheus-container
        image: bitnami/prometheus
        state: started
        volumes:
          - ~/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
        restart_policy: unless-stopped
        published_ports:
          - "9090:9090"
```


(ansible image running)

## Monitor Resources for appserver & gateway 

Sebelum kita memonitoring semua server, kita perlu memastikan node exporter di semua server itu telah terinstall node exporter serta target di prometheus mengarah ke ketiga node exporter tersebut.

Appserver (gambar metric)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.1%20Metric%20app.png?raw=true)

Gateway (gambar metric)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.2%20Metric%20gate.png?raw=true)

Prometheus Targets

![alt text](https://github.com/DitoIhkam/sr-devops/blob/main/3%3A%20Monitoring%20and%20Logging%20with%20Prometheus%20and%20Grafana/img/prom.png?raw=true)


## Create a fully working dashboard in grafana (Template/DIY)

Setelah kita sudah memastikan resource monitor berjalan dengan baik, selanjutnya kita akan membuat dashboard untuk memonitoring kedua server tersebut menggunakan template grafana.

Pertama login dan setting grafana agar target mengarah ke prometheus yang sudah kita set

(gambar setting prometheus)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.4%20Graf.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.5%20Graf%20Dashboard.png?raw=true)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.6%20Graf%20setting%20prom.png?raw=true)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.7%20prom%20settingggs.png?raw=true)

lalu kita buat dashboard menggunakan settingan import 1860 seperti dibawah
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.8%20import%201.png?raw=true)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.9%20import%202.png?raw=true)





## Grafana Alerts

Setelah membuat dashboard untuk memonitoring, terakhir saya akan membuat alert yang mana nanti akan ada peringatan masuk ketika cpu, ram, dan memory usage melebihi batas yang telah ditentukan.

dibagian kiri atas, klik garis tiga dan pindah ke menu alerting, lalu pilih contact point, dan set integration ke discord serta masukkan webhook url discord. Apabila sudah dimasukkan webhooknya, bisa di test send notification bila perlu untuk mengetahui sudah terhubung atau belum ke discord

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.10%20edit%20contact%20point.png?raw=true)


Setelah itu, kita setting notification policy agar mengarah ke discord dan bukan ke email ataupun lainnya

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.11%20arah%20contact%20point.png?raw=true)

ini menu dimana kita menambahkan alert rules

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.12%20cara%20menambahkan%20alert%20rules.png?raw=true)

lalu kita ke menu alert rules untuk menyetel cpu, ram, dan disk dibawah ini

CPU usage ove 80% & RAM usage over 10%
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.13%20script%20promql.png?raw=true)

Disk usage over 80% & Hasil Discord
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.14%20script%20dan%20hasi%3B.png?raw=true)

