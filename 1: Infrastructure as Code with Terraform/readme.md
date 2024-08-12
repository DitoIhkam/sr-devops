
Baik, berikut terlampir untuk testnya. Deadline waktu 5 hari setelah pengiriman test. Apabila sudah kirim test mohon konfirmasi dan kirimkan link Github ke saya by WA atau email: widdy@redikru.com. Terima kasih🙏🏻


https://bit.ly/SrDevOpsTest 


jenkins diilangin


# Question 1: Infrastructure as Code with Terraform
Objective: Set up a basic infrastructure using Terraform.

## Provisioning idcoudhost using terraform

Pertama, untuk menyediakan server idcloudhost menggunakan Terraform, kita akan membuat source code didalam file ekstensi .tf di masing2 folder dengan spesifikasi sebagai berikut :

* Appserver - 2 vCPU, 2GB RAM
	* Berfungsi untuk web/aplikasi sederhana
* Gateway - 2 vCPU, 2GB RAM
	* Berfungsi untuk reverse proxy dan load balancing
* Monitoring - 2 vCPU, 2GB RAM
	* Berfungsi untuk memonitoring semua server
semua server berkapasitas 20GB Storage 

Saya akan melakukan installasi dan menjalankan beberapa kebutuhan software/aplikasi seperti :
1. Copy ssh ke semua virtual machine (menggunakan terraform)
2. Disable Password
3. mengganti mirror idch menjadi archive/non mirror
4. Install & Run 
* Docker engine (Semua Server)
* Node exporter (Semua Server)
* Grafana (Server Monitoring)
* Prometheus (Server Monitoring)




### Provisioning server with terraform

Sekarang saya akan membuat script terraform agar vm didalam idcloudhost tersedia secara otomatis, setelah saya menginstall Terraform.

1. appserver.tf

```
terraform {
  required_providers {
    idcloudhost = {
      source  = "bapung/idcloudhost"
      version = "0.1.3"
    }
  }
}

provider "idcloudhost" {
  # Configuration options
  auth_token = "H4DgjZQTQh9JIenbqc64HHQycLOdfQG7"
}

resource "idcloudhost_vm" "appserver" {
  name           = "ditoihkam-app"       # Tambahkan tanda kutip ganda yang hilang di bagian ini
  os_name        = "ubuntu"
  os_version     = "20.04"
  vcpu           = "2"
  memory         = "2048"
  disks          = "20"
  username       = "ditoihkams"
  initial_password = "Katasand1"    # Tambahkan tanda kutip ganda yang hilang di bagian ini
  public_key     = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC161240SqH/DxKtpWIcD8j8eeuhQJXVsRRIjhucAsXuF3gs4Trbyl4RYflSW8+r+T+P/GP3graT/LEDq6KbTXeEjBm5INzixBoWySWPa1JxckqJ12dPvkQAJwJ6Rs1ZUiG7rfbOnD4gq1pbt7NHY8SAN1rx2JT0NJIGZtFWUbSuwVWW0C0FdUTPdFD3hScHgkt/45HGRQzmN4QiKX40mymLwEQ8E7wwB8DHrNZsFEnSeFrkpGRkKjVe0jgxrmDF3wvaZXacnjqN7gi1z4A24De1a1cQcElaGqsgsX90SuY+zbfRkCKzCHi+1alS1Bck3MMxKyBowK/SG6mEAVW5KI83smpug9eLQpTpIlIS3QEupobwj4095kAa+JpvaJpMdyjPc59Q77OjeBcKYBNYDSyByA+/yIhHKdEE5vij9ozphk57peHe+77mdrTNRp9+25UswCqZFrl128Ew2aGff+qo7VBscwsdZaAq0+9XLgLxPKs0J/v7Fa9GUsq4YUmduM= ditoihkams@DESKTOP-EN5FRGH"
  billing_account_id = "1200201911"
}

resource "idcloudhost_floating_ip" "ipsatu" {
  name              = "appserverIP"
  billing_account_id = "1200201911"
  assigned_to       = idcloudhost_vm.appserver.id  # Koreksi nama atribut dan tambahkan ".appserver"
}
```

