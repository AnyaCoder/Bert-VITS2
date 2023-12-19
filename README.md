---
# 详细文档见https://modelscope.cn/docs/%E5%88%9B%E7%A9%BA%E9%97%B4%E5%8D%A1%E7%89%87
domain: #领域：cv/nlp/audio/multi-modal/AutoML
# - cv
tags: #自定义标签
-
datasets: #关联数据集
  evaluation:
  #- damotest/beans
  test:
  #- damotest/squad
  train:
  #- modelscope/coco_2014_caption
models: #关联模型
#- damo/speech_charctc_kws_phone-xiaoyunxiaoyun

## 启动文件(若SDK为Gradio/Streamlit，默认为app.py, 若为Static HTML, 默认为index.html)
deployspec:
  entry_file: webui.py
license: Apache License 2.0
---
#### Prerequisite 

For inference:

> - GRAM >= 2 GiB
> - RAM >= 16 GiB
> - CUDA or CPU

For training:

> - GRAM >= 6 GiB (1 batch size or so)
> - RAM >= 24 GiB
> - CUDA supported (AMD ROCm : linux only)

#### For Quick Start


1. Clone the repository

```bash
 git clone https://www.modelscope.cn/studios/SpicyqSama007/Bert-VITS2-v2.3-clap.git
```

2. Configure necessary environments

```bash
# make sure that you've installed anaconda/miniconda, CUDA (tool kit),
# minimum GRAM >= 2GB
# (python virtual env is ok too, similiar)
# here is the example for miniconda

# open 'miniconda' terminal
# we need python>=3.10 for compatibility
conda create -n vits python=3.10
conda activate vits
cd /d "{path of the project e.g. 'D:/Py/Bert-VITS2-v2.3-clap' }"

# install torch first (CUDA ver >= 12.1)
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# then the requirement
pip install -r requirements.txt

# extra for asr : audio wave -> corresponding transcription
pip install funasr modelscope openi
```

3. Configure yaml file

```bash
# auto-generate a config file
python config.py
# then , a file name 'config.yml' generated at the root of the project
mkdir -p Data/mix/models
# your model
mv G_xxx.pth Data/mix/models
mv config.json Data/mix/

# or manually open 'config.yml'
vi config.yml
```


```yaml
# after `vi config.yml` we get into it
...

# 不填或者填空则路径为相对于项目根目录的路径

# fill the path (relatively to the root)
dataset_path: "Data/mix"
# it means the dataset path is in '{...}/Bert-VITS2-v2.3-clap/Data/mix'

...

# webui webui配置
# 注意， “:” 后需要加空格
webui:
# 推理设备 device
device: "cuda"
# 模型路径 path to the model
model: "models/G_xxxx.pth"
# 配置文件路径
config_path: "config.json"
# 端口号
port: 7860
# 是否公开部署，对外网开放
share: false
# 是否开启debug模式
debug: false
# 语种识别库，可选langid, fastlid
language_identification_library: "langid"

```

​	After configuration, let's start webui.py

```bash
python webui.py
```



#### For Quick Finetuning

> Same as the first 2-steps in 'For Quick Start'

3. if you understand Chinese, you can use my UI, as what the name literally means

```bash
python all_process.py
```

   Or :

3. Configure yaml file

```yaml
# after `vi config.yml` we get into it
# or manually open yml file

...

# 不填或者填空则路径为相对于项目根目录的路径
# fill the path (relatively to the root)
dataset_path: "Data/mix"
# it means the dataset path is in '{...}/Bert-VITS2-v2.3-clap/Data/mix'

...


resample:
  sampling_rate: 44100
  in_dir: "audios/raw" # relatively in '/datasetPath/in_dir'
  out_dir: "audios/wavs"



preprocess_text:
  # Format for single line in the list: 
  # {wav_path}|{speaker_name}|{language}|{text}。
  transcription_path: "filelists/{custom}.list"
  cleaned_path: ""
  train_path: "filelists/train.list"
  val_path: "filelists/val.list"
  config_path: "config.json"
  val_per_lang: 4
  max_val_total: 12
  clean: true

# train 训练配置
# 注意， “:” 后需要加空格
train_ms:

  model: "models"
  # 配置文件路径
  config_path: "config.json"
  # 训练使用的worker，不建议超过CPU核心数
  num_workers: 16
  # 关闭此项可以节约接近50%的磁盘空间，但是可能导致实际训练速度变慢和更高的CPU使用率。
  spec_cache: True
  # 保存的检查点数量，多于此数目的权重会被删除来节省空间。
  keep_ckpts: 8

 
```

After editing , save and quit.

4. Then, we should configure the training 'config.json'

```bash
# or manually open
vi Data/mix/config.json
# if there is no config.json, copy one piece from 'configs/config.json'
cp configs/config.json Data/mix/

# pay attention to the following part
```

```json
 "data": {
    "training_files": "Data/mix/filelists/train.list",
    "validation_files": "Data/mix/filelists/val.list",
    "max_wav_value": 32768.0,
    ...
 }
```

Then, save and quit.

5. Come back to the Terminal ( it's fast and easy), type the following command.

```bash
# your source audio files (only .wav)
mv "{your_audios_folder}" Data/mix/audios/raw
# "{custom}.list" contains your transcriptions of the wav files
mkdir -p Data/mix/filelists
mv "{custom}.list"  Data/mix/filelists
python resample.py
python preprocess_text.py
python bert_gen.py
python clap_gen.py

torchrun \
    --nnodes=1\
    --nproc_per_node=2\
    train_ms.py
```

6. If all normal and successful, we can get the trained models in `Data/mix/model/G_xxxx.pth`

   Tips: 

   > 1. You can terminate the training process at any time by yourself. (after 'Saving models...')
   > 2. Most of the problems take place with 'incorrect path '
