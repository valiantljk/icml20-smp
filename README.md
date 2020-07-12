## Docker Container for Modular-RL

### Build a docker image for icml20: modular-rl
```Shell
docker build -t modular-rl .
```
### Run a container for test
```Shell
docker run -it modular-rl bash 
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