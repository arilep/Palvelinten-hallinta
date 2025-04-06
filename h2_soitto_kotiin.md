### h2 Soitto kotiin [Tehtävänanto](https://terokarvinen.com/palvelinten-hallinta/#h2-soitto-kotiin)

## Tehtävissä käytetty ympäristö

### PC

- Käyttöjärjestelmä: Windows 11 Home 64-bit
- Suoritin: AMD Ryzen 7 5800X 8-Core Processor
- Muisti: 32GB RAM
- Näytönohjain: NVIDIA GeForce RTX 3080
- Virtualisointiohjelmisto: Oracle VM VirtualBox 7.0

- Internet yhteys: DNA Netti 600 M

### Virtuaalikone

- ISO Image: Debian live 12.9.0 amd 64 xfce
- Version: Debian 12 Bookworm (64-bit)
- Base Memory: 4000 MB
- Processors: 1
- Virtual HD: 60 GB
- Video Memory: 128 MB

## x) Tiivistelmät

### Karvinen 2021: Two Machine Virtual Network With Debian 11 Bullseye and Vagrant

- Nykyinen Debian stable = 12-Bookworm (11-bullseye vanhentunut)
- Vagrantin avulla VirtualBox -koneiden automaattinen määritys.
- Automatisoitu SSH-kirjautuminen.
- Graafiselle käyttöliittymälle (Grpahical User Interface; GUI) ei tarvetta.

Debian ja Ubuntu järjestelmille asennus:

```
$ sudo apt-get update
$ sudo apt-get install vagrant virtualbox
```
Vagrantilla myös MAC ja Windows tuki, asennus onnistuu [täältä](https://developer.hashicorp.com/vagrant/install)

SSH-kirjautuminen `$ vagrant ssh t001`

Virtuaalikoneen poisto `$ vagrant destroy` (Huom. poistaa SEKÄ koneen, ETTÄ tallennetut tiedostot)

Virtuaalikoneen käynnistys `$ vagrant up`

### Karvinen 2018: Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux

#### Tärkeimmät Salt -komennot

- Master asennus
```
master$ sudo apt-get update
master$ sudo apt-get -y install salt-master
master$ hostname -I
10.0.0.88
```

- Slave asennus
```
slave$ sudo apt-get update
slave$ sudo apt-get -y install salt-minion
```

- Hyväksy slave-avain master-koneella
```
master$ sudo salt-key -A
Unaccepted Keys:
imaminion
Proceed? [n/Y]
Key for minion imaminion accepted.
```

- Testaus
```
master$ sudo salt '*' cmd.run 'whoami'
user:
 root
```
### Karvinen 2023: Salt Vagrant - automatically provision one master and two slaves

#### Infra koodina

```
$ sudo mkdir -p /srv/salt/hello # Salt:n tilatiedostojen (state) kansio
$ sudoedit /srv/salt/hello/init.sls # luodaan .sls = Salt State File
```
```
# Tiedoston /tmp/infra-as-code tulisi olla olemassa
$ cat /srv/salt/hello/init.sls
/tmp/infra-as-code:
  file.managed
```
```
# '*' - aja komento kaikilla slave-koneilla
$ sudo salt '*' state.apply hello
```

#### Top.sls

Top -tiedosto määrittää mitä tiloja (states) ajetaan milläkin slave-koneilla
```
$ sudo salt '*' state.apply hello^C
$ sudoedit /srv/salt/top.sls
# Top tiedoston sisältö
$ cat /srv/salt/top.sls
base:
  '*':
    - hello
```
Moduuleja (state.apply:ssa) ei tarvitse enää nimetä:
```
$ sudo salt '*' state.apply
```

## a) Hello Vagrant!

Vagrantin asennus osoitteesta https://developer.hashicorp.com/vagrant/install

![image](https://github.com/user-attachments/assets/f66010bc-1bcd-410a-8371-8cad48818477)


## b) Linux Vagrant
## c) Kaksin kaunihimpi
## d) Herra-orja verkossa
## e) Kokeile vähintään kahta tilaa verkon yli

### Lähteet

