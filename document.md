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
- konfiguraatiotiedossa ilmoitettu vault_password_file osoittaa tiedostoon, jotta ei tarvitse manuaalisesti syöttää komentoa --ask-become-pass playbookia ajaessa.

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

- Tähäna asti tehty projekti näyttää seuraavanlaiselta:
<img width="600" height="276" alt="kuva" src="https://github.com/user-attachments/assets/f717c368-b7ac-4c57-86e1-d2daa42a0ff4" />

- playbookia ajaessa huomattiin, että tulos jäi "changed =1" tilaan. vähän googlatessa huomattiin, että ongelma korjautuu kirjaamalla `main.yml` tiedostoon `update_password: on_create`. vika johtui hashista, joka päivittyi joka kerta playbookia ajaessa muuttaen hashia.

## SSH ja sudoless automatisointi
Lisäsin projektiin SSH-avaimella kirjautumisen sekä sudon käytön ilman salasanaa.

Toteutin tämän Ansible playbookilla, jossa luodaan käyttäjä, lisätään SSH-avain ja määritellään sudo-oikeudet ilman salasanan kyselyä. Tämä mahdollistaa turvallisen ja automatisoidun pääsyn järjestelmään.

<img width="789" height="370" alt="Näyttökuva 2026-05-05 173949" src="https://github.com/user-attachments/assets/007a71b0-a00f-434f-bcdf-8af1a7ab2d5d" />


Ajoin playbookin komennolla:

ansible-playbook -i localhost, -c local users.yml

<img width="825" height="484" alt="Näyttökuva 2026-05-05 173057" src="https://github.com/user-attachments/assets/672d3d2f-3d47-471f-9f01-957f1c8ac959" />

Yllä näkyy, että kaikki tehtävät suoritettiin onnistuneesti ilman virheitä.

Tämän jälkeen testasin käyttäjän toiminnan. Vaihdoin käyttäjään komennolla:

sudo su - anssi

<img width="810" height="47" alt="Näyttökuva 2026-05-05 173244" src="https://github.com/user-attachments/assets/9b5742a9-c22b-4102-bcec-72c1daf217d2" />

Käyttäjä luotiin onnistuneesti ja siihen päästiin siirtymään.

Seuraavaksi testasin sudo-oikeudet komennolla:

sudo ls

<img width="753" height="112" alt="Näyttökuva 2026-05-05 173445" src="https://github.com/user-attachments/assets/c016794d-03c5-4223-a696-2aa46862d8fb" />

Komento toimi ilman salasanakyselyä, mikä osoittaa että sudoless on määritelty oikein.

Tämä ratkaisu on idempotentti, eli playbook voidaan ajaa useita kertoja ilman että järjestelmä menee rikki.

# Lähteet
- Jeremy Canfield, 2025. | Ansible: create user account: https://www.freekb.net/Article?id=2538
- Root RouteWay, 2023. | Efficiently Manage Users and Groups with Ansible: A Step-By-Step Guide: https://medium.com/@RootRouteway/efficiently-manage-users-and-groups-with-ansible-a-step-by-step-guide-d72a2b625b60
- Karvinen, Tero 2026. | Palvelinten hallinta: https://terokarvinen.com/palvelinten-hallinta/
