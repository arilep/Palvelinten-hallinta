### h1 Viisikko [Tehtävänanto](https://terokarvinen.com/palvelinten-hallinta/#h1-viisikko)

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

### Karvinen 2023: Run Salt Command Locally

- Salt Slave asennus `sudo apt-get update` `sudo apt-get -y install salt-minion`
- Tärkeät komennot
    - pkg.installed / pkg.removed
    - file.managed / file.absent
    - service.running / service.dead
    - user.present / user.absent
    - cmd.run
    - sudo salt-call --local sys.state_doc

### Karvinen 2018: Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux

- Master
    - sudo apt-get update
    - sudo apt-get -y install salt-master
    - huomaa tulimuuri 4505/tcp ja 4506/tcp
- Slave
    - sudo apt-get update
    - sudo apt-get -y install salt-minion
- Accept Slave Key on Master
    - sudo salt-key -A
- Testaus
    - sudo salt '*' cmd.run 'whoami'

### Karvinen 2006: Raportin kirjoittaminen

- Toistettava
    - Samassa ympäristössä tulisi päätyä samaan lopputulokseen raportin vaiheita noudattaen
    - Ympäristön (raudan) raportointi!
- Täsmällinen
    - Komennot
    - Kellonajat
    - Vikatilanteet
    - Raportti menneessä aikamuodossa
- Helppolukuinen
    - Väliotsikot
    - Huolellinen kirjoitusasu
- Lähdeviittaukset
- Vakiotekstit
    - Lisenssit (GNU versio 2 tai uudempi)
- Mokat (rangaistavat vilpit)
    - Sepittäminen
    - Tekstin / kuvien plagiointi
 
### VMWare Inc: Salt Install Guide: Linux (DEB)

- Neljä vaihetta
    - Keyrings kansion luonti
    - Julkisen avaimen lataus
    - apt repo konfigurointi
    - Salt-minion asennus

## b) Salt-minion asennus

(Osio aloitettu: 31.3.2025 klo 22.05)

Aloitin seuraamalla VMware, Inc. Salt Project -sivuston ohjeita [linkki](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html)

Ajoin ensin päivitykset: `sudo apt-get update`

Tämän jälkeen keyrings kansion luonti: `mkdir -p /etc/apt/keyrings`

Julkisen avaimen lataus: `curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp` 

Salt:n apt repo konfigurointia varten: `curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources` 

Ja tämän jälkeen sivuston ohjeiden mukaan paketinhallinnan päivitykset: `sudo apt update`

Itse salt-minion asennus: `sudo apt-get -y install salt-minion`

Lopuksi tarkistin että asennus onnistui:

