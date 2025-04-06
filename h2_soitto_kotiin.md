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

Käynnistin koneen uudelleen asennuksen päätteeksi.

Tämän jälkeen siirryin Windowsin komentoriville ja tarkastin, että viimeisin versio Vagrantista tuli asennettua.

![image](https://github.com/user-attachments/assets/60a65784-cbcc-4fa0-8735-21a624ddfb19)

Osioon käytetty aika: ~15min

## b) Linux Vagrant

Aloitin käynnistämällä PowerShell:n järjestelmänvalvojan oikeuksin ja loin uuden kansion.

![image](https://github.com/user-attachments/assets/090f1161-95a3-4c26-8f5f-0cffba3bdcfc)

Siirryin polkuun program files\vagrant\PH1 ja ajoin komennon `vagrant init debian/bookworm64` ja sen jälkeen `vagrant up`.

![image](https://github.com/user-attachments/assets/bc0e7b7c-2877-4d16-994b-e9a25258009d)


![image](https://github.com/user-attachments/assets/2ece3ad8-65e8-4ef2-a464-bde6e5f4abed)

Tämän jälkeen otin SSH-yhteyden virtuaalikoneeseen.

![image](https://github.com/user-attachments/assets/853e1275-ca20-489c-86ae-c025a69c7c14)

Osioon käytetty aika: ~20min

## c) Kaksin kaunihimpi

Aloitin luomalla uuden kansion projektille.

![image](https://github.com/user-attachments/assets/27ac17c0-c063-4bed-ba97-e6b9c10079d7)

Ensin loin tiedostopohjan, jota lähteä muokkaamaan `vagrant init debian/bookworm64`.

Avasin tiedoston notepadilla `notepad Vagrantfile`, tyhjensin sisällön ja lisäsin [ohjeiden mukaisesti](https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/#vagrantfile) uuden sisällön tiedostoon, mutta muokkasin config.vm.box = "debian/bullseye64" --> config.vm.box = "debian/bookworm64".

![image](https://github.com/user-attachments/assets/65aa7113-8716-4038-b2f9-c511b10f0524)

Tämän jälkeen koneet käyntiin `vagraant up`.

Asennusprosessi vei muutaman minuutin. Sitten testaamaan yhteyksiä.

ssh-yhteyden luominen koneeseen t001:

![image](https://github.com/user-attachments/assets/85a2c0f9-66dc-40d4-91d9-66a9298213d6)

pingataan konetta t002 (192.168.88.102) ja googlen public DNS (8.8.8.8)

![image](https://github.com/user-attachments/assets/f302b3bf-3699-408a-8e67-b3e23bd435c9)

Sitten ssh-yhteys koneeseen t002 ja toistin samat testit:

![image](https://github.com/user-attachments/assets/5d5da1ba-9d04-4403-9b0f-a82bbf426f50)

Osioon käytetty aika: ~25min

## d) Herra-orja verkossa
## e) Kokeile vähintään kahta tilaa verkon yli

### Lähteet

Karvinen, T. 4.11.2021. Two Machine Virtual Network With Debian 11 Bullseye and Vagrant. Luettavissa: https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/.

Karvinen, T. 28.3.2018. Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux. Luettavissa: https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/?fromSearch=salt%20quickstart%20salt%20stack%20master%20and%20slave%20on%20ubuntu%20linux.

Karvinen, T. 28.3.2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file.

HashiCorp. s.a. Install Vagrant. Luettavissa: https://developer.hashicorp.com/vagrant/docs/installation.

HashiCorp. s.a. Vagrant Version. Luettavissa: https://developer.hashicorp.com/vagrant/docs/cli/version.






