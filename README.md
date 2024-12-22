# Working with Raspberry PI OS

## Installing OS on SD Card
* Install Raspberry PI OS Imager from https://www.raspberrypi.com/software/.
* Plug the memory card with the computer.  
  NOTE: Don't mistakenly plug the memory card with Raspberry PI and try to install the OS.
* Run the Imager program.
* From Imager settings, enable and enter hostname, SSH, Wifi, eventually configure everything.  
   Note: if you already have another active PI device, then make sure hostname are different.

 
**How to choose the correct version of OS**
- If you use the OS headless (i.e. without monitor), then install the *lite* version. Otherwise Recommended version is good to go.
- If you want a bootable memory card to use in both 32bit and 64bit Raspberry pi devices, then you have to install 32bit version of OS.


## How to connect and work with Raspberry PI from the computer
* To enable headless (aka. command-line/remote/SSH) access, after installing the OS, create an empty file `ssh` in the root directory of the memory card.
* Plug the memory card to Raspberry PI.
* Connect to the power either using a 5v charger or to the computer usb.
* Wait for a couple of minutes to load the OS. It will automatically connect to the WIFI. Note: to change or add WIFI information, check **Usefull Notes** below.
* Go to the Router admin panel and get the IP address of the RPI to use as the hostname.
* Putty can be used to access the RPI from command line or VSCode has extensions for that. Install entire **Remote Development** suite VSCode extensions (It will install a set of extensions. By Microsoft) to access and to do development in Raspberry PI. Also the extension **SSH-FS** is a useful one.
* Start creating a new remote connection on the extension.
* When prompted for ssh user@hostname, enter in that format
  `ssh -p 22 <os_username>@<rpi_ip_address>`.


## Update and upgrade Linux package information
First things first:
```shell
sudo apt-get update
sudo apt-get upgrade
```

`sudo` gives root privilege.


## Install Git
```shell
sudo apt install git
```

Enter the command below to verify the installation:
```
git --version
```



## Install Node.js

Install Node.js from the NodeSource Repository, a third party service which resolves the installation process.

```
curl -sL https://deb.nodesource.com/setup_<version>.x | sudo bash -
```

```
sudo apt-get install -y nodejs
```
Go to https://deb.nodesource.com/ for command with latest version.

> NOTE  
> * Try avoiding the process of downloading installer, extracting etc. For new Linux users, it can be a mess.
> * ARMv6 processor is not supported by NodeSource; even for Node-v10.


### NodeJS Install Steps for ARMv61 processor (RPi Zero W)

1. Go to the _unoffical_ builds download page of Node.js site and select the version of NodeJS you want to install. Copy the link of the ARMv6 version of NodeJS (Select the link with the .xz extension).

1. Download the file `wget <link>`. Example  
  ```sh
  wget https://unofficial-builds.nodejs.org/download/release/v14.13.0/node-v14.13.0-linux-armv6l.tar.xz
  ```

1. Extract the binary from the tarball file using the following command `tar xvfJ <file_name.tar.xz>`. Example  
  ```sh
  tar xvfJ node-v14.13.0-linux-armv6l.tar.xz
  ```

1. Copy the contents of the extracted tarball file to the `usr/local` directory: `sudo cp -R <extracted tar folder>/* /usr/local`. Example:  
  ```sh
  sudo cp -R node-v14.13.0-linux-armv6l/* /usr/local
  ```

1. Reboot and check everything is working correctly `node -v && npm -v`.

**Extra steps**

1. You might have to do the following if you receive a *command not found* error.
  First, open the `.profile` file using nano: `sudo nano ~/.profile`

1. Add the following line to the end of the file and hit Ctrl+X to save, and then hit 'y' and enter to confirm the changes: `PATH=$PATH:/usr/local/bin`.

1. Reboot and check is it working.


**NOTE**: If running `node` command shows *exec format error* then undo the previous steps and try again. **Make sure** to add `PATH` variable to `.profile` and then reboot.


### Command to install Python package installer (pip3) and python packages
   ```sh
   sudo apt install python3-pip
   ```

   ```sh
   pip3 install smbus2
   ```

## Execute certain command on boot
* Edit `/etc/rc.local` with root permission and commands before `exit` command.

Example
```
   node /home/pi/projects/raspberry-pi-projects/home-iot/app.js &
```

To know about the ending `&`, check *How to run a process in the background* in this doc.


