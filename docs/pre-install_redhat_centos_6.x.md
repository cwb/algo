# Algo pre-install steps for Red Hat/CentOS 6.x (currently 6.8)

Many people prefer RedHat or CentOS 6 (or similar variants like Amazon Linux) for to their stability and lack of systemd. Unfortunately, there are a number of dated libraries, notably Python 2.6, that prevent Algo from running without errors. This script will prepare a RedHat, CentOS, or similar VM to deploy to Algo cloud instances.

## Step 1: Prep for RH/CentOS 6.8/Amazon

```
yum -y -q update
yum -y -q install epel-release
```

Enable any kernel updates:

``reboot`` 

## Step 2: Install Ansible & launch Algo

Fix GPG key warnings during Ansible rpm install:

``rpm --import https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-6``

Fix GPG key warning during offical Software Collections (SCL) package install:

``rpm --import https://raw.githubusercontent.com/sclorg/centos-release-scl/master/centos-release-scl/RPM-GPG-KEY-CentOS-SIG-SCLo``

RedHat/CentOS 6.x uses Python 2.6 by default, which is explicitly deprecated and produces many warnings and errors, so we must install a safe, non-invasive 2.7 tool set which has to be expressly enabled (and will not survive login sessions and reboots):

```
# Install the Software Collections Library (to enable Python 2.7)
yum -y -q install centos-release-SCL

# 2.7 will not be used until explicitly enabled, per login session		
yum -y -q install python27-python-devel python27-python-setuptools python27-python-pip
yum -y -q install openssl-devel libffi-devel automake gcc gcc-c++ kernel-devel wget unzip ansible nano 

# Enable 2.7 default for this session (needs re-run beween logins & reboots)
# shellcheck disable=SC1091
source /opt/rh/python27/enable
# We're now defaulted to 2.7 

# Upgrade pip itself
pip -q install --upgrade pip
# # python-devel needed to prevent setup.py crash, pycrypto 2.7.1 needed for latest security patch
pip -q install pycrypto       
pip -q install setuptools --upgrade

wget -q https://github.com/trailofbits/algo/archive/master.zip
unzip master.zip 
cd algo-master || echo "No Algo directory found"

# Install the local Algo dependencies (must be run from algo-master)
pip -q install -r requirements.txt

# Edit the userlist and any other settings you desire
nano config.cfg
# Now you can run the Algo installer!
./algo
```

## Post-install OSX

* Copy ./configs/*mobileconfig to your local Mac
* Install the VPN profile on your Mac (10.10+ required)
  * ``/usr/bin/profiles -I -F ./x.x.x.x_NAME.mobileconfig``
* To remove: ```/usr/bin/profiles -D -F ./x.x.x.x_NAME.mobileconfig```

The VPN connection will now appear under Networks (which can be pinned to the top menu bar if preferred)
