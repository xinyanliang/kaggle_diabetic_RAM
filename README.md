# Kaggle Diabetic Retinopathy Detection
### Introduction
This codes are an extension for the Kaggle Diabetic Retinopathy Detection competitation with the support of RAM (Regression Activation Map) to localize the ROI which contributing to the specific severities of DR. A breif stats of the performance of the model is summarized as 

|                                  |          |          | 
|----------------------------------|----------|----------| 
|                                  |Baseline  |  ours    | 
| Test Kappa (Public Leaderboard)  | 0.85425  | 0.85038  | 
| Test Kappa (Private Leaderboard) | 0.84479  | 0.84118  | 
| Parameter # (net-5)              | 12.4M    | 9.7M     | 
| Training time (second/epoch)     | 422.1    | 367.3    | 
| Parameter # (net-4)              | 12.5M    | 9.8M     | 
| Training time (second/epoch)     | 451.7    | 398.2    | 
| Support RAM for visual explaination                   | No       | Yes    |

We heavily adopt the solution from https://github.com/sveitser/kaggle_diabetic, bravos to the author!

The example RAM generated from the neural networks on 128 and 256 pixel images area as below.
![levelRAM](https://www.dropbox.com/s/8nc89ovymsn0f49/levelRAM.png?dl=1)

For the mild-conditioned patients, RAM learned to discover the narrowing of the retinal arteries associated with
reduced retinal blood flow (Figure (d)), where the vessel
shows dark red. The dysfunction of the neurons of the inner
retina, followed in later stages (moderate) by changes in the
function of the outer retina are captured in Figure (c), as
such dysfunction protects the retina from many substances
in the blood (including toxins and immune cells), leading to
the leaking of blood constituents into the retinal neuropile.
When the patients are round the next stage (severe), as the
basement membrane of the retinal blood vessels thickens,
capillaries degenerate and lose cells leading to loss of blood
flow and progressive ischemia and microscopic aneurysm-
s which appear as balloon-like structures jutting out from
the capillary walls. RAM, as shown in Figure (b), learned
to converge its focus on the border where the balloon-like
structures occurs. As the disease progresses to the proliferative stage, the lack of oxygen in the retina causes fragile,
new, blood vessels to grow along the retina and in the clear,
gel-like vitreous humour that fills the inside of the eye. In
Figure (a), RAM shows the model put its attention on the
grey dots scattering around, which undoubtly demonstrate
the proliferative stage. We also note that if the patient has
no DR and the score predicted by the model is smaller than
0.5, then the RAM uniformly shows the dot-like focus near
the pupil ((e)).

More detailed about RAM please see the CAM.ipynb and our report:

**Diabetic Retinopathy Detection via Deep Convolutional Networks for Discriminative Localization and Visual Explanation.**<br />
Zhiguang Wang, Jianbo Yang <br />
https://arxiv.org/pdf/1703.10757


### Installation

Extract train/test images to `data/train` and `data/test` respectively and
put the `trainLabels.csv` file into the `data` directory as well.

Install python2 dependencies via,
```
pip install -r requirements.txt
```
You need a CUDA capable GPU with at least 4GB of video memory and
[CUDNN](https://developer.nvidia.com/cudnn) installed.

If you'd like to run a deterministic variant you can use the `deterministic`
branch. Note that the branch has its own `requirements.txt` file.
In order to achieve determinism cuda-convnet is used for convolutions instead
of cuDNN. The deterministic version increases the GPU memory requirements
to 6GB and takes about twice as long to run.

The project was developed and tested on arch linux and hardware with a i7-2600k CPU,
GTX 970 and 980Ti GPUs and 32 GB RAM. You probably need at least 8GB of RAM as
well as up to 160 GB of harddisk space (for converted images, network
parameters and extracted features) to run all the code in this repository.

### Usage
#### Generating the kaggle solution
A commented bash script to generate our final 2nd place solution can be found
in `make_kaggle_solution.sh`.

Running all the commands sequentially will probably take 7 - 10 days on recent
consumer grade hardware. If you have multiple GPUs you can speed things up
by doing training and feature extraction for the two networks in parallel.
However, due to the computationally heavy data augmentation it may be far less than
twice as fast especially when working with 512x512 pixel input images.

You can also obtain a quadratic weighted kappa score of 0.839 on the private
leaderboard by just training the 4x4 kernel networks and by performing only 20
feature extraction iterations with the weights that gave you the best MSE
validation scores during training. The entire ensemble only achieves a
slightly higher score of 0.845.

#### Scripts
All these python scripts can be invoked with `--help` to display a brief help
message. They are meant to be executed in the order,

- `convert.py` crops and resizes images
- `train_nn.py` trains convolutional networks
- `transform.py` extracts features from trained convolutional networks
- `blend.py` blends features, optionally blending inputs from both patient eyes

##### convert.py
Example usage:
```
python convert.py --crop_size 128 --convert_directory data/train_tiny --extension tiff --directory data/train
python convert.py --crop_size 128 --convert_directory data/test_tiny --extension tiff --directory data/test
```
```
Usage: convert.py [OPTIONS]

Options:
  --directory TEXT          Directory with original images.  [default: data/train]
  --convert_directory TEXT  Where to save converted images.  [default: data/train_res]
  --test                    Convert images one by one and examine them on screen.  [default: False]
  --crop_size INTEGER       Size of converted images.  [default: 256]
  --extension TEXT          Filetype of converted images.  [default: tiff]
  --help                    Show this message and exit
```
##### train_nn.py
Example usage:
```
python train_nn.py --cnf configs/c_128_5x5_32.py
python train_nn.py --cnf configs/c_512_5x5_32.py --weights_from weigts/c_256_5x5_32/weights_final.pkl
```
```
Usage: train_nn.py [OPTIONS]

Options:
  --cnf TEXT           Path or name of configuration module.  [default: configs/c_512_4x4_tiny.py]
  --weights_from TEXT  Path to initial weights file.
  --help               Show this message and exit.
```

##### transform.py
Example usage:
```
python transform.py --cnf config/c_128_5x5_32.py --train --test --n_iter 5
python transform.py --cnf config/c_128_5x5_32.py --n_iter 5 --test_dir path/to/other/image/files
python transform.py --test_dir path/to/alternative/test/files
```
```
Usage: transform.py [OPTIONS]

Options:
  --cnf TEXT           Path or name of configuration module.  [default: configs/c_512_4x4_32.py]
  --n_iter INTEGER     Iterations for test time averaging.  [default: 1]
  --skip INTEGER       Number of test time averaging iterations to skip. [default: 0]
  --test               Extract features for test set. Ignored if --test_dir is specified.  [default: False]
  --train              Extract features for training set.  [default: False]
  --weights_from TEXT  Path to weights file.
  --test_dir TEXT      Override directory with test set images.
  --help               Show this message and exit.
```
##### blend.py
Example usage:
```
python blend.py --per_patient # use configuration in blend.yml
python blend.py --per_patient --feature_file path/to/feature/file
python blend.py --per_patient --test_dir path/to/alternative/test/files

```
```
Usage: blend.py [OPTIONS]

Options:
  --cnf TEXT            Path or name of configuration module.  [default: configs/c_512_4x4_32.py]
  --predict             Make predictions on test set features after training. [default: False]
  --per_patient         Blend features of both patient eyes.  [default: False]
  --features_file TEXT  Read features from specified file.
  --n_iter INTEGER      Number of times to fit and average.  [default: 1]
  --blend_cnf TEXT      Blending configuration file.  [default: blend.yml]
  --test_dir TEXT       Override directory with test set images.
  --help                Show this message and exit.
```

#### Configuration

- The convolutional network configuration is done via the files in the `configs` directory.
- To select different combinations of extracted features for blending edit  `blend.yml`.
- To tune parameters related to blending edit `blend.py` directly.
- To make predictions for a different test set either
  + put the resized images into the `data/test_medium` directory
  + or edit the `test_dir` field in your config file(s) inside the `configs` directory
  + or pass the `--test_dir /path/to/test/files` argument to `transform.py` and `blend.py`


#### 运行问题解决方案

1. 安装sharedarray报错,具体描述间 [link](https://github.com/sveitser/kaggle_diabetic/issues/7) 

解决方案

```python
python -c "import numpy; print numpy.__file__"
```
将打印出的numpy路径替换到下面命令中所对应的numpy路径中
``` shell
export CFLAGS=-I/home/student/anaconda3/envs/lxy_p2/lib/python2.7/site-packages/numpy/core/include
```

2. 改代码必须运行在GPU上，否者报错.

解决方案：

安装完指定版本Theano后，在自己的home目录下创建文件 .theanorc

```
[global]
floatX = float32
device = gpu

```

# kaggle_diabetic_RAM




