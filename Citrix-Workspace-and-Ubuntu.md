# Citrix Installation and Configuration for Ubuntu OS
## This document shows you how to install Citrix Receiver to you Ubuntu OS and configure to connect the X network

## Steps:
 1. Go to [Install, Uninstall, and Update](https://docs.citrix.com/en-us/citrix-workspace-app-for-linux/install.html) page of Citrix Workspace.

 2. Under the **Manual install** section, you will see *Download the following packages from the Citrix Downloads page.* Click on it.

 3. Currently, the link opens the *Citrix Workspace app 2203 for Linux* page. Go down and find *Available Downloads*. Go *Debian Packages -> Full Packages (Self-Service Support) -> Citrix Workspace app for Linux (x86_64)* and click *Download File*.

 4. Run the following command and install "GDebi" app:<br>
 `sudo apt install gdebi`

 5. Install the downloaded *Citrix Workspace app for Linux (x86_64)* Debian file via GDebi:<br>
 `sudo gdebi ~/Downloads/icaclient_22.3.0.24_amd64.deb`

    **Do not forget to change the package version of icaclient. If your default download folder is different from '~/Downloads/' you also need to change path of the file.**

 6. We need to use web browser to make a bridge between Citrix Workspace and X network machine. To use Firefox for this purpose, we need also share Firefox certificates with Citrix Workspace. In order to make it, use the following command:<br>
 `sudo ln -s /usr/share/ca-certificates/mozilla/* /opt/Citrix/ICAClient/keystore/cacerts`

 7. That's all for setup processes. Go to X network and enjoy to use it on Ubuntu OS :)

 ## Note: "ctxusb" is not installed because machines of X network does not support the USB connection.
