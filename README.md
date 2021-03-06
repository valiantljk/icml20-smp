# Profiling Modular-rl on Cloud
This goal of this repo is to take the recent ICML'20 [paper](https://www.cs.cmu.edu/~dpathak/papers/modular-rl.pdf)/[code](https://github.com/huangwl18/modular-rl) as an exmaple,
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
### Create Conda Env
```Shell
cd /modular-rl
brew install libomp 
brew install mpich
brew install ffmpeg
conda create -n modular-rl python=3.6
conda init
conda activate modular-rl
pip install -r requirements.txt
```
### Configure Mujoco
```Shell
brew install wget
brew install gcc@6
cd /Downloads
wget https://www.roboti.us/download/mujoco200_macos.zip
unzip mujoco200_macos.zip -d /Users/username/.mujoco
cd /Users/username/.mujoco
mv mujoco200_macos mujoco200
```
### Register Mujoco
```Shell
wget https://www.roboti.us/getid/getid_osx
chmod a+x getid_osx
```
you get something like 'OSX_AAAA_BBBBB'
then go to [Roboti](https://www.roboti.us/license.html) , put the computer id and pro key(need to apply first),and get a licnese 'mjkey.txt', save it in /Users/username/.mujoco
### Test
```Shell
conda activate modular-rl
cd /src
python main.py --expID 001 --td --bu --morphologies walker_7_main --max_timesteps 20000
```
### Profiling with torch.utils.bottleneck
```Shell
python -m torch.utils.bottleneck main.py --expID 001 --td --bu --morphologies walker_7_main --max_timesteps 20000
```


Running on Cloud seems easier (but I don't recommend google cloud, it's not user-friendly at all, I use it bc it has $300 credit for new users)

## Run on Google Cloud
### VM Instance
16 vCPU, 60 G Mem, P100
### 0 Install Dependencies

```Shell
sudo apt-get install -y libglew-dev libgl1-mesa-dev libgl1-mesa-glx libosmesa6-dev software-properties-common 
net-tools unzip wget  xpra xserver-xorg-dev nvidia-smi
sudo reboot
```

### 1 Install Anaconda

```shell
wget https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
chmod +x Anaconda3-2020.02-Linux-x86_64.sh
./Anaconda3-2020.02-Linux-x86_64.sh
#reopen terminal
#might need to add more disks for the vm instance
conda create -n /mnt/disks/data/modular-rl python=3.6
```

### 2 Install MPICH 
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
### 3 Install Mujoco

```Shell
sudo mkdir -p /home/username/.mujoco
wget https://www.roboti.us/download/mujoco200_linux.zip -O mujoco.zip
sudo apt-get install unzip
sudo unzip mujoco.zip -d /home/username/.mujoco
sudo mv /home/username/.mujoco/mujoco200_linux /home/username/.mujoco/mujoco200
rm mujoco.zip
```

### 4 Register Mujoco
get 'compute id'
```Shell
wget https://www.roboti.us/getid/getid_linux
chmod +x getid_linux
./getid_linux
```
you get something like 'LINUX_AAAA_BBBBB'
then go to [Roboti](https://www.roboti.us/license.html) and get a licnese 'mjkey.txt', save it in /home/username/.mujoco


### 5 Install Modula-Rl
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
python main.py --expID 001 --td --bu --morphologies walker_7_main
  ```

### Expected Output  
```Shell
*** training from scratch ***
--------------------------------------------------
ExpID: 1, FPS: 21.88, TotalT: 81, EpisodeNum: 1, SampleNum: 82, ReplayBSize: 82
walker_7_main === EpisodeT: 81, Reward: 82.41
--------------------------------------------------
ExpID: 1, FPS: 22.10, TotalT: 209, EpisodeNum: 2, SampleNum: 211, ReplayBSize: 211
walker_7_main === EpisodeT: 128, Reward: 185.34
--------------------------------------------------
ExpID: 1, FPS: 22.08, TotalT: 258, EpisodeNum: 3, SampleNum: 261, ReplayBSize: 261
walker_7_main === EpisodeT: 49, Reward: 40.91
```

### Performance Comparision [Not Apple to Apple]
| Platform      | Configuration                 |  FPS   | 
| ------------- | ----------------------------- | ------ |
| MacAir        | 1.6 GHz 2 Core, i5, 8GB Mem   |   4    |
| MacPro        | 2.6 GHz 6-Core, i7, 32G Mem   |   9.5  |
| Google Cloud  | 16 vCPU, 60GB Mem, P100 GPU   |   20   |

### Visualization
- [Install ffmpeg](https://tecadmin.net/install-ffmpeg-on-linux/)

- To visualize all ``walker`` environments with the both-way SMP model from experiment ``expID 001``:
```Shell
python visualize.py --expID 001 --td --bu --morphologies walker_7_main
```
- To Download the video, click the top-right of the google cloud's ssh window, there is a 'download' option. enter filename to download the video, which is in results/videos
