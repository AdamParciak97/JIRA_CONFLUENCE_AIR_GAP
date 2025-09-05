# JIRA_CONFLUENCE_AIR_GAP

## Architecture:

| Machine | Jira | Confluence | Postgres | PROXY|
| --------------- | --------------- | --------------- |--------------- |--------------- |
| CPU | 16vCPU | 16vCPU2 | 8vCPU | 4vCPU|
| RAM | 32GB | 32GB | 24GB | 8BG |
| DISK | 300GB | 300GB | 300GB | 100 GB|
| JAVA | OpenJDK11 | OpenJDK11 |  |  |
| System | Oracle Linux 8.10 | Oracle Linux 8.10 | Oracle Linux 8.10 | Oracle Linux 8.10 |

## Adressing. To ensure proper addressing of automation device addresses, please use the following key:
### 192.168.10.180 – Jira;
### 192.168.10.181 – Confluence;
### 192.168.10.182 – Postgres;
### 192.168.10.57 - Proxy.

## Download jira.bin, confluence.bin and OpenJDK17U-jdk_x64_linux_hotspot_17.0.8.1_1.tar.gz
## Rpm for postgres is on this project

## Create machine on Vsphere
### During installation, select the server installation option.
<img width="609" height="388" alt="image" src="https://github.com/user-attachments/assets/8d4c3d1d-cec3-4899-8271-4e651c6afa25" />

### In the network configuration, you must enter the address, mask, gateway, and DNS addresses.
<img width="691" height="305" alt="image" src="https://github.com/user-attachments/assets/57cddc8c-9c0b-4349-ab60-c826cb4cbdd0" />

### You must set a hostname for each machine.
<img width="783" height="152" alt="image" src="https://github.com/user-attachments/assets/2e0caf35-c6fa-4ecc-b0ab-ae790ab5c005" />

### Create a new user (super_admin) with sudo privileges. We also set a password for the root account.
<img width="818" height="423" alt="image" src="https://github.com/user-attachments/assets/d07a4cfa-52a7-4d54-9f22-af775232b013" />

### The final step is to split the partitions. They should look like this:
<img width="647" height="548" alt="image" src="https://github.com/user-attachments/assets/2ae3965a-31e4-497b-acd2-a2743aab01d0" />

### Click Begin Installation and wait for the installation to complete.

## If we have repostiries on other machine, we can update software to new package. This should be done by editing the local.repo file in the /etc/yum.repos.d directory. The local.repo file looks like this.

<img width="582" height="282" alt="image" src="https://github.com/user-attachments/assets/62a39af9-e515-44ee-917e-791df85f5b9c" />

#### This command use to check repository and update software
```bash
dnf repolist
sudo dnf update -y
```

## Java must be installed on each machine. Transfer the file to the machine using scp or WinSCP. Then, add the command line to the Java installation.
```bash
mkdir -p /opt/java
sudo tar -xzf OpenJDK17U-jdk_x64_linux_hotspot_17.0.8.1_1.tar.gz -C /opt/java

sudo tee /etc/profile.d/jdk17.sh > /dev/null <<EOF
export JAVA_HOME=/opt/java/jdk-17.0.8.1+1
export PATH=\$JAVA_HOME/bin:\$PATH
EOF

source /etc/profile.d/jdk17.sh
java -version
```

## Modify NTP on machine. Open file chrony conf.
```bash
sudo nano /etc/chrony.conf
```

### Edit file like this:
```bash
server 192.168.10.100 iburst
```

### Restart and automatic start
```bash
sudo systemctl enable --now chronyd
sudo systemctl restart chronyd
```

### Check NTP 
```bash
sudo chronyc tracking
```

# POSTGRES MACHINE
## Copy file rpm to directory /home/super_admin
## Install rpm
```bash
cd /home/super_admin
sudo dnf install ./*rpm -y
```
<img width="980" height="206" alt="image" src="https://github.com/user-attachments/assets/a706296d-9620-49a5-9643-bd9db4784477" />

## Next we need to initialize the postgres database and add to autostart
```bash
sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
sudo systemctl enable --now postgresql-15
```

## The next step is to change the password of the postgres user.
```bash
sudo su - postgres
psql
\password

\q
exit
```
<img width="889" height="296" alt="image" src="https://github.com/user-attachments/assets/ecd4d681-1b3e-4a3d-855c-82857081449b" />

## The next step is to create users and the Jira and Confluence databases. We do this by logging into the Postgres user account and then entering database mode.
```bash
sudo su - postgres
psql

CREATE USER jira WITH PASSWORD 'SilneHaslo1';
CREATE DATABASE jiradb WITH ENCODING 'UNICODE' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE=template0 OWNER=jira;

CREATE USER confluence WITH PASSWORD 'SilneHaslo2';
CREATE DATABASE confluencedb WITH ENCODING 'UNICODE' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE=template0 OWNER=confluence;

\q
exit
```

