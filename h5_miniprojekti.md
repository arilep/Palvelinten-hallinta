### h5 Miniprojekti [Tehtävänanto](https://terokarvinen.com/palvelinten-hallinta/#h5-miniprojekti)

## Tehtävissä käytetty ympäristö

- Käyttöjärjestelmä: Windows 11 Home 64-bit
- Suoritin: AMD Ryzen 7 5800X 8-Core Processor
- Muisti: 32GB RAM
- Näytönohjain: NVIDIA GeForce RTX 3080

- Internet yhteys: DNA Netti 600 M
- Oracle VirtualBox 7.1.8
- Vagrant 2.4.3

## Miniprojekti info

Projektini ideana oli luoda moduuli 'all-in-one' -periaatteella. Halusin itselleni helposti ja nopeasti kasattavan työskentely-ympäristön, josta löytyvät useimmin käyttämäni työkalut/ohjelmat.

Sisältö ja idea lyhyesti:

Vagrantfile
- Pystyyn kaksi virtuaalikonetta: toiseen koneeseen salt-master, toiseen salt-minion
- Slave koneelle määritetään master-koneen IP polkuun /etc/salt/minion
- Forwarded_port -rivi Netdata -ohjelmaa varten

Moduuli
- Asentaa seuraavat:
  - Netdata
  - Micro
  - Git
  - Apache2
  - Palomuuri (ufw)
- Lisäksi käynnistä netdata service, apache2 service, reiät palomuuriin (portit 22, 80) ja ota palomuuri käyttöön.

## Toteutus: Vagrantfile

