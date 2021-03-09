<div align="center">

  <h3 align="center">Better internet speed measurement</h3>

  <p align="center">
    Follow this guide if you would like to get more precise information about your internet speed.
  </p>

</div>



<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#warning-about-data-usage">Warning about data usage</a></li>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#resources">Resources</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

I had problem with my internet connectivity especially with the speed which was far away from the speed for which I was paying for. I just wanted to have more professional measurements of my internet connection, so my network operator could not reject my complaints so easily.  

Key aspects of testing:
* testing will be done on dedicated testing device, which will have this as only task
* testing device will have fully updated operation system right before doing tests 
* measurement will be done each 5 minutes
* testing device will be connected to the broadband router by ethernet cable through gigabit port on the tested device
* testing device will be the only one device connected to the broadband router by ethernet cable
* wifi will be turned off on the broadband router


With this setup the internet provider cant say that slow speed could be caused by:
* low wifi speed in the house (distance, interference)
* other devices consuming bandwidth


### Built With

* [Raspberry Pi 4 B](https://www.raspberrypi.org/)
* Raspberry Pi OS Lite 32 bit without desktop environment
* [speedtest](https://www.speedtest.net/apps/cli)
* [nmap](https://nmap.org/)
* Python programming language



<!-- GETTING STARTED -->
## Getting Started

### Warning about data usage

Please be aware that this measurement will use a lot of data. Be careful If you have metered data plan. Just one measurement with 20mbit/s connection will use approx. 30MB just for download and approx. 4MB for upload. One measurement each 5 minutes. Do your math.

### Prerequisites

* fully built raspberry pi - there is a lot of options how you can build your Raspberry. You can choose nice case or go completely without the case. You can choose 2GB, 4GB or 8GB RAM version. Same applies for the size of sd card. Choose whatever suits you the best. As long as the base is Raspberry Pi 4B then you are golden. Don't forget about power adapter.
* sd card reader
* ethernet cable connected from your broadband router to raspberry
* make sure that no other devices are connected to your router except raspberry during measurement -> I did this by turning off the wifi on the broadband router and disconnected all other ethernet cables connected to it -> **BE REALLY SURE THAT YOU KNOW YOU ARE DOING** - for example - your home security cameras will be offline so your home could be unprotected or there could be other use-cases when some devices are essential to be connected to the internet.   

### Installation

1. Download [Raspberry Pi Imager](https://www.raspberrypi.org/software/) and open it
2. For the operating system choose Raspberry Pi OS Lite - Debian port with no desktop environment -> it is in Raspberry Pi Os (other) section
3. Choose the SD card and hit the Write button
4. Turn on the raspberry & login with these credentials:
```shell
raspberrypi login: pi 
Password: raspberry
```
5. We will do some configuration on our raspberry - we will enable SSH access so we could connect to raspberry from other computer (easier for copy & paste of the commands) and also we will set correct timezone:
```shell
sudo raspi-config

-> Select Interfacing Options
-> Navigate to and select SSH
-> Choose Yes
-> Select Ok

-> Select Localisation Options
-> Navigate to and select Timezone
-> Choose your geographic area
-> Choose the city of your timezone
-> Choose Finish (use right arrow)
```

6. Check IP adress of your raspberry & make a note of it - it should be something like 192.168.0.15
```shell
hostname -I
```

7. Connect to raspberry from your computer - please choose appropriate tutorial and then all following commands write into the newly opened session

* [Linux & Mac OS](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md)
* [Windows 10](https://www.raspberrypi.org/documentation/remote-access/ssh/windows10.md) 
* [Windows](https://www.raspberrypi.org/documentation/remote-access/ssh/windows.md) 

8. Install speedtest and nmap

```shell
sudo apt-get install gnupg1 apt-transport-https dirmngr
export INSTALL_KEY=379CE192D401AB61
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $INSTALL_KEY
echo "deb https://ookla.bintray.com/debian generic main" | sudo tee  /etc/apt/sources.list.d/speedtest.list
sudo apt-get update
sudo apt-get install speedtest nmap
```

9. Run speedtest manually for the first time and on license question answer 2 times YES

```shell
speedtest
```

10. Next we will open the file for our script

```
cd ~
sudo nano speedtest.py
```

11. This code will take care of calling speedtest and nmap and write results into the file - please replace 192.168.0.* with your network prefix:

```shell
import os
import re
import subprocess
import time

response = subprocess.Popen('speedtest -f csv', shell=True, stdout=subprocess.PIPE).stdout.read().decode('utf-8')

hosts_count_response = subprocess.Popen('sudo nmap -sP -PR 192.168.0.*', shell=True, stdout=subprocess.PIPE).stdout.read().decode('utf-8')
hosts_count = re.findall('(\d+) hosts up', hosts_count_response, re.MULTILINE)
hosts_count = hosts_count[0]


try:
    f = open('/home/pi/speedtest.csv', 'a+')
    if os.stat('/home/pi/speedtest.csv').st_size == 0:
            f.write('"Date","Time","Server name","Server id","Latency (ms)","Jitter (ms)","Packet loss %","Download (B/s)","Upload (B/s)","Downloaded Bytes","Uploaded Bytes","Share url","Hosts on network"\r\n')
except:
    pass

f.write('"{}","{}",{},"{}"\r\n'.format(time.strftime('%d.%m.%y'), time.strftime('%H:%M'), response, hosts_count))
```

12. Try to check if script works by calling it and waiting for a moment for showing the results from newly created csv file:
```shell
python3 /home/pi/speedtest.py && cat /home/pi/speedtest.csv
```

13. Check if there is last value `Hosts on network` showing exactly 2 (your raspberry + router itself):
```shell
cat /home/pi/speedtest.csv
```

14a. If there is 0 for your `Hosts on network` then you have your network prefix wrong, try to alter network prefix on this command until you get satisfying result and then change prefix used in script with correct one mentioned in step 11. (you will trigger editing the script by repeating the step 10.) 

```shell
sudo nmap -sP -PR 192.168.0.*
```

14b. If there is more than 2 it means there are other devices except your broadband router and raspberry. I think that having only testing device and broadband router is essential in case you want to complain to your internet provider. Ideally try to lower this number as much as possible as was mentioned in `Prerequisites` section.

15. If something was outputted from that file, we are ready to call this command periodically. We will use the cron to do this. Start editing crontab with this command:

```shell
crontab -e
-> hit enter on editor choose question - easiest editor to use will be used
```

16. Put this into editor at the end of the file and then save and exit the editor: 

```shell
*/5 * * * * python3 /home/pi/speedtest.py
```

17. Wait for few minutes and then make sure that new results are added to the end of file:
```shell
cat /home/pi/speedtest/speedtest.csv
```

18. Your can leave your raspberry doing this measurement for as long as you need - I recommend at least leave it to measure at least 24 hours straight. Exit ssh session:
```shell
exit
```

19. Copy results to your computer after desired period of time - change `ip-address` with your raspberry IP address and `~/Desktop` to your desired path on your computer:
```shell
scp pi@ip-address:/home/pi/speedtest/speedtest.csv ~/Desktop
-> use the same password as you used in first steps of this tutorial
```

20. Turn off periodic measurement by connecting to raspberry and open cron table:
```shell
ssh pi@ip-address
-> use the same password as you used in first steps of this tutorial

crontab -e
-> and comment out previously created record so it will look like this:

#*/5 * * * * python3 /home/pi/speedtest.py
-> save and exit
```


## Other resources
* [Internet speed in your browser](https://www.yournetspeed.com/)

