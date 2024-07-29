<p align="center" width="100%">
<img src="docs\share.jpg"  width="80%" height="80%">
</p>


# ShareGemini: Scaling Up Video Caption Data for Multimodal Large Language Models
[![share_gemini-data](https://img.shields.io/badge/share_gemini-data-yellow)](https://huggingface.co/datasets/Share14/ShareGemini)

To enable Multimodal Large Language Models (MLLMs) to understand videos, a large volume of high-quality caption data is crucial for modality alignment. Existing publicly available video caption datasets, however, are insufficient for this purpose, suffering from limited size, short captions, and restricted diversity. Recent benchmarks like [Video-MME](https://video-mme.github.io/) have demonstrated the remarkable video comprehension capabilities of Gemini. Inspired by this, an expert group leverages the Gemini-1.5-Pro API to annotate captions for existing public video datasets ([Webvid](https://github.com/m-bain/webvid), [Kinetic-400](https://github.com/cvdfoundation/kinetics-dataset), [InternVid](https://github.com/OpenGVLab/InternVideo/tree/main/Data/InternVid), [HD-VILA](https://github.com/microsoft/XPretrain/tree/main/hd-vila-100m)). Furthermore, we investigate the scaling laws of these collected caption data by training [Chat-UniVi-7B](https://github.com/PKU-YuanGroup/Chat-UniVi) with different levels of data scale.



## Release
- [2024/07/29] 🔥 ShareGemini-K400 is available [here](https://huggingface.co/datasets/Share14/ShareGemini/tree/main). 
- [2024/06/18] 🔥 ShareGemini-Webvid-core100k is available [here](https://huggingface.co/datasets/Share14/ShareGemini/tree/main). The caption data being prepared for release includes:
  * ShareGemini-Webvid
  * ShareGemini-InternVid
  * ShareGemini-K400
  * ShareGemini-HDVILA



## Data Scaling on ShareGemini-Webvid
We explore data scaling for the annotated 530k ShareGemini-Webvid dataset through the following steps:

1. **Embedding Extraction**: [InternVideo2](https://github.com/OpenGVLab/InternVideo/tree/main/InternVideo2) is used to extract embeddings from the 530k videos.
2. **Clustering & Pruning**: The resulting video embeddings are clustered using the [ToMe](https://github.com/facebookresearch/ToMe) algorithm, generating data subsets of core-400k, core-200k, core-100k, and core-50k.
3. **Training**: An additional training stage is introduced between Stage-1 and Stage-2 of the baseline [Chat-UniVi-7B](https://github.com/PKU-YuanGroup/Chat-UniVi). Different caption subsets above are utilized respectively in this stage to train both the connector and the LLM.
4. **Evaluation**: Comprehensive evaluations are performed on the [Video-MME](https://video-mme.github.io/) benchmark.

### Visulization of Clustering 
Here are the visulization of some clustered videos. It can be observed that videos within the same cluster exhibit significant similarities, while videos from different clusters show notable differences.
| Cluster | Image | 
|---|---|
| cluster_1 | <img src="docs\manwoman.jpg" width="100%" align="center"> |
| cluster_2 | <img src="docs\pool.jpg" width="100%" align="center"> |
| cluster_3 | <img src="docs\rice.jpg" width="100%" align="center"> |
| cluster_4 | <img src="docs\yellow.jpg" width="100%" align="center"> |
    

### Results on Video-MME
The models in 2nd to 6th row differ from the baseline only by the inclusion of an additional training stage using ShareGemini-Webvid. All other aspects, including model architecture, hyperparameters, and data, remain identical. "SG-WV" in the tables represents ShareGemini-Webvid.
#### W/o Subtitles
|   | Model                         | LLM Params | Overall (%) | Short Video (%) | Medium Video (%) | Long Video (%) |
|---|-------------------------------|------------|-------------|-----------------|------------------|----------------|
| 1 | Chat-UniVi-1.5                 |     7B    |  41.2       |      46.3       |       40.3       |     36.9       |
| 2 | Chat-UniVi-1.5 **+50k SG-WV**  |     7B    |  41.8 (+0.6)|      47.6 (+1.3)|       40.2 (-0.1)|     37.7 (+0.8)|
| 3 | Chat-UniVi-1.5 **+100k SG-WV** |     7B    |  43.2 (+2.0)|      49.1 (+2.8)|       41.3 (+1.0)|     39.1 (+2.2)|
| 4 | Chat-UniVi-1.5 **+200k SG-WV** |     7B    |  43.0 (+1.8)|      49.3 (+3.0)|       42.0 (+1.7)|     37.7 (+0.8)|
| 5 | Chat-UniVi-1.5 **+400k SG-WV** |     7B    |  42.5 (+1.3)|      50.2 (+3.9)|       40.0 (-0.3)|     37.3 (+0.4)|
| 6 | Chat-UniVi-1.5 **+530k SG-WV** |     7B    |  42.6 (+1.4)|      49.1 (+2.8)|       39.9 (-0.4)|     38.8 (+1.7)|

#### W/ Subtitles
|   | Model                         | LLM Params | Overall (%) | Short Video (%) | Medium Video (%) | Long Video (%) |
|---|-------------------------------|------------|-------------|-----------------|------------------|----------------|
| 1 | Chat-UniVi-1.5                 |     7B    |  46.3       |      51.4       |       45.2       |      42.3       |
| 2 | Chat-UniVi-1.5 **+50k SG-WV**  |     7B    |  47.6 (+0.7)|      54.2 (+2.8)|       45.7 (+0.5)|      42.8 (+0.5)|
| 3 | Chat-UniVi-1.5 **+100k SG-WV** |     7B    |  47.9 (+1.6)|      52.8 (+1.4)|       47.3 (+2.1)|      43.4 (+1.1)|
| 4 | Chat-UniVi-1.5 **+200k SG-WV** |     7B    |  47.4 (+1.1)|      55.0 (+3.6)|       46.2 (+1.0)|      41.1 (-1.2)|
| 5 | Chat-UniVi-1.5 **+400k SG-WV** |     7B    |  47.6 (+1.3)|      55.0 (+3.6)|       44.8 (-0.4)|      43.0 (+0.7)|
| 6 | Chat-UniVi-1.5 **+530k SG-WV** |     7B    |  47.3 (+1.0)|      53.0 (+1.6)|       46.1 (+0.9)|      42.9 (+0.6)|


### Discussion
We dive into the key findings observed during the evaluation of Video-MME:

- **Effectiveness of Pre-training**: The results demonstrate that pre-training Chat-UniVi-1.5 with a large corpus of high-quality captions significantly enhances its video understanding capabilities, aligning with our initial research motivation.

- **Data Scaling Robustness**: Pretraining with ShareGemini-Webvid enhances the model's performance across Short Video, Medium Video, and Long Video categories on Video-MME. The most robust data scaling improvements are observed with short videos, where model accuracy consistently increases with larger data volumes. This aligns with the fact that Webvid predominantly consists of short videos (< 30s). The upcoming ShareGemini-HDVILA (average duration 5 min) will further improve the model's understanding of medium and long videos.

- **Data Scaling Saturation**: Scaling ShareGemini-Webvid data to 100k instances results in saturation of performance improvements for Chat-UniVi-1.5-7B. This saturation may be attributed to the limited capacity of the 7B model. 

## Citation
If you find it useful for your research and applications, please cite the related webpage using this BibTeX:
```
@misc{sharegemini,
  title={ShareGemini: Scaling Up Video Caption Data for Multimodal Large Language Models},
  url={https://github.com/Share14/ShareGemini},
  author={Share},
  month={June},
  year={2024}
}
```




