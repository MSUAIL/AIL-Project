# Documentation

Historically I am not great at documenting my work but I'll try to improve here. 
Some times I will be more verbose / accurate with my work, other times I may just post a URL to a guide I referenced. Best of luck. 

### Setup

#### Initial setup

Got a RPi 4B as the host system

Using the Raspberry Pi Imager on a separate device, I flashed the 128Gb MicroSD with Raspberry Pi OS 64bit (Full) (Debian Bookworm)

* Likely could have gotten away with a smaller OS but I like *some* of the features the Full installation comes with. 
* Set up with a Device name / password, as well as a user name / password. Did not change any default networking settings in the boot loader.
* Took a bit over a hour to flash

Inserted the MicroSD into the Pi and logged in

#### Altering the user structure

The default user, "u_senseme", was initially a sudo user. I set up Z-wave with this sudo user. Then I removed u_senseme from the sudoers file:

removed it from the sudo group using user `root`: 

```bash
userdel u_senseme sudo
```

I've added a new **group** called `admin` (which is defined in /etc/sudoers to have the same permissions as %sudo for now)

I then added a user `u_admin` using:
```bash
useradd u_admin`
```
and assigned u_admin to groups with:
```bash
usermod -aG admin u_admin 
usermod -aG users u_admin` # this too...
usermod -aG sambashare u_admin` # I set up a simple samba sharing server to add and remove files to the pi easier... it doesnt work. *Will Uninstall*
usermod -aG adm u_admin` # for access to some extra log files
```

Then I had to alter some of the files in the /etc/sudoers.d directory.

After checking the contents of each file, I realized which one was keeping u_senseme an elevated user.

After cd-ing into this sudoers.d directory, I used:
```bash
mv 010_pi-nopasswd 010_pi-nopasswd~
``` 
to prevent that file from being read but without deleting it. (As the documentation says to do)

#### Installing Z-Wave-js-ui

Roughly followed the https://zwave-js.github.io/zwave-js-ui guide for setting up Z-Wave
    * Installed a packaged version of Z-Wave (Listed under other options) -- Not very comfortable with docker... it changes too many firewall settings for my little brain to comprehend.


Make a home directory for the new app
```bash
mkdir ~/zwave-js-ui
```
Enter it
```bash
cd ~/zwave-js-ui
```
Pull the latest package from Github
```bash
# download latest version
curl -s https://api.github.com/repos/zwave-js/zwave-js-ui/releases/latest  \
| grep "browser_download_url.*zip" \
| cut -d : -f 2,3 \
| tr -d \" \
| wget -i -
```
Now if you `ls` you will see four .zip files... Only need one (the guide unzips them all?)
```bash
unzip zwave-js-ui-v*arm64.zip
```

Now you can 'manually' host the Z-Wave server with:
```bash
cd ~/zwave-js-ui
./zwave-js-ui
```

The instance should be accessible by accessing this url in your browser:
```
http://localhost:8091
```

##### Setting up the zwave service

**From the u_admin user**

```bash
sudo vim /etc/systemd/system/zwave.service
```
and entered this data:

```conf
[Unit]
Description=Z-Wave JS custom service
After=network.target

[Service]
Type=simple
User=u_senseme
WorkingDirectory=/home/u_senseme/zwave-js-ui
ExecStart=/home/u_senseme/zwave-js-ui/zwave-js-ui

[Install]
WantedBy=multi-user.target
```
Save and exit from editor

ran these commands to refresh systemd and enable the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable zwave.service
sudo systemctl start zwave.service
sudo systemctl status zwave.serivce
```

Now, zwave is a systemd controlled process. It will start on boot, and can be stopped / restarted with `systemctl`

to read the logs:

```bash
sudo journalctl -u zwave.service
```

##### Errors I encountered:

* Once I launched Z-Wave then restarted, I was getting an error preventing the app from launching. I checked the log files and found a message that the instance was failing to connect to `/dev/zwave`.  

    * I knew this was an odd device and checked the `/dev/zwave` folder. Sure enough no zwave device. 
    * I then used `lsusb` to verify that the controller was being seen. 
    * Used `ls /dev/` (with the controller plugged in)
    * Used `ls /dev/` (with the controller disconnected)
    * Found the different device: `/dev/ttyACM0`
* Navigated to the `~/zwave-js-ui/store` and found a `settings.json` file which seemed to have relevant contents
* Made a copy with `cp settings.json settings.json.bkp`
* It is a .JSON file so its ugly.... but inside was a setting zwave > port : `/dev/zwave`. I changed the value from `/dev/zwave` to `/dev/ttyACM0` and saved the file.
*relaunched the instance and got back to the dashboard

**Root Cause**

Was running zwave from the wrong directory. need to be in the zwave-js-ui folder before running the executable. 


#### Installing Home Assistant Core

Once again, not a fan of using docker and I didn't install the Home Assistant OS (which might have been easier, but I'd rather suffer)

* Followed this guide pretty closely:
    * https://www.home-assistant.io/installation/raspberrypi/#install-home-assistant-core



#### Installing mosquitto ? 

TBD


### Connecting Devices