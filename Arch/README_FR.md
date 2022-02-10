# Miroir Arch Juline

Langues disponibles : 

[Français](https://github.com/juliiine/cygogne-mirror/blob/main/Arch/README_FR.md)
[English](https://github.com/juliiine/cygogne-mirror/blob/main/Arch/README.md)

Ce guide décrit comment utiliser mon miroir sur votre système Arch, et vous explique aussi comment en créer un.

N'hésitez-pas à me dire si vous utilisez mon miroir, afin que je puisse l'améliorer en fonction de l'usage que vous en faites.
Vous pouvez aussi modifier le fichier `readme.md`, m'aider à traduire le projet ou me suggérer des améliorations.

## Configurer Juline en miroir principal pour Pacman

La configuration se résume au changement d'un seul fichier, donc c'est assez simple. 
A l'aide de votre éditeur favori, modifier le fichier `mirrorlist` situé dans `/etc/pacman.d/` :
```
nano /etc/pacman.d/mirrorlist
```
Il est possible que Reflector ait déjà crée des entrées "Server =" dans votre fichier, si c'est le cas il faut juste les commenter.
Sinon, il faut ajouter une entrée Server, avec une ligne dediée dans le fichier et peu importe l'ordre:
```
Server = <url>
```
Une fois que vous avez ajouté la ligne, on doit changer la valeur de la ligne `Server` :
Il faut changer `<url>` par `https://arch.juline.tech/$repo/os/$arch`, ce qui doit vous donner ceci :
```
Server = https://arch.juline.tech/$repo/os/$arch
```
Enregistrer le fichier `mirrorlist`.

Le miroir est donc configuré, et sera utilisé pour tout les téléchargements de Pacman.
Vous pouvez taper `paru -Syyu` (si vous avez `paru` installé) ou `pacman -Syyu` pour tester l'update des paquets.

Si vous voulez créer votre propre miroir, suivez les instructions ci-dessous.

## Créer votre propre miroir

Il faut garder à l'esprit que ce genre de "miroirs perso" utilise la bande passante d'un serveur tier, et que je n'encourage pas la création de ce genre de miroir si l'objectif est pour vous seul. Il est toutefois possible qu'il n'y ait pas de miroir disponible près de chez vous, et l'équipe d'Arch vous encourage à créer ce genre de miroir et de le soumettre pour qu'il soit disponible de façon officielle. 

Voici la méthode si vous souhaitez continuer malgré tout ....

### Pré-requis

- Bande passante : Il est recommandé d'avoir au moins 5Mb/s en débit montant, où plus vous avez mieux c'est.
- Stockage : L'hébergement d'un miroir requiert au moins 350 Go de stockage sur votre disque.
- Un accès HTTP/HTTPS/RSYNC à votre serveur. 
- Je part du principe que vous avez déjà un serveur web configuré et fonctionnel.
- L'équipe Arch recommande d'update votre serveur toutes les heures.
- `rsync`
- `cron` ou une alternative.
- Un système Arch pour tester le bon fonctionnement de votre miroir.
- Beaucoup de temps (en fonction de votre connexion)

### Téléchargement initial

Cette partie est celle qui prends le plus de temps, et aussi la plus complexe de ce guide.
Il faut d'abord déterminé où sera stocké l'ensemble du miroir
En général c'est `/var/www/` ou `/var/www/html/` si vous avez choisi `apache` comme serveur web.
Dans notre cas, j'ai choisi `/var/www/html/arch/`.

Je vous recommande de créer un hôte virtuel `VirtualHost` pour rediriger les requêtes sur le bon répertoire.
C'est indispensable si vous souhaitez héberger d'autres sites sur un même serveur.
Lisez le manuel de votre serveur web pour savoir comment l'on configure des `VirtualHost`.

Une fois que votre répertoire a été choisi et est accessible via le web, il est temps de télécharger TOUT le nécessaire depuis un serveur Tier 1 qui doit supporter Rsync.
Actuellement, tout cela pèse près de 350Go, donc soyez patients.

Pour télécharger depuis un serveur Tier 1 (Liste des miroirs Tier 1: https://archlinux.org/mirrors/tier/1/) vers votre répertoire choisi, tapez la commande suivante avec le mien en example :
```
rsync -av rsync://arch.juline.tech/archlinux /var/www/html/arch/
```
Changer `/var/www/html/arch/` pour correspondre à votre cas.
L'arguement `-v` à la commande peut être ignoré si vous ne voulez pas le mode verbeux.

Après ce gros téléchargement, votre miroir est désormais utilisable dans le fichier `mirrorlist` (VOir chapitre précédent).
Attention, si vous vous arrêter ici, votre miroir ne sera pas mis à jour par la suite.

### Tâche Cron pour update votre miroir

Les instructions qui suivent concernent [`cron`](https://github.com/cronie-crond/cronie), lisez le manuel si vous utilisez un autre outil.

Créer une nouvelle tâche planifiée :
```
crontab -e
```
Pour update toutes les heures :
```
00 */1 * * * rsync -av <tier 1 mirror supports Rsync> /var/www/html/arch/
```
Vous pouvez changer `1` en fonction de ce que vous voulez (ex : `3` pour toutes les 3 heures).

Tout devrait être fonctionnel.

### Améliorations et bonus

#### Journalisation :

Je vous recommande de journaliser votre tâche cron pour observer les changements qui ont été faits : 

```
00 */1 * * * rsync -av <tier 1 mirror supports Rsync> /var/www/html/arch/ >> /home/user/log.txt
```
 #### Script pour faciliter la modification :

Voici un exemple de script correpondant à la tâche :

```
#!/bin/bash

rsync -av <tier 1 mirror supports Rsync> /var/www/html/arch/ >> /home/user/log.txt
```
Pour que dans cron on utilise que le script :

```
00 */4 * * * /home/juliiine/rsync_mirror.sh
```
#### Envoyer un mail à chaque update :

Pour cela, vous pouvez utiliser un serveur `postfix`, mais il y a plus simple.
J'utilise [Swaks](https://github.com/jetmore/swaks), un outil utilisant Perl permettant l'envoi de mails, notamment dans des scripts.
Vous avez besoin d'une vraie adresse mail et des informations SMTP de votre fournisseur mail.
J'utilise une adresse OVH pour cet usage.

Un exemple de Swaks (avec les paramètres OVH du serveur sortant) :

```
swaks -t receiver@address.com -s ssl0.ovh.net:587 -tls -au arch@yourdomain.com -ap superpassword -f arch@yourdomain.com --body "The body of your mail" --h-Subject "Planified update is good" --attach /home/user/logofrsyncoutput.log
```

## Contribuer

N'hésitez-pas à faire des remarques, des propositions d'améliorations, ou de l'aide pour traduire le projet dans d'autres langues.
Vous pouvez également m'informer si vous utilisez mon miroir, afin que je puisse l'améliorer en fonction.

## Nos contributeurs

- [Juline](https://github.com/juliiine) : Créatrice initiale du projet
- [Ahraon](https://github.com/Ahraon) : Développeur et traducteur principal du projet 