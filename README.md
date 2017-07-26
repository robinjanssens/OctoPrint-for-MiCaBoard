# Running OctoPrint on the MiCa7688 MEGA

## Create Overlay

If you want to run OctoPrint you will need more storage than is provided so we need to enlarge the storage space by creating an overlay onto a microSD card. If you already done this you can skip this part.

After inserting the SD card you can check `dmesg` to make sure it is detected correctly. (optional)

Mount the SD card `mount /dev/mmcblk0p1 /mnt`

Check the mounting by typing `df`<br>
You should see a line like `/dev/mmcblk0p1   <blocks>   <used>   <available>  <use%> /mnt`

copy existing overlay to the mounted SD card `tar -C /overlay -cvf - . | tar -C /mnt -xf -`

Unmount SD card `umount /mnt`

Create fstab<br>
`block detect > /etc/config/fstab; \
   sed -i s/option$'\t'enabled$'\t'\'0\'/option$'\t'enabled$'\t'\'1\'/ /etc/config/fstab; \
   sed -i s#/mnt/mmcblk0p1#/overlay# /etc/config/fstab; \
   cat /etc/config/fstab;`

Reboot `reboot`

Now check if the overlay is correctly mounted by typing `df`.<br>
It should look like this `overlayfs:/overlay   <blocks>   <used>   <available>  <used>% /` with the same values as `/dev/mmcblk0p1   <blocks>   <used>   <available>  <use%> /overlay`.

You now have extended the storage capacity of your board.

## Installing OctoPrint

We need to install Python first.<br>
`opkg update`<br>
`opkg install python python-pip unzip`<br>
`pip install --upgrade setuptools`

We also need to install the certificate tools to be able to clone or download from github.<br>
`opkg install ca-bundle ca-certificates`

Expand /tmp folder by moving it to the overlay.<br>
`mkdir /overlay/tmp`<br>
`rm -rf /overlay/tmp/*`<br>
`cp -a /tmp/* /overlay/tmp/`<br>
`umount -l /tmp`<br>
`[ $? -ne 0 ] && {`<br>
`umount -l /tmp`<br>
`}`<br>
`mount /overlay/tmp/ /tmp`

Now we are going to download and install OctoPrint.

Go to the desired install location. I chose `cd /usr/share`.

Clone this git repo `git clone git@github.com:robinjanssens/OctoPrint-for-MiCaBoard.git`.

Enter the directory `cd OctoPrint-for-MiCaBoard`.

Install requirements `pip install -r requirements.txt`.

We need to make the run file executable `chmod 755 ./run`.

Test OctoPrint `./run`.

OctoPrint is now running on port 5000. Go to you web browser and check if it is running properly.

## Make OctoPrint run on startup

Create a symlink 'octoprint' that links to the run script `ln -s /usr/share/OctoPrint-for-MiCaBoard/run /usr/bin/octoprint`.

You can choose the port to run the OctoPrint web interface on by using the following parameter `--port=8080`. Keep in mind port 80 is already running the connection web interface.

Add OctoPrint to rc.local to start it at startup.
```
# Put your custom commands here that should be executed once
# the system init finished. By default this file does nothing.

octoprint --port=8080 &

exit 0
```

To prevent another restart you can run OctoPrint by typing `octoprint --port=8080 &`

## Program the Arduino firmware

First you need to download and configure your firmware e.g.: [Marlin](https://github.com/MarlinFirmware/Marlin).

Add `https://raw.githubusercontent.com/MiCa-boards/MiCa7688_Arduino/master/package_micaboards_index.json` to `File > Preferences > Additional Boards Manager URLs`

Go to `Tools > Board:  > Boards Manager...` at the bottom you will see `MiCa AVR Boards` press "Install".

Now at the bottom of the `Tools > Board:` menu there will be the MiCa Boards listed.

Choose the `MiCa7688 Mega`.

If your board is running on the same network as your computer you will see the IP address appear in the `Tools > Port` menu.

As everything went as planned you can now press `Upload` and flash the firmware over the air.

## Program hex file manually

If OTA programming doesn't work via the Arduino IDE you can program manually via SSH.

Export hex file in arduino<br>
`File > Preferences > check 'Show verbose output during compilation and upload'`

Compile and look for the path to the hex file in the compilation output.<br>
On Windows I had something like this
`C:\Users\<yourname>\AppData\Local\Temp\arduino_build_15087\Marlin.ino.hex`.

Send hex file to MiCa board.<br>
`scp C:\Users\<yourname>\AppData\Local\Temp\arduino_build_15087\Marlin.ino.hex root@<ipadress>:~/marlin.hex`

Login and program hex file.<br>
`ssh root@<ipaddress>`<br>
`/usr/bin/run-avrdude ~/marlin.hex -v -patmega2560 -cstk500v2`

## References

- [https://github.com/foosel/OctoPrint](https://github.com/foosel/OctoPrint)
- [https://github.com/MiCa-boards](https://github.com/MiCa-boards)
- [https://wiki.openwrt.org/doc/howto/extroot](https://wiki.openwrt.org/doc/howto/extroot)
- [https://community.onion.io/topic/1569/octoprint-3d-print-server-on-omega2](https://community.onion.io/topic/1569/octoprint-3d-print-server-on-omega2)