2. gateway.tf
```
terraform {
  required_providers {
    idcloudhost = {
      source  = "bapung/idcloudhost"
      version = "0.1.3"
    }
  }
}

provider "idcloudhost" {
  # Configuration options
  auth_token = "H4DgjZQTQh9JIenbqc64HHQycLOdfQG7"
}

resource "idcloudhost_vm" "gateway" {
  name           = "ditoihkam-gateway"       # Tambahkan tanda kutip ganda yang hilang di bagian ini
  os_name        = "ubuntu"
  os_version     = "20.04"
  vcpu           = "2"
  memory         = "2048"
  disks          = "20"
  username       = "ditoihkams"
  initial_password = "Katasand1"    # Tambahkan tanda kutip ganda yang hilang di bagian ini
  public_key     = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC161240SqH/DxKtpWIcD8j8eeuhQJXVsRRIjhucAsXuF3gs4Trbyl4RYflSW8+r+T+P/GP3graT/LEDq6KbTXeEjBm5INzixBoWySWPa1JxckqJ12dPvkQAJwJ6Rs1ZUiG7rfbOnD4gq1pbt7NHY8SAN1rx2JT0NJIGZtFWUbSuwVWW0C0FdUTPdFD3hScHgkt/45HGRQzmN4QiKX40mymLwEQ8E7wwB8DHrNZsFEnSeFrkpGRkKjVe0jgxrmDF3wvaZXacnjqN7gi1z4A24De1a1cQcElaGqsgsX90SuY+zbfRkCKzCHi+1alS1Bck3MMxKyBowK/SG6mEAVW5KI83smpug9eLQpTpIlIS3QEupobwj4095kAa+JpvaJpMdyjPc59Q77OjeBcKYBNYDSyByA+/yIhHKdEE5vij9ozphk57peHe+77mdrTNRp9+25UswCqZFrl128Ew2aGff+qo7VBscwsdZaAq0+9XLgLxPKs0J/v7Fa9GUsq4YUmduM= ditoihkams@DESKTOP-EN5FRGH"
  billing_account_id = "1200201911"
}

resource "idcloudhost_floating_ip" "ipdua" {
  name              = "gatewayIP"
  billing_account_id = "1200201911"
  assigned_to       = idcloudhost_vm.gateway.id  # Koreksi nama atribut dan tambahkan ".gateway"
}
```

3. monitoring.tf
```
terraform {
  required_providers {
    idcloudhost = {
      source  = "bapung/idcloudhost"
      version = "0.1.3"
    }
  }
}

provider "idcloudhost" {
  # Configuration options
  auth_token = "H4DgjZQTQh9JIenbqc64HHQycLOdfQG7"
}

resource "idcloudhost_vm" "monitoring" {
  name           = "ditoihkam-monitoring"       # Tambahkan tanda kutip ganda yang hilang di bagian ini
  os_name        = "ubuntu"
  os_version     = "20.04"
  vcpu           = "2"
  memory         = "2048"
  disks          = "20"
  username       = "ditoihkams"
  initial_password = "Katasand1"    # Tambahkan tanda kutip ganda yang hilang di bagian ini
  public_key     = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC161240SqH/DxKtpWIcD8j8eeuhQJXVsRRIjhucAsXuF3gs4Trbyl4RYflSW8+r+T+P/GP3graT/LEDq6KbTXeEjBm5INzixBoWySWPa1JxckqJ12dPvkQAJwJ6Rs1ZUiG7rfbOnD4gq1pbt7NHY8SAN1rx2JT0NJIGZtFWUbSuwVWW0C0FdUTPdFD3hScHgkt/45HGRQzmN4QiKX40mymLwEQ8E7wwB8DHrNZsFEnSeFrkpGRkKjVe0jgxrmDF3wvaZXacnjqN7gi1z4A24De1a1cQcElaGqsgsX90SuY+zbfRkCKzCHi+1alS1Bck3MMxKyBowK/SG6mEAVW5KI83smpug9eLQpTpIlIS3QEupobwj4095kAa+JpvaJpMdyjPc59Q77OjeBcKYBNYDSyByA+/yIhHKdEE5vij9ozphk57peHe+77mdrTNRp9+25UswCqZFrl128Ew2aGff+qo7VBscwsdZaAq0+9XLgLxPKs0J/v7Fa9GUsq4YUmduM= ditoihkams@DESKTOP-EN5FRGH"
  billing_account_id = "1200201911"
}

resource "idcloudhost_floating_ip" "iptiga" {
  name              = "monitoringIP"
  billing_account_id = "1200201911"
  assigned_to       = idcloudhost_vm.monitoring.id  # Koreksi nama atribut dan tambahkan ".monitoring"
}
```
lalu setelah kode dibuat, jalankan perintah terraform di masing2 folder dengan perintah sebagai berikut :

