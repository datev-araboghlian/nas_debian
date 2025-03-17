# nas_debian
Déploiement d’un Serveur NAS sous Debian

⸻

Déploiement d’un Serveur NAS sous Debian

1. Introduction

Objectif du Projet

L’objectif de ce projet est de mettre en place un serveur NAS robuste et flexible sous Debian, en privilégiant une configuration sans interface graphique pour optimiser les ressources.

Fonctionnalités clés
	•	RAID 5 : Tolérance aux pannes et optimisation des performances
	•	Synchronisation avec Rsync : Sauvegarde automatisée et sécurisée
	•	Partage sécurisé : Accès via SFTP & Samba
	•	Administration via Webmin : Interface Web pour gérer le serveur

⸻

2. Configuration du RAID 5

Pourquoi RAID 5 ?
	•	Tolérance aux pannes grâce à la redondance
	•	Bonne performance en lecture
	•	Utilisation optimisée des disques

Mise en place du RAID 5

sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sda /dev/sdb /dev/sdd
sudo mkfs.ext4 /dev/md0
sudo mkdir /mnt/raid && sudo mount /dev/md0 /mnt/raid

Configuration automatique
	•	Récupération de l’UUID

sudo blkid /dev/md0

	•	Ajout à /etc/fstab

UUID=ton-uuid /mnt/raid ext4 defaults 0 2

	•	Sauvegarde de la configuration

sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf
sudo update-initramfs -u



⸻

3. Ajout d’un disque de secours au RAID 5

Pourquoi ?
	•	Remplacement rapide en cas de panne

Procédure

sudo mdadm --detail /dev/md0
sudo mdadm --add /dev/md0 /dev/sde
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u



⸻

4. Synchronisation avec Rsync

Pourquoi Rsync ?
	•	Sauvegarde incrémentielle efficace
	•	Sécurisé avec SSH

Installation et configuration

sudo apt update && sudo apt install rsync -y
ssh-keygen -t rsa
ssh-copy-id user@destination_ip
rsync -avz /mnt/raid/ user@destination_ip:/mnt/backup/

Automatisation avec Cron
	•	Exécution quotidienne à 2h

0 2 * * * rsync -avz /mnt/raid/ user@destination_ip:/mnt/backup/



⸻

5. Configuration SFTP

Sécurisation des accès aux fichiers

Installation et configuration

sudo apt install openssh-server

Ajout de règles pour les utilisateurs dans /etc/ssh/sshd_config

Match User swan
ChrootDirectory /mnt/raid
ForceCommand internal-sftp -d /swan
AllowTcpForwarding no

Gestion des droits

sudo mkdir -p /mnt/raid/{swan,sana,datev}
sudo chown root:root /mnt/raid
sudo chmod 755 /mnt/raid
sudo chown -R swan:swan /mnt/raid/swan

Redémarrer SSH

sudo systemctl restart ssh



⸻

6. Partage Samba (SMB)

Pourquoi Samba ?
	•	Accès aux fichiers depuis Windows/Linux
	•	Gestion des permissions par utilisateur

Installation et configuration

sudo apt install samba -y

Ajout d’un partage pour l’utilisateur Swan dans /etc/samba/smb.conf

[swan]
path = /mnt/raid/swan
valid users = swan
read only = no

Ajout des utilisateurs Samba

sudo smbpasswd -a swan

Redémarrer Samba

sudo systemctl restart smbd nmbd



⸻

7. Installation de Webmin

Pourquoi Webmin ?
	•	Interface graphique pour administrer Debian
	•	Accès à distance sécurisé

Installation et accès

sudo apt update
curl -o webmin-setup-repo.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repo.sh
sh webmin-setup-repo.sh
sudo apt install -y webmin

Accès via navigateur

https://<IP_SERVEUR>:10000



⸻

8. Sécurité et Gestion des Utilisateurs

Principales mesures de sécurité
	•	Accès sécurisé via SSH et SFTP
	•	Droits d’accès stricts aux fichiers
	•	Synchronisation et sauvegarde régulières

Gestion des utilisateurs
	•	Sessions personnalisées avec accès privé
	•	Séparation des permissions (Admin et Utilisateurs)

⸻

9. Conclusion

Récapitulatif
	•	RAID 5 pour la tolérance aux pannes
	•	Synchronisation et sauvegarde automatisées
	•	Partage sécurisé via SFTP et Samba
	•	Administration simplifiée avec Webmin

Évolutions possibles
	•	Virtualisation et extensions NAS
	•	Automatisation avancée
	•	Intégration Cloud
