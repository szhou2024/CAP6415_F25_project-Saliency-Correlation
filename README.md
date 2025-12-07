# CAP6415_F25_project-Saliency-Correlation
Large-scale receptive field to saliency correlation study. [Click here to go to Preface section](./#Preface)
## Setup Instructions (PLEASE FOLLOW SEQUENTIALLY)
For all with ⬇️ download image, you can just right click save as link for ease. Otherwise, just click to be directed to that link.

### (Recommended) If you DON'T want to manually process all 5000 images from scratch:
1. Download the main notebook in the repository. Link provided here for ease: [⬇️Main Notebook](https://github.com/szhou2024/CAP6415_F25_project-Saliency-Correlation/blob/main/EigenCAM_on_fastRCNN.ipynb)
2. Download backup.npy in results subfolder. Link provided here for ease: [⬇️backup](https://github.com/szhou2024/CAP6415_F25_project-Saliency-Correlation/blob/main/results/backup.npy)
3. In the notebook, change save_path variable to where you downloaded to '.../your_path/backup.npy'
4. In the notebook, set recovery_mode = True
5. Run the entire notebook

### If you DO want to manually process all 5000 images from scratch:
1. Download the main notebook in the repository. Link provided here for ease: [⬇️Main Notebook](https://github.com/szhou2024/CAP6415_F25_project-Saliency-Correlation/blob/main/EigenCAM_on_fastRCNN.ipynb)
2. Download the two datasets. First link below is the original images in SALICON validation set. Second link is the ground truth maps
   
  [⬇️SALICON Source Images](https://s3.amazonaws.com/salicon-dataset/2015r1/val.zip)

  [Ground Truth Saliency Maps](https://drive.usercontent.google.com/download?id=1YWiwRNHokV2TIUBsNhAeRo1aqz8Iq0In&export=download&authuser=0)

3. For Images, extract entire val folder locally to a directory.
4. In the notebook, change path_to_images variable to that directory.
5. For Ground Truths, extract just the val folder locally to a directory.
6. In the notebook, change path_to_maps variable to that directory.
7. Run entire notebook

## Requirements
Python 3.9.6 utilized in the project, but no issues running on versions after this. 
See requirements.txt in repository if you want to use that, but notebook has a !pip section that will install the requirements and versions.

## Usage 
Recommended way to recreate results file is to simply follow the first section of setup instructions above (uses backup.npy).
If you want to recreate the entire forward pass process, follow the second section of setup instructions above.
After you're done setting up, simply run the entire notebook.

## Preface
Many SOTA models have come out to predict human fixation/saliency, including the ever popular DeepGaze models. These models combine image processing with fixation sequencing to come out with predictions that do quite a good job on benchmark datasets.

However, instead of a "Deep Gaze," I would like to explore a framework that can similuate a "quick gaze" and compare saliency that way. There are inherent flaws in both methodology and comparison of "quick gaze" to ground truth human fixation/saliency maps. Nevertheless, I believe it will be interesting to explore.

## Abstract
The goal of this project is to compare similarities and differences between CAM generated saliency maps versus SALICON human saliency maps. To represent the aforementioned "quick gaze," I decided to implement quick  

## Methodology/Pipeline
As described above, this project produces saliency maps by combining FasterRCNN + FPNs with EigenCAM to generate saliency maps. Pipeline for image processing is below:

**Source Image** → Input Tensor → FasterRCNN + FPN model → Extract FPN layers → EigenCAM on all 5 FPN layers → **Saliency Map**

The resulting saliency map is compared to ground truth human saliency maps on the same image, and the metrics stored in a dictionary.

Some notable things:
* Normally, with grad-cam library, you would pass a list of layers to hook onto and produce averaged CAMs across the layers. Unfortunately, FPNs are not easy to hook in PyTorch, so the workaround is the following:
  
   * Flatten all 5 FPN layers into 1 giant "layer"
   * Resize all spatial dimensions to match (default using fasterrcnn_reshape_transform is pool layer dimensions)
   * Conduct PCA as EigenCAM would on all channels in this layer
     
* The model itself (FasterRCNN + FPNs) is designed to create FPNs for the purpose of object detection. However, this should not matter much, as I am simply using the backbone and not propogating the model forward past FPNs
* Ground truth saliency maps apply a gaussian blur of σ~19. I do the same to initial EigenCAM output, and in the notebook, it will be referred to as "smoothed CAM"
* Standard correlation metric (used in papers/benchmarks) inflate correlation because 0's are being processed as not null values, but 0's itself. This renders intersection masking that standard libraries have useless. Instead, I apply a cc_mask, which is correlation based on union mask, but 0's are counted as nulls instead of 0.

## Results
For a more detailed look at the results, click below to navigate to it:

[Click here to go to results](./results)

To summarize:
* Results generally reflect model backbone training bias (imagenet datasets)
* Saliency maps on single object, non-occluded images tend to be highly correlated to ground truths.
* Maps on high texture and multiple different object classes tend to be negatively correlated with ground truths.
* The standard text bias is not present in CAMs, but is present in ground truths.
* Human bias is not 

## Acknowledgements
* Jacob Gil for his popular EigenCAM for YOLO5 notebook. Implementation of this project is based heavily this framework, with some adjustments. Link the notebook below:

[[https://github.com/jacobgil/pytorch-grad-cam/blob/master/tutorials/EigenCAM%20for%20YOLO5.ipynb](https://github.com/jacobgil/pytorch-grad-cam/blob/master/tutorials/EigenCAM%20for%20YOLO5.ipynb)](https://github.com/jacobgil/pytorch-grad-cam/blob/master/tutorials/Class%20Activation%20Maps%20for%20Object%20Detection%20With%20Faster%20RCNN.ipynb)

* SALICON dataset providers
* Jacob Gil again for his popular grad-cam Python library. While documentation is not the most intuitive, the library itself is incredibly flexible and powerful. Link to library below:

[https://github.com/jacobgil/pytorch-grad-cam/blob/master/pytorch_grad_cam/eigen_cam.py](https://github.com/jacobgil/pytorch-grad-cam/blob/master/pytorch_grad_cam/eigen_cam.py)

## References/Sources
* Muhammad, M. B., & Yeasin, M. (2020). Eigen-CAM: Class Activation Map using Principal Components. arXiv. https://arxiv.org/abs/2008.00299

* Simonyan, K., Vedaldi, A., & Zisserman, A. (2013). Deep inside convolutional networks: Visualising image classification models and saliency maps. arXiv. https://arxiv.org/abs/1312.6034

* Li, X., Zhao, L., Wei, L., Yang, M.-H., Wu, F., Zhuang, Y., Ling, H., & Wang, J. (2015). DeepSaliency: Multi-Task Deep Neural Network Model for Salient Object Detection. arXiv. https://arxiv.org/abs/1506.01497

* Girshick, R. (2015). Fast R-CNN. arXiv. https://arxiv.org/abs/1504.08083 

## License
This project is purely for academic purposes as part of CAP6415 coursework requirement.
