## h6 Bonus

Tehtävät ovat Tero Karvisen opintojaksolta [Palvelinten Hallinta 2025 kevät](https://terokarvinen.com/palvelinten-hallinta/) [^1]

---

#### Laite jolla tehtävät tehdään:

- Apple MacBook Pro M2 Max
- macOS Sequoia 15.3.1
- Parallels Desktop

---

## Vapaaehtoinen bonus: luettele ja linkitä tähän tekemäsi vapaaehtoiset tehtävät

h4:sta minulla oli jäänyt tekemättä d, e ja f. Tein tehtäviä varten seuraavan Vagrantfilen:

```bash
$minion = <<MINION
sudo apt-get update
mkdir -p /etc/apt/keyrings
cp /vagrant_data/SaltProjectKey.gpg.pub /etc/apt/keyrings/salt-archive-keyring.pgp
cp /vagrant_data/salt.sources /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get -qy install salt-minion
echo "master: 192.168.51.4">/etc/salt/minion
sudo systemctl restart salt-minion.service
MINION

$master = <<MASTER
sudo apt-get update
mkdir -p /etc/apt/keyrings
cp /vagrant_data/SaltProjectKey.gpg.pub /etc/apt/keyrings/salt-archive-keyring.pgp
cp /vagrant_data/salt.sources /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get -qy install salt-master
sudo systemctl start salt-master.service
MASTER

Vagrant.configure("2") do |config|

  config.vm.box = "bento/debian-12"
  config.vm.box_version = "202502.21.0"

  config.vm.define "master" do |master|
      master.vm.hostname = "master"
      master.vm.network "private_network", ip: "192.168.51.4"
      master.vm.synced_folder ".", "/vagrant_data"
      master.vm.provision :shell, inline: $master
  end

  config.vm.define "minion001" do |minion001|
      minion001.vm.hostname = "minion001"
      minion001.vm.network "private_network", ip: "192.168.51.5"
      minion001.vm.synced_folder ".", "/vagrant_data"
      minion001.vm.provision :shell, inline: $minion
  end

  config.vm.define "minion002" do |minion002|
      minion002.vm.hostname = "minion002"
      minion002.vm.network "private_network", ip: "192.168.51.6"
      minion002.vm.synced_folder ".", "/vagrant_data"
      minion002.vm.provision :shell, inline: $minion
  end

  config.vm.define "minion003" do |minion003|
      minion003.vm.hostname = "minion003"
      minion003.vm.network "private_network", ip: "192.168.51.7"
      minion003.vm.synced_folder ".", "/vagrant_data"
      minion003.vm.provision :shell, inline: $minion
  end

end
```


### h4: c) Vapaaehtoinen, haastavahko tässä vaiheessa: Asenna ja konfiguroi Apache ja Name Based Virtual Host. Sen tulee näyttää palvelimen etusivulla weppisivua. Weppisivun tulee olla muokattavissa käyttäjän oikeuksin, ilman sudoa.

https://github.com/pinkkila/palvelinten-hallinta/blob/main/h4-pkg-file-service.md#c-asenna-ja-konfiguroi-apache-ja-name-based-virtual-host-sen-tulee-näyttää-palvelimen-etusivulla-weppisivua-weppisivun-tulee-olla-muokattavissa-käyttäjän-oikeuksin-ilman-sudoa

---

### h4: d) Vapaaehtoinen, haastava: Caddy. Asenna Caddy tarjoilemaan weppisivua. Weppisivun tulee näkyä palvelimen etusivulla (localhost). HTML:n tulee olla jonkun käyttäjän kotihakemistossa, ja olla muokattavissa normaalin käyttäjän oikeuksin, ilman sudoa.




---

### h4: e) Vapaaehtoinen, haastava: Nginx. Asenna Nginx (lausutaan engine-X) tarjoilemaan weppisivua. Weppisivun tulee näkyä palvelimen etusivulla (localhost). HTML:n tulee olla jonkun käyttäjän kotihakemistossa, ja olla muokattavissa normaalin käyttäjän oikeuksin, ilman sudoa.




---

### h4: f) Vapaaehtoinen, haastava: PostgreSQL. Asenna PostgreSQL-tietokannanhallintajärjestelmä. Anna jollekin käyttäjälle oma tietokanta. Osoita testillä, että se toimii.

Miniprojektista [^2] minulla oli jo seuraava moduuli init.sls:

```yaml
postgresql:
  pkg.installed
          
myuser:
  postgres_user.present:
    - password: secret
    - encrypted: scram-sha-256
    - require:
        - pkg: postgresql
  
shop_db:
  postgres_database.present:
    - owner: myuser
    - require:
      - postgres_user: myuser
```

Karvisen ohjeessa [^3] neuvotaan kätevä tapa luoda tietokanta käyttämällä tietokannan nimessä ja käyttäjässä linux käyttäjän nimeä, jolloin autentikointi on automaattista.   

Kokeillaan tätä ensin minion003:ssa täysin käsipelillä (komennot Karvisen ohjeesta [^3]:

```bash
vagrant@minion003:~$ sudo -u postgres createdb $(whoami)
could not change directory to "/home/vagrant": Permission denied
vagrant@minion003:~$ sudo -u postgres createdb $(whoami)
could not change directory to "/home/vagrant": Permission denied
createdb: error: database creation failed: ERROR:  database "vagrant" already exists
vagrant@minion003:~$ sudo -u postgres createuser $(whoami)
could not change directory to "/home/vagrant": Permission denied
vagrant@minion003:~$ pwd
/home/vagrant
vagrant@minion003:~$ psql
psql (15.12 (Debian 15.12-0+deb12u2))
Type "help" for help.

vagrant=>
```

Seuraavaksi kokeilin seuraavaa ja törmäsin samaan tilanteeseen kuin Miniprojektissa `permission denied for schema public`, eli varmaankin owner tulisi lisätä.

```bash
vagrant@minion003:~$ psql
psql (15.12 (Debian 15.12-0+deb12u2))
Type "help" for help.

vagrant=> \d
Did not find any relations.
vagrant=> \create table animal (id bigserial primary key, animal_type text);
invalid command \create
Try \? for help.
vagrant=> \CREATE TABLE animal (id bigserial primary key, animal_type text);
invalid command \CREATE
Try \? for help.
vagrant=> \l
                                                 List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 vagrant   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
(4 rows)

vagrant=> CREATE TABLE animal (id bigserial primary key, animal_type text);
ERROR:  permission denied for schema public
LINE 1: CREATE TABLE animal (id bigserial primary key, animal_type t...
```

Ja oletetusti (komento [^5]: 

```bash
vagrant=>  ALTER DATABASE vagrant OWNER TO vagrant;
ERROR:  must be owner of database vagrant
```

Kuten edellistä aiemmasta tulosteesta näkee vagrant databasen Owner on postgres. 

```bash
vagrant@minion003:~$ cat /etc/passwd
# removed lines 
postgres:x:104:111:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash

vagrant@minion003:~$ psql -U postgres
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "postgres"
vagrant@minion003:~$ psql -U postgres -d vagrant
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "postgres"
```

Tämän ihan hyvän oloisen ohjeen [^4] mukaan komennolla `sudo -u postgres psql`, eli vaihdetaan käyttäjä `sudo -u postgres`, jolla psql on kaiketi ehkä ihan ok tapa tehdä tämä ownerin vaihto.  

Myös tämä tieto oli selkeyttävä siinä mielessä, että se mitä aiemman pystyi olettamaan on myös varmarmastikin niin postgresin asunnus luo postgers super userin jne.:

"The PostgreSQL installation creates a "UNIX USER" called postgres, who is ALSO the "Default PostgreSQL's SUPERUSER". The UNIX USER postgres cannot login interactively to the system, as its password is not enabled" [^4].

```bash
vagrant@minion003:~$ sudo -u postgres psql
could not change directory to "/home/vagrant": Permission denied
psql (15.12 (Debian 15.12-0+deb12u2))
Type "help" for help.

postgres=# ALTER DATABASE vagrant OWNER TO vagrant;
ALTER DATABASE
postgres=#
```

```bash
vagrant@minion003:~$ psql
psql (15.12 (Debian 15.12-0+deb12u2))
Type "help" for help.

vagrant=> CREATE TABLE animal (id bigserial primary key, animal_type text);
CREATE TABLE
vagrant=> SELECT * FROM animal;
 id | animal_type
----+-------------
(0 rows)

vagrant=> INSERT INTO animal VALUES (99, 'cat');
INSERT 0 1
vagrant=> SELECT * FROM animal;
 id | animal_type
----+-------------
 99 | cat
(1 row)

vagrant=>
```

Nyt päästään tekemään sama Salt:lla. Muokkasin Miniprojektissa käyttämääni `init.sls` tiedoston seuraavaksi:

```yaml
postgresql:
  pkg.installed
          
vagrant:
  postgres_user.present:
    - require:
      - pkg: postgresql
  
vagrant_db:
  postgres_database.present:
    - name: vagrant
    - owner: vagrant
    - require:
      - postgres_user: vagrant
```

```bash
vagrant@master:~$ sudo mkdir -p /srv/salt/postgres
vagrant@master:~$ cd /srv/salt/postgres/
vagrant@master:/srv/salt/postgres$ sudoedit init.sls
vagrant@master:~$ sudo salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
minion001
minion002
minion003
Proceed? [n/Y] y
Key for minion minion001 accepted.
Key for minion minion002 accepted.
Key for minion minion003 accepted.
```

```bash
vagrant@master:/srv/salt/postgres$ sudo salt 'minion001' state.apply postgres
minion001:
----------
          ID: postgresql
    Function: pkg.installed
      Result: True
     Comment: The following packages were installed/updated: postgresql
     Started: 06:56:28.032034
    Duration: 10075.217 ms
     Changes:
              ----------
              libcommon-sense-perl:
                  ----------
                  new:
                      3.75-3
                  old:
              libjson-perl:
                  ----------
                  new:
                      4.10000-1
                  old:
              libjson-xs-perl:
                  ----------
                  new:
                      4.030-2+b1
                  old:
              libllvm14:
                  ----------
                  new:
                      1:14.0.6-12
                  old:
              libpq5:
                  ----------
                  new:
                      15.12-0+deb12u2
                  old:
              libsensors-config:
                  ----------
                  new:
                      1:3.6.0-7.1
                  old:
              libsensors5:
                  ----------
                  new:
                      1:3.6.0-7.1
                  old:
              libtypes-serialiser-perl:
                  ----------
                  new:
                      1.01-1
                  old:
              libxslt1.1:
                  ----------
                  new:
                      1.1.35-1+deb12u1
                  old:
              libz3-4:
                  ----------
                  new:
                      4.8.12-3.1
                  old:
              postgresql:
                  ----------
                  new:
                      15+248
                  old:
              postgresql-15:
                  ----------
                  new:
                      15.12-0+deb12u2
                  old:
              postgresql-client-15:
                  ----------
                  new:
                      15.12-0+deb12u2
                  old:
              postgresql-client-common:
                  ----------
                  new:
                      248
                  old:
              postgresql-common:
                  ----------
                  new:
                      248
                  old:
              ssl-cert:
                  ----------
                  new:
                      1.1.2
                  old:
              sysstat:
                  ----------
                  new:
                      12.6.1-1
                  old:
----------
          ID: vagrant
    Function: postgres_user.present
      Result: True
     Comment: The user vagrant has been created
     Started: 06:56:38.108761
    Duration: 347.015 ms
     Changes:
              ----------
              vagrant:
                  Present
----------
          ID: vagrant_db
    Function: postgres_database.present
        Name: vagrant
      Result: True
     Comment: The database vagrant has been created
     Started: 06:56:38.456471
    Duration: 148.484 ms
     Changes:
              ----------
              vagrant:
                  Present

Summary for minion001
------------
Succeeded: 3 (changed=3)
Failed:    0
------------
Total states run:     3
Total run time:  10.571 s
vagrant@master:/srv/salt/postgres$ sudo salt 'minion001' state.apply postgres
minion001:
----------
          ID: postgresql
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 06:56:50.345396
    Duration: 9.136 ms
     Changes:
----------
          ID: vagrant
    Function: postgres_user.present
      Result: True
     Comment: User vagrant is already present
     Started: 06:56:50.355186
    Duration: 209.234 ms
     Changes:
----------
          ID: vagrant_db
    Function: postgres_database.present
        Name: vagrant
      Result: True
     Comment: Database vagrant is already present
     Started: 06:56:50.564847
    Duration: 72.494 ms
     Changes:

Summary for minion001
------------
Succeeded: 3
Failed:    0
------------
Total states run:     3
Total run time: 290.864 ms
```

Testataan toimiiko ja näyttää tomivan:

```bash
vagrant@minion001:~$ psql
psql (15.12 (Debian 15.12-0+deb12u2))
Type "help" for help.

vagrant=> CREATE TABLE animal (id bigserial primary key, animal_type text);
CREATE TABLE
vagrant=> INSERT INTO animal VALUES (99, 'cat');
INSERT 0 1
vagrant=> SELECT * FROM animal;
 id | animal_type
----+-------------
 99 | cat
(1 row)

vagrant=>
```


---

### Lähteet

[^1]: Tero Karvinen. Palvelinten Hallinta: https://terokarvinen.com/palvelinten-hallinta/ 

[^2]: pinkkila: h5 Miniprojekti https://github.com/pinkkila/palvelinten-hallinta/blob/main/h5-miniprojekti.md

[^3]: Tero Karvinen. PostgreSQL Install and One Table Database – SQL CRUD tutorial for Ubuntu: https://terokarvinen.com/2016/postgresql-install-and-one-table-database-sql-crud-tutorial-for-ubuntu/

[^4]: Chua Hock-Chuan. Getting Started with PostgreSQL: https://www3.ntu.edu.sg/home/ehchua/programming/sql/PostgreSQL_GetStarted.html

[^5]: The PostgreSQL Global Development Group. ALTER DATABASE: https://www.postgresql.org/docs/current/sql-alterdatabase.html