# Profiling Modular-rl
This goal of this repo is to take the recent ICML'20 paper as an exmaple,
* get familar with Google Cloud development environment
* understand deep reinforcement leanring
* understand the 'one policy for all' idea in this paper
* figure out the right toolset to profile application performance on the cloud
* figure out the problem related to dl application's performance bottleneck on the cloud

There are issues in running in Docker container and running on a local mac:

## Run in a container (Issue: mujoco key is not valid in container)
```Shell
docker build -t modular-rl .
docker run -it modular-rl bash 
```

## Run on Mac (Issue: training takes long time)
* Install Ananconda Navigator
* Create Environments 'modular-rl'
* Launch the environment
```Shell
cd /modular-rl
brew install libomp 
pip install -r requirements.txt
```

Running on Cloud seems easier (but I don't recommend google cloud, it's not user-friendly at all, I use it bc it has $300 credit for new users)

## Run on Google Cloud
### VM Instance
16 vCPU, 60 G Mem, P100 
### Install Anaconda
```shell
sudo apt-get install -y nvidia-smi
sudo reboot
wget https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
chmod +x Anaconda3-2020.02-Linux-x86_64.sh
./Anaconda3-2020.02-Linux-x86_64.sh
#reopen terminal
#might need to add more disks for the vm instance
conda create -n /mnt/disks/data/modular-rl python=3.6
```
### Install MPICH 
```Shell
export MPICH_VERSION="3.4a3"
export MPICH_CONFIGURE_OPTIONS="--disable-fortran --with-device=ch4:ofi"
wget http://www.mpich.org/static/downloads/${MPICH_VERSION}/mpich-${M
ICH_VERSION}.tar.gz 
tar xfz mpich-${MPICH_VERSION}.tar.gz
cd mpich-${MPICH_VERSION}
./configure ${MPICH_CONFIGURE_OPTIONS}
make -j 16
sudo make install
```
### Install Mujoco

```Shell
sudo mkdir -p /home/btrace_lu/.mujoco
wget https://www.roboti.us/download/mujoco200_linux.zip -O mujoco.zip
sudo apt-get install unzip
unzip mujoco.zip -d /home/btrace_lu/.mujoco
sudo unzip mujoco.zip -d /home/btrace_lu/.mujoco
sudo mv /home/btrace_lu/.mujoco/mujoco200_linux /home/btrace_lu/.mujoco/mujoco200
rm mujoco.zip
```

### Register Mujoco
get 'compute id'
```Shell
wget https://www.roboti.us/getid/getid_linux
chmod +x getid_linux
./getid_linux
```
you get something like 'LINUX_AAAA_BBBBB'
then go to https://www.roboti.us/license.html and get a licnese 'mjkey.txt', save it in /root/.mujoco


### Install Modula-Rl
```Shell
cd /mnt/disks/data/modular-rl/
git clone https://github.com/valiantljk/icml20-smp.git
export TMPDIR=/mnt/disks/data/tmp
cd icml20-smp/modular-rl
sudo apt-get install libx11-dev
pip install -r requirements.txt 
```


Then follow the readme in modular-rl, e.g., 

### Train with existing environment
- Train both-way SMP on ``Walker++`` (12 variants of walker):
```Shell
python main.py --expID 001 --td --bu --morphologies walker
  ```

### Visualization
- To visualize all ``walker`` environments with the both-way SMP model from experiment ``expID 001``:
```Shell
python visualize.py --expID 001 --td --bu --morphologies walker
```
