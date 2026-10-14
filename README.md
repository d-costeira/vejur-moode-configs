# vejur-moode-configs
saved configurations and radio stations for vejur, moode audio player
installed on Raspberry Pi 4B+

https://github.com/moode-player/moode/blob/master/www/setup.txt
https://www.headphonesty.com/2021/09/introduction-to-moode-audio/

Resolution to double song meme:
https://moodeaudio.org/forum/showthread.php?tid=2948



## Flash the SD card, do the following:
* go to the Raspberry Pi website and download the Raspberry Pi Imager software
		https://www.raspberrypi.com/software/
* install Moode onto the SD card via the Raspberry Pi Imager
  * it's just easier


## CONFIGURE WIFI CONNECTION
NOTE - Not necessary if you used the Raspberry Pi Imager and configured Wifi at that step, it should automatically connect to your wifi network
* connect to the Moode wifi (password: moodeaudio)
* on a browser go to `http://moode.local`
> m > Configure > Network > Wireless (WiFi)
* Enter the wifi network + password it should connect to
* SAVE


## VERIFY SSH ACCESS
If you did not enable SSH when flashing Moode with the Raspberry Pi Imager, then enable it with these commands:
> m > Configure > System > Local Services
* Turn SSH term server ON
* SSH into the device using
```
ssh [user]@vejur.local   or
ssh [user]@moode.local
```
* replace `[user]` with the username you configured in Raspberry Pi Imager, default is `pi`
* password is `moodeaudio` (or whatever you set it to in Raspberry Pi imager)
* run the following commands to update the device software:
```
sudo apt update && sudo apt full-upgrade -y
```


## INSTALL AND CONFIGURE ARGON ONE SCRIPT
SSH into the device and run the following command:
```
curl https://download.argon40.com/argon1.sh | bash
(note: the | symbol is ALT+124)
```
* Run the command `argonone-config` to change the fan configuration.
	(Recommend setting fan to 40% at 55C, 70% at 60C, 100% at 65C)
* Recommend running the following commands to change Power Button behavior:
```
sudo nano /etc/argon/argonpowerbutton.py
```
(file may vary - we want the file that controls power button behavior)

* Find the line that begins with `def argonpowerbutton_monitorevent`
* Change the IF statement for `os.system("shutdown now -h")` to read:
```
                elif pulsetime >=4 and pulsetime <=10:
                        os.system("shutdown now -h")
```
  (you may have to change the following lines too if there are more conditional statements)
* CTRL-X then Y then ENTER to save changes


## INSTALL IR REMOTE SCRIPT FOR NANOSOUND ONE CASE
Okay look this script is an outdated mess so we'll have to make significant adjustments
* Check that you have the correct dependencies installed
```
python3 -c "import RPi.GPIO, mpd; print('ok')"
```
* If it returns "ok", you're ok - if it doesn't run this command: `sudo apt install -y python3-mpd python3-rpi.gpio`
* Run the following commands:
```
git clone https://github.com/nanomesher/Nanomesher_NanoSound_moode.git
cd Nanomesher_NanoSound_moode
```
* Run the following commands to make necessary changes to the IR Remote script:
```
	# change script to run on python 3 instead of python 2
sed -i -e 's|^#! */usr/bin/python$|#!/usr/bin/python3|' -e 's/^import ConfigParser$/import configparser as ConfigParser/' nanosound_moode/nanodac_lirc.py

	# change script to use the correct HOME directory instead of the default /home/pi
sed -i "s|/home/pi|$HOME|g" nanosound_lirc.service nanosound_moode/nanodac_lirc.py

	# change script to use the correct audio volume control script location - /var/www/vol.sh is outdated
sed -i 's|/var/www/vol.sh|/var/www/util/vol.sh|g' nanosound_moode/nanodac_lirc.py

	# make the IR Remote script actually executable
chmod +x nanosound_moode/nanodac_lirc.py
```
* Run the following command to make the service script start after `mpd.service` and restart after 10 seconds if it fails:
```
sudo sed -i -e 's/^After=mpd$/After=mpd.service/' -e 's/^StandardOutput=null$/Restart=on-failure\nRestartSec=10/' nanosound_lirc.service
```
* !! DO NOT RUN THE `install.sh` SCRIPT !! The installation will be fucked.  Matter of fact:
```
rm install.sh
```
* Run the following commands to install and start the IR Remote Service:
```
sudo cp nanosound_lirc.service /lib/systemd/system/
sudo systemctl daemon-reload && sudo systemctl enable --now nanosound_lirc
```
* Test that the service is running:
```
systemctl status nanosound_lirc
```
After you configure Moode and are able to play music, test the IR Remote included with the NanoSound ONE case, reboot the device, and verify the service still works after restarting.


