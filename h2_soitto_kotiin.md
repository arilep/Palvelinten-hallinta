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

SSH-kirjautuminen `$ vagrant ssh koneen_nimi`

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

Vagrantin asennus osoitteesta: https://developer.hashicorp.com/vagrant/install

Käytössäni on Windows-kone, joten valitsin listalta sopivan. 2.4.3 on Vagrantin viimeisin versio.

![image](https://github.com/user-attachments/assets/f66010bc-1bcd-410a-8371-8cad48818477)

Asennus oli hyvin simppeli, Vagrant setup wizardin käynnistyttyä aloitusnäkymästä valitsin 'Next'.

Seuraavassa ikkunassa oli ohjelmiston lisenssisopimus tietoa, laitoin täpän kohtaan 'I accept the terms in the License Agreement' ja 'Next'.

Tämän jälkeen säilytin asennuksen oletussijainnin C-asemalla ja jatkoin 'Next'. Asennuksen päätteeksi käynnistin koneen uudelleen kuten asennusohjelmisto suositteli.

Koneen käynnistyttyä siirryin Windowsin komentoriville ja tarkastin, että viimeisin versio Vagrantista tuli asennettua.

![image](https://github.com/user-attachments/assets/60a65784-cbcc-4fa0-8735-21a624ddfb19)

Osioon käytetty aika: ~15min

## b) Linux Vagrant

Aloitin käynnistämällä PowerShell:n järjestelmänvalvojan oikeuksin ja loin uuden kansion 'PH1' samaan polkuun, mihin Vagrant asennettiin.

![image](https://github.com/user-attachments/assets/090f1161-95a3-4c26-8f5f-0cffba3bdcfc)

Siirryin polkuun program files\vagrant\PH1 ja ajoin komennon `vagrant init debian/bookworm64`.

Komennolla alustetaan hakemisto Vagrant ympäristöksi ja luodaan Vagrantfile -tiedosto, mikäli sellaista ei ole vielä olemassa. Tiedosto sisältää virtuaalikoneiden määritykset.

`vagrant up` -komento luo ja konfiguroi virtuaalikoneen / -koneet Vagrantfile -tiedoston määritysten mukaisesti.

Ajoin komennon, asennus osuus vei muutaman minuutin:

![image](https://github.com/user-attachments/assets/bc0e7b7c-2877-4d16-994b-e9a25258009d)

![image](https://github.com/user-attachments/assets/2ece3ad8-65e8-4ef2-a464-bde6e5f4abed)

Tämän jälkeen otin SSH-yhteyden virtuaalikoneeseen ja testasin toimintaa muutamalla peruskomennolla:

![image](https://github.com/user-attachments/assets/853e1275-ca20-489c-86ae-c025a69c7c14)

Osioon käytetty aika: ~25min

## c) Kaksin kaunihimpi

Aloitin luomalla uuden kansion projektille.

![image](https://github.com/user-attachments/assets/27ac17c0-c063-4bed-ba97-e6b9c10079d7)

Ensin loin tiedostopohjan, jota lähteä muokkaamaan `vagrant init debian/bookworm64`.

Avasin tiedoston notepadilla `notepad Vagrantfile`, tyhjensin sisällön ja lisäsin [ohjeiden mukaisesti](https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/#vagrantfile) uuden sisällön tiedostoon, mutta muokkasin config.vm.box = "debian/bullseye64" --> config.vm.box = "debian/bookworm64".

![image](https://github.com/user-attachments/assets/65aa7113-8716-4038-b2f9-c511b10f0524)

Tämän jälkeen koneet käyntiin `vagrant up`.

Asennusprosessi vei muutaman minuutin. Sitten testaamaan yhteyksiä.

ssh-yhteyden luominen koneeseen t001:

![image](https://github.com/user-attachments/assets/85a2c0f9-66dc-40d4-91d9-66a9298213d6)

Tarkoituksena on testata/osoittaa että verkkoyhteys / yhteys koneiden t001 ja t002 välillä toimii. Pingataan konetta t002 (192.168.88.102) ja googlen public DNS (8.8.8.8).

![image](https://github.com/user-attachments/assets/f302b3bf-3699-408a-8e67-b3e23bd435c9)

Sitten ssh-yhteys koneeseen t002 ja toistin samat testit:

![image](https://github.com/user-attachments/assets/5d5da1ba-9d04-4403-9b0f-a82bbf426f50)

Osioon käytetty aika: ~35min

## d) Herra-orja verkossa

Aloitin osion siirtymällä polkuun C:\Program Files\Vagrant\twohost, johon luotiin virtuaalikoneet t001 ja t002 edellisessä osiossa.

Otin ensin ssh-yhteyden koneeseen t002 `vagrant ssh t002` ja asensin saltin vaiheittain [Salt install guide](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html) -ohjeiden mukaisesti.

`curl` -komentoa ajettaessa sain virheilmoituksen "-bash: curl: command not found", eli curlia ei ollut asennettuna.

Asensin curlin `sudo apt-get install -y curl`, jonka jälkeen sain asennuksen suoritettua loppuun.

Koneesta t002 tulee slave-kone, joten asensin salt-minionin komennolla `sudo apt-get -y install salt-minion`.

Lopuksi vielä tarkistin että Salt:n asennus onnistui:

![image](https://github.com/user-attachments/assets/81f2c5a3-55c2-428d-8598-8bfea51eb8c6)

Toistin samat vaiheet koneella t001, johon asensin salt-minionin tilalta salt-masterin `sudo apt-get -y install salt-master`.

![image](https://github.com/user-attachments/assets/1fd6ea61-4111-4117-a7fb-d114ffa6210a)

Tämän jälkeen asensin micron `sudo apt-get install -y micro` ja kävin laittamassa slave -koneen t002 /etc/salt/minion -tiedostoon masterin osoitteen `sudo micro minion` polussa /etc/salt/minion.

Osoitteen kävin kaivamassa Vagrantfile -tiedostosta:

![image](https://github.com/user-attachments/assets/dba57180-5f0a-4159-9c80-1dff2428fb58)


Lisätyt tiedot:

![image](https://github.com/user-attachments/assets/c5f920de-ad73-4fa0-b80b-f186e61e1614)

Tämän jälkeen slave-avainten hyväksyntä masterilla.

Otin ssh-yhteyden koneeseen t001 ja ajoin komennon `sudo salt-key -A`, mutta sain kuitenkin seuraavanlaisen ilmoituksen:

![image](https://github.com/user-attachments/assets/07c8fb2e-aa67-4f46-9bd5-402f9c7f1cf8)

Kokeilin käynnistää virtuaalikoneet uudelleen. `vagrant halt` --> `vagrant up`

Tarkistin vielä koneiden tilat komennolla `vagrant status`. t002 kone ei jostain syystä suostunut käynnistymään.

![image](https://github.com/user-attachments/assets/37102dc6-fcbb-4cf5-a10f-d130cd4156fa)

Sain koneen kuitenkin ylös ajamalla erikseen komennon `vagrant up t002`

![image](https://github.com/user-attachments/assets/054cf54b-4839-4f49-9652-304be4f382df)

Tämän jälkeen t002 koneella vielä päivitys `sudo apt-get update` ja potkaisu: `sudo systemctl restart salt-minion.service`

Tämän jälkeen sain vihdoin komennon ajettua läpi:

![image](https://github.com/user-attachments/assets/ccc518c1-463d-4c4b-be09-522d397a6d05)

Sitten testasin, että master koneella onnistuu slave koneen "komennus":

![image](https://github.com/user-attachments/assets/325ebceb-8f5a-405c-9ce7-2e6f6aad1a92)

Osioon käytetty aika: ~40min

## e) Kokeile vähintään kahta tilaa verkon yli

Testasin micron asennusta ajamalla: `sudo salt '*' state.single pkg.installed micro`, micron olin asentanut jo aiemmassa osiossa.

![image](https://github.com/user-attachments/assets/4ce36f84-e4cb-4b95-a5fb-4d18ed844085)

Kommentti osiosta löytyykin maininta: "All specified packages are already installed" ja summarysta "Succeeded: 1".

Paketinhallinnan päivitys näytti myöskin menevän läpi:

![image](https://github.com/user-attachments/assets/23fb8feb-e3b4-4a69-b5d8-fb2708eb9d43)

Tree:n asennus:

![image](https://github.com/user-attachments/assets/1595dd67-943c-4cc8-9b17-c11137758de6)

Tree olikin jo näköjään asennettuna. Poistetaan ja asennetaan uudestaan:

![image](https://github.com/user-attachments/assets/bf3d7e56-0250-4154-9e91-392785922fb1)

Osioon käytetty aika: ~15min

### Lähteet

HashiCorp. s.a. Install Vagrant. Luettavissa: https://developer.hashicorp.com/vagrant/docs/installation.

HashiCorp. s.a. Vagrant Version. Luettavissa: https://developer.hashicorp.com/vagrant/docs/cli/version.

HashiCorp. s.a. Vagrant Init. Luettavissa: https://developer.hashicorp.com/vagrant/docs/cli/init.

HashiCorp. s.a. Vagrant Up. Luettavissa: https://developer.hashicorp.com/vagrant/docs/cli/up.

Karvinen, T. 26.3.2025. Palvelinten Hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/.

Karvinen, T. 28.3.2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file.

Karvinen, T. 4.11.2021. Two Machine Virtual Network With Debian 11 Bullseye and Vagrant. Luettavissa: https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/.

Karvinen, T. 28.3.2018. Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux. Luettavissa: https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/?fromSearch=salt%20quickstart%20salt%20stack%20master%20and%20slave%20on%20ubuntu%20linux.

VMware, Inc. s.a. Salt Install Guide: Linux (DEB). Luettavissa: https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html
