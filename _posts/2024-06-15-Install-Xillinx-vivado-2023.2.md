---
title: Install Xillinx vivado 2023.2 on Ubuntu 22.04
author: Gieun Jeong
date: 2024-06-15 02:00:00 +0900
categories: [Verilog]
tags: [Verilog, vivado]
# pin: true
# math: true
# mermaid: true
# image:
#   path: /commons/devices-mockup.png
#   lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
#   alt: Responsive rendering of Chirpy theme on multiple devices.****
---

## Environment
I'm going to install Xillinx Vivado 2024.1 on my remote server where OS version is Ubuntu 22.04.4. I've installed vivado in below environments.

- Remote server: Ubuntu 22.04
- Local OS: MacOS Sonoma 14.1
- Vivado: Xillinx vivado 2023.2 for Linux

## prerequisite
- If you use Linux, run below commands.
```bash
sudo apt-get update
sudo apt upgrade
sudo apt-get install libtinfo5 libncurses5 libxrender1 -y # Libraries for running vivado
sudo apt install libncurses5-dev libncursesw5-dev
sudo apt-get install -y xauth # For X11 forwarding
sudo apt install x11-apps # For X11 forwarding
```
- You should verify x11 forwarding parameters. At first, run below command to open `ssh_config`.
```bash
vim /etc/ssh/ssh_config
```
- Then, you can find that `ForwardX11` and `ForwardX11Trusted` variables are commented. Remove the comment and change them as following.
```md
ForwardX11 yes
ForwardX11Trusted yes
```
- Reconnect to remote server with `X` option.
```bash
ssh -X username@<server_address>
```
- Check whether X11 forwading is successfully configured by running below command.
```bash
xclock
```

## Download Xillinx vivado 2024.1
- Go to this link [Ofiicial page for Xillinx Vivado 2024.1](https://www.xilinx.com/support/download.html). Then, download **Linux self extracting verison**, which could be `.bin` file. You're required to make an account for AMD.
The downloaded file name should be `FPGAs_AdaptiveSoCs_Unified_2024.1_0522_2023_Lin64.bin` or something similar.
- Next, **copy** the `.bin` file to your remote server.

## Install Vivado on remote server
- You first need to check your super user password which is required to change to root account. Vivado should be installed under root account.
```bash
sudo passwd # configure password for root account
```
- Change to root account.
```bash
su
```
- Then you can see below path on your CLI. Which indicates you successfully change to root account!
```
root@username:/home/username#
```
- If you enter `ls -alh` command, you can see that there is no execution permission for `FPGAs_AdaptiveSoCs_Unified_2024.1_0522_2023_Lin64.bin` file. We need to provide permission.
```bash
# Before providing the permission
# -rw-r--r--  FPGAs_AdaptiveSoCs_Unified_2024.1_0522_2023_Lin64.bin

chmod +x FPGAs_AdaptiveSoCs_Unified_2024.1_0522_2023_Lin64.bin

# After providing the permission
# -rwxr-xr-x  FPGAs_AdaptiveSoCs_Unified_2024.1_0522_2023_Lin64.bin
```
- I want to install vivado on new directory. I will temporarily log out of the root account, create a new directory, and then proceed. If I make a new directory in root account, the new directory would be possessed to only for the root account. I don't want this.
```bash
exit # log out of the root account
mkdir tools # make a new directory
su
```
- Create account token and configuration file. Your AMD email is required in progress. I'm going to install in CLI. That's the reason why I generate token and configuration file. If your local OS is Ubuntu, you may not need to below process.
```bash
./FPGAs_AdaptiveSoCs_Unified_2024.1_0522_2023_Lin64.bin -- -b AuthTokengen # token gen
./FPGAs_AdaptiveSoCs_Unified_2024.1_0522_2023_Lin64.bin -- -b ConfigGen # config gen
```
- You can edit `config.txt` file. You may choose which tools to install. Go to configuration file and edit if needed.
```bash
vim /root/.Xilinx/install_config.txt
```
- All configurations are done. Run below command to start installing.
```bash
./FPGAs_AdaptiveSoCs_Unified_2024.1_0522_2023_Lin64.bin -- -a XilinxEULA,3rdPartyEULA -b Install  -c /root/.Xilinx/install_config.txt

# After installing, run below commands
exit # log out of root
source /home/<username>/tools/Xilinx/Vivado/2024.1/settings64.sh
vivado # run vivado
```
- If you can successfully see the vivado, Let's add source scirpt to `.bashrc`.
```bash
cd ~ # move to home directory
vim .bashrc

# Copy and paste below command to .bashrc file.
source /home/<username>/tools/Xilinx/Vivado/2024.1/settings64.sh
```
