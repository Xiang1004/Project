# Object Detection - Distance Estimation
#### Purpose
- 開發一個基於 RGB 影像的物件偵測模型，不僅能準確辨識各類物件，還能同步預測其距離資訊。此模型將應用於先進駕駛輔助系統 (ADAS, Advanced Driver Assistance Systems)。

#### Method
- YOLOv9: Learning What You Want to Learn Using Programmable Gradient Information 

    https://arxiv.org/abs/2402.13616

    https://github.com/WongKinYiu/yolov9?tab=readme-ov-file

- Dist-YOLO: Fast Object Detection with Distance Estimation

    https://github.com/billcao2000/yolo-distance

    https://github.com/billcao2000/yolov8-distance


### Modifications
```
- Learning Rate (Warm up) (Cosine annealing)
- Loss Function (distance loss : MSE -> Huber -> Log-cosh)
- Architecture (distance output head : add batchnorm、residual)
- Remove PGI & adjust loss weight
```

### Performance
|     Bbox                 | Precision | Recall | F1 Score | Dist. error |
|--------------------------|:---------:|:------:|:--------:|:-----------:|
|        Original          |   86.6%   |  62.7% |   0.727  |    54.9%    |
|        Augmentation      |   88.1%   |  49.2% |   0.631  |    28.0%    |
|       Pre-process        |   91.5%   |  67.3% |   0.776  |    25.0%    |
|       bbox to dist       |   90.0%   |  69.9% |   0.787  |    14.6%    |
| dist loss add iou weight |   90.1%   |  81.0% |   0.853  |     6.5%    |
| new datasets structure   |   92.7%   |  89.2% |   0.909  |     4.4%    |

------------------------------------------------------------------

#### Loss function
|  Distance loss  | Dist. error |
|-----------------|:-----------:|
|      MSE        |    31.2%    |
|     Huber       |    32.1%    |
|    Log-Cosh     |    29.9%    |
------------------------------------------------------------------

#### Distance detection head
|             |   F1 Score   | Dist. error |
|-------------|:------------:|:-----------:|
|    ReLU     |    82.2%     |    10.8%    |
|    SiLU     |    83.6%     |    10.8%    |
|  Short cut  |    83.6%     |    10.8%    |
|  Residual   |    83.6%     |    10.8%    |
------------------------------------------------------------------

#### Training Distance Loss weight
|         | F1 Score | Dist. error |
|---------|:--------:|:-----------:|
| 1:1:1:1 |   0.822  |    12.5%    |
| 1:1:2:2 |   0.822  |    10.8%    |
| 1:1:5:4 |   0.810  |    10.6%    |
| 1:1:1:3 |   0.837  |    10.7%    |

#### Remove Outliers
Pedestrian： 0-110 m -> ±1.5 σ <br>
Cycle, Car： 0-10 m -> ±1.5 σ 、10-110 m -> ±2.0 σ

|  Bbox | Precision | Recall | F1 Score | Dist. error |
|-------|:---------:|:------:|:--------:|:-----------:|
|  Old  |   90.0%   |  72.7% |   0.803  |    12.1%    |
|  New  |   90.0%   |  70.9% |   0.793  |    10.7%    |
------------------------------------------------------------------


### Ineffective
- Distance head architecture (More Residual block) 
- Activation Function (SiLU -> ReLU)
- Optimizer (SGD -> Adam、AdamW)


## Other Algorithm

### Reparameterization
#### Purpose
- 移除programmable gradient information，減少參數量
#### Method
- 以gelan架構取代yolov9，移除auxiliary architecture
---

### Visualization
#### Purpose
- Demo 2D 預測的結果，以利更直觀分析結果和修正模型
#### Method
- OpenCV
- Matplotlib
---

### Evaluation
#### Purpose
- 計算 2D inference 的 Precision / Recall / F1 score / Dist. error
- 最佳化信心分數閾值，得出最佳F1 score
#### Method
- IoU (2D : area)
- Distance (per 10m)
- Bbox Size (Tiny / Small / Medium / Large / Extra Large)
- Class (Pedestrian / Cycle / Car)
---

### Exporter
#### Purpose
- 將訓練好的模型轉成多種格式，以便適應不同的部署環境或框架需求。
- PyTorch(.pt)、TorchScript(.torchscript)、ONNX(.onnx)、TensorFlow(.pb)
#### Method
- 讀取原始模型權重和架構，根據目標格式重新構建模型結構。
- example : pt file -> torchscript file (torch.load、torch.jit.trace)
---

### Data Clean
#### Purpose
- 將ground truth因人為標記、修正錯誤的物件影像挑出，重新標記。
#### Method
- 透過模型inference後，產出FP(img_id、class_id、bbox(x,y,w,h))和FN(img_id、obj_id)
---