![image](https://github.com/user-attachments/assets/b86030e7-e741-4ef5-aff4-2103dbef200f)


(Osio valmis: 31.3.2025 klo 22:18)

## c) Saltin tilafunktiot (pkg, file, service, user, cmd)

(Osio aloitettu: 31.3.2025 klo 22:19)

### Pkg

pkg.installed - komento viittaa tilaan, jossa ohjelma (tässä tapauksessa tree) olisi jo asennettuna.

pkg.removed - komento taasen viittaa tilaan, jossa ohjelma olisi poistettu.

Ajoin komennon:
`sudo salt-call --local -l info state.single pkg.installed tree`

![image](https://github.com/user-attachments/assets/e56189ce-7f60-40e0-89d3-a151f3e4e164)

Summary osion 'changed=1' kertoo, että tarvittavat muutokset (ohjelman asennus) on tehty. Seuraavalla ajokerralla tämän tiedon pitäisi jäädä pois.

pkg.removed 

Ajoin komennon:
`sudo salt-call --local -l info state.single pkg.removed tree`

![image](https://github.com/user-attachments/assets/bee4bced-23ff-4ba1-a972-d424a873548f)

Jälleen summary osion 'changed=1' kertoo, että muutokset on tehty.

### File

file.managed - komento pyrkii tilanteeseen, jossa tiedosto on jo olemassa.

file.absent - komento vastoin pyrkii tilanteeseen, jossa tiedostoa ei ole.

/tmp -polkuun 'helloap' tiedosto:
`sudo salt-call --local -l info state.single file.managed /tmp/helloap`

/tmp -polkuun tiedosto 'moiap' joka sisältää tekstin (contents) "foo":

`sudo salt-call --local -l info state.single file.managed /tmp/moiap contents="foo"`

![image](https://github.com/user-attachments/assets/4a61e54c-8199-4678-9d6f-625a9e895fc5)

/tmp -polun 'helloap' -tiedoston tulisi olla poistettu:
`sudo salt-call --local -l info state.single file.absent /tmp/helloap`

![image](https://github.com/user-attachments/assets/2d3539a9-f3b5-435c-81ae-011cd60c7894)

### Service

service.running - esimerkissä apache2 tulisi olla asennettuna ja toiminnassa.

service.dead - esimerkissä apache2 ei toiminnassa.

`sudo salt-call --local -l info state.single service.running apache2 enable=True`

![image](https://github.com/user-attachments/assets/1bf9f749-5ed3-4d76-a188-4cd1d121405e)

Aiempi komento tuottaa herjan (failed: 1), sillä apache2 ei ole koneessa asennettuna.

Asensin apachen virtuaalikoneelle:
`sudo apt-get install apache2`

Jonka jälkeen uusi ajo:
`sudo salt-call --local -l info state.single service.running apache2 enable=True`

![image](https://github.com/user-attachments/assets/b62518ca-7183-4955-9f1b-dae04981f253)

Huomaa Succeeded: 1.

Tämän jälkeen service.dead, jolloin tulisi ilmetä jälleen changed = 1 -merkintä:
`sudo salt-call --local -l info state.single service.dead apache2 enable=False`

![image](https://github.com/user-attachments/assets/f711eb74-9a1e-408a-8738-e6927697a1b7)


### User

user.present - komennolla pyritään tilanteeseen, jossa käyttäjä 'testikayttaja' on olemassa.

user.absent - käyttäjää 'testikayttaja' ei ole.

present -komento:
`sudo salt-call --local -l info state.single user.present testikayttaja`

![image](https://github.com/user-attachments/assets/7fd45442-1ef2-4cef-9cf8-8da1e738df3d)

Huomaa: New user testikayttaja created

absent -komento:
`sudo salt-call --local -l info state.single user.absent testikayttaja`

![image](https://github.com/user-attachments/assets/c465f5bc-455c-4973-b3fb-4bb8bb1138be)

--> käyttäjä sekä group 'testikayttaja' poistettu.

### Cmd

cmd.run - komennon tarkoitus ajaa komento 'touch tmp foo' ja tuloksena tulisi olla luotu tiedosto (creates="/tmp/foo").

Ajoin komennon:
`sudo salt-call --local -l info state.single cmd.run 'touch /tmp/foo' creates="/tmp/foo"`

![image](https://github.com/user-attachments/assets/39ab8f4b-e3fb-4bf2-b537-abd275ee04b8)


(Osio valmis: 31.3.2025 klo 22.36)

## d) Idempotentti

(Osio aloitettu: 31.3.2025 klo 22:37)

Idempotentilla tarkoitetaan tilaa, jossa ajamalla saman komennon n-kertaa, tulisi lopputuloksen olla sama kuin ensimmäisen suorituksen jälkeen. Lopputuloksena tulisi olla Succeeded=1 ja ainoastaan ensimmäisella ajokerralla changed=1.

Asennetaan micro -komennolla: `sudo salt-call --local -l info state.single pkg.installed micro`

Ensimmäinen ajo:

![image](https://github.com/user-attachments/assets/96b4a581-9813-4728-8540-c5af040fe802)

Toinen ajo:

![image](https://github.com/user-attachments/assets/88aca39a-46da-42e6-be29-18a6b0b4ac3a)

Viides ajo:

![image](https://github.com/user-attachments/assets/1f8bccbb-6ac8-4591-95e2-7fa264933e13)


(Osio valmis: 31.3.2025 klo 22.49)

### Lähteet

Karvinen, T. 26.3.2025. Palvelinten Hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/.

Karvinen, T. 28.10.2021. Run Salt Command Locally. Luettavissa: https://terokarvinen.com/2021/salt-run-command-locally/.

Karvinen, T. 28.3.2018. Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux. Luettavissa: https://terokarvinen.com/2018/03/28/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/.

Karvinen, T. 4.6.2006. Raportin kirjoittaminen. Luettavissa: https://terokarvinen.com/2006/06/04/raportin-kirjoittaminen-4/.

VMware, Inc. s.a. Salt Install Guide: Linux (DEB). Luettavissa: https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html

VMware,Inc. s.a. Verify a Salt install. Luettavissa: https://docs.saltproject.io/salt/install-guide/en/latest/topics/verify-install.html#verify-install.
