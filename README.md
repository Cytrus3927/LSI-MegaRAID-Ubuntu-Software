# LSI MegaRAID Ubuntu/Windows Software

This repository contains installation instructions and helper scripts for **LSI MegaRAID Storage Manager (MSM)** on Ubuntu/Linux servers and Windows 10/11 clients.  
**Note:** The software itself is proprietary Broadcom software and is **not included** in this repository. Users must download the original packages from Broadcom.

---

## Table of Contents

- [Client Installation (Windows 10/11)](#client-installation-windows-1011)
- [Server Installation (Ubuntu/Linux)](#server-installation-ubuntulinux)
- [Licensing](#licensing)

---

## Client Installation (Windows 10/11)

### **Step: 1** Install MegaRaid Storage Manager

1. Download official MSM package [Download](https://docs.broadcom.com/docs-and-downloads/17.05.06.00_MSM_Windows.zip)
2. Unpack package
3. Run `setup.exe` from MSM directory 
4. Follow installation instructions; if no LDAP server is available, choose **Do not use LDAP**  
5. Finish installation do not run the application yet

### **Step: 2** Set-up Java for MSM

1. Download **Temurin JDK 8 32-bit LTS version (ZIP)** here: [Download](https://adoptium.net/en-GB/temurin/releases?version=8&os=any&arch=any)  
2. Extract to `C:/Java/`  
3. Open the file with administration privileges:  
`C:\Program Files (x86)\MegaRAID Storage Manager\startupui.bat`
4. Change the Java path: <br/>
From: <br/>
 `start %JRE_HOME%\bin\javaw` <br/>
To: <br/>
`start C:\Java\bin\javaw` <br/>
(replace with your actual Java path) <br/>
If your directory contains spaces, you need to do somethink like this <br/>
`start "" "C:\Program Files\Java\jre1.8.0_431\bin\javaw.exe" -DVENUS=true -classpath ".;GUI.jar;monitorgui.jar;DebugLog.jar;jaxen-1.1.jar;jdom-1.1.jar" GUI.VivaldiStartupDialog ajsgyqkj=71244`
(replace with your actual Java path)

>[!WARNING]
> If MegaRAID Storage Manager does not start correctly or you encounter errors related to Java
> 1. Add Java as a system environment variable <br/>
>    - Open System Properties → Advanced → Environment Variables <br/>
>    - Under System Variables, click New <br/>
>    - Set Variable name: JRE_HOME <br/>
>    - Set Variable value: the path to your Java installation directory (`C:\Java`) <br/>
>    - Save the variable <br/>

### **Step: 3** Run MegaRaid Storage Manager

## Server Installation (Ubuntu/Debian)

Tested on Ubuntu Server 25.10 with OpenJDK 21 (headless) — everything worked correctly without issues.
However, according to Broadcom’s documentation, the server officially recommends using Java 8.

### **Step: 1** Create .deb packages for Debian based distros

1. Install alien for `.rpm` package repacking <br/>
`sudo apt install alien`

2. Download the official MegaRaid Storage Manager package from Broadcom [Download](https://docs.broadcom.com/docs-and-downloads/17.05.06.00_MSM_Linux-x64.zip) <br/>
Extract it to a folder
3. Convert `.rpm` into `.deb` package <br/>
 `sudo alien --scripts *.rpm`

You should get those files:

```
lib-utils2*.deb
megaraid-storage-manager*.deb
sas-ir-snmp*.deb
sas-snmp*.deb
```

### **Step: 2** Install MegaRaid Storage Manager

1. Install actual package and it's dependencies

```
sudo -i
cd /path/to/unpacked/megaraid/software/
export setuptype="d"
export upgradesetuptype="n"
touch /etc/redhat-release
dpkg --install lib-utils*.deb
dpkg --install megaraid-storage-manager*.deb
exit
sudo rm /etc/redhat-release
```

2. Set root password for MSM app <br/>
`sudo passwd`


### **Step: 3** Modify MegaRaid scripts to match Debian format

1. Edit init script <br/>
Open <br/>
`/etc/init.d/vivaldiframeworkd` <br/>
and replace: <br/>
`for ((i=0, i < 20, i++))` <br/>
with: <br/>
```for i in `seq 0 19` ```


2. Edit startup script <br/>
Open <br/>
`/usr/local/MegaRAID Storage Manager/Framework/startup.sh` <br/>
and replace: <br/>
```LD_LIBRARY_PATH=`pwd`:/opt/lsi/Apache:/opt/lsi/Pegasus:/opt/lsi/openssl;export LD_LIBRARY_PATH``` <br/>
with: <br/>
```LD_LIBRARY_PATH=`pwd`:/opt/lsi/Pegasus:/usr/sbin/openssl;export LD_LIBRARY_PATH```


### **Step: 4** Add MSM as a system service

1. Add service to system <br/>
`systemctl enable vivaldiframeworkd.service`

2. Start service <br/>
`service vivaldiframeworkd start`

3. Check if any error occured <br/>
`service vivaldiframeworkd status` <br/>
`journalctl -xe`

### **Step: 5** Allow MSM Client to access server via ufw firewall

`sudo ufw allow 3071/tcp 5571/tcp `


### **Step: 6** (Optional) Add storcli tool as system command
`sudo ln -s /usr/local/MegaRaid\ Storage\ Manager/StorCLI/storcli64 /usr/local/sbin/storcli` <br/>
>Note: storcli works only with sudo ex. `sudo storcli show`, if you run it as normal user you probably gets 0 cards detected even if you have one in your system


### Licensing
The MegaRAID Storage Manager software is proprietary Broadcom software. This repository does not include the MSM binaries. Software is provided AS IS without any warranty. Users must obtain the official packages from Broadcom and comply with their license.


Disclaimer

This software and these instructions are provided AS IS, without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement.

In no event shall the author(s) or copyright holder(s) be liable for any claim, damages, or other liability, whether in an action of contract, tort, or otherwise, arising from, out of, or in connection with the software or the use or other dealings in the software.

Use at your own risk.
