## h5 Miniprojekti

Tehtävät ovat Tero Karvisen opintojaksolta [Palvelinten Hallinta 2025 kevät](https://terokarvinen.com/palvelinten-hallinta/) [^1]

---

#### Laite jolla tehtävät tehdään:

- Apple MacBook Pro M2 Max
- macOS Sequoia 15.3.1
- Parallels Desktop

---

## Projektin tavoite

Valmiin projektin tarkoitus on mallintaa Backend for Frontend patternilla toteutettua sovelluskokonaisuuttaa siten, että koodin infrastruktuuria hallitaan automatisoidusti Salt:lla.

Kokonaisuus tulee sisältämään kolme Sprig Boot sovellusta ja yhden Next.js sovelluksen. Jokainen sovellus tulee olemaan omalla virtuaalikoneellaan ja jokaisella on omat Backend for Frontend patternin mukaiset tehtävät.

Yksittäiset sovellukset ovat:

- Resource Server — Spring Boot
- Authorization Server — Spring Boot
- Backend for Frontend — Spring Boot
- Frontend — Next.js

---

## Vagrantfile

Tein ensin seuraavat Vagrant valmistelut:

- latasin konelle Salt Project asentamiseen tarvittavan public keyn ja salt.sourcen GitHubista:

```
https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public
```

```
https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources
```

- Loin Vagrantfilen:

```
vagrant init bento/debian-12 --box-version 202502.21.0
```

Aloitin tällä:

```bash
Vagrant.configure("2") do |config|

  config.vm.box = "bento/debian-12"
  config.vm.box_version = "202502.21.0"

  config.vm.define "master" do |master|
      master.vm.hostname = "master"
  end

  config.vm.define "resource" do |resource|
      resource.vm.hostname = "resource"
  end

  config.vm.define "auth" do |auth|
      auth.vm.hostname = "auth"
  end

  config.vm.define "bff" do |bff|
      bff.vm.hostname = "bff"
  end

  config.vm.define "frontend" do |frontend|
      frontend.vm.hostname = "frontend"
  end

end
```

Lopullinen Vagrantfile. Käytin apuna Karvisen ohjetta [^2], Giangin tunnilla esittelemää toteutusta [^3], Paralellin dokumentaatiota [^4] networkingin kanssa ja Vagrantin dokumentaatiota synced folderin kanssa [^5].

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

  config.vm.define "resource" do |resource|
      resource.vm.hostname = "resource"
      resource.vm.network "private_network", ip: "192.168.51.5"
      resource.vm.synced_folder ".", "/vagrant_data"
      resource.vm.provision :shell, inline: $minion
  end

  config.vm.define "auth" do |auth|
      auth.vm.hostname = "auth"
      auth.vm.network "private_network", ip: "192.168.51.6"
      auth.vm.synced_folder ".", "/vagrant_data"
      auth.vm.provision :shell, inline: $minion
  end

  config.vm.define "bff" do |bff|
      bff.vm.hostname = "bff"
      bff.vm.network "private_network", ip: "192.168.51.7"
      bff.vm.synced_folder ".", "/vagrant_data"
      bff.vm.provision :shell, inline: $minion
  end

  config.vm.define "frontend" do |frontend|
      frontend.vm.hostname = "frontend"
      frontend.vm.network "private_network", ip: "192.168.51.8"
      frontend.vm.synced_folder ".", "/vagrant_data"
      frontend.vm.provision :shell, inline: $minion
  end

end
```

```
vagrant up --provider=parallels
```

Tämän jälkeen masterissa:

```
sudo salt-key -A
```

```bash
vagrant@master:~$ sudo salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
auth
bff
frontend
resource
Proceed? [n/Y] y
Key for minion auth accepted.
Key for minion bff accepted.
Key for minion frontend accepted.
Key for minion resource accepted.
```

---

## Java 21

Seuraavaksi kaikkiin minioneihin paitsi frontendiin pitäisi asentaa Java 21 ja Debianin package-managerista ei löydy sitä. 

```bash
vagrant@master:~$ apt-cache search openjdk-21-jdk
vagrant@master:~$ apt-cache search openjdk-17-jdk
default-jdk - Standard Java or Java compatible Development Kit
default-jdk-headless - Standard Java or Java compatible Development Kit (headless)
openjdk-17-jdk - OpenJDK Development Kit (JDK)
openjdk-17-jdk-headless - OpenJDK Development Kit (JDK) (headless)
```

```
sudo mkdir -p /srv/salt/jdk
```

```
sudoedit /srv/salt/jdk/init.sls
```

Aiempien asentelujeni peruteella menin Oraclen sivuille [^6] katsomaan miten saisin ladattua jdk-21 Debianiin. Koska käytän ARM konetta valittavaksi jäi tarpallon lataaminen. Latasin ensin jdk:n suoraan toiselle Debianin virutaalikoneelle ja kävin asentamisen sillä läpi. Macissa vaihtelen HOME_JAVA:n jdk:ta .zshr tiedostossa ja saman pystyi tekemään myös Debianin .bashrc tiedostossa, mutta sen ongelmaksi tulee varmastikkin se, että HOME_JAVA on user:n muuttuja ja koska salt ajaa kaiken sudo:na ei se ole ehkä toimiva ratkaisu. Kysyin tähän ChatGPT:ltä [^7] vinkkiä ja päädyin vinkin ja tämän Stack Overflow keskustelun [^8] perusteella laittamaan /etc/profile.d/java.sh kansioon konfiguroinnin JAVA_HOME:sta ja PATH:sta. En ehtinyt ihan kunnolla selvittää tuota profide.d hakemisto ja vaikka toisen hyvänoloisen keskustelun [^9], niin en ihan sisäistänyt sitä. Täytyy myöhemmin opiskella tarkemmin.     

Kun sourcessa on verkko-osoite tarvitsee se Saltin virheilmoituksen mukaan joko tarkistaa hashilla tai sitten skipata. Täytyy myöhemmin tutkia tätäkin, mutta nyt skippasin. 

Käytin taas Linoden ohjetta Salt:sta ja otin sieltä `makedirs` (tekee puuttuvat kansiot) ja `require` (ei suorita ennen kuin requiren on suoritettu) [^10]. 

`creates` cmd.run statessa aiheuttaa sen, että cmd.run ajetaan vain, jos tiedostoa ei ole olemassa [^11]


```yaml
/opt/jdk/jdk-21_linux-aarch64_bin.tar.gz:
  file.managed:
    - source: https://download.oracle.com/java/21/latest/jdk-21_linux-aarch64_bin.tar.gz
    - skip_verify: True
    - makedirs: true
      
unpack-jdk-tar:
  cmd.run:
    - name: tar -xzf /opt/jdk/jdk-21_linux-aarch64_bin.tar.gz -C /opt/jdk
    - creates: /opt/jdk/jdk-21.0.7
    - require:
        - file: /opt/jdk/jdk-21_linux-aarch64_bin.tar.gz
    
/etc/profile.d/java.sh:
  file.managed:
    - contents: |
        export JAVA_HOME=/opt/jdk/jdk-21.0.7
        export PATH=$JAVA_HOME/bin:$PATH
    - makedirs: true
```

```
sudo salt 'resource' state.apply jdk
```

```bash
vagrant@master:/srv/salt/jdk$ sudo salt 'resource' state.apply jdk
resource:
----------
          ID: /opt/jdk/jdk-21_linux-aarch64_bin.tar.gz
    Function: file.managed
      Result: True
     Comment: File /opt/jdk/jdk-21_linux-aarch64_bin.tar.gz updated
     Started: 17:42:04.733589
    Duration: 3733.008 ms
     Changes:
              ----------
              diff:
                  New file
              mode:
                  0644
----------
          ID: unpack-jdk-tar
    Function: cmd.run
        Name: tar -xzf /opt/jdk/jdk-21_linux-aarch64_bin.tar.gz -C /opt/jdk
      Result: True
     Comment: Command "tar -xzf /opt/jdk/jdk-21_linux-aarch64_bin.tar.gz -C /opt/jdk" run
     Started: 17:42:08.467162
    Duration: 2444.746 ms
     Changes:
              ----------
              pid:
                  2576
              retcode:
                  0
              stderr:
              stdout:
----------
          ID: /etc/profile.d/java.sh
    Function: file.managed
      Result: True
     Comment: File /etc/profile.d/java.sh is in the correct state
     Started: 17:42:10.912038
    Duration: 1.04 ms
     Changes:

Summary for resource
------------
Succeeded: 3 (changed=2)
Failed:    0
------------
Total states run:     3
Total run time:   6.179 s
vagrant@master:/srv/salt/jdk$ sudo salt 'resource' state.apply jdk
resource:
----------
          ID: /opt/jdk/jdk-21_linux-aarch64_bin.tar.gz
    Function: file.managed
      Result: True
     Comment: File /opt/jdk/jdk-21_linux-aarch64_bin.tar.gz is in the correct state
     Started: 17:42:18.072320
    Duration: 185.274 ms
     Changes:
----------
          ID: unpack-jdk-tar
    Function: cmd.run
        Name: tar -xzf /opt/jdk/jdk-21_linux-aarch64_bin.tar.gz -C /opt/jdk
      Result: True
     Comment: /opt/jdk/jdk-21.0.7 exists
     Started: 17:42:18.258009
    Duration: 410.44 ms
     Changes:
----------
          ID: /etc/profile.d/java.sh
    Function: file.managed
      Result: True
     Comment: File /etc/profile.d/java.sh is in the correct state
     Started: 17:42:18.668491
    Duration: 1.196 ms
     Changes:

Summary for resource
------------
Succeeded: 3
Failed:    0
------------
Total states run:     3
Total run time: 596.910 ms
```

Menin vielä varmuuden vuoksi resource-minioniin tarkistamaan:

```bash
vagrant@resource:~$ java --version
java 21.0.7 2025-04-15 LTS
Java(TM) SE Runtime Environment (build 21.0.7+8-LTS-245)
Java HotSpot(TM) 64-Bit Server VM (build 21.0.7+8-LTS-245, mixed mode, sharing)
vagrant@resource:~$ echo $JAVA_HOME
/opt/jdk/jdk-21.0.7
```

Ajoin staten myös auth- ja bff-minineille. 

---

## Postgres




---

## Spring Boot sovellukset

---

## Next.js sovellus

---

### Lähteet

[^1]: Tero Karvinen. Palvelinten Hallinta: https://terokarvinen.com/palvelinten-hallinta/ 

[^2]: Tero Karvinen. Salt Vagrant - automatically provision one master and two slaves: https://terokarvinen.com/2023/salt-vagrant/

[^3]: gianglex. H2 - Soitto kotiin: https://github.com/gianglex/Courses/blob/5978adafc55e712df670b3d976f1225698663abb/Palvelinten-Hallinta/h2-soitto-kotiin.md

[^4]: Parallels. Parallels + Vagrant - Private Networks: https://parallels.github.io/vagrant-parallels/docs/networking/private_network.html

[^5]: HashiCorp. Synced Folders - Basic Usage: https://developer.hashicorp.com/vagrant/docs/synced-folders/basic_usage

[^6]: Oracle. 3 Installation of the JDK on Linux Platforms: https://docs.oracle.com/en/java/javase/21/install/installation-jdk-linux-platforms.html#GUID-ADC9C14A-5F51-4C32-802C-9639A947317F

[^7]: OpenAI. ChatGPT: Version 1.2025.112, Model GPT‑4o mini.

[^8]: Stack Overflow: How can I found where I defined JAVA_HOME: https://stackoverflow.com/questions/43229474/how-can-i-found-where-i-defined-java-home

[^9]: Stack Exchange. What do the scripts in /etc/profile.d do?: https://unix.stackexchange.com/questions/64258/what-do-the-scripts-in-etc-profile-d-do

[^10]: Linode LLC. Configure Apache with Salt Stack: https://www.linode.com/docs/guides/configure-apache-with-salt-stack/

[^11]: Salt Project. SALT.STATES.CMD: https://docs.saltproject.io/en/3006/ref/states/all/salt.states.cmd.html