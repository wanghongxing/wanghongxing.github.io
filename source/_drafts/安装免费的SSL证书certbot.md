**certbot**



The EPEL repository can be added to a RHEL 7 system with the following command:

$ sudo rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

Adding the *optional* and *extras* repositories is also recommended:

$ sudo subscription-manager repos --enable "rhel-*-optional-rpms" --enable "rhel-*-extras-rpms"

$ sudo yum update





With the EPEL repository added to your RHEL installation, simply install the *snapd* package:

$ sudo yum install -y snapd

Once installed, the *systemd* unit that manages the main snap communication socket needs to be enabled:

$ sudo systemctl enable --now snapd.socket

To enable *classic* snap support, enter the following to create a symbolic link between /var/lib/snapd/snap and /snap:



$ sudo ln -s /var/lib/snapd/snap /snap





**Install Certbot**

**Run this command on the command line on the machine to install Certbot.**

sudo snap install --classic certbot



Prepare the Certbot command Execute the following instruction on the command line on the machine to ensure that the certbot command can be run. 


 sudo ln -s /snap/bin/certbot /usr/bin/certbot