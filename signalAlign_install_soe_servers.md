# Install SignalAlign on SOE server

Welcome to the workflow that will test your patience and endurance but at the end you will finally have an installed version of SignalAlign on your server which is probably you need for your project. Otherwise, I have no idea why would you want to install this software. Please send me an email at `shafin@ucsc.edu` if you are stuck. I highly suggest each individual set-up their own version of SignalAlign as it depends on some shared library which makes it impossible to make the binaries portable.

## Start a screen
This process takes forever and an hour to finish. I suggest running this in a screen so you don't lose progress. If you don't know how use screen, I suggest taking help from google.
```bash
screen -S signalalign_install
```

## Step 1: Install Anaconda
Because you don't have root privilege. **Make sure you have `wget, gcc,and bzip2` installed**. If not, then please ask the admin to install them. The UCSC SOE servers have them available for all users.

```bash
wget https://repo.anaconda.com/archive/Anaconda3-5.3.1-Linux-x86_64.sh
bash Anaconda3-5.3.1-Linux-x86_64.sh
```
You should see an output like the following:

```
Welcome to Anaconda3 5.3.1

In order to continue the installation process, please review the license
agreement.
Please, press ENTER to continue
```

Press `ENTER` to continue and then press `ENTER` to scroll through the license. Once you’re done reviewing the license, you’ll be asked to approve the license terms:
```
output
Do you accept the license terms? [yes|no]
```
Type yes to accept the license and you’ll be prompted to choose the installation location.
```
Anaconda3 will now be installed into this location:
/home/user/anaconda3

    - Press ENTER to confirm the location
    - Press CTRL-C to abort the installation
    - Or specify a different location below
```
I highly recommend not changing the location, but if you know what you are doing then go ahead and change the location. Press `ENTER` to confirm the location and installation process will continue.

*Now wait forever and 30 minutes*, then it'll ask you this:

```
Do you wish the installer to initialize Anaconda3
in your /home/kishwar/.bashrc ? [yes|no]
```

##### NOTE:
I have heard that in few cases the `~/.bashrc` does not get updated, don't panic if it doesn't, if you follow the rest of it, it should still work.

You should type `yes`.

```
To install Visual Studio Code, you will need:
  - Administrator Privileges
  - Internet connectivity

Visual Studio Code License: https://code.visualstudio.com/license

Do you wish to proceed with the installation of Microsoft VSCode? [yes|no]
>>>
```
You wouldn't need Microsoft VS, so type `no`.

This should complete anaconda installation.

Finally, activate conda by running:
```bash
source $HOME/anaconda3/etc/profile.d/conda.sh
source ~/.bashrc
```

To verify the install, run:
```bash
conda info
```

## Step 2: Set up environment variables

Set up the envoironment variables.
#### Sanity check:
```bash
echo $HOME
# output should be your home directory
ls $HOME/anaconda3/
# output should be list of directories inside anaconda3
ls $HOME/anaconda3/bin/
# output should be a list of binaries installed by anaconda3
ls $HOME/anaconda3/lib/
# output should be a list of libraries
ls $HOME/anaconda3/include/
# output should be a list of .h files
```

#### Set up library paths
We are going to use these installed libraries and binaries for SignalAlign. So next you should add these directories to you path variable. If everything looks right, then simply run this:
```bash
echo "PATH=$HOME/anaconda3/bin:$HOME/anaconda3/lib:$HOME/anaconda3/include:$PATH" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=$HOME/anaconda3/lib:$LD_LIBRARY_PATH" >> ~/.bashrc
```

We are also going to change some compiler flags, but this should not be a default behavior, so we will create a separate bash file for this.
```bash
echo "export CPATH=\"$HOME/anaconda3/include\"" >> ~/.bashrc_sa
echo "export C_INCLUDE_PATH=\"$HOME/anaconda3/include\"" >> ~/.bashrc_sa
echo "export LDFLAGS=\"-L$HOME/anaconda3/lib\"" >> ~/.bashrc_sa
echo "export CPPFLAGS=\"-I$HOME/anaconda3/include\"" >> ~/.bashrc_sa
```

Now you **MUST** run:
```bash
source ~/.bashrc
source ~/.bashrc_sa
```
This will set the environment variables for you. And if you do `echo $PATH` you can see the PATH variable now includes your anaconda paths which is what we need.