## Restart network or the OS when connection is lost
* Create a shell script `/usr/local/bin/checkwifi.sh`.
   ```sh
   ping -c4 <router_ipv4> > /dev/null
 
   if [ $? != 0 ]
   then
     echo "No network connection, restarting wlan0"
     /sbin/ifdown 'wlan0'
     sleep 5
     /sbin/ifup --force 'wlan0'
   fi
   ```

   * Open the crontab editor by typing:
   ```sh
   crontab -e
   ```

   Add the following line:
   ```sh
   */5 * * * * /usr/bin/sudo -H /usr/local/bin/checkwifi.sh >> /dev/null 2>&1
   ```
   This will run the script in *checkwifi.sh* every 5 minutes, writing its output to `/dev/null` so it won't clog your syslog.

   * To reboot the PI, write the following script instead in *checkwifi.sh*:
   ```sh
   ping -c4 <router_ipv4> > /dev/null
    
   if [ $? != 0 ]
   then
     sudo /sbin/shutdown -r now
   fi
   ```


## Enable the 1-Wire Interface
*(Courtesy: circuitbasics.com)*  
We’ll need to enable the One-Wire interface before the Pi can receive data from the sensor. Once you’ve connected the DS18B20, power up your Pi and log in, then follow these steps to enable the One-Wire interface:

* At the command prompt, enter
```sh
sudo nano /boot/firmware/config.txt
```
then add this text to the bottom of the file:
```sh
dtoverlay=w1-gpio
```
Note: If any `dtoverlay=` is already there, doesn't matter; add one more at bottom.

* Exit *nano*, and reboot the Pi with `sudo reboot`.

* Log in to the Pi again, and at the command prompt enter `sudo modprobe w1-gpio`. Then enter `sudo modprobe w1-therm`.
  > * Modules are pieces of code which extend the functionality of the operating system kernel without the need to reboot. Once loaded, modules reside in memory. 
  > * To remove the module use `sudo modprove -r <module_name>`

* Change directories to the `/sys/bus/w1/devices` directory.

* Enter `ls` to list the devices. Something like `28-xxxxxxxxxxx w1_bus_master1` will be displayed.

* Go to `28-xxxxxxxxxxx` directory and type `cat w1_slave` to see the temperature value from sensor.


### [Extra] Install LocalTunnel (proxy tool)
1. Install localtunnel globally to use localtunnel command `lt` from anywhere
  ```cmd
  npm install -g localtunnel
  ```
  If not installed globally then full path to *lt* executable have to be used, which will be something like `/home/pi/.../mode_modules/.bin/lt`.
  
2. Forward a local port specifying a subdomain:
  ```cmd
  lt --subdomain <subdomain_name> --port <port_number>
  # example
  lt --subdomain my-unique-subdomain-name --port 8080
  ```



## TroubleShooting
* **Trouble**: Problem connecting to RPI with VSCode remote explorer with previous ssh config.  
  **Shoot**: Remove the previous ssh SHA1 string from the computer. On Windows, it is in `os drive/users/<username>/.ssh/known_hosts`.
  
* **Trouble** Where to look for system logs, crash/error/warning logs:  
  **Shoot**:
  - The command `dmesg` will return most of the activity of the current boot. `dmesg` returns every event after the boot and how long after the boot, in seconds.
  - The files `/var/log/messages` `/var/log/syslog` and `/var/log/kern.log` will return pretty much every event you could ever need to know to figure out what happened.



## Useful Notes
* Use `node` command to add a node.js app on device startup. Running with `npm` command will run an extra `npm` process.

* Installing Node.js with NodeSource will resolve the *ARM* architecture and other compatibilities and downloads the correct version of the Node.js distribution. So Node.js installed with one model Raspberry PI may not run on other model(s). Like RPI3 is ARMv71, so Node.js ARMv71 distribution has to be installed to run on that machine. But after that this installation will not run on RPI Zero W, which is ARMv61. ARMv61 distribution of Node.js has to be installed too.

* To change or add WIFI information diretly from the memory card: Wifi network information saved in `/etc/wpa_supplicant/wpa_supplicant.conf`.
   To add new network information, add following block of code at the end of the file:
   ```conf
   network={
      ssid="<ssid_name>"
      psk="<plain_text_password>"
   }
   ```
   > IMPORTANT
   > *Don't use apostroph (') in wifi passwork.*


## Useful Linux terminal commands

