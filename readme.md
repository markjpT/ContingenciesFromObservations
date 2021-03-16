# Contingencies From Observations

# Purposes

1. Serve as the accompanying code for ICRA 2021 paper: Contingencies from Observations.
2. A framework for running scenarios with Precog models in Carla.

## Installing Carla

This repository requires Carla 0.9.8. Please navigate to carla.org to download the correct packages, or do the following:
```bash
# Downloads hosted binaries
wget http://carla-assets-internal.s3.amazonaws.com/Releases/Linux/CARLA_0.9.8.tar.gz

# Unpack CARLA 0.9.8 download
tar -xvzf CARLA_0.9.8.tar.gz -C /path/to/your/desired/carla/install
```

Once downloaded, make sure that `CARLAROOT` is set to point to your copy of CARLA:
```bash
export CARLAROOT=/path/to/your/carla/install
```

CARLAROOT should point to the basae directory, such that the output of `ls $CARLAROOT` shows the following files:
```bash
CarlaUE4     CHANGELOG   Engine  Import           LICENSE                        PythonAPI  Tools
CarlaUE4.sh  Dockerfile  HDMaps  ImportAssets.sh  Manifest_DebugFiles_Linux.txt  README     VERSION
```

## Setup

```bash
conda create -n precog python=3.6.6
conda activate precog
# make sure to source this every time after activating, and make sure $CARLAROOT is set beforehand
source precog_env.sh
pip install -r requirements.txt
```
Note that `CARLAROOT` needs to be set and `source precog_env.sh` needs to be run every time you activate the conda env in a new window/shell.

## Running experiments with Precog model

```bash
cd Experiment
python scenario_runner.py \
--enable-inference \
--enable-control \
--enable-recording \
--checkpoint_path [absolute path to model checkpoint] \
--model_path [absolute path to model folder] \
--replan 4 \
--planner_type 0 \
--scenario 0 \
--location 0
```

## Collecting data

First collect data in Carla.

```bash
cd Experiment
python scenario_runner.py \
--enable-collecting \
--scenario 0 \
--location 0  
```

Episode data will be stored to Experiment/Data folder.
Then run

```bash
cd Experiment
python Utils prepare_data.py
```

This will convert the episode data objects into json file per frame, and store them in Data/JSON_output folder.

## Training model

Organize the json files into the following structure:

```md
Custom_Dataset
---train
   ---feed_Episode_1_frame_90.json
   ...
---test
   ...
---val
   ...
```

Modify relevant precog/conf files to insert correct absolute paths.

```md
Custom_Dataset.yaml
esp_infer_config.yaml
esp_train_config.yaml
shared_gpu.yaml
sgd_optimizer.yaml # set training hyperparameters
```

Then run

```bash
export CUDA_VISIBLE_DEVICES=0;
python $PRECOGROOT/precog/esp_train.py \
dataset=Custom_Dataset \
main.eager=False \
bijection.params.A=2 \
optimizer.params.plot_before_train=True \
optimizer.params.save_before_train=True
```


## License
[MIT](https://choosealicense.com/licenses/mit/)