1. Terraform init (menginisialisasi konfigurasi Terraform agar bisa menjalankan perintah terraform lainnya)
2. Terraform validate (memvalidasi source code terraform tidak ada error)
3. Terraform plan (opsional, melihat rencana untuk kode kedepannya)
4. Terraform apply (untuk menjalankan terraform)


(gambar terraform init, validate, plan, dan apply)


## NGINX Installation


Setelah server tersedia, yang pertama saya sediakan disini yaitu menginstall NGINX serta menyediakan reverse proxy nya di server gateway mengggunakan `ansible` yang sudah terinstall. Reverse proxy sudah harus disediakan karna kita sekaligus mengcopy reverse proxy ke dalam server. Source code dan penjalanan kode ada dibawah ini

* Reverse Proxy
```
server {
    server_name ditoihkamp.studentdumbways.my.id;

    location / {
        proxy_pass http://27.112.78.5:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    server_name api.ditoihkamp.studentdumbways.my.id;

    location / {
        proxy_pass http://27.112.78.5:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    server_name node-appserver.ditoihkamp.studentdumbways.my.id;

    location / {
        proxy_pass http://27.112.78.5:9100;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    server_name node-gateway.ditoihkamp.studentdumbways.my.id;

    location / {
        proxy_pass http://103.226.139.168:9100;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    server_name node-monit.ditoihkamp.studentdumbways.my.id;

    location / {
        proxy_pass http://103.176.79.201:9100;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    server_name jenkins.ditoihkamp.studentdumbways.my.id;

    location / {
        proxy_pass http://103.176.79.201:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    server_name graf.ditoihkamp.studentdumbways.my.id;

    location / {
        proxy_pass http://103.176.79.201:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    server_name prom.ditoihkamp.studentdumbways.my.id;

    location / {
        proxy_pass http://103.176.79.201:9090;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# appserver 27.112.78.5
# gateway 103.226.139.168
# monitoring 103.176.79.201
```

Juga untuk ansible playbook disini script untuk penginstallan nginx dan copy paste reverse proxy ke vm gateway

```
- hosts: gateway
  become: true
  tasks:
    - name: Installing nginx
      apt:
        name: nginx
        state: latest
        update_cache: yes

    - name: Start nginx
      service:
        name: nginx
        state: started

    - name: Copy reverse-proxy
      copy:
        src: ~/srdevops/vm2/reverse-proxy.conf
        dest: /etc/nginx/sites-enabled

    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
```


(nginx playbook image)



saya disini juga menyetel ip dns ip di cloudflare kearah server gateway semua seperti ini agar ketika mengakses web tidak menggunakan ip

(dns image settings)


juga untuk ssh management, disini saya menggunakan ssh config agar kita tidak perlu mengingat nomor ip yang sulit/ip diganti menggunakan alias yang mudah kita ingate
```
host gateway
    HostName (ip gateway)
    User ditoihkam

host appserver
    Hostname (ip appserver)
    User ditoihkam
    ProxyCommand ssh gateway -W %h:%p
    IdentityFile /home/ditoihkam/.ssh/id_rsa

host monitoring
    Hostname (ip monitoring)
    User ditoihkam
    ProxyCommand ssh gateway -W %h:%p
    identityFile /home/ditoihkam/.ssh/id_rsa
```

berikut untuk hasil dari installation seperti grafana, node exporter, prometheus

(gambar web nodeexporter ,prometheus, dan grafana)



