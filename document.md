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

