### h4 Pkg-file-service [Tehtävänanto](https://terokarvinen.com/palvelinten-hallinta/#h4-pkg-file-service)

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

### Karvinen 2018: Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port

- Hallitse suurta määrää demoneja configuration management system:llä
- Package-file-service -malli:
    - Asenna ohjelmisto
    - Korvaa konfiguraatio tiedosto
    - Käynnistä demoni uudelleen, jotta uusi konfiguraatio astuu voimaan



## a) Apache easy mode

Note: käytin tämän viikon harjoituksissa aiemman [viikkotehtävän](https://github.com/arilep/Palvelinten-hallinta/blob/main/h2_soitto_kotiin.md) c) osiossa luotuja koneita t001 (master) ja t002 (slave).

Aloitin käynnistämällä virtuaalikoneet `vagrant up` ja tarkistamalla virtuaalikoneiden tilan `vagrant status`

t002 ei käynnistynyt, joten erikseen potkaisu `vagrant up t002` jolloin molemmat koneet olivat ylhäällä.

Sitten otin yhteyden t001 koneeseen `vagrant ssh t001` ja ajoin vielä päivitykset `sudo apt-get update`

"Taikurin hihat tyhjät", osoitin ettei apachea oltu vielä asennettu:

![image](https://github.com/user-attachments/assets/d34383f1-6305-489c-a073-c4b03668dcf1)

Todennettu --> Service could not be found.

Asensin apachen komennolla `sudp apt-get install apache2 -y`, jonka jälkeen ajoin uudelleen `sudo systemctl status apache2` tarkistaakseni tilan:

![image](https://github.com/user-attachments/assets/ead2b55e-c6fc-42f0-91e8-8bc1cae64b41)

Active: active (running), eli palvelu on ylhäällä.

Tämän jälkeen korvasin testisivun, joka löytyy polusta /var/www/html/index.html:

![image](https://github.com/user-attachments/assets/dbacfd89-1e6c-4b44-8476-9540496d494f)

Tämän jälkeen demonin potkaisu `sudo systemctl restart apache2` ja sitten testaus `curl http://localhost` (curl asennettu aikaisemmin)

![image](https://github.com/user-attachments/assets/b8aff5da-5cf5-4881-b5d5-9794b723cc31)

Sitten sls-tiedoston ääreen, loin uuden kansion polkuun /srv/salt

![image](https://github.com/user-attachments/assets/a63c4535-afdc-443a-8899-105338c49288)

käytin microa tekstieditorina `export EDITOR=micro` ja sitten loin tiedoston `sudoedit init.sls`

sls-tiedostoon lisäsin seuraavan pätkän:

![image](https://github.com/user-attachments/assets/0b31b9f6-01f3-4000-a5d1-3ae30b9fcb66)

Rivit 1-2 Apachen asennus, 4-6 index.html -tiedoston korvaaminen, 8-11 käynnistää apache-palvelun jollei se ole jo käynnissä

Testataan yhteys t002 -koneeseen ja katsotaan apache2 status:

![image](https://github.com/user-attachments/assets/14dc2a12-2a52-4868-9c40-bfee1f544377)

Jälkimmäinen palautti 'false' = palvelu ei ole käynnissä, apachea ei ole asennettu

Sitten orja töihin

`sudo salt '*' state.apply apache2`



![image](https://github.com/user-attachments/assets/3f2e4047-feae-4fbf-8ce9-4af425162fb8)

Apache2 asennus onnistui

![image](https://github.com/user-attachments/assets/4a325fe3-dfd5-488c-85e9-d7a37de60075)

Index.html korvattu

![image](https://github.com/user-attachments/assets/77486131-cc86-4b47-af69-9cd900ce7193)

Palvelu on jo päällä

![image](https://github.com/user-attachments/assets/f2e2ca08-9647-4667-88ed-38afe3264b4d)

Summaryssä changed = 2 koska service.running Result: True

Toinen ajo testin vuoksi:

![image](https://github.com/user-attachments/assets/88d968d2-1ed2-40c8-a720-73943c6142e9)

= Idempotentti, ei muutoksia

Tarkistukset vielä koneella t002 `vagrant ssh t002`

Testisivu korvattu:

![image](https://github.com/user-attachments/assets/9079a228-6e59-4674-9478-6c8a5bdb2550)

Apache2 status:

![image](https://github.com/user-attachments/assets/f9e3ad03-ba97-408e-833d-694e185337eb)

curl localhost:

![image](https://github.com/user-attachments/assets/8e404c69-a11f-4a81-859b-f360fa8e1903)

Osioon käytetty aika ~1,5h

## b) SSHouto

Aloitin tarkistamalla tilan `sudo systemctl status sshd`

![image](https://github.com/user-attachments/assets/bbb9b5ba-5b29-4b25-bc46-1e9a35bcd1a7)

Active: active (running) = palvelu ylhäällä ja Server listening on :: port 22 = kuuntelee porttia 22.

Lisäsin konfiguraatio tiedostoon (polussa /etc/ssh/sshd_config) portit 22 ja 1234:

![image](https://github.com/user-attachments/assets/9a00a138-7f67-4d91-9792-351856045cff)

![image](https://github.com/user-attachments/assets/ed3e101d-5a77-4642-ac7d-f23f79922898)

Ensin potkaisu `sudo systemctl restart sshd` ja sitten uudelleen `sudo systemctl status sshd`

![image](https://github.com/user-attachments/assets/25ac4dfd-17b1-4165-ab4a-3020fa72a9b1)

Server listening on :: port 22 & Server listening on :: port 1234, eli näyttää toimivan

testaus portti kerrallaan:

![image](https://github.com/user-attachments/assets/12d7f04a-ebb8-4f19-880d-d6c91601d532)

### Automaatio osuuteen

Uusi kansio moduulille ja sshd_config -tiedoston kopiointi polkuun:

![image](https://github.com/user-attachments/assets/492f20aa-1d38-45ff-bee7-f2a8dae998a8)

init.sls tiedosto moduulille `export EDITOR=micro` ja `sudoedit /srv/salt/sshmodule/init.sls` Tero Karvisen [ohjeiden](https://terokarvinen.com/2018/04/03/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=karvinen%20salt%20ssh) mukaan:

![image](https://github.com/user-attachments/assets/48e7dc11-4f5c-4cb2-86e6-a87d06f7897d)

Sitten orja töihin `sudo salt '*' state.apply sshmodule`

![image](https://github.com/user-attachments/assets/79a86a9b-45aa-4ed5-a134-45ddca370831)

openssh-server asennus onnistui, sshd_config tiedosto päivitetty ja palvelu ylhäällä.

koneella t002 vielä tarkistukset:

![image](https://github.com/user-attachments/assets/c4ebdbdc-056f-4b12-b4dd-50e623f554d8)

Osioon käytetty aika ~1,5h

### Lähteet

Karvinen, T. 26.3.2025. Palvelinten Hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/.

Karvinen, T. 3.4.2018. Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port. Luettavissa: https://terokarvinen.com/2018/04/03/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=karvinen%20salt%20ssh.

Karvinen, T. 10.4.2018. Name Based Virtual Hosts on Apache - Multiple Websites to Single IP Address. Luettavissa: https://terokarvinen.com/2018/04/10/name-based-virtual-hosts-on-apache-multiple-websites-to-single-ip-address/.

Salt Project. s.a. How do I use Salt states? Luettavissa: https://docs.saltproject.io/en/3006/topics/tutorials/starting_states.html.

Suse. s.a. Custom Salt States. Luettavissa: https://documentation.suse.com/suma/4.3/en/suse-manager/specialized-guides/salt/salt-custom-states.html.