## The final step on the Postgres machine is to configure remote access from the Jira and Confluence machines. Edit two files in the var/lib/pgsql/15/data directory. 
### The first file is postgresql.conf. Change line 60 to the following: It will listen on all interfaces.
```bash
cd var/lib/pgsql/15/data
```
<img width="741" height="190" alt="image" src="https://github.com/user-attachments/assets/fb85d7c0-bdc5-4721-8f99-e6f4f5dd61df" />

### The second file is pg_hba.conf, we add a line in the database section, in this case line 90. This will allow us to log in from external IP addresses.
```bash
cd var/lib/pgsql/15/pg_hba.conf
```
<img width="663" height="231" alt="image" src="https://github.com/user-attachments/assets/56a7a916-c1f9-4959-9b31-66a269868d3e" />

## Restart service postgresql-15
```bash
sudo systemctl restart postgresql-15
```

# Jira
## Copy file jira.bin to machine to directory /home/super_admin. Then we give it execute permissions and run the file.
```bash
cd /home/super_admin
chmod +x atlassian-jira-software-*-x64.bin
./atlassian-jira-software-*-x64.bin
```

## During installation, select the Custom installation option. Once installed, go to your browser and enter http://192.168.10.180:8080.

### Select the "I'll set it up myself" option. Then, provide the database configuration information. Perform a connection test. Port 5432/TCP must be added to the Jira and Postgres machines for communication. Otherwise, the firewall will not allow it through.
<img width="553" height="661" alt="image" src="https://github.com/user-attachments/assets/d0855ee3-a827-47de-a563-434a0ab78e1e" />

<img width="745" height="446" alt="image" src="https://github.com/user-attachments/assets/05039b40-6625-4b9a-8b16-afb5f39b896a" />

### The next step is to upload a license. Without it, we cannot proceed to the next configuration stage. A trial license has been generated for testing purposes.
<img width="980" height="189" alt="image" src="https://github.com/user-attachments/assets/20e9288f-6c3f-4572-90e7-5916eb426f24" />

### The next step is to configure the administrator account.
<img width="840" height="478" alt="image" src="https://github.com/user-attachments/assets/431689b5-429d-4e4c-ad8e-9b5705ab2cf0" />

### Select Finish. Configure the rest of the settings, select the Polish language, and choose an avatar. Then, you'll see a window to choose one of the options below.
<img width="980" height="309" alt="image" src="https://github.com/user-attachments/assets/cd6bc2ed-69fc-41a5-991f-f305df483e0e" />

# Confluence
## Copy file confluence.bin to machine to directory /home/super_admin. Then we give it execute permissions and run the file.
```bash
cd /home/super_admin
chmod +x atlassian-confluence-*-x64.bin
./atlassian-confluence-*-x64.bin
```
## During installation, select the Custom installation option. Once installed, go to your browser and enter http://192.168.10.181:8090
### Selecet Production Installation
<img width="755" height="510" alt="image" src="https://github.com/user-attachments/assets/5e82c240-6664-467b-ad49-66e4488471d8" />

### Import licence
<img width="546" height="412" alt="image" src="https://github.com/user-attachments/assets/e9558f88-8796-4bf0-aa06-8070568cf5cc" />

### Configure connection do database postgres
<img width="654" height="516" alt="image" src="https://github.com/user-attachments/assets/87eb98ca-ddd3-42d8-b044-4a42a59b5724" />

### Last steps
<img width="726" height="527" alt="image" src="https://github.com/user-attachments/assets/93682737-1c7f-468e-8aa7-cfa2d849290b" />

<img width="640" height="365" alt="image" src="https://github.com/user-attachments/assets/36e02bc1-59a8-4f22-aced-dbea3f65888a" />

<img width="599" height="188" alt="image" src="https://github.com/user-attachments/assets/61594cf5-3d08-4b3a-a40f-f2fd359ff5a7" />


# PROXY
### Install Oracle Linux 8 on machine from ISO file
### Download docker image nginx on docker desktop
### Create directory in opt
```bash
sudo mkdir -p /opt/nginx-proxy/{certs,logs}
cd /opt/nginx-proxy/certs
```
### Create CA
```bash
openssl genrsa -out labCA.key 4096

openssl req -x509 -new -nodes -key labCA.key -sha256 -days 3650 \
  -subj "/C=PL/O=Lab/CN=Lab Root CA" -out labCA.crt
```

