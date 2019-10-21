itescia:
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2f6MQuKEqzDAGyKNP1o+Qfb4SSSKw+BmZWbyhkMkouPAY/MnEet8IFav+Nz3hgYBauo2wvHQ0ElzO1aa57fboL0r2aTHZcGMZAAPQnZu8CSP1+9yuXzmJNmXVkYn8ohjMZbsfhF5QLnq7okv9c9ZQiePvrOa1/FUgKq03d8gHagmCXp0oU37VG7Gg1Och2P0qwbVDRfXhR8P9LV4/ZFmXGsMFEVqjhqPMgC15Sdtuurwz1WdcFrQyqU1YtxwyKQZgWoruEpE3sl9QgG+fI6TutB5hoMYHRuepyAmTefAAqm42buI0FeamkWVtp7/+8ixoSOb/JNabYQXHFm5Hehbv itescia@master

root: 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCyN+7ps5dW5MDtftbDCvU5PUvGaHRrA9oyyfjWACe6rr6QvzygB0kPs1U1nAyXN9pyxtFQ5HV4WjN2Kz1jeaynX375pi7dUvCAyVoTAElctVyN/LriXwL11XE4J5Fg9FgP2a4TzDmQ/tKwNrwbb6hUIcC/aMmaeGrQGyTQipvgwYDE6ZNPWcMTKivXSHtNZlWUicz2p4vOye3dnTPA3csnKPygJS6i3oXyGbjll7FwRQBLaJuxo8YQRtrqNS4bjsJixBw38RouJjrAVnIlLa1DwYsB2+lkZaWkJ38tnmakh8Pd/MRvqDFluiDAZ/vuRifi0doa78Or/YHy0dxsRtMB root@master

-Installer Git et Ansible sur chaque poste (Master, Slave et Backup)
-Récupérer le projet à l'aide de la commande git clone (https://github.com/Gregitescia/recette_Ansible.git)
-Déployer la clé publique du serveur master sur les autres machines (Slave et Backup) en root
	~/.ssh/authorized_keys
	
	
itescia@master:~/recette_Ansible/roles/common/tasks$ vim user.yml

 - name: authorized key
  authorized_key:
    user: itescia
    key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2f6MQuKEqzDAGyKNP1o+Qfb4SSSKw+BmZWbyhkMkouPAY/MnEet8IFav+Nz3hgYBauo2wvHQ0ElzO1aa57fboL0r2aTHZcGMZAAPQnZu8CSP1+9yuXzmJNmXVkYn8ohjMZbsfhF5QLnq7okv9c9ZQiePvrOa1/FUgKq03d8gHagmCXp0oU37VG7Gg1Och2P0qwbVDRfXhR8P9LV4/ZFmXGsMFEVqjhqPMgC15Sdtuurwz1WdcFrQyqU1YtxwyKQZgWoruEpE3sl9QgG+fI6TutB5hoMYHRuepyAmTefAAqm42buI0FeamkWVtp7/+8ixoSOb/JNabYQXHFm5Hehbv itescia@master'
    state: present

- name: authorized key
  authorized_key:
    user: root
    key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCyN+7ps5dW5MDtftbDCvU5PUvGaHRrA9oyyfjWACe6rr6QvzygB0kPs1U1nAyXN9pyxtFQ5HV4WjN2Kz1jeaynX375pi7dUvCAyVoTAElctVyN/LriXwL11XE4J5Fg9FgP2a4TzDmQ/tKwNrwbb6hUIcC/aMmaeGrQGyTQipvgwYDE6ZNPWcMTKivXSHtNZlWUicz2p4vOye3dnTPA3csnKPygJS6i3oXyGbjll7FwRQBLaJuxo8YQRtrqNS4bjsJixBw38RouJjrAVnIlLa1DwYsB2+lkZaWkJ38tnmakh8Pd/MRvqDFluiDAZ/vuRifi0doa78Or/YHy0dxsRtMB root@master'
    state: present


Modifier le fichier /etc/hosts
	
172.16.203.204 master
192.168.41.128 master
172.16.203.85 slave
192.168.35.128 slave
172.16.203.41 backup
192.168.202.128 backup

	
Créer un fichier roles/common/tasks/dns.yml

---
- name: ajout adresses IP dans /etc/hosts
  lineinfile:
    path: /etc/hosts
    regexp: '^127\.0\.0\.1'
    line: 127.0.0.1 localhost
    line: 172.16.203.204 master
    line: 192.168.41.128 master
    line: 172.16.203.85 slave
    line: 192.168.35.128 slave
    line: 172.16.203.41 backup
    line: 192.168.202.128 backup

	
	
Ajouter dans le fichier hosts.txt 

[master]
172.16.203.204 


[slave]
172.16.203.85 

[backup]
172.16.203.41 


[all:vars]
ansible_user=root
ansible_python_interpreter=/usr/bin/python3	


	
Ajouter dans le fichier /roles/common/tasks/user.yml

---
- name: création d'un utilisateur
  user:
          name: "{{item.name}}"
          password: "{{item.password}}" 
          createhome: yes
  loop: "{{users}}"

Modifier le fichier roles/common/vars/main.yml
---
users: 
        - name: client1
          password: client1
        - name: client2
          password: client2
        - name: client3
          password: client3
		  
		  
		  
Créer le fichier requirements.yml et ajouter les lignes suivantes:

---
# role mysql from galaxy
- src: geerlingguy.mysql
#
# role backup from galacy
- src: ome.mysql_backup
#
#role apache from galaxy
- src : geerlingguy.apache



Executer le fichier requirements.yml à l'aide Commande : ansible-galaxy install -r requirements.yml



Modifier le fichier apache.yml

---
- hosts: all
  become: yes
  vars_files:
    - vars/main.yml

  pre_tasks:
    - name: création du documentroot
      file: path="{{item.documentroot}}" state=directory
      loop: "{{apache_vhosts}}"     
      
  roles:
    - { role: geerlingguy.apache }
	
	

Ajouter dans le fichier /vars/main.yml

- name: client1
    encoding: latin1
    collation: latin1_general_ci
  - name: client1
    encoding: latin1
    collation: latin1_general_ci
  - name: client2
    encoding: latin1
    collation: latin1_general_ci

mysql_users:
  - name: itescia
    host: "%"
    password: itescia
    priv: "db_itescia.*:ALL"
	
# Configuration Apache
apache_listen_port: 8080
apache_vhosts:
  - {servername: "www.client1.com", documentroot: "/var/www/vhosts/srv_client1"}


Voir dans :  /etc/apache2/sites-available/vhosts.conf  et dans /var/www/vhosts

Master / Slave

vim /etc/mysql/my.cnf
blind-adress = 172.16.203.204
CREATE USER "slave"@"172.16.203.85" IDENTIFIED BY "< itescia >" ;


Grant replication slave on *.* to "slave"@"ip address" ;
FLUSH PRIVILEGES ;

Sauvegarde et backup des bdd


