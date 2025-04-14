### h3 Infraa koodina [Tehtävänanto](https://terokarvinen.com/palvelinten-hallinta/#h3-infraa-koodina)

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

### Karvinen 2014: Hello Salt Infra-as-Code

- /srv/salt/ hakemisto jaettu kaikille slave-koneille.
- /srv/salt/example/ polusta löytyisi siis kaikki tarvittava moduulia varten (salt-koodi, tiedostot ja pohjat (=template)).
- aloituspisteenä toimii init.sls kyseisessä polussa.

### Salt contributors: Salt overview (Rules of YAML, YAML simple structure, Lists and dictionaries - YAML block structures)

- Oletusrenderöijä Saltissa useimmiten YAML.
- YAML on merkintäkieli, joka kääntää YAML-tietorakenteen Python-tietorakenteeksi Saltia varten.
- YAML koostuu kolmesta peruselementistä:
    - Skalaarit (Scalars)
    - Listat (Lists)
    - Sanakirjat (Dictionaries)
- YAML jäsennelty lohkorakenteisiin (Blocks)
- Sisennys kontekstin määrityksessä: ominaisuudet ja listat sisennetään yhdellä tai useammalla välilyönnillä. Kaksi välilyöntiä on yleinen käytäntö.
- Lista tai sanakirjalohkon jono: aloitetaan '-' ja välilyönnillä.

## a) Hei infrakoodi!