### Create file SAN
```bash
cat > san.cnf <<'EOF'
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = dn
req_extensions     = v3_req

[ dn ]
C = PL
O = Lab
CN = 192.168.10.57

[ v3_req ]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[ alt_names ]
IP.1 = 192.168.10.57
EOF
```

### Create key and CSR for proxy
```bash
openssl genrsa -out proxy.key 2048
openssl req -new -key proxy.key -out proxy.csr -config san.cnf
```

### Sign CSR
```bash
openssl x509 -req -in proxy.csr -CA labCA.crt -CAkey labCA.key -CAcreateserial \ -out proxy.crt -days 825 -sha256 -extensions v3_req -extfile san.cnf
```

### Create docker-compose.yml
```bash
version: "3.8"

services:
  nginx:
    image: nginx:stable-perl
    container_name: ngnix-reverse-proxy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
      - ./logs:/var/log/nginx
```

### Create nginx.conf
```bash
user  nginx;
worker_processes auto;

events { worker_connections 4096; }

http {
  include       /etc/nginx/mime.types;
  default_type  application/octet-stream;

  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
  access_log /var/log/nginx/access.log main;
  error_log  /var/log/nginx/error.log warn;

  map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
  }

  # --- Backendy ---
  upstream jira_upstream {
    server 192.168.10.180:8080;
    keepalive 64;
  }
  upstream confluence_upstream {
    server 192.168.10.181:8090;
    keepalive 64;
  }

  # --- HTTP -> HTTPS ---
  server {
    listen 80;
    return 301 https://$host$request_uri;
  }

  # --- HTTPS (proxy 192.168.10.57) ---
  server {
    listen 443 ssl http2;
    server_name _;

    ssl_certificate     /etc/nginx/certs/proxy.crt;
    ssl_certificate_key /etc/nginx/certs/proxy.key;
    # ssl_trusted_certificate /etc/nginx/certs/labCA.crt;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    client_max_body_size 200m;

    # --- ROOT -> /jira/ (tylko strona główna) ---
    location = / {
      return 301 /jira/;
    }

    # --- /login.jsp -> wewnętrznie do /jira/login.jsp (BEZ 301!) ---
    location = /login.jsp {
      rewrite ^ /jira/login.jsp last;
    }

    # ========================
    #         JIRA
    # ========================
    location = /jira { return 301 /jira/; }

    location ^~ /jira/ {
      # Jira ma contextPath=/jira → wysyłamy dokładnie /jira/... do backendu
      proxy_pass http://jira_upstream;

      proxy_http_version 1.1;
      proxy_read_timeout 300;
      proxy_connect_timeout 60;
      proxy_send_timeout 60;

      proxy_set_header Host              $host;
      proxy_set_header X-Real-IP         $remote_addr;
      proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Host  $host;
      proxy_set_header X-Forwarded-Port  443;
      proxy_set_header X-Forwarded-Proto https;

      # Jeśli Jira da 'Location: /coś', dodaj prefiks /jira (nie dotyka query stringów)
      proxy_redirect off;
    }

    # ========================
    #      CONFLUENCE
    # ========================
    location = /confluence { return 301 /confluence/; }

    location ^~ /confluence/ {
      # Confluence zwykle ma path=/confluence → wysyłamy /confluence/... do backendu
      proxy_pass http://confluence_upstream;

      proxy_http_version 1.1;
      proxy_read_timeout 300;
      proxy_connect_timeout 60;
      proxy_send_timeout 60;

      proxy_set_header Host              $host;
      proxy_set_header X-Real-IP         $remote_addr;
      proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Host  $host;
      proxy_set_header X-Forwarded-Port  443;
      proxy_set_header X-Forwarded-Proto https;

      proxy_set_header Upgrade           $http_upgrade;
      proxy_set_header Connection        $connection_upgrade;

      proxy_redirect ~^(/[^/].*)$ /confluence$1;
    }
  }
}
```

### Run docker compose
```bash
cd /opt/nginx-proxy
docker compose up -d
docker logs -f ngnix-reverse-proxy
```

### Modify file "server.xml" on jira and confluence

## Jira

```bash
nano /opt/atlassian/jira/conf/server.xml
```

