# projektin dokumentointi:

- aivan ensiksi luotiin kansio nimeltä ansible-project, jonka sisälle tehtiin tiedostot ansible.cfg, hosts.ini sekä site.yml.
- hosts.ini tiedostoon on kirjattu:
```
localhost

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

- ansible.cfg:
```
[defaults]
inventory = hosts.ini
vault_password_file = .vaultpass.txt
display_args_to_stdout = true
```
- konfiguraatiotiedossa ilmoitettu vault_password_file osoittaa tiedostoon, jotta ei tarvitse manuaalisesti syöttää komentoa `--ask-become-pass` playbookia ajaessa.

- site.yml:
```
- hosts: all
  become: yes
  vars_files:
    - roles/config/files/secret.yml
  roles:
    - config
```

- tätä ennen käytin komennon `EDITOR=micro`, jotta seuraavasta tulisi helpompaa:
- `ansible-vault create secret.yml`

- ensimmäisellä kerralla ansible-vault pyytää tekemään salasanan itse vaultin käyttöön, ja sen jälkeen sain luotua salasanan tulevalle käyttäjälle. tämä salasana sitten kryptattiin.
- itse vaultin salasana kirjattiin tiedostoon `micro .vaultpass.txt` tämä pysyy omalla koneella, eikä jaeta esim. gittiin.

- sen jälkeen alettiin järjestelemään kansioita nätiksi: tehtiin neljä kansiota: `roles`, `config`, `tasks` ja `files` mkdir -komennolla.
- `config` lisättiin `roles` kansion sisään, ja loput configin sisälle.
- `files` kansioon lisättiin aiemmin tehty kryptattu salasana secret.yml
- `tasks` kansioon luotiin `main.yml`, jossa itse play on.
- main.yml sisältö:

<img width="800" height="219" alt="kuva" src="https://github.com/user-attachments/assets/21fc5f97-075a-46ac-b463-c2d581fbb57f" />

- eli tehdään käyttäjä anssi, viitataan kryptattuun salasanaan, `state: present` varmistaa, että jos käyttäjää ei ole olemassa, ansible tekee sen.
- loput tiedostosta lisättiin sen jälkeen, kun oltiin varmistettu että playbook toimii: kirjataan käyttäjän shell, asetetaan kotihakemisto, ryhmä (tässä tapauksessa "it"), sekä asetetaan `uid`.
- `append: yes` tarkoittaa, että käyttäjä pysyy aiemmissa ryhmissään, jos lisätään esim. ryhmään it.

- Tähän asti tehty projekti näyttää seuraavanlaiselta:
<img width="600" height="276" alt="kuva" src="https://github.com/user-attachments/assets/f717c368-b7ac-4c57-86e1-d2daa42a0ff4" />

- playbookia ajaessa huomattiin, että tulos jäi "changed =1" tilaan. vähän googlatessa huomattiin, että ongelma korjautuu kirjaamalla `main.yml` tiedostoon `update_password: on_create`. vika johtui hashista, joka päivittyi joka kerta playbookia ajaessa muuttaen hashia.

- Päätettiin lisätä uudelle käyttäjälle kotihakemistoonsa oma ansible kansio (työympäristö), ja laitettiin sinne tervehdysviesti.
- Tämä toteutui luomalla ensin polkuun `/roles/config/files` uusi tiedosto nimeltä `paths.yml`, joka määrittää käyttäjän kotihakemiston:
<img width="700" height="88" alt="image" src="https://github.com/user-attachments/assets/ee19d0d0-c111-4422-a553-2aa590d2d861" />

- itse play `main.yml` tiedostossa näyttää seuraavanlaiselta:
<img width="550" height="150" alt="image" src="https://github.com/user-attachments/assets/c457f768-d125-4602-89cc-f49547cec455" />

- viittaukset polkuihin helpottavat tulevaisuudessa, jos esim. polkua tai käyttäjää haluttaisiin muuttaa.

- lisätään playhin vielä tervehdysviesti (testataan että ympäristö toimii): `main.yml` lisätään aiemman kohdan perään tämä:

<img width="443" height="119" alt="image" src="https://github.com/user-attachments/assets/46851e14-5206-471a-ad3b-8999da3cea45" />

- tämäkin tiedosto löytyy `roles/config/files` hakemistosta tekstitiedostona `helloansible.txt`:
<img width="790" height="44" alt="image" src="https://github.com/user-attachments/assets/03aec757-7c15-4107-be81-6fa4f2f06c13" />


- lisäksi haluttiin testata vielä oikeuksien antoa tiettyihin kansioihin ja applikaatioihin. esimerkiksi otettiin apache2. `main.yml` tuli lisättyä seuraavat pätkät:
<img width="424" height="305" alt="image" src="https://github.com/user-attachments/assets/9b7f9a62-667b-4ad7-a2e3-7f5460a0e400" />

- ja lisättiin `create anssi account` groups kohtiin www-data:
<img width="185" height="54" alt="image" src="https://github.com/user-attachments/assets/9428e844-7760-4493-8ebd-18c91e22e5e7" />

- näin assin on mahdollista hallita apachea.
- `mode: 2775` tarkoittaa että omistaja ja ryhmä voivat kirjoittaa, lukea ja ajaa kansion, muut lukea ja ajaa. `2` eli setGid varmistaa, että kansioon luodut tiedostot perivät kansion ryhmän. `0644` taas tarkoittaa että omistaja ja ryhmä voivat lukea ja kirjoittaa, muut vain lukea.


## SSH ja sudoless käyttäjän hallinta

Tässä tehtävässä konfiguroin käyttäjän hallinnan Ansiblella. Käyttäjälle *anssi* lisättiin SSH-avain ja määriteltiin sudo-oikeudet ilman salasanaa.

<img width="730" height="194" alt="Näyttökuva 2026-05-05 190451" src="https://github.com/user-attachments/assets/c95e11eb-d52c-4e60-a92e-b49fe8cbe6b4" />



Ansible playbookin ajo

Ajoin playbookin komennolla:

```bash
ansible-playbook site.yml -c local
```
Kaikki tehtävät suoritettiin onnistuneesti ilman virheitä.

<img width="1004" height="668" alt="Näyttökuva 2026-05-05 184215" src="https://github.com/user-attachments/assets/bc7fc36a-18ce-4441-b3fc-bc2e49c7845e" />


SSH-yhteyden testaus

Lisäsin SSH-avaimen käyttäjälle anssi Ansiblella. Tämä tarkoittaa, että kirjautuminen voidaan tehdä ilman salasanaa käyttäen julkisen ja yksityisen avaimen paria.

Testasin yhteyden komennolla:

```bash
ssh anssi@localhost