* Update & Upgrade PI OS
   ```
   sudo apt-get update
   sudo apt-get upgrade
   ```

   If problem occurred on upgrade, run this:
   ```
   sudo apt-get upgrade --fix-missing
   ```
   
   - Firmware update command `sudo rpi-eeprom-update` only available in RPi-4.
 
 
* List current directory
   ```
   ls
   ```
* Make directory
   ```
   mkdir <dir_name>
   ```
* Removing files and directories
  
  - Remove non-empty directory:
   ```
   rm -r <dir_name>
   ```
   
  - Remove all files of current directory, with recursive (-r) and force (-f) arguments:
   ```
   rm -r -f *.*
   ```
   
   - To delete or skip one by one, type the following:
   ```sh
   rm -i mydir/*
   ```
   After each file name displays, type `y` and press Enter to delete the file. Or to keep the file, just press Enter.
   
   
* Size of current directory  
   `-s` to display only the total size, `-h` to display sizes in a human-readable format.
   ```
   sudo du -sh
   ```
   Specific directory
   ```
   sudo du -sh /home/pi
   ```

* Open a file in terminal with *root* permission:
   ```
   sudo nano <file_name>
   ```

* Shutdown, reboot etc
   ```
   sudo shutdown -h now
   ```
   ```
   sudo reboot
   ```

* List running processes
   All processes
   ```
   ps -e
   ```

   Filtering processes
   ```
   ps -e | grep <partial_process_name>
   ```
   Example:
   ```sh
   ps -e | grep node
   ```

   Real-time process listing by CPU usage
   ```
   top
   ```

* How to run a process in the background, permanently
   ```
   nohup <command_and_arguments> &
   ```
   `nohup` is short for "no hang-up". Ending '&' will run the command in the background.


* Kill a process:
  ```sh
  kill -9 <process_id>
  # or
  kill -9 <process_name>
  ```
  - `-9` denotes SIGKILL; to see full list of *kill* parameters, use `kill -l`


* Top 10 processes sorted by CPU usage. This will count the header line as one record, that's why `-11`
   ```sh
   ps -eo pmem,time,stat,pid,pcpu,command --sort -pcpu | head -11
   ```

* Memory status
   ```sh
   free -h
   ```

* Run Tailscale vpn
   ```
   sudo tailscale up
   ```
  - Seems like Tailscale doesn't support ARMv61 process (i.e. RPi Zero W)


* Raspberry PI OS configurations
   ```
   sudo rasp-config
   ```

* GPU temperature
   ```sh
   vcgencmd measure_temp
   ```

   Regular expression to get only the value:
   ```sh
   vcgencmd measure_temp | grep  -o -E '[[:digit:]].*'
   ```

* CPU Temperature
   ```sh
   cat /sys/class/thermal/thermal_zone0/temp
   ```

* Cpu Information
   ```sh
   cat /proc/cpuinfo
   ```

  Use grep to filter information:
   ```sh
   cat /proc/cpuinfo | grep Model
   ```
   
* Check under-voltage warning:
   ```sh
   dmesg | grep voltage
   ```

   Also Cpu throttle status, good value is `0x0`:
   ```sh
   vcgencmd get_throttled
   ```
   
   | Bit Position |	Meaning
   | -------------| ------------
   | 0 |	Under-voltage detected
   | 1 |	Arm frequency capped
   | 2 |	Currently throttled
   | 3 |	Soft temperature limit active
   | 16 |	Under-voltage has occurred
   | 17 |	Arm frequency capping has occurred
   | 18 |	Throttling has occurred
   | 19 |	Soft temperature limit has occurred

  For example, a throlled value shows 0x80008. Now convert this hex value to binary in two steps - hex to decimal, decimal to binary.
  
  ```js
  let n = parseInt('0x80000', 16) // must use quotes. without quotes it shows different value.
  n.toString(2) // result: 10000000000000000000
  ```
  
  Here, in the value '10000000000000000000' 19th bit is turned on which means *soft temperature limit has occurred*.

* Save any command output to a file `<command_and_arguments> > <file_name>`.
   Example:
   ```sh
   top -b -n1 > output.txt
   ```
   File will be saved in current directory.

* System log path `/var/log/messages`. Additionally enable watch dog to do vigorous logging.

* Download/copy over SSH:
  ```
  scp username@remotehost:file_name.ext /my/local/destination/path
  ```