```bash

<?xml version="1.0" encoding="utf-8"?>
<!--
  Apache License, Version 2.0
-->
<Server port="8005" shutdown="SHUTDOWN">
    <Listener className="org.apache.catalina.startup.VersionLoggerListener"/>
    <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on"/>
    <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener"/>
    <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener"/>
    <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener"/>

    <Service name="Catalina">
        <!-- ===========================================================================================================
             DEFAULT connector (no proxy) — DISABLED
             =========================================================================================================== -->
        <!--
        <Connector port="8080" relaxedPathChars="[]|" relaxedQueryChars="[]|{}^&#x5c;&#x60;&quot;&lt;&gt;"
                   maxThreads="150" minSpareThreads="25" connectionTimeout="20000" enableLookups="false"
                   maxHttpHeaderSize="8192" protocol="HTTP/1.1" useBodyEncodingForURI="true" redirectPort="8443"
                   acceptCount="100" disableUploadTimeout="true" bindOnInit="false"/>
        -->

        <!-- ===========================================================================================================
             HTTPS - Proxying Jira via Nginx/Apache over HTTPS — ENABLED
             =========================================================================================================== -->
        <Connector port="8080"
                   relaxedPathChars="[]|" relaxedQueryChars="[]|{}^&#x5c;&#x60;&quot;&lt;&gt;"
                   maxThreads="150" minSpareThreads="25" connectionTimeout="20000" enableLookups="false"
                   maxHttpHeaderSize="8192" protocol="HTTP/1.1" useBodyEncodingForURI="true" redirectPort="8443"
                   acceptCount="100" disableUploadTimeout="true" bindOnInit="false"
                   secure="true" scheme="https"
                   proxyName="192.168.10.57" proxyPort="443"/>

        <!-- ===========================================================================================================
             AJP connector — DISABLED (niepotrzebny)
             =========================================================================================================== -->
        <!--
        <Connector port="8009" URIEncoding="UTF-8" enableLookups="false" protocol="AJP/1.3"/>
        -->

        <Engine name="Catalina" defaultHost="localhost">
            <Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">

                <!-- Context path ustawiony na /jira -->
                <Context path="/jira"
                         docBase="${catalina.home}/atlassian-jira"
                         reloadable="false" useHttpOnly="true">
                    <Resource name="UserTransaction" auth="Container" type="javax.transaction.UserTransaction"
                              factory="org.objectweb.jotm.UserTransactionFactory" jotm.timeout="60"/>
                    <Manager pathname=""/>
                    <JarScanner scanManifest="false"/>
                    <Valve className="org.apache.catalina.valves.StuckThreadDetectionValve" threshold="120"/>
                </Context>

            </Host>
            <Valve className="org.apache.catalina.valves.AccessLogValve"
                   pattern="%a %{jira.request.id}r %{jira.request.username}r %t &quot;%m %U%{sanitized.query}r %H&quot; %s %b %D &quot;%{sanitized.referer}r&quot; &quot;%{User-Agent}i&quot; &quot;%{jira.request.assession.id}r&quot;"/>
        </Engine>
    </Service>
</Server>

```

## Confluence

```bash
nano /opt/atlassian/confluence/conf/server.xml
```

```bash
<Connector port="8090"
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           proxyName="192.168.10.57"
           proxyPort="443"
           scheme="https"
           secure="true"
           URIEncoding="UTF-8" />

<Context path="/confluence" docBase="../confluence" debug="0" reloadable="false"/>
```

### Restart service Jira and Confluence
```bash
sudo service jira stop && sudo service jira start
sudo service confluence stop && sudo service confluence start
```

### Change on GUI baseURL
<img width="1150" height="431" alt="image" src="https://github.com/user-attachments/assets/69cf9db8-b271-4541-98c5-195ff22b1d3f" />

<img width="1379" height="363" alt="image" src="https://github.com/user-attachments/assets/3cfc17a4-4f79-4ea9-b37c-f465165f8d2d" />


# Integration
<img width="462" height="113" alt="image" src="https://github.com/user-attachments/assets/2c3672d7-abe3-423e-a65b-a17773113e47" />

### Next select the option to configure Jira with Confluence. Click the hyperlink and enter the administrator password.
<img width="539" height="295" alt="image" src="https://github.com/user-attachments/assets/a6e6727b-4e0d-45a8-afb5-64a34dc51e63" />

### Create links
<img width="980" height="257" alt="image" src="https://github.com/user-attachments/assets/ca9377ef-bb19-49b7-bc3c-1db9291a8e50" />

<img width="546" height="464" alt="image" src="https://github.com/user-attachments/assets/e5de22b6-3ebc-49f0-909e-d2c92166e9ee" />

<img width="394" height="527" alt="image" src="https://github.com/user-attachments/assets/f454eb1b-88b4-4e2d-8222-87600bd1035e" />

### Connecting
<img width="980" height="131" alt="image" src="https://github.com/user-attachments/assets/9c0bf886-41c6-4d54-89b4-13b7398269b2" />
<img width="980" height="125" alt="image" src="https://github.com/user-attachments/assets/03e1794f-6dbc-4d00-99bb-85149ba3dbd8" />
