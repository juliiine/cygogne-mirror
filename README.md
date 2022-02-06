# Cygogne Gentoo Mirror

This repo describes how to set my mirror in your Gentoo system, and also show how to create a mirror.
Specifics things to know about my mirror : 

- 1Gbits bandwith fiber edition.
- 2 Tb stockage allowed (actually uses only 480Gb).
- 2 CPU / 1G RAM (it's sooo fine).
- Refresh every hour from Master Gentoo Mirror.
- HTTPS only.
- No-log policies, I don't care what you emerges.
- Uses Debian 11.
- 100% French and Weeb powered (maybe not so cool).
- Cygogne-it is not a cool name, I already know that. 


 Please don't hesitate to tell me if you use my mirror, so I can improve it if there is more and more users.
 You can also contribute to modify the `readme.md` file, translate it or make me any feedback and suggestions.

## Set Cygogne-IT to default mirror for portage

It's pretty simple because you only need to edit one file. 
Take your favorite editor and edit the `make.conf` located in `/etc/portage/` :
```
nano /etc/portage/make.conf
```
Normally, you shouldn't have a `GENTOO_MIRRORS="something"` line in your file.
If you already have it, it's ok.
Otherwise it must be added, the placement does not matter but it must be placed at the beginning of the line :
```
GENTOO_MIRRORS="something"
```
After you have the line required, you can change the value to the `GENTOO_MIRRORS` variable :
Change `something` to `https://cygogne-it.fr` and you should have this :
```
GENTOO_MIRRORS="https://cygogne-it.fr"
```
Save your `make.conf` file and exit.

The mirror is now set to here, and will be used for any download from portage now.
You can run `eix-sync` (if you have `eix` installed) or `emerge --sync` to refresh packages.

If you want to set your own mirror, please read below.

## Set up your own mirror

Please keep in mind that kind of "personnal mirrors" uses bandwith from Tier 1 mirror, and I don't encourage anyone to set a mirror like this one for the fun, or only for your usage. If there is not a lot of Gentoo mirrors in your area, there is no problem to create one, and it's recommended to submit your mirror to the Gentoo team. 

If you still want to continue even this warning ...

### Requirements

- Bandwith : You need to have a minimum of 5 MBit/s upstream capacity on a permanent connection, higher is better.
- Storage : Hosting a mirror requires a minimum of 550 GiB available disk space.
- HTTP and/or HTTPS access to the site. 
- I assume that you already have a web server in place.
- The Gentoo team recommends to sync from master every four hours.
- `rsync`
- `cron` or any task scheduler.
- A Gentoo system to test your mirror.
- A loooot of time (depends of your connection)

### Initial fetch

This part is maybe the hardest and takes a lot of time.
Please choose a location where the mirror will be stocked. 
Often, it's `/var/www/` or `/var/www/html/` with an `apache` setup.
In my case, I've choosed `/var/www/html/gentoo/`.

I also recommend to create a `VirtualHost` to redirect to my Gentoo directory.
It's useful when you have multiple website already installed on your server.
Read your web server manual to know how to set `VirtualHost`.

When your desired directory is configured and accessible, it's time to download EVERYTHING from Gentoo Master Mirror.
Actually, it takes up to 480Gb to download, so please be patient.

To fetch from Gentoo Master Mirror to your desired directory :
```
rsync -av rsync://masterdistfiles.gentoo.org/gentoo/ /var/www/html/gentoo/
```
Change `/var/www/html/gentoo/` to the location you have already choosed.
The `-v` argument can be skipped if you don't want verbose mode.

After this huge download, your mirror can be now set in `make.conf` file (see previous chapter).
Be careful, if you stop right here, your mirror will be never updated.

### Cron setup to refresh your mirror

The instruction below is for [`cron`](https://github.com/cronie-crond/cronie) usage, please refer to the manual if tou use an another tool.

Create a new task in crontab :
```
crontab -e
```
To refresh every 4 hours :
```
00 */4 * * * rsync -av rsync://masterdistfiles.gentoo.org/gentoo/ /var/www/html/gentoo/
```
You change change `4` to what you want (ex : `1` to refresh every hour).

Everything should be working now.

### Enhancement and bonus

#### Log :

I recommend to log your rsync task, to see what have been updated since last time : 

```
00 */4 * * * rsync -av rsync://masterdistfiles.gentoo.org/gentoo/ /var/www/html/gentoo/ >> /home/user/log.txt
```
 #### Script for easy call :

I've created a script to run simple line in my cron task :

```
#!/bin/bash

rsync -av rsync://masterdistfiles.gentoo.org/gentoo/ /var/www/html/gentoo/ >> /home/user/log.txt
```
to have in cron :

```
00 */4 * * * /home/juliiine/rsync_mirror.sh
```
#### Send a mail every time refresh is terminated :

For this, you can use a `postfix` server, but honestly there is moooore simple.
I use [Swaks](https://github.com/jetmore/swaks), a perl-based tool to send simple mail in things like script.
You need an real email address and the SMTP configuration from your provider.
I use an OVH email address to send me mail in my script.

An exemple of usage of Swaks (with OVH out server settings) :

```
swaks -t receiver@address.com -s ssl0.ovh.net:587 -tls -au gentoo@yourdomain.com -ap superpassword -f gentoo@yourdomain.com --body "The body of your mail" --h-Subject "Planified update is good" --attach /home/user/logofrsyncoutput.log
```

## Contribute

Feel free to fill an issue if you have suggestions about this page, or if you want to translate it in other languages.
I also like to know if you use this mirror as a daily driver to see how it can be improved.
