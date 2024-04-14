Created by: Nguyen Quoc Thai

Date: 12/04/2024

Reference: [yolov7 github (author: WongKinYiu)](https://github.com/WongKinYiu/yolov7)

# Table of content
1. [About](#about)
2. [Propose approach](#propose-approach)

    2.1. [Current progress](#current-progress)
3. [Set up Environment](#set-up-environment)

## About

With the idea reducing number of models when deploy on edge device, this repo introduces about the yolov7 backbone custom for the multihead image classification. Since 2 (or more) heads perform the classifications seperately, it require a complete change for the whole project.

## Propose approach

For the new idea, it costs lots of effort to deeply understand the model architecture and function for customizing it. The below is the step by step propose approach for customization.

1. **Model Architecture Modification:** Extend the classification head of the YOLOv8 architecture to include multiple independent heads, one for each attribute you want to predict. Each head would need to be fully connected to the appropriate feature layers and have a number of outputs equal to the number of classes for that particular attribute.

2. **Custom Dataset Preparation:** Prepare your dataset so that each image has multiple labels associated with it, corresponding to each of the heads (Make, Model, and Color). The data format should reflect the multi-label nature of your task.

3. **Loss Function Adaptation:** Modify the loss function to handle multiple classification tasks. You would likely need a separate loss term for each head, and the overall loss would be a weighted sum of these individual losses.

4. **Training Procedure:** Update the training scripts to handle multi-head learning. This includes accommodating the modified data loading process and ensuring gradients are backpropagated appropriately for each head.

5. **Evaluation Metrics:** Implement evaluation metrics that can assess performance for each head separately, as well as overall model performance.

6. **Inference Adjustments:** Modify the inference code to handle the multi-head outputs appropriately, ensuring that each discrete prediction is correctly associated with its corresponding attribute.

### Current progress

Since the limitation in the GPU, I cannot meet the deadline for it. Currently I am on progress of step by step customizing and debugging the code. Some changes were done:

- In the *config file* (yolov7\cfg\training\yolov7-custom.yaml) by adding a new head with the same as the old one since they perform the same function. *nc1* and *nc2* represent to the number of classes for each head.

- Since the output of model was changed, it needs some modifications in the *train.py* file. *nc1* and *nc2* were added and the input of class *Model* is also changed relatively.

- The *yolo.py* file is changed in the *Model* class to meet the requirement of *train.py*.

- Now I am on progress of modifying below functions: *loss*, *forward_one* and analyzing the output provided by the model to modify the class *dataset*.

## Set up Environment

1. Git clone the yolov7 github
```
git clone git clone https://github.com/SkalskiP/yolov7.git
```

2. 
```
# Check if your computer has the GPU
nvidia-smi
```
If not, please set up the cuda tool-kit suitable with current OS and version. For me (Windows 10), can set via [link](https://developer.nvidia.com/cuda-12-1-0-download-archive?target_os=Windows&target_arch=x86_64&target_version=10&target_type=exe_local)

3. Install the python=3.9 and the requirements packages or with conda:
```
conda create -n <env_name> python=3.9
conda activate <env_name>
cd yolov7
pip install -r requirements.txt
# Since yolov7 works with numpy < 2.24
pip install "numpy<1.24.0"
```

4. Download dataset for training. Since it is under the modifications, using the **COCO128** for small debug and train to analyze the model behavior. Dataset can be download from Roboflow [web-page](https://universe.roboflow.com/team-roboflow/coco-128/dataset/2). After downloading, please set-up the folder structure as below:

```
yolov7
    ├───cfg
    ├───coco128
   ...
    
```

5. Download the pretrained weight
```
# Powershell-cli
Invoke-WebRequest -Uri https://github.com/WongKinYiu/yolov7/releases/download/v0.1/yolov7.pt -OutFile ./yolov7.pt
```

```
# Bashshell-cli
curl -O  https://github.com/WongKinYiu/yolov7/releases/download/v0.1/yolov7.pt
```

6. Perform the train step and debug:

```
python train.py --batch 16 --epochs 2 --data ./coco128/data.yaml --weights 'yolov7.pt' --device 0 --cfg ./cfg/training/yolov7-custom.yaml --workers 1
```