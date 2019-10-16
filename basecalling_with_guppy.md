# How to run guppy basecaller

This is the workflow I follow to basecall ONT reads using guppy basecaller:

<b>NOTE: To install guppy you need administrative privilege.</b>

## Check if guppy_basecaller is already installed in your machine
```bash

```

## Install guppy on a Linux machine:

#####  Install ONT dependency packages
```bash
sudo apt-get update
sudo apt-get install wget lsb-release
export PLATFORM=$(lsb_release -cs)
wget -O- https://mirror.oxfordnanoportal.com/apt/ont-repo.pub | sudo apt-key add -
echo "deb http://mirror.oxfordnanoportal.com/apt ${PLATFORM}-stable non-free" | sudo tee /etc/apt/sources.list.d/nanoporetech.sources.list
sudo apt-get update
```

#####  Install the CPU-ONLY guppy_basecaller
```bash
apt-get install ont-guppy-cpu
```

#####  Or, Install the GPU enabled guppy_basecaller
For this version to work, you will need appropriate CUDA drivers to be installed on your system.
```bash
apt-get install ont-guppy
```

## STEP 1: copy your local FAST5 files to a server
I highly recommend to run time-consuming stuff in a screen which is a terminal multiplexer. So, first, start a screen on your <b>local machine</b>.
```bash
screen -S upload_fast5
```
This will start a screen where you can run your command with the fear of losing progress if you do anything else.

Inside the screen, run `scp` to copy your fast5 files:
```bash
scp -r /path/to/local/fast5_dir/ username@server.ip:/path/to/remote/fast5_dir/
```
Now you can <b>disconnect from the screen</b> by pressing `Ctrl` + `A` + `D`.<br/>
You can list the screens using:
```bash
screen -ls
```
And, reconnect to the screen using:
```bash
screen -rd <screen_name>
# in our case the screen_name is upload_fast5, so the command will be:
screen -rd upload_fast5
```

## STEP 2: Prepare for a run
Log in to your remote server.
```bash
ssh username@server.ip
```
##### Start a screen
```bash
screen -S guppy_basecalling
```

##### Download sample dataset (If you don't have your own data)
```bash
mkdir guppy_basecaller_tutorial
cd guppy_basecaller_tutorial
wget https://openstack.cebitec.uni-bielefeld.de:8080/swift/v1/nanopore_course_data/Data_Group1.tar.gz
wget https://openstack.cebitec.uni-bielefeld.de:8080/swift/v1/nanopore_course_data/Results_Group1.tar.gz
tar -xzvf Data_Group1.tar.gz
tar -xzvf Results_Group1.tar.gz
rm Data_Group1.tar.gz
rm Results_Group1.tar.gz
ll
# you will see a list of files that you just downloaded
```
To inspect fast5 files:
https://denbi-nanopore-training-course.readthedocs.io/en/latest/basecalling/inspect_h5.html

## STEP 3: Run guppy basecaller (CPU)
After installing `guppy_basecaller`, you should be able to see the installed files here:
```bash
```
