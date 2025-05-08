# 5980-Project
For fulfillment of CSCI 5980 Ai for Sequential Decision Making (Spring 2025)

# Home Robot Setup

This guide provides instructions to set up the Home Robot project environment.

## 1. Create Conda Environment

Create a new conda environment using the provided `environment.yaml` file:

```bash
conda env create -f environment.yaml
```

Activate the environment:

```bash
conda activate home-robot
```

## 2. Clone Home Robot Repository

Clone the `home-robot` repository from Facebook Research:

```bash
git clone https://github.com/facebookresearch/home-robot.git
```

Navigate into the cloned directory:

```bash
cd home-robot
export HOME_ROBOT_ROOT=$(pwd)
```

## 3. Checkout Challenge Branch

Checkout the specific branch for the Home Robot Challenge 2024:

```bash
git checkout home-robot--ovmm-challenge-2024
```

## 4. Install Dependencies

Run the script to install additional dependencies:

```bash
./install_deps.sh
```

## 5. Download Data

Run the script to download necessary data:

```bash
./download_data.sh
```

## 6. Run Habitat OVMM Example

Navigate to the example directory and run the Python script:

```bash
python projects/habitat_ovmm/eval_baselines_agent.py --env_config projects/habitat_ovmm/configs/env/hssd_demo.yaml
```


Results are saved to `datadump/images/eval_hssd/`.


## Training DD-PPO skills

First setup data directory
```
cd /path/to/home-robot/src/third_party/habitat-lab/

# create soft link to data/ directory
ln -s /path/to/home-robot/data data
```

To train on a single machine use the following script:
```
#/bin/bash

export MAGNUM_LOG=quiet
export HABITAT_SIM_LOG=quiet

set -x
python -u -m habitat_baselines.run \
   --exp-config habitat-baselines/habitat_baselines/config/ovmm/rl_skill.yaml \
   --run-type train benchmark/ovmm=<skill_name> \
   habitat_baselines.checkpoint_folder=data/new_checkpoints/ovmm/<skill_name>
```
Here `<skill_name>` should be one of `gaze`, `place`, `nav_to_obj` or `nav_to_rec`.

To run on a cluster with SLURM using distributed training run the following script. While this is not necessary, if you have access to a cluster, it can significantly speed up training. To run on multiple machines in a SLURM cluster run the following script: change `#SBATCH --nodes $NUM_OF_MACHINES` to the number of machines and `#SBATCH --ntasks-per-node $NUM_OF_GPUS` and `$SBATCH --gres $NUM_OF_GPUS` to specify the number of GPUS to use per requested machine.

```
#!/bin/bash
#SBATCH --job-name=ddppo
#SBATCH --output=logs.ddppo.out
#SBATCH --error=logs.ddppo.err
#SBATCH --gres gpu:1
#SBATCH --nodes 1
#SBATCH --cpus-per-task 10
#SBATCH --ntasks-per-node 1
#SBATCH --mem=60GB
#SBATCH --time=12:00
#SBATCH --signal=USR1@600
#SBATCH --partition=dev

export MAGNUM_LOG=quiet
export HABITAT_SIM_LOG=quiet
set -x
python -u -m habitat_baselines.run \
   --exp-config habitat-baselines/habitat_baselines/config/ovmm/rl_skill.yaml \
   --run-type train benchmark/ovmm=<skill_name> \
   habitat_baselines.checkpoint_folder=data/new_checkpoints/ovmm/<skill_name>
```


## Running evaluations


### Evaluate with ground truth semantics
```
# Evaluation on complete episode dataset with GT semantics
python projects/habitat_ovmm/eval_baselines_agent.py

# Print out the metrics
python projects/habitat_ovmm/scripts/summarize_metrics.py
```

### Evaluate with DETIC
Ensure `GROUND_TRUTH_SEMANTICS:0` in `configs/env/hssd_eval.yaml` before running the above command

### Evaluate on specific episodes
```
python projects/habitat_ovmm/eval_baselines_agent.py habitat.dataset.episode_ids="[151,182]"
```

### Evaluate all baseline variants
1. First generate all possible configs using the base config `configs/agent/hssd_eval.yaml`. Configs will be saved under `projects/habitat_ovmm/configs/agent/generated`
```
python projects/habitat_ovmm/scripts/gen_configs.py
```

2. Run evaluation using the generated config files
```
python projects/habitat_ovmm/eval_baselines_agent.py --baseline_config_path projects/habitat_ovmm/configs/agent/generated/<dir_name>/<manip>_m_<nav>_n_<perception><viz?>.yaml
```