```

<img width="909" height="294" alt="Näyttökuva 2026-05-05 191936" src="https://github.com/user-attachments/assets/a98abdc9-c3f0-40d6-bb52-29343377abb0" />


Kirjautuminen onnistui ilman salasanaa, mikä osoittaa että SSH-avain toimii oikein. Tämä parantaa turvallisuutta, koska salasanoja ei tarvitse käyttää eikä niitä voida arvata tai vuotaa.

SSH-avaimiin perustuva kirjautuminen on yleinen käytäntö palvelinympäristöissä.

Sudo ilman salasanaa

Määrittelin sudo-oikeudet ryhmälle "it" Ansiblella. Tämä tehtiin lisäämällä seuraava määritys:
<img width="730" height="194" alt="Näyttökuva 2026-05-05 190451" src="https://github.com/user-attachments/assets/8f5da495-9785-426a-8e19-ff8fb0ba4d95" />

Määritys "%it ALL=(ALL) NOPASSWD: ALL" tarkoittaa, että kaikki ryhmän "it" käyttäjät voivat suorittaa sudo-komentoja ilman salasanaa.

Testasin tämän komennolla:

```bash
sudo su - anssi
sudo ls /
```
<img width="735" height="123" alt="Näyttökuva 2026-05-05 184308" src="https://github.com/user-attachments/assets/759801c9-3d37-40ce-a919-23d97dfd42c1" />

Komennot toimivat ilman salasanakyselyä, mikä osoittaa että sudoless-konfiguraatio toimii oikein.

Tämä ratkaisu helpottaa järjestelmän hallintaa, koska käyttäjien oikeuksia voidaan hallita ryhmätasolla eikä yksittäisiä käyttäjiä tarvitse määritellä erikseen.

### Lopputuloksen konfiguraatio:

- playbook työn lopussa näyttää seuraavanlaiselta:
<img width="546" height="379" alt="image" src="https://github.com/user-attachments/assets/96d4e11f-b880-4613-8cc8-05c00b3d2b4d" />

- ja playn recap näyttää ratkaisun olevan idempotentti:
<img width="1246" height="101" alt="image" src="https://github.com/user-attachments/assets/d5a47052-2d86-477a-aca4-7dd5d9db52da" />


# Lähteet
- Jeremy Canfield, 2025. | Ansible: create user account: https://www.freekb.net/Article?id=2538
- Root RouteWay, 2023. | Efficiently Manage Users and Groups with Ansible: A Step-By-Step Guide: https://medium.com/@RootRouteway/efficiently-manage-users-and-groups-with-ansible-a-step-by-step-guide-d72a2b625b60
- Karvinen, Tero 2026. | Palvelinten hallinta: https://terokarvinen.com/palvelinten-hallinta/
