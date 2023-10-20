# Create a Debian-based VM machine hosting a Minecraft Server

It is possible to leverage an e2-medium machine of the Google Cloud machine pool to host a Debian-based 10GB boot disk, 2 vCPU, and 4GB RAM. 

## Goals of the project
1. Create an application server
2. Install software and dependency
3. Configure networking features
4. Finalize a backup cronjob
5. Schedule shutdown and startup script


## 1. Create an application server - VM constraints
- Name: `mc-server`
- Region and zone: custom
- Boot disk: `Debian GNU/Linux 11 (bullseye)`
- Access scopes for Storage: R/W
- Persistent disk: `50 GB SSD Persistent Disk` 
- Encryption type: `Google-managed` 
- Network tag: `minecraft-server`
- External IP Address: `static`

## 2. Install software and dependency

```shell
sudo apt-get update
sudo apt-get install -y default-jre-headless wget screen
```

### - Prepare the persistent disk

Enable the SSH connection through the VM and set up a new folder as a mount point.

```shell
sudo mkdir -p /home/minecraft
```
Format the disk.

```shell
sudo mkfs.ext4 -F -E lazy_itable_init=0, lazy_journal_init=0, discard /dev/disk/by-id/google-minecraft-disk
```

Mount the disk in the folder created above.

```shell
sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft
```

### - Install software

Download the up-to-date Minecraft server. 

```shell
sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar
```
Try to start your server.

```shell
sudo java -Xmx1024M -Xms1024M -jar server.jar nogui
```
But it won't run unless you set `eula=true` in the following file

```shell
sudo nano eula.txt
```
Finally, to start your Minecraft server you can type.
```shell
sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui
```
If you want to leave this process in the background you can press CTRL+A and CTRL+D. 

## 3. Configure networking features

In order to enable external traffic it is mandatory to set a new firewall rule from the Firewall tab in the VPC network menu. Then, create a new Firewall rule. 

- Name: `minecraft-rule`
- Target: `Specified target tags`
- Target tags: `minecraft-server`
- Source filter: `IPv4 ranges`
- Source IPv4 ranges: `0.0.0.0/0`
- Protocols and ports: `Specified protocols and ports`
- Protocol: `TCP`
- Port: `25565`

### - Exploiting the Minecraft Server Status to test the application
Visit https://mcsrvstat.us/ and test your server. 

1. Go to Compute Engine > VM instances and note the `External IP Address` of your VM server.
 

2. Paste your External IP Address on the web page. 
3. Click on `Get server status`. 

## 4. Finalize a backup cronjob

Set a persistent variable name for your bucket. 

```shell
export YOUR_BUCKET_NAME=<Enter your bucket name here>

echo YOUR_BUCKET_NAME=$YOUR_BUCKET_NAME >> ~/.profile
```

Then, create the bucket via Cloud SDK
```shell
gcloud storage buckets create gs://$YOUR_BUCKET_NAME-minecraft-backup
```

Copy the file `backup.sh` into `/home/minecraft/backup.sh`. Then, make the script executable. 

```shell
sudo chmod 755 /home/minecraft/backup.sh
```

Now, we can create the crontab. 
```shell
sudo crontab -e
```
Type the number of the nano editor and append at the bottom of the file the following: 

```shell
0 */4 * * * /home/minecraft/backup.sh
```

This line set up a cronjob to run backups every 4 hours.

## 5. Schedule shutdown and startup script

Instead of manually going through the process of mounting the persistent disk and initiating the server application within a screen, you have the option to streamline this by utilizing metadata scripts. These scripts will automatically generate a startup script and a shutdown script, taking care of these tasks on your behalf.

Click `edit` on your VM instances and looking for the `Metadata` section. Then, click `Add Item`

- startup-script-url: `https://storage.googleapis.com/cloud-training/archinfra/mcserver/startup.sh`
- shutdown-script-url: `https://storage.googleapis.com/cloud-training/archinfra/mcserver/shutdown.sh`