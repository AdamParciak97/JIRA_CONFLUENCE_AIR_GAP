# JIRA_CONFLUENCE_AIR_GAP

## Architecture:

| Machine | Jira | Confluence | Postgres |
| --------------- | --------------- | --------------- |--------------- |
| CPU | 16vCPU | 16vCPU2 | 8vCPU |
| RAM | 32GB | 32GB | 24GB3 |
| DISK | 300GB | 300GB | 300GB |
| JAVA | OpenJDK11 | OpenJDK11 |  |
| System | Oracle Linux 8.10 | Oracle Linux 8.10 | Oracle Linux 8.10 |

## Adressing. To ensure proper addressing of automation device addresses, please use the following key:
### 192.168.10.180 – Jira;
### 192.168.10.181 – Confluence;
### 192.168.10.182 – Postgres.

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
