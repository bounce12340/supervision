<div align="right">

🌐 **語言：** English | [繁體中文](README.zh-TW.md)

</div>

<div align="center">
  <p>
    <a align="center" href="" target="https://supervision.roboflow.com">
      <img
        width="100%"
        src="https://media.roboflow.com/open-source/supervision/rf-supervision-banner.png?updatedAt=1678995927529"
      >
    </a>
  </p>

<br>

[notebooks](https://github.com/roboflow/notebooks) | [inference](https://github.com/roboflow/inference) | [autodistill](https://github.com/autodistill/autodistill) | [maestro](https://github.com/roboflow/multimodal-maestro)

<br>

[![version](https://badge.fury.io/py/supervision.svg)](https://badge.fury.io/py/supervision)
[![downloads](https://img.shields.io/pypi/dm/supervision)](https://pypistats.org/packages/supervision)
[![license](https://img.shields.io/pypi/l/supervision)](LICENSE.md)
[![python-version](https://img.shields.io/pypi/pyversions/supervision)](https://badge.fury.io/py/supervision)
[![codecov](https://codecov.io/gh/roboflow/supervision/graph/badge.svg?token=HMNJ5FVZ36)](https://codecov.io/gh/roboflow/supervision)

[![snyk](https://snyk.io/advisor/python/supervision/badge.svg)](https://snyk.io/advisor/python/supervision)
[![colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/roboflow/supervision/blob/main/demo.ipynb)
[![gradio](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Spaces-blue)](https://huggingface.co/spaces/Roboflow/Annotators)
[![discord](https://img.shields.io/discord/1159501506232451173?logo=discord&label=discord&labelColor=fff&color=5865f2&link=https%3A%2F%2Fdiscord.gg%2FGbfgXGJ8Bk)](https://discord.gg/GbfgXGJ8Bk)

<div align="center">
    <a href="https://trendshift.io/repositories/124"  target="_blank"><img src="https://trendshift.io/api/badge/repositories/124" alt="roboflow%2Fsupervision | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>
  </div>

</div>

## 👋 歡迎

**我們為您編寫可重複使用的電腦視覺工具。** 無論您是想要從硬碟載入資料集、在圖片或影片上繪製偵測框，或是計算某個區域內有多少偵測結果。您都可以信賴我們！🤝

## 💻 安裝

在 [**Python>=3.9**](https://www.python.org/) 環境中使用 pip 安裝 supervision 套件。

```bash
pip install supervision
```

閱讀我們的[指南](https://roboflow.github.io/supervision/)了解更多關於 conda、mamba 以及從原始碼安裝的資訊。

## 🔥 快速開始

### 模型

Supervision 的設計與模型無關。只要插入任何分類、偵測或分割模型即可。為了您的便利，我們為最流行的函式庫如 Ultralytics、Transformers、MMDetection 或 Inference 建立了[連接器](https://supervision.roboflow.com/latest/detection/core/#detections)。其他整合如 `rfdetr` 已經直接回傳 `sv.Detections`。

使用 `pip install pillow rfdetr` 安裝此範例所需的選用相依套件。

```python
import supervision as sv
from PIL import Image
from rfdetr import RFDETRSmall

image = Image.open(...)
model = RFDETRSmall()
detections = model.predict(image, threshold=0.5)

len(detections)
# 5
```

<details>
<summary>👉 更多模型連接器</summary>

- inference

    使用 [Inference](https://github.com/roboflow/inference) 需要 [Roboflow API KEY](https://docs.roboflow.com/api-reference/authentication#retrieve-an-api-key)。

    ```python
    import supervision as sv
    from PIL import Image
    from inference import get_model

    image = Image.open(...)
    model = get_model(model_id="rfdetr-small", api_key="ROBOFLOW_API_KEY")
    result = model.infer(image)[0]
    detections = sv.Detections.from_inference(result)

    len(detections)
    # 5
    ```

</details>

### 註解器

Supervision 提供廣泛的高度可自訂[註解器](https://supervision.roboflow.com/latest/detection/annotators/)，讓您可以為您的使用案例組合完美的視覺化效果。

```python
import cv2
import supervision as sv

image = cv2.imread(...)
detections = sv.Detections(...)

box_annotator = sv.BoxAnnotator()
annotated_frame = box_annotator.annotate(scene=image.copy(), detections=detections)
```

### 資料集

Supervision 提供一組[工具](https://supervision.roboflow.com/latest/datasets/core/)，讓您可以在其中一種支援的格式中載入、分割、合併和儲存資料集。

```python
import supervision as sv
from roboflow import Roboflow

project = Roboflow().workspace("WORKSPACE_ID").project("PROJECT_ID")
dataset = project.version("PROJECT_VERSION").download("coco")

ds = sv.DetectionDataset.from_coco(
    images_directory_path=f"{dataset.location}/train",
    annotations_path=f"{dataset.location}/train/_annotations.coco.json",
)

path, image, annotation = ds[0]
# 按需載入圖片

for path, image, annotation in ds:
    # 按需載入圖片
    pass
```

<details close>
<summary>👉 更多資料集工具</summary>

- 載入

    ```python
    dataset = sv.DetectionDataset.from_yolo(
        images_directory_path=...,
        annotations_directory_path=...,
        data_yaml_path=...,
    )

    dataset = sv.DetectionDataset.from_pascal_voc(
        images_directory_path=...,
        annotations_directory_path=...,
    )

    dataset = sv.DetectionDataset.from_coco(
        images_directory_path=...,
        annotations_path=...,
    )
    ```

- 分割

    ```python
    train_dataset, test_dataset = dataset.split(split_ratio=0.7)
    test_dataset, valid_dataset = test_dataset.split(split_ratio=0.5)

    len(train_dataset), len(test_dataset), len(valid_dataset)
    # (700, 150, 150)
    ```

- 合併

    ```python
    ds_1 = sv.DetectionDataset(...)
    len(ds_1)
    # 100
    ds_1.classes
    # ['dog', 'person']

    ds_2 = sv.DetectionDataset(...)
    len(ds_2)
    # 200
    ds_2.classes
    # ['cat']

    ds_merged = sv.DetectionDataset.merge([ds_1, ds_2])
    len(ds_merged)
    # 300
    ds_merged.classes
    # ['cat', 'dog', 'person']
    ```

- 儲存

    ```python
    dataset.as_yolo(
        images_directory_path=...,
        annotations_directory_path=...,
        data_yaml_path=...,
    )

    dataset.as_pascal_voc(
        images_directory_path=...,
        annotations_directory_path=...,
    )

    dataset.as_coco(
        images_directory_path=...,
        annotations_path=...,
    )
    ```

- 轉換

    ```python
    sv.DetectionDataset.from_yolo(
        images_directory_path=...,
        annotations_directory_path=...,
        data_yaml_path=...,
    ).as_pascal_voc(
        images_directory_path=...,
        annotations_directory_path=...,
    )
    ```

</details>

## 🎬 教學

想要學習如何使用 Supervision？探索我們的[操作指南](https://supervision.roboflow.com/develop/how_to/detect_and_annotate/)、[端到端範例](./examples)、[速查表](https://roboflow.github.io/cheatsheet-supervision/)和[食譜](https://supervision.roboflow.com/develop/cookbooks/)！

## 💜 使用 Supervision 建立

您使用 supervision 建立了酷炫的東西嗎？[讓我們知道！](https://github.com/roboflow/supervision/discussions/categories/built-with-supervision)

## 📚 文件

造訪我們的[文件](https://roboflow.github.io/supervision)頁面，了解 supervision 如何協助您更快、更可靠地建立電腦視覺應用程式。

## 🏆 貢獻

我們熱愛您的參與！請參閱我們的[貢獻指南](.github/CONTRIBUTING.md)開始參與。感謝 🙏 所有貢獻者！

---

*此為英文版的繁體中文翻譯。如需最新資訊和完整細節，請參閱[英文版 README](README.md)。*