## UPDATE KEY SETTINGS

> m > Configure > System

> General
* Change host name and Browser Title to "vejur" and "vejur"

> System Modifications
* Set Worker Response to 'Boosted'
* Set CPU Governor to 'Performance'
* Turn HDMI Port OFF
* Expand File System

> Local Display
* configure local display or turn everything off

> m > Configure > Audio > MPD Options > Edit MPD settings
* Set the Output Device to USB to use with a DAC
* Set the Volume Type to "Fixed (0db output)"
* Set "Audio Buffer" to 64Mb
* Set "Output Buffer Size" to 512Mb
* Set "Input Cache" to some value idk
* SAVE

> m > Configure > Audio > MPD options > General
* Set "Autoplay after Start" to ON


## CHANGE APPEARNACE
> m > Preferences > Appearance
* Set "Theme" to Default
* Set "Alpha Blend" to 0.50
* Set "Cover Backdrop" to YES
* Set "Cover Scale" to 1.25
* Set "Adaptive Coloring" to YES

> m > Preferences > Playback
* Set "Now-playing icon" to WAVEFORM
* Set "Search Site" to DISABLED

> m > Preferences > Cover Art
* Set "Search Priority" to EMBEDDED
* Set "Audio formats" to ALL
* Set "Thumbnail Resolution" to 400px
* Set "Tag view thumbs" to NO

> m > Preferences > Library
* Set "One Touch Album" to SHOW TRACKS
* Set "Album View Sort" to ARTIST/YEAR
* SET "Genre Column" to COMPOSER
* Set "Show Sample Rate" to HD ONLY
* Set "Thumbnail Columns" to 7/3
* Set "Comment Tag" to NO


## CONFIGURE RADIOS
in the Now Playing screen, click the album art, then select "Radio" in the top left drop menu
* click the gear icon to go into settings
*  Set "Sort tab" to NAME
*  Set "Group method" to SORT TAG
*  SAVE

Manually add missing radio stations


## OPTIONAL - CONFIGURE NETWORK LIBRARY
> m > Configure > Library > Music Sources
* click CREATE
* Type SMB
* SCAN for paths, then select WINTERMUTE/Music/Music
  * userID: guest
  * password: [none]
* SAVE

> m > Configure > Library
* Set "Automatically update the MPD database on USB insert or remove" to NO
* Set "Ignore CUE Files" to YES

> m > Update Library


## OPTIONAL - SET UP SYNCED FOLDER FROM WINTERMUTE
* SSH into vejur
* create the folder where WINTERMUTE will be mounted
* run the following command
```
sudo mkdir /mnt/wintermute_music
```
* set it to auto-mount WINTERMUTE on boot
* run the following command
```
sudo nano /etc/fstab
```
* add the following line:
```
//wintermute.local/Music/Music /mnt/wintermute_music cifs guest,x-systemd.automount,iocharset=utf8,ro,vers=2.1 0 0
```
* CTRL-X then Y then ENTER
* run the following command to create the sync script:
```
sudo nano ~/wintermute_boot_sync.sh
```
* copy the contents in the `wintermute_boot_sync.txt file`
* CTRL-X then Y then ENTER
* run the following command to make the script executable
```
sudo chmod +x ~/wintermute_boot_sync.sh
```
* run the following command to open the list of scripts ran at boot
```
sudo nano /etc/rc.local
```
* add the following line above 'exit 0'
	note: '&' at the end makes it run in the background
```
# sync with Wintermute
/home/vejur/wintermute_boot_sync.sh &
```
* CTRL-X then Y then ENTER

* NOTE: don't forget to regenerate the music database
> m > Configure > Library > Music Database
> REGENERATE Music Database

## SET UP SPOTIFY
> m > Configure > Audio > Audio Renderers > Spotify Renderer
* Set Spotify to ON and change the name to Vejur Spotify
* Set Resume MPD to "Yes"


## SET UP BLUETOOTH
> m > Configure > Audio > Audio Renderers
* This is where you can do the memes.
	
* -- COME BACK TO THIS LATER
