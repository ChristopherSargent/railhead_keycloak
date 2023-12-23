![alt text](rh_logo_small.jpg)
# Railhead Keycloak Project
This repository contains the code and configuration instructions for MFA to Microsoft's Active Directory via a Keycloak container deployed from podman. For additional details, please email at [christopher.sargent@sargentwalker.io](mailto:christopher.sargent@sargentwalker.io).

# Prerequisites
# [Download Windows Server 2022](https://go.microsoft.com/fwlink/p/?LinkID=2195167&clcid=0x409&culture=en-us&country=US)
# [Install Windows Server 2022 Instructions](https://medium.com/@yasithkumara/creating-a-virtual-windows-server-in-windows-10-with-hyper-v-9f3bd58c0ba)
# [Active Directory Install Instructions](https://medium.com/@yasithkumara/how-to-create-a-domain-and-a-domain-controller-in-a-windows-server-2019-virtual-machine-e70587e2fbe2)
# [Ubbuntu 20.04 STIG Hardened FIPS Enabled](https://docs.google.com/document/d/1nEIavbELGl8xjHjZX4p22q5m32HCLkLH/edit#heading=h.gjdgxs)
* Take post set up snapshot on all servers 
* Create cas.local domain 

# Test VMs
* 10.1.2.85 AD server
* 10.1.2.86 keycloak01
* 10.1.2.87 keycloak02

# Install Keycloak via [podman](https://docs.podman.io/en/stable/Introduction.html)
* Note podman is in the default Ubuntu 22.04 repos but not Ubuntu 20.04
1. ssh cas@10.1.2.86
2. sudo -i 
3. apt install software-properties-common uidmap
4. sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"
5. apt update 
6. apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 4D64390375060AA4
```
Executing: /tmp/apt-key-gpghome.NKPCfh5qZD/gpg.1.sh --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 4D64390375060AA4
gpg: out of core handler ignored in FIPS mode
gpg: key 4D64390375060AA4: public key "devel:kubic OBS Project <devel:kubic@build.opensuse.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: out of core handler ignored in FIPS mode

```
7. apt update 
8. apt install podman -y 
9. podman version
```
Version:      3.4.2
API Version:  3.4.2
Go Version:   go1.15.2
Built:        Thu Jan  1 00:00:00 1970
OS/Arch:      linux/amd64
```
10. mkdir keycloak && cd keycloak 
11. apt install python3-pip -y 
12. pip3 install podman-compose
13. apt install git -y && git init 
14. git add .
15. vim keycloak.yml 
```
version: '3'

volumes:
  postgres_data:
      driver: local

services:
  postgres:
      image: postgres
      volumes:
        - ./postgres_data:/var/lib/postgresql/data
      environment:
        POSTGRES_DB: keycloak
        POSTGRES_USER: keycloak
        POSTGRES_PASSWORD: password
  keycloak:
      image: quay.io/keycloak/keycloak:latest
      environment:
        DB_VENDOR: POSTGRES
        DB_ADDR: postgres
        DB_DATABASE: keycloak
        DB_USER: keycloak
        DB_SCHEMA: public
        DB_PASSWORD: password
        KEYCLOAK_ADMIN: admin
        KEYCLOAK_ADMIN_PASSWORD: 31Nst31n!40
#        KEYCLOAK_USER: admin
#        KEYCLOAK_PASSWORD: Pa55w0rd
        PROXY_ADDRESS_FORWARDING: 'true'
        # Uncomment the line below if you want to specify JDBC parameters. The parameter below is just an example, and it shouldn't be used in production without knowledge. It is highly recommended that you read the PostgreSQL JDBC driver documentation in order to use it.
        #JDBC_PARAMS: "ssl=true"
      ports:
        - 8010:8080
      depends_on:
        - postgres
      command:
      - start-dev

```
16. podman-compose -f keycloak.yml up -d
* podman logs -t keycloak_keycloak_1 
17. http://10.1.2.86:8010/ select admin console 

![Screenshot](resources/keycloak01.png)

![Screenshot](resources/keycloak02.png)

# Notes
```
podman exec -it keycloak_keycloak_1 bash
cd /opt/keycloak/bin
./kc.sh build --features="preview"
exit 
podman restart keycloak_keycloak_1 
```