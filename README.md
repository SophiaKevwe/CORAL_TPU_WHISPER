# EDGE AI
Working with edge devices, specifically  the coral dev board

## Table of contents
* Google Coral Development Board
* Openai-whisper-live-transcription using dev board

## Coral Development Board
This is a step-by-step guide to setting up a coral development board. The setup requires flashing Mendel Linux to the board and then accessing the board's shell terminal.
We will also cover performing inferencing on the coral development board, with and without the coral dev board.

## Setup for Coral Development Board
Steps to set up the coral development board.
### Installation of Git Bash
* For compatibility with the command-line instructions, Git Bash terminal is recommended.
* The Git Bash installer can be gotten from [here](https://git-scm.com/downloads).
* After installing Git Bash, open it and run the following commands to make Python accessible:
```
echo "alias python3='winpty python3.exe'" >> ~/.bash_profile
source ~/.bash_profile
```
### Flashing of the board
* A microSD card would be required to flash the board with Mendel Linux.
* Download and unzip the SD card image: [enterprise-eagle-flashcard-20211117215217.zip](https://dl.google.com/coral/mendel/enterprise/enterprise-eagle-flashcard-20211117215217.zip) ,the ZIP contains one file named flashcard_arm64.img.
* Use a program such as balenaEtcher to flash the flashcard_arm64.img file to your microSD card. This takes 5-10 minutes, depending on the speed of your microSD card and adapter.
* While the card is being flashed, make sure the Dev Board is completely unplugged, and change the boot mode switches to boot from SD card like [this](https://coral.ai/static/docs/images/devboard/devboard-bootmode-sdcard.jpg)
* Once the card is flashed, safely remove it from your computer and insert it into the Dev Board (the card's pins face toward the board). The board should not be powered on yet.
* Power up the board by connecting your 2-3 A power cable to the USB-C port labeled "PWR" . The board's red LED should turn on.
Caution: Do not attempt to power the board by connecting it to your computer.
* When the board boots up, it loads the SD card image and begins flashing the board's eMMC memory. It should finish in 5-10 minutes, depending on the speed of your microSD card.
You'll know it's done when the board shuts down and the red LED turns off.
* When the red LED turns off, unplug the power and remove the microSD card.
* Change the boot mode switches to eMMC mode, like [this](https://coral.ai/static/docs/images/devboard/devboard-bootmode-emmc.jpg).
* Connect the board to power and it should now boot up Mendel Linux.
Booting up for the first time after flashing takes about 3 minutes (subsequent boot times are much faster).

### Installing the MDT(Microsoft Deployment Toolkit)
* You can install MDT on your host computer by typing this on your bash
```
python3 -m pip install --user mendel-development-tool
```
* You might see a warning that mdt was installed somewhere that's not in your PATH environment variable. If so, be sure you add the given location to your PATH, as appropriate for your operating system. If you're using Git Bash, you'll need an alias to run MDT. Run this in your Git Bash terminal:
```
echo "alias mdt='winpty mdt'" >> ~/.bash_profile
source ~/.bash_profile
```
### Connecting to the board's shell via MDT
* Connect a USB-C cable from your computer to the board's other USB port (labeled "OTG").
* Now make sure MDT can see your device by running this command from your host computer:
```
mdt devices
```
You should see the output showing your board hostname and IP address:
```
orange-horse        (192.168.100.2)
```
* If you don't see your device printed, it's probably because the system is still setting up Mendel. So instead run
```
mdt wait-for-device.
```
* Now to open the device shell, run this command:
```
mdt shell
```
* After a moment, you should see the board's shell prompt.

### Connecting to the internet
* Either connect an Ethernet cable to the board or select a Wi-Fi network by running the following command in the device shell:
```
nmtui
```
* Then select Activate a connection and select a network from the list under Wi-Fi (wlan0).
* Alternatively, use the following command to connect to a known network name:
```
nmcli dev wifi connect <NETWORK_NAME> password <PASSWORD> ifname wlan0
```
* Verify your connection with this command:
```
nmcli connection show
```
* The DEVICE name is wlan0 for a Wi-Fi connection or eth0 for an Ethernet connection.

### Updating the Mendel software
Make sure you have the latest software by running the following commands.
```
sudo apt-get update
sudo apt-get dist-upgrade
```

### Installing other versions of Python from Source on the pi
You can change the version as you wish but this guide is for Python Version 3.9(any version higher than python 3.7)
#### Update and Upgrade:
First, make sure your coral dev board is up to date by running the following commands:
```
sudo apt update
sudo apt upgrade
```
#### Install Dependencies:
* Install the dependencies required to build Python from source:
```
sudo apt install build-essential libsqlite3-dev sqlite3 bzip2 libbz2-dev zlib1g-dev libssl-dev libreadline-dev libncurses5-dev libncursesw5-dev libgdbm-dev liblzma-dev libffi-dev uuid-dev libffi-dev liblzma-dev libsqlite3-dev libncurses5 libgdbm-compat-dev libgdbm-dev libssl-dev libreadline-gplv2-dev
```
#### Download and Compile Python:
* Now, you can download and compile Python 3.9. Here's how:
```
cd ~
wget https://www.python.org/ftp/python/3.9.7/Python-3.9.7.tgz
tar -xf Python-3.9.7.tgz
cd Python-3.9.7
./configure --enable-optimizations
make -j 4  # Adjust the number based on your Pi's resources
sudo make altinstall
```
#### Verify Installation:
* After the installation is complete, you can verify that Python 3.9 is installed by running:
```
python3.9 --version
```
#### Cleanup:
* You can remove the downloaded files to free up space:
```
cd ~
rm -rf Python-3.9.7 Python-3.9.7.tgz
```
* To make use of the Python version you need to run your programs created in a virtual environment.

### Installing a virtual environment(If not already installed)(Optional):
Run the following command to install the virtualenv package:
```
sudo apt install python3.9-venv
```
#### Create a Virtual Environment:
Navigate to the directory where you want to create your virtual environment. For example, you might create a Venv directory within your home folder:
```
cd ~
mkdir venv
```
Now, create the virtual environment by running:
```
python3.9 -m venv venv/myproject
```
Replace myproject with the name of your project. This will create a virtual environment named myproject within the venv directory

#### Activate the Virtual Environment:
To activate the virtual environment, run:
```
source venv/myproject/bin/activate
```
Now you can run your python programs with your new python version in the virtual envrionment.

## Running a demo app
* For a video demo of the Edge TPU performance, run the following command from the Dev Board terminal:
```
edgetpu_demo --stream
```
* Then on your desktop (that's connected to the Dev Board)—if you're connected to the board using MDT over USB—open 192.168.100.2:4664 in a browser. If you're instead connected to the board by other means (such as SSH over LAN or with an Ethernet cable), type the appropriate IP address into your browser with port 4664.
* You should then see a video playing in your browser. The footage of the cars is a recording, but the MobileNet model is executing in real-time on your Dev Board to detect each car.
* If you have a monitor attached to the Dev Board, you can instead see the demo on that screen:
```
edgetpu_demo --device
```


# openai-whisper-live-transcription
Real-time transcription of audio from a microphone
Credit to https://github.com/JohnZolton/scribe