Pohjana käytin Tero Karvisen mallia, jonka löytää [täältä.](https://terokarvinen.com/2023/salt-vagrant/)

Vagrantfilen kasaamiseen löysin erinomaisia ohjeita [tästä linkistä.](https://developer.hashicorp.com/vagrant/docs/provisioning/shell)

Koska tekstiä oli paljon, hyödynsin ChatGPT:tä sisällön tarkistamiseen (kirjoitusvirheet) ja antamaan parannusehdotuksia promptilla "tarkista sisältö kirjoitusvirheiden varalta".

Olennaiset muutokset:

- config.vm.box = "debian/bullseye64" muutettu muotoon config.vm.box = "debian/bookworm64"
- kaksi konetta kolmen sijaan (koneita voi lisätä tarpeen mukaan)
- Saltin asennus vaatii Debian 12 Bookworm uuden pakettivaraston asentamista, nämä lisätty [Salt install guiden](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html) mukaan.
- Curlin asennus jotta Salt asennus voidaan suorittaa
- `echo "master: 192.168.60.101" | sudo tee -a /etc/salt/minion` komennolla lisätään /etc/salt/minion -tiedostoon suoraan Salt masterin IP-osoite
- `pcslave.vm.network "forwarded_port", guest: 19999, host: 19999` ja `pcmaster.vm.network "forwarded_port", guest: 19999, host: 29999` rivit lisätty, jotka Netdata -ohjelmisto vaatii.

Vagrantfile muokkausten jälkeen:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
# Copyright 2019-2021 Tero Karvinen http://TeroKarvinen.com

$pcslave = <<PCSLAVE
set -o verbose
sudo apt-get update
sudo apt-get -y install curl
mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get -y install salt-minion
echo "master: 192.168.60.101" | sudo tee -a /etc/salt/minion
sudo systemctl restart salt-minion
echo "Done - set up test environment - https://terokarvinen.com/search/?q=vagrant"
PCSLAVE

$pcmaster = <<PCMASTER
set -o verbose
sudo apt-get update
sudo apt-get -y install curl
mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources
sudo apt-get update
sudo apt-get -y install salt-master
sudo systemctl restart salt-master
echo "Done - set up test environment - https://terokarvinen.com/search/?q=vagrant"
PCMASTER

Vagrant.configure("2") do |config|
	config.vm.synced_folder ".", "/vagrant", disabled: true
	config.vm.synced_folder "shared/", "/home/vagrant/shared", create: true
	config.vm.box = "debian/bookworm64"

	config.vm.define "pcmaster" do |pcmaster|
		pcmaster.vm.provision :shell, inline: $pcmaster
		pcmaster.vm.hostname = "pcmaster"
		pcmaster.vm.network "private_network", ip: "192.168.60.101"
		pcmaster.vm.network "forwarded_port", guest: 19999, host: 29999
	end

	config.vm.define "pcslave", primary: true do |pcslave|
		pcslave.vm.provision :shell, inline: $pcslave
		pcslave.vm.hostname = "pcslave"
		pcslave.vm.network "private_network", ip: "192.168.60.102"
		pcslave.vm.network "forwarded_port", guest: 19999, host: 19999
	end
	
end
```
## Toteutus: Miniprojekti

Aloitin projektini luomalla uuden kansion Powershell:ssä 'miniprojekti' polkuun C:\Program Files\Vagrant komennolla `mkdir miniprojekti` ja siirtymällä polkuun `cd miniprojekti`

Tämän jälkeen alustus komennolla `vagrant init debian/bookworm64` ja korvasin Vagrantfile:n sisällön `notepad .\Vagrantfile`, jonka jälkeen asennukseen komennolla `vagrant up`

Asennus kesti noin ~2 minuuttia

Ajoin komennon `vagrant status` tarkistaakseni, että molemmat koneet olivat varmasti ylhäällä

![image](https://github.com/user-attachments/assets/ec47c1f5-dd81-4531-b0f2-243989962a5c)

sekä pcmaster että pcslave "running" tilassa, joten kaiken pitäisi olla ok.

Tämän jälkeen yhteys master koneeseen `vagrant ssh pcmaster`

Vagrantfile:ssa on valmiiksi määritettynä Salt masterin IP-osoite, joten avainten hyväksyntä sujui ilman, että sitä täytyi käydä erikseen lisäämässä.

Testasin samalla että pingaus pcslave koneeseen onnistuu.

![image](https://github.com/user-attachments/assets/8f6dfffb-131c-489d-a063-7f1c58517693)

Seuraavaksi loin uuden kansion ja init.sls -tiedoston moduulia varten.

Kansion luonti ja siirtyminen polkuun

`sudo mkdir -p /srv/salt/munrojut && cd /srv/salt/munrojut` 

init.sls -tiedoston luominen

`sudoedit init.sls`

init.sls tiedostoon lisätty sisältö:

```
# install netdata
install_netdata:
  pkg.installed:
    - name: netdata

# ensures that the Netdata service is running
netdata_service:
  service.running:
    - name: netdata
    - enable: True

# install micro
install_micro:
  pkg.installed:
    - name: micro

# install git
install_git:
  pkg.installed:
    - name: git

# install apache2
install_apache2:
  pkg.installed:
    - name: apache2

# ensures that the apache2 service is running
apache2_service:
  service.running:
    - name: apache2
    - enable: True

# install firewall
install_ufw:
  pkg.installed:
    - name: ufw

# allow ssh
allow_ssh:
  cmd.run:
    - name: ufw allow 22/tcp

# allow http
allow_http:
  cmd.run:
    - name: ufw allow 80/tcp

# enable firewall
enable_ufw:
  cmd.run:
    - name: ufw enable

```

Tallennus ctrl+S ja editorista poistuminen ctrl+X

### Idempotenttiudesta!

Moduuli sisältää kolme cmd.run -komentoa, joiden osalta moduuli EI ole idempotentti, muilta osin kyllä.

Komentoja en halunnut jättää pois, mutta en löytänyt toista tapaa toteuttaa näitä siten, että moduuli olisi idempotentti.

### Moduulin ajo

Seuraavaksi ajoin moduulin komennolla `sudo salt '*' state.apply munrojut`

Ensimmäisen ajon summary:

![image](https://github.com/user-attachments/assets/5a82d644-4750-44ea-96d8-fd292f8bc94f)

changed=8, sillä netdata_service ja apache2_servicen osalta Comment -osiossa mainitaan, että "The service X is already running"

seuraavalla ajolla päädytään changed=3, johtuen siitä, että moduuli ei ole täysin idempotentti:

![image](https://github.com/user-attachments/assets/44a0a4aa-f6bd-4d84-b8bb-a382ad43ddbd)

### Netdata korjaus

Netdatan konfiguraatio tiedostoon liittyen löysin tietoa muun muassa [täältä](https://dietpi.com/forum/t/netdata-after-fresh-install-config-cannot-load-cloud-config/18374)

Vaikka Netdata -asennus onnistuu, täytyy pcslave koneella käydä tekemässä vielä pienet korjaukset, johon en löytänyt muunlaista ratkaisua:

Yhteys slave-koneeseen komennolla `vagrant ssh pcslave`

jonka jälkeen netdatan konffi-tiedostoon komennolla `sudoedit /etc/netdata/netdata.conf`

![image](https://github.com/user-attachments/assets/961ac10b-7a6a-4a27-90f2-09f53ee64d5c)

"bind socket to IP = 127.0.0.1" -rivin IP-osoitteen muokkasin vastaamaan slave-koneen IP:tä 192.168.60.102

Lisäksi palomuuriin täytyy tehdä vielä reikä porttia 19999 varten `sudo ufw allow 19999`. Tämän olisi voinut tietty lisätä suoraan moduuliin, täytynee fiksata myöhemmin.

Sitten vielä demonin potkaisu komennolla `sudo systemctl restart netdata`

Netdatan näkymä selaimessa (http://192.168.60.102:19999/):

![image](https://github.com/user-attachments/assets/5086c9c1-758c-4b4a-8bd9-b9c2bee26062)

### Testaukset ja mietteet

Tarkastin vielä, että micro, git, apache2 ja palomuuri ovat asennettuina ja toiminnassa:

![image](https://github.com/user-attachments/assets/1bbaaa00-05ff-4e6b-95bc-c0e4b64d2626)

Netdatan osalta toimivuus tulikin testattua jo aiemmassa osiossa.

Kokonaisuus on helposti laajennettavissa ja ohjelmistoja / työkaluja valmiiseen projektiin on helppo lisätä jälkikäteen lisää. Samoten myös koneiden määrää mikäli kokee tarpeelliseksi.

Moduuliin on aikomuksena lisätä myöhemmin myös .bashrc aliaksia helpottamaan/nopeuttamaan omaa käyttöä, mutta toistaiseksi tähän ei vielä aika riittänyt.

### Käytetty aika

- Projektin suunnittelu

Tiedonhaku ja suunnittelu ~1-2 tuntia

- Vagrantfile (työläin)

Tiedonhaku, testaus ja kasaaminen ~3-4 tuntia

- Moduuli

Tiedonhaku, testaus ja kasaaminen ~3 tuntia

### Lähteet

PhoenixAp. 23.1.2023. What is the .bashrc Fine in Linux? Luettavissa: https://phoenixnap.com/kb/bashrc#:~:text=bashrc%3F-,.,shell%20experience%20and%20automate%20tasks.

DigitalOcean. 25.8.2025. UFW Essentials: Common Firewall Rules and Commands. Luettavissa: https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands

DietPi. Netdata after fresh install: CONFIG: cannot load cloud config. Luettavissa: https://dietpi.com/forum/t/netdata-after-fresh-install-config-cannot-load-cloud-config/18374.

Hashicorp. s.a. Shell Provisioner. Luettavissa: https://developer.hashicorp.com/vagrant/docs/provisioning/shell.

Karvinen, T. 28.3.2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/.

Karvinen, T. 26.3.2025. Palvelinten Hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/.

NetData. s.a. Monitor your entire infrastructure in high-resolution and in real time. Luettavissa: https://www.netdata.cloud/.

NetData. s.a. Securing Netdata Agents. Luettavissa: https://learn.netdata.cloud/docs/netdata-agent/configuration/securing-agents/.

Salt Project. s.a. Salt install guide. Luettavissa: https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html.
