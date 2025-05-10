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

```
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
      master.vm.synced_folder ".", "/vagrant_data"
      master.vm.provision :shell, inline: $master
  end

  config.vm.define "minion001" do |minion001|
      minion001.vm.hostname = "minion001"
      minion001.vm.synced_folder ".", "/vagrant_data"
      minion001.vm.provision :shell, inline: $minion
  end

  config.vm.define "minion002" do |minion002|
      minion002.vm.hostname = "minion002"
      minion002.vm.synced_folder ".", "/vagrant_data"
      minion002.vm.provision :shell, inline: $minion
  end

  config.vm.define "minion003" do |minion003|
      minion003.vm.hostname = "minion003"
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


---

### Lähteet

[^1]: Tero Karvinen. Palvelinten Hallinta: https://terokarvinen.com/palvelinten-hallinta/ 
