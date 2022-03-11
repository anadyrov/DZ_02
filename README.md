Описание/Пошаговая инструкция выполнения домашнего задания:
• сделать в GCE инстанс с Ubuntu 20.04 -- Выполнено
-- Пересоздал postgres@postgres2022-19840601
gcloud compute instances create postgres2022-19840601 --project=melodic-realm-343102 --zone=us-central1-a --machine-type=e2-medium --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --service-account=337881949322-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=postgres2022-19840601,image=projects/ubuntu-os-cloud/global/images/ubuntu-2004-focal-v20220308,mode=rw,size=10,type=projects/melodic-realm-343102/zones/us-central1-a/diskTypes/pd-ssd --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
Created [https://www.googleapis.com/compute/v1/projects/melodic-realm-343102/zones/us-central1-a/instances/postgres2022-19840601].
NAME: postgres2022-19840601
ZONE: us-central1-a
MACHINE_TYPE: e2-medium
PREEMPTIBLE:
INTERNAL_IP: 10.128.0.7
EXTERNAL_IP: 35.225.1.18
STATUS: RUNNING

• поставить на нем Docker Engine  -- Выполнено
anadyrov@postgres2022-19840601:~$ sudo docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
6bb9b9dfdbb7   postgres:14   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   docker

• сделать каталог /var/lib/postgres  -- Выполнено

• развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres -- Выполнено
psql (14.2 (Debian 14.2-1.pgdg110+1))
Type "help" for help.

postgres=# SHOW data_directory;
      data_directory      
--------------------------
 /var/lib/postgresql/data
(1 row)

• развернуть контейнер с клиентом postgres  -- Выполнено
sudo docker run -it --rm --network docker --name pg-client postgres:14 psql -h postgres2022-19840601 -U postgres

• подключится из контейнера с клиентом к контейнеру с сервером и сделать  
таблицу с парой строк  -- Выполнено
postgres=# CREATE DATABASE test_03;
CREATE DATABASE
postgres=# \c test_03
You are now connected to database "test_03" as user "postgres".
test_03=# CREATE TABLE test_03 (i int, amount int);
INSERT INTO test_03 VALUES (1,10), (1,20), (2,100), (2,200); 
CREATE TABLE
INSERT 0 4
test_03=# 
test_03=# select * from public.test_03;
 i | amount 
---+--------
 1 |     10
 1 |     20
 2 |    100
 2 |    200
(4 rows)

• подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP
Локально на ноутбуке развернут Centos
[root@docker docker]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"

Подключаюсь с него:

[root@docker docker]# psql -p 5432 -U postgres -h 35.225.1.18 -d postgres -W
Password:
psql (14.2)
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 test_03   | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
(4 rows)

postgres=#

• удалить контейнер с сервером
anadyrov@postgres2022-19840601:~$ sudo docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED       STATUS       PORTS                                       NAMES
6bb9b9dfdbb7   postgres:14   "docker-entrypoint.s…"   2 hours ago   Up 2 hours   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   docker
anadyrov@postgres2022-19840601:~$ psql -h localhost -U postgres -d postgres
Password for user postgres: 
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1), server 14.2 (Debian 14.2-1.pgdg110+1))
WARNING: psql major version 12, server major version 14.
         Some psql features might not work.
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 test_03   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
(4 rows)

anadyrov@postgres2022-19840601:~$ sudo docker stop 6bb9b9dfdbb7
6bb9b9dfdbb7
anadyrov@postgres2022-19840601:~$ sudo docker rm 6bb9b9dfdbb7
6bb9b9dfdbb7
anadyrov@postgres2022-19840601:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

• создать его заново
anadyrov@postgres2022-19840601:~$ sudo docker run --name docker --network docker -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
27d89f6766d7ebc9860ca4241efb9f1b43bcda6057bb5976345627f7546165f7
anadyrov@postgres2022-19840601:~$ sudo docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS         PORTS                                       NAMES
27d89f6766d7   postgres:14   "docker-entrypoint.s…"   10 seconds ago   Up 9 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   docker

• подключится снова из контейнера с клиентом к контейнеру с сервером
anadyrov@postgres2022-19840601:~$ psql -h localhost -U postgres -d test_03
Password for user postgres: 
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1), server 14.2 (Debian 14.2-1.pgdg110+1))
WARNING: psql major version 12, server major version 14.
         Some psql features might not work.
Type "help" for help.

test_03=# select * from public.test_03;
 i | amount 
---+--------
 1 |     10
 1 |     20
 2 |    100
 2 |    200
(4 rows)

anadyrov@postgres2022-19840601:~$ sudo su
root@postgres2022-19840601:/home/anadyrov# ls /var/lib/postgres/
PG_VERSION  global        pg_dynshmem  pg_ident.conf  pg_multixact  pg_replslot  pg_snapshots  pg_stat_tmp  pg_tblspc    pg_wal   postgresql.auto.conf  postmaster.opts
base        pg_commit_ts  pg_hba.conf  pg_logical     pg_notify     pg_serial    pg_stat       pg_subtrans  pg_twophase  pg_xact  postgresql.conf       postmaster.pid
• проверить, что данные остались на месте     -- Выполнено