Note: käytin tämän viikon harjoituksissa edellisen [viikkotehtävän](https://github.com/arilep/Palvelinten-hallinta/blob/main/h2_soitto_kotiin.md) c) osiossa luotuja koneita t001 (master) ja t002 (slave).

Aloitin työskentelyn ottamalla ssh-yhteyden master koneeseen t001 `vagrant ssh t001` ja sen jälkeen ajamalla päivitykset `sudo apt-get update`.

Tämän jälkeen Micron asennus `sudo apt-get -y install micro` seuraavia vaiheita varten. `export EDITOR=micro` -komennolla Micro oletustekstieditoriksi tälle istunnolle.

Seuraavaksi loin uuden kansion moduulia varten:

![image](https://github.com/user-attachments/assets/9dd530ce-a64f-4142-8821-16fdb48bf13e)

Kyseiseen polkuun init.sls tiedosto komennolla `sudoedit init.sls` ja lisätty sisältö:

![image](https://github.com/user-attachments/assets/e71e8384-a464-4a76-b642-159838569362)

file.managed -komennolla pyritään tilanteeseen jossa tiedosto on jo olemassa. Tässä tapauksessa tiedosto 'helloap' polussa /tmp/helloap

![image](https://github.com/user-attachments/assets/5afb1b15-bf82-4f6c-b5bb-b5cf841e3cf5)

Tarkistin vielä, että kansio todella löytyy kyseisestä polusta:

![image](https://github.com/user-attachments/assets/80e305f5-d636-4a12-8845-3dd9e41d25f1)

Osioon käytetty aika: ~30min

## b) sls-tiedosto verkon yli orjalla

Ajoin t001 koneella komennon `sudo salt '*' state.apply hello`

![image](https://github.com/user-attachments/assets/9cbb6fc0-a9cf-492b-913d-254d8ed1a23d)

Herjasta huomio kiinnittyi kohtaan "The master is not responding."

Tarkistin palvelun tilan:

![image](https://github.com/user-attachments/assets/742bb06e-ca46-440a-935d-b85edc8337a9)

"Active: failed" kertoo, ettei salt-master ole käynnissä. Ajoin komennon `sudo systemctl restart salt-master.service` jotta se lähtee pyörimään.

![image](https://github.com/user-attachments/assets/574d5dc9-50a2-4e31-b9f8-f318ce8bde4c)

Ilmoituksen perusteella lähti toimimaan, seuraavaksi tarkistamaan. Siirryin koneelle t002 `vagrant ssh t002` tarkistamaan, että hakemisto löytyy. Kansio löytyi oikeasta polusta:

![image](https://github.com/user-attachments/assets/69bf2301-0cfe-44b0-88ae-71241be84348)


Osioon käytetty aika: ~20 min

## c) sls-tiedosto kahdella eri tilafunktiolla

Palasin t001 koneen ääreen `vagrant ssh t001` ja loin uuden kansion `mkdir -p /srv/salt/matrix`

![image](https://github.com/user-attachments/assets/fd92cbe0-082f-44d3-ad41-a18214f9654f)

Sitten loin tähän hakemistoon uuden init.sls -tiedoston, jota lähteä muokkaamaan `export EDITOR=micro` `sudoedit init.sls`

pkg.installed - asentaa cmatrix -ohjelman

user.present - luo käyttäjän testuser

![image](https://github.com/user-attachments/assets/29067704-6253-4948-b57c-a107d3ef43c1)

Sitten ajoin komennon `sudo salt '*' state.apply matrix`

![image](https://github.com/user-attachments/assets/195906c3-e3a8-42d0-a444-20fb1aedd94e)

Käyttäjän luominen onnistui, mutta cmatrix -ohjelman asennuksessa meni jotain pieleen (Failed: 1).

Lyhyen googlaamisen jälkeen päädyin osoitteeseen https://askubuntu.com/questions/1096930/sudo-apt-update-error-release-file-is-not-yet-valid.

Sivustolla eräässä kommentissa todettiinkin että "This is a timezone issue." Joten päädyin tarkistamaan koneen tietoja `date` -komennolla. Kellonajat olivatkin pielessä joten korjasin ne kummallakin koneella erikseen.

![image](https://github.com/user-attachments/assets/7072028c-3bb1-4eda-94ef-165259e0b242)

Vastaavan komennon kävin ajamassa myös koneella t002, jonka jälkeen kokeilin ajaa komennon `sudo salt '*' state.apply matrix` uudelleen.

![image](https://github.com/user-attachments/assets/4158421a-f6b0-4726-979f-6c4931e5d614)

![image](https://github.com/user-attachments/assets/288fcbeb-5997-4096-ad71-a1e8b4168555)

Tällä kertaa komento näyttikin menevän läpi onnistuneesti (changed=1, käyttäjä testuser saatiin luotua jo aiemmin).

Ajamalla komennon läpi useamman kerran changed=x kenttä jää puuttumaan, eli sls-tiedosto on idempotentti:

![image](https://github.com/user-attachments/assets/0a4ddcf7-895e-4d76-b5a0-4c9e78fdb402)


Sitten todentamaan t002 koneella, että toivotut muutokset menivät läpi.

Matrix -ohjelmisto lähti käyntiin `cmatrix` komennolla:

![image](https://github.com/user-attachments/assets/45893106-1fa3-4318-9ced-96e149df488a)

testuser -käyttäjähakemisto löytyi polusta /home/

![image](https://github.com/user-attachments/assets/0512ef37-b203-4c68-b5f5-5be1d306ef84)

testuser alimmalla rivillä:

![image](https://github.com/user-attachments/assets/fa753e3c-9a53-4499-8374-2925900d4545)


Osioon käytetty aika: ~45min



### Lähteet

AskUbuntu. sudo apt update error: "Release file is not yet valid". Luettavissa: https://askubuntu.com/questions/1096930/sudo-apt-update-error-release-file-is-not-yet-valid.

Hostinger.com. Linux list users: How to display all users in the terminal?. Luettavissa: https://www.hostinger.com/tutorials/how-to-list-users-in-linux#:~:text=You%20can%20list%20user%20accounts,users%2C%20use%20the%20users%20command.

Karvinen, T. 26.3.2025. Palvelinten Hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/.

Karvinen, T. 3.4.2024. Hello Salt Infra-as-Code. Luettavissa: https://terokarvinen.com/2024/hello-salt-infra-as-code/.

Linux.com. Fun Linux Terminal Commands !. Luettavissa: https://www.linux.com/training-tutorials/fun-linux-terminal-commands/.

Salt Project. s.a. Salt overview. Luettavissa: https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml.

Salt Project. s.a. Start the master and minion services. Luettavissa: https://docs.saltproject.io/salt/install-guide/en/latest/topics/start-salt-services.html.

Salt Project. s.a. Management of user accounts. Luettavissa: https://docs.saltproject.io/en/3006/ref/states/all/salt.states.user.html
