# Face Analysis for ComfyUI

This extension uses [DLib](http://dlib.net/) or [InsightFace](https://github.com/deepinsight/insightface) to perform various operations on human faces.

The most obvious is to calculate the similarity between two faces. The best way to evaluate generated faces is to first send a batch of 3 reference images to the node and compare them to a forth reference (all actual pictures of the person). That will give you a baseline number that you can use to compare to generated images.

> [!IMPORTANT]  
> **2025.04.14** - I do not use ComfyUI as my main way to interact with Gen AI anymore as a result I'm setting the repository in "maintenance only" mode. If there are crucial updates or PRs I might still consider merging them but I do not plan any consistent work on this repo.

## Installation

You need to install either InsightFace or Dlib (or both).

For DLIB download [Shape Predictor](https://huggingface.co/matt3ounstable/dlib_predictor_recognition/resolve/main/shape_predictor_68_face_landmarks.dat?download=true), [Face Predictor 5 landmarks](https://huggingface.co/matt3ounstable/dlib_predictor_recognition/resolve/main/shape_predictor_5_face_landmarks.dat?download=true), [Face Predictor 81 landmarks](https://huggingface.co/matt3ounstable/dlib_predictor_recognition/resolve/main/shape_predictor_81_face_landmarks.dat?download=true) and the [Face Recognition](https://huggingface.co/matt3ounstable/dlib_predictor_recognition/resolve/main/dlib_face_recognition_resnet_model_v1.dat?download=true) models and place them into the `dlib` directory.

Precompiled Dlib for Windows can be found [here](https://github.com/z-mahmud22/Dlib_Windows_Python3.x).

![face analysis](./face_analysis.jpg)

The extension also supports [AuraFace](https://huggingface.co/fal/AuraFace-v1/tree/main) that is a free alternative to InsightFace. Download all the files and place them under `models/insightface/models/auraface/`

## Nodes

### Face Bounding Box Advanced

An enhanced version of **Face Bounding Box** with multi-face selection modes, pre-filtering, and structured JSON output.

#### Inputs

| Name | Type | Description |
|------|------|-------------|
| `analysis_models` | ANALYSIS_MODELS | Face analysis model loaded by the **Face Analysis Models** node |
| `image` | IMAGE | Input image(s), batch supported |
| `filter_to_top_n_by_size` | INT | Global pre-filter: keep only the N largest faces before sorting and selection. `0` or `-1` = disabled |
| `padding` | INT | Extra pixels added around each detected face bbox |
| `padding_percent` | FLOAT | Extra padding as a fraction of face width/height, applied on top of `padding` |
| `sort_mode` | `size` / `position_horizontal` | `size`: sort faces largest-first. `position_horizontal`: sort faces left-to-right by center X |
| `index` | INT | Select the Nth face from the sorted list. `-1` returns all faces |

#### Outputs

| Name | Type | Description |
|------|------|-------------|
| `indexed_face` | IMAGE | Face(s) selected by `index`. Returns all faces when `index = -1` |
| `face_0` | IMAGE | 1st face in sorted order. Falls back to itself if fewer faces detected |
| `face_1` | IMAGE | 2nd face in sorted order. Falls back to `face_0` if not available |
| `face_2` | IMAGE | 3rd face in sorted order. Falls back to `face_1` if not available |
| `data_json` | STRING | JSON summary of the detection results (see below) |

#### data_json format

```json
{
  "filter_to_top_n_by_size": 3,
  "sort_mode": "position_horizontal",
  "num_faces": 3,
  "indexed_face": {
    "requested_index": 1,
    "actual_rank": 1,
    "x": 150, "y": 25, "width": 95, "height": 110
  },
  "faces": [
    {"rank": 0, "x": 10,  "y": 20, "width": 100, "height": 120},
    {"rank": 1, "x": 150, "y": 25, "width": 95,  "height": 110},
    {"rank": 2, "x": 300, "y": 30, "width": 88,  "height": 105}
  ]
}
```

- `num_faces`: face count after `filter_to_top_n_by_size` is applied
- `indexed_face`: the face selected by `index` (`null` when `index = -1`)
- `actual_rank` may differ from `requested_index` when the index exceeds the number of detected faces (clamped to last)
- `faces`: all faces after filtering, sorted by `sort_mode` — independent of `index`