## Step 3: Install Samtools, bwa and python
We are going to install `SignalAlign` and all the dependencies in the same directory. Create that and install samtools in that directory:
```bash
cd $HOME
mkdir signalalign_install
cd signalalign_install
```

#### Samtools
To install samtools, you'll need ncurses installed:
```bash
conda install -c anaconda ncurses
conda install -c biobuilds ncurses-devel
conda install -c dgursoy gcc-5
conda install openssl=1.0
conda install git
conda install -c anaconda zlib
```

Then download and install samtools
```bash
cd $HOME/signalalign_install
wget https://github.com/samtools/samtools/releases/download/1.9/samtools-1.9.tar.bz2
tar -vxjf samtools-1.9.tar.bz2
cd samtools-1.9
./configure --prefix=$HOME/signalalign_install/samtools-1.9/
make -j 16
make install
```

Now update the environment variable to make this samtools the default samtools.
```bash
echo "PATH=$HOME/signalalign_install/samtools-1.9/bin/:$PATH" >> ~/.bashrc_sa
source ~/.bashrc_sa
```

Verify the install by running:
```bash
which samtools
samtools --version
```

#### bwa

```bash
cd $HOME/signalalign_install
wget https://github.com/lh3/bwa/releases/download/v0.7.17/bwa-0.7.17.tar.bz2
tar -vxjf bwa-0.7.17.tar.bz2
cd bwa-0.7.17
make -j 16
```

Now update the environment variable to make this samtools the default samtools.
```bash
echo "PATH=$HOME/signalalign_install/bwa-0.7.17/:$PATH" >> ~/.bashrc_sa
source ~/.bashrc_sa
```

Verify the install by running:
```bash
which bwa
```

#### Install python (This is required even if you have python installed)
Now download and install a custom python.

```bash
cd $HOME/signalalign_install
wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tgz
tar -xzf Python-3.6.8.tgz
find $HOME/signalalign_install/  -type d | xargs chmod 0755
cd Python-3.6.8/
./configure --prefix=$HOME/signalalign_install/Python-3.6.8/
make -j 16
make install
```

Set it to default
```bash
echo "PATH=$HOME/signalalign_install/Python-3.6.8/bin:$PATH" >> ~/.bashrc_sa
echo "alias python=python3" >> ~/.bashrc_sa
source ~/.bashrc_sa
```
Verify the install:
```bash
which python3
```


#### Install pip
```bash
cd $HOME/signalalign_install
wget https://bootstrap.pypa.io/get-pip.py
python3 get-pip.py --user
```

#### Install python dependencies:
```bash
python3 -m pip install numpy
python3 -m pip install setuptools
python3 -m pip install py3helpers==0.3.4
python3 -m pip uninstall python-dateutil
python3 -m pip install python-dateutil==2.6.1
```

## Step 4: Finally, install SignalAlign
#### Sanity check:
```bash
source ~/.bashrc
source ~/.bashrc_sa
which samtools
# should point to $HOME/signalalign_install/samtools-1.9/bin/samtools
which bwa
# should point to $HOME/signalalign_install/bwa/bwa
which python
# should point to $HOME/signalalign_install/Python-3.6.8/bin/python
which python3
# should point to $HOME/signalalign_install/Python-3.6.8/bin/python3
gcc --version
# should be 5.2
```
#### Install SignalAlign
```bash
source ~/.bashrc
source ~/.bashrc_sa
cd $HOME/signalalign_install
git clone --recursive https://github.com/kishwarshafin/signalAlign.git
cd signalAlign/
make
python3 -m pip install -e . --user
# if this gives sonLib error then run make one more time, sonLib is just weird sometimes
```

This should install SignalAlign in this directory: `/home/kishwar/signalalign_install/signalAlign/bin/`. You can make this globally execcible like other modules:
```bash
echo "PATH=$HOME/signalalign_install/signalAlign/bin/:$PATH" >> ~/.bashrc_sa
source ~/.bashrc_sa
```

Now you can run:
```bash
python3 runSignalAlign -h
signalMachine
```



## Step 5: What happens when you log-out and then log back again
Each time you log out of a server and then log in back again, your environment variables get reset. As we have used conda installed libraries instead of the globally installed ones, it is necessary to source the bash files each time you want to use `SignalAlign`. So, run:
```bash
source ~/.bashrc
source ~/.bashrc_sa
```
Then all the path variables will be reset to what you had and you can run SignalAlign globally again.
