# Miroir Gentoo Cygogne

Ce guide décrit comment utiliser mon miroir sur votre système Gentoo, et vous explique aussi comment en créer un.
Caractéristiques principales de mon miroir: 

- Liaison fibrée 1 Go.
- 2 To de stockage dediée (actuellement seul 480Go sont pris).
- 2 CPU / 1G RAM (C'est large pour cet usage).
- Syncro toutes les heures piles depuis le serveur miroir maître Gentoo.
- Accès via HTTPS.
- Je ne détient aucun log, je m'en fout de ce que vous installez.
- Tourne sous Debian 11.
- 100% Français et Weeb friendly (pas une si bonne nouvelle).
- Cygogne-it est pas si fou comme nom, je sais. 


 N'hésitez-pas à me dire si vous utilisez mon miroir, afin que je puisse l'améliorer en fonction de l'usage que vous en faites.
 Vous pouvez aussi modifier le fichier `readme.md`, m'aider à traduire le projet ou me suggérer des améliorations.

## Configurer Cygogne en miroir principal pour Portage

La configuration se résume au changement d'un seul fichier, donc c'est assez simple. 
A l'aide de votre éditeur favori, modifier le fichier `make.conf` situé dans `/etc/portage/` :
```
nano /etc/portage/make.conf
```
Normalement, vous ne devriez pas avoir de ligne `GENTOO_MIRRORS="something"` dans votre fichier.
Si vous l'avez déjà, c'est bon.
Sinon, il faut l'ajouter, avec une ligne dediée dans le fichier et peu importe l'ordre:
```
GENTOO_MIRRORS="truc"
```
Une fois que vous avez ajouté la ligne, on doit changer la valeur de la ligne `GENTOO_MIRRORS` :
Il faut changer `truc` par `https://cygogne-it.fr`, ce qui doit vous donner ceci :
```
GENTOO_MIRRORS="https://cygogne-it.fr"
```
Enregistrer le fichier `make.conf`.

Le miroir est donc configuré, et sera utilisé pour tout les téléchargements de Portage.
Vous pouvez taper `eix-sync` (si vous avez `eix` installé) ou `emerge --sync` pour tester l'update des paquets.

Si vous voulez créer votre propre miroir, suivez les instructions ci-dessous.

## Créer votre propre miroir

Il faut garder à l'esprit que ce genre de "miroirs perso" utilise la bande passante du miroir maître de Gentoo, et que je n'encourage pas la création de ce genre de miroir si l'objectif est pour vous seul. Il est toutefois possible qu'il n'y ait pas de miroir disponible près de chez vous, et l'équipe de Gentoo vous encourage à créer ce genre de miroir et de le soumettre pour qu'il soit disponible de façon officielle. 

Voici la méthode si vous souhaitez continuer malgré tout ....

### Pré-requis

- Bande passante : Il est recommandé d'avoir au moins 5Mb/s en débit montant, où plus vous avez mieux c'est.
- Stockage : L'hébergement d'un miroir requiert au moins 550 Go de stockage sur votre disque.
- Un accès HTTP/HTTPS/RSYNC à votre serveur. 
- Je part du principe que vous avez déjà un serveur web configuré et fonctionnel.
- L'équipe Gentoo recommande d'update votre serveur toutes les 4H.
- `rsync`
- `cron` ou une alternative.
- Un système Gentoo pour tester le bon fonctionnement de votre miroir.
- Beaucoup de temps (en fonction de votre connexion)

### Téléchargement initial

Cette partie est celle qui prends le plus de temps, et aussi la plus complexe de ce guide.
Il faut d'abord déterminé où sera stocké l'ensemble du miroir
En général c'est `/var/www/` ou `/var/www/html/` si vous avez choisi `apache` comme serveur web.
Dans notre cas, j'ai choisi `/var/www/html/gentoo/`.

Je vous recommande de créer un hôte virtuel `VirtualHost` pour rediriger les requêtes sur le bon répertoire.
C'est indispensable si vous souhaitez héberger d'autres sites sur un même serveur.
Lisez le manuel de votre serveur web pour savoir comment l'on configure des `VirtualHost`.

WUne fois que votre répertoire a été choisi et est accessible via le web, il est temps de télécharger TOUT le nécessaire depuis le serveur maître.
Actuellement, tout cela pèse près de 480Go, donc soyez patients.

Pour télécharger vers votre répertoire choisi, tapez la commande suivante :
```
rsync -av rsync://masterdistfiles.gentoo.org/gentoo/ /var/www/html/gentoo/
```
Changer `/var/www/html/gentoo/` pour correspondre à votre cas.
L'arguement `-v` à la commande peut être ignoré si vous ne voulez pas le mode verbeux.

Après ce gros téléchargement, votre miroir est désormais utilisable dans le fichier `make.conf` (VOir chapitre précédent).
Attention, si vous vous arrêter ici, votre miroir ne sera pas mis à jour par la suite.

### Tâche Cron pour update votre miroir

Les instructions qui suivent concernent [`cron`](https://github.com/cronie-crond/cronie), lisez le manuel si vous utilisez un autre outil.

Créer une nouvelle tâche planifiée :
```
crontab -e
```
Pour update toutes les 4 heures :
```
00 */4 * * * rsync -av rsync://masterdistfiles.gentoo.org/gentoo/ /var/www/html/gentoo/
```
VOus pouvez changer `4` en fonction de ce que vous voulez (ex : `1` pour toutes les heures).

Tout devrait être fonctionnel.

### Améliorations et bonus

#### Journalisation :

Je vous recommande de journaliser votre tâche cron pour observer les changements qui ont été faits : 

```
00 */4 * * * rsync -av rsync://masterdistfiles.gentoo.org/gentoo/ /var/www/html/gentoo/ >> /home/user/log.txt
```
 #### Script pour faciliter la modification :

Voici un exemple de script correpondant à la tâche :

```
#!/bin/bash

rsync -av rsync://masterdistfiles.gentoo.org/gentoo/ /var/www/html/gentoo/ >> /home/user/log.txt
```
Pour que dans cron on utilise que le script :

```
00 */4 * * * /home/juliiine/rsync_mirror.sh
```
#### Envoyer un mail à chaque update :

POur cela, vous pouvez utiliser un serveur `postfix`, mais il y a plus simple.
J'utilise [Swaks](https://github.com/jetmore/swaks), un outil utilisant Perl permettant l'envoi de mails, notamment dans des scripts.
Vous avez besoin d'une vraie adresse mail et des informations SMTP de votre fournisseur mail.
J'utilise une adresse OVH pour cet usage.

Un exemple de Swaks (avec les paramètres OVH du serveur sortant) :

```
swaks -t receiver@address.com -s ssl0.ovh.net:587 -tls -au gentoo@yourdomain.com -ap superpassword -f gentoo@yourdomain.com --body "The body of your mail" --h-Subject "Planified update is good" --attach /home/user/logofrsyncoutput.log
```

## Contribuer

N'hésitez-pas à faire des remarques, des propositions d'améliorations, ou de l'aide pour traduire le projet dans d'autres langues.
Vous pouvez également m'informer si vous utilisez mon miroir, afin que je puisse l'améliorer en fonction.