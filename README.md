# user management with ansible
[ see document.md for doc (finnish only) ]
- User creation and management made simple with ansible. This ansible-playbook covers safe user generation using ansible-vault, locking / unlocking of shell, setting up home & work environment, as well as managing ssh keys and permissions. All you have to do is run the playbook.
<img width="532" height="48" alt="kuva" src="https://github.com/user-attachments/assets/eec354c5-ec07-4a33-9a72-6a0d28e04af2" />

- demo: https://youtu.be/ShA2Ik32igY
### how to use

- in order to use this playbook, make sure you create your own ansible-vault password, and replace the one in files/secret.yml with your own. to change the name of the account, go to roles/config/tasks/main.yml and change the name to your desired user.
- install with `git clone`, or download zip.
- there's an experimental play in tasks, so to run you need to have apache installed.
