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

#### Clone with HTTP

```bash
 git clone https://www.modelscope.cn/studios/SpicyqSama007/Bert-VITS2-v2.3-clap.git
```