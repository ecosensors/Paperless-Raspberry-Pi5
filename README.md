# Paperless-Raspberry-Pi5

(That summary is being writting)

Installation of the OpenSource EDM [Paperless-ngx](https://docs.paperless-ngx.com/) on a Rasoberry Pi5 with Docker.

That summary will show you how to

* Install the raspberry
* install Docker
* install Paperless-ngx
* install and configure Samba
* backup your Paperless documents to a remote Synology NAS

Note: I do not have so much experience with Docker and paperless. You might need to adapt the step according to your needs. Feel free to suggest corrections and improvements. I assume you have a minimum experience with Raspberry and command lines.

## Materiel

* Raspberry Pi5
* Raspberry Pi Official cable Micro-HDMI to HDMI
* Keyboard and mouse
* [Raspberry Pi M.2 HAT+] (https://www.raspberrypi.com/products/m2-hat-plus/) with a SSD Disk
* Raspberry power cable
* A Synology NAS or similar

## Usefull commands
```
docker compose exec webserver document_exporter ../export -fpz
docker compose up -d
sudo find ~/ -type f -name "export*"
sudo nano /etc/fstab
systemctl daemon-reload
getent group |grep docker
sudo mount -t cifs -o username=user,password=yourpassword,domain=WORKGROUP //192.168.1.114/paperless /home/pierrot/paperless-ngx/export/
unzip export-2026-01-19.zip -d export-2026-01-19
```

## Installation of the Raspberry

After you have assembled the SSD hat onto the Raspberry Pi, connect an Ethernet cable from your home router to the Ethernet port on your Raspberry Pi. Connect a keyboard and an Micro-HDMI cable to a monitor. Power on your Raspberry Pi and hold down the Shift key.

(In progress)


## Install Docker
```
sudo apt install apt-transport-https ca-certificates curl gpg
```

Add Docker’s GPG Repo Key
```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
```

Add Docker Repository
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/debian trixie stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
```

### Check
```
apt-cache policy | grep docker
```

### Install
```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

That’s all. Docker should now be installed; the service should be started and enabled to run automatically on boot by default. Let’s verify.

```sudo systemctl is-active docker```


Add the user to the docker group
```
sudo useradd pierrot docker
getent group
```

then close and open the terminal
```
docker ps
```

## Install Paperless-ngx

```
bash -c "$(curl --location --silent --show-error https://raw.githubusercontent.com/paperless-ngx/paperless-ngx/main/install-paperless-ngx.sh)"
```

### Very usefull configuration
By default, all scanned documents are stored in paperless-ngx/media/document/original/ from 00000001.pdf to 000000x.pdf.
It's much more useful to customize the folder structure so that if you no longer wish to use Paperless, you can keep your documents in a readable and reusable format.

Edit the file `sudo nano paperless-ngx/docker-compose.env` and add the line at the bottom

```
PAPERLESS_FILENAME_FORMAT={{correspondent}}/{{created_year}}/{{document_type}}/{{title}}
```

You can find other placeholders here : https://docs.paperless-ngx.com/advanced_usage/#file-name-handling if you prefer to have another tree.

Then run the command (you can run it after you have configured the backup, bellow)
```
docker compose up -d
```

From this point, by default, all new documents with me saved /compagny/year/document_type/title.

You can create additional storage paths, if you need to specify different path/tree.

## Install Samba

```
sudo apt install samba samba-common-bin
```
### Configuration

```
sudo nano /etc/samba/smb.conf
```

Add the following

```
[consume-ngx]
path = /home/pierrot/paperless-ngx/consume # change it to your real path
writeable = yes
browseable = yes
public = no
```

Add a Samba user
```
sudo smbpasswd -a <USERNAME>
sudo smbpasswd -a pierrot
```

Restart Samba
```
sudo systemctl restart smbd
```

## Backup on a Synology NFS Share drive

```
sudo apt-get install nfs-kernel-server nfs-common
```

edit paperless-ngx/docker-compose.xml and change the line
```
#- ./export:/usr/src/paperless/export
- export:/usr/src/paperless/export
```

and add the line starting with export, bellow the line redisdata 
```
volumes:
  redisdata:
  export:
    driver: local
    driver_opts:
      type: "nfs"
      o: "addr=192.168.1.114,rw" # Change with your IP address
      device: ":/volume1/paperless" # Change with the path to your share
```

when it's done, you have to run the command

```
docker compose up -d
```

