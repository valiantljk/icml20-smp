## Profiling Modular-rl

### Run in a container (Issue: mujoco key is not valid in container)
```Shell
docker build -t modular-rl .
docker run -it modular-rl bash 
```

### Run on Mac(Issue: training takes long time)
* Install Ananconda Navigator
* Create Environments 'modular-rl'
* Launch the environment
```Shell
cd /modular-rl
brew install libomp 
pip install -r requirements.txt
```
### Run on Google Cloud
16 vCPU, 60 G Mem, P100 
```shell
sudo apt-get install -y nvidia-smi
sudo reboot
wget https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
chmod +x Anaconda3-2020.02-Linux-x86_64.sh
./Anaconda3-2020.02-Linux-x86_64.sh
#reopen terminal
conda create -n modular-rl python=3.6
conda activate modular-rl
git clone https://github.com/valiantljk/icml20-smp.git
cd icml20-smp
cd modular-rl
pip install -r requirements.txt # might need to add more [disks](https://cloud.google.com/compute/docs/disks/add-persistent-disk#formatting) for the vm instance
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
