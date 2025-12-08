# CAP6415_F25_project-Saliency-Correlation
Large-scale receptive field to saliency correlation study. [Click here to go to Preface section](./#Preface)
______
______
## Setup Instructions (PLEASE FOLLOW SEQUENTIALLY)
For all with ⬇️ download image, you can just right click save as link for ease. Otherwise, just click to be directed to that link. There are 3 ways to reproduce the results. The first two ways do not process the 5000 images, and use the results I have provided via backup.npy. If you wish to fully reproduce, follow the instructions of the last method.

### (Recommended) Using Git:
1. Download the repository using:
   `git clone https://github.com/szhou2024/CAP6415_F25_project-Saliency-Correlation-Study.git`
2. Change directory using:
   `cd CAP6415_F25_project-Saliency-Correlation-Study`
3. Make note of download path (can check using pwd). This is the **your_path** used in step 5
4. Open the EigenCAM_on_fastRCNN.ipynb notebook using whichever method you wish (Jupyter, VSC, Colab etc)
5. In the notebook, change **save_path** variable to '**.../your_path**/results/backup.npy'
6. In the notebook, make sure to set **recovery_mode** = True (should be default)
7. Run the entire notebook

### Manually, if you DON'T want to process all 5000 images from scratch :
1. Download the main notebook in the repository. Link provided here for ease: [⬇️Main Notebook](https://github.com/szhou2024/CAP6415_F25_project-Saliency-Correlation/blob/main/EigenCAM_on_fastRCNN.ipynb)
2. Download backup.npy in results subfolder. Link provided here for ease: [⬇️backup](https://github.com/szhou2024/CAP6415_F25_project-Saliency-Correlation/blob/main/results/backup.npy)
3. In the notebook, change **save_path** variable to where you downloaded to '**.../your_path**/backup.npy'
4. In the notebook, set **recovery_mode** = True
5. Run the entire notebook

### Manually, if you DO want to manually process all 5000 images from scratch:
1. Download the main notebook in the repository. Link provided here for ease: [⬇️Main Notebook](https://github.com/szhou2024/CAP6415_F25_project-Saliency-Correlation/blob/main/EigenCAM_on_fastRCNN.ipynb)
2. Download the two datasets. First link below is the original images in SALICON validation set. Second link is the ground truth maps
   
  [⬇️SALICON Source Images](https://s3.amazonaws.com/salicon-dataset/2015r1/val.zip)

  [Ground Truth Saliency Maps](https://drive.usercontent.google.com/download?id=1YWiwRNHokV2TIUBsNhAeRo1aqz8Iq0In&export=download&authuser=0)

3. For Images, extract entire val folder locally to a directory.
4. In the notebook, change **path_to_images** variable to that directory.
5. For Ground Truths, extract just the val folder locally to a directory.
6. In the notebook, change **path_to_maps** variable to that directory.
7. Run entire notebook
______
______
## Requirements
Python 3.9.6 utilized in the project, but no issues running on versions after this. 
See requirements.txt in repository if you want to use that, but notebook has a !pip section that will install the requirements and versions.
______
______
## Usage 
Recommended way to recreate results file is to simply follow the first section of setup instructions above (uses backup.npy).
If you want to recreate the entire forward pass process, follow the second section of setup instructions above.
After you're done setting up, simply run the entire notebook.

## Preface
Many SOTA models have come out to predict human fixation/saliency, including the ever popular DeepGaze models. These models combine image processing with fixation sequencing to come out with predictions that do quite a good job on benchmark datasets.

However, instead of a "Deep Gaze," I would like to explore a framework that can similuate a "quick gaze" and compare saliency that way. There are inherent flaws in both methodology and comparison of "quick gaze" to ground truth human fixation/saliency maps. Nevertheless, I believe it will be interesting to explore.
______
______
## Abstract
The goal of this project is to compare similarities and differences between CAM generated saliency maps versus SALICON human saliency maps. To represent the aforementioned "quick gaze," I decided to implement FasterRCNN + FPNs as the model to process image, and EigenCAM to create a final saliency map.

In a nutshell, the model uses ResNet50 backbone for feature extraction, and then a Feature Pyramid Network (FPN) that produces representations at multiple spatial resolutions. Like an encoder, the FPN represents 'latent representations' that encode what the model has learned about the image at different zoom levels. This would be similar to how human eyes can focus on fine details or see the big picture. The hallmark of FPNs is the ability to capture both fine-grained spatial details (early layers) and high-level semantic information (deep layers).

EigenCAM is a gradient-free method of generating class activation maps (CAMs). Unlike gradient-based CAM methods, EigenCAMs only require forward pass, do not require backpropagation, and do not require class-specific information. EigenCAMs, as the name suggests, perform PCA on the feature maps of a model to identify the most salient spatial regions for the model's predictions. Since the feature maps in this project are the channels of FPN layers, it reduces all the channels to a single spatial heatmap by computing a weighted linear combination of the most dominant activations across all scales. In a nutshell, EigenCAM on FPNs abstractly represents a "blurry impression" saliency map.
______
______
## Methodology/Pipeline
As described above, this project produces saliency maps by combining FasterRCNN + FPNs with EigenCAM to generate saliency maps. Pipeline for image processing is below:

**Source Image** → Input Tensor → FasterRCNN + FPN model → Extract FPN layers → EigenCAM on all 5 FPN layers → **Saliency Map**

The source image dimensions are **480x640** and each FPN layer has **256 channels** of spatial dimensions, ranging from finest (4x downsampling to 120x160) to most coarse (64x downsampling to 8x10). The resulting saliency map is compared to ground truth human saliency maps on the same image, and the metrics stored in a dictionary. 

An important thing to note is that the "EigenCAM on all 5 FPN layers" step in the pipeline requires a massive workaround. In order for EigenCAM to calculate PCA, it expects uniformly shaped tensors. As such, I follow Jacob Gil's methodology and flatten all five layers into one, then resize to same lowest layer dimension (pool layer of 8x10 in this case). The resulting tensor is shape = (1, 1280, 8, 10) and EigenCAM produces a corresponding square matrix for singular value decomposition (SVD) processing. This workaround leads to some inevitable pitfalls, which I will discuss in results section. [Click here to see more about this](./results/Bottom%2025%20Correlated#model--eigencam-design) 

Some notable things:
* Normally, with grad-cam library, you would pass a list of layers to hook onto and produce averaged CAMs across the layers. Unfortunately, FPNs are not easy to hook in PyTorch, so the workaround is the following:
  
   * Flatten all 5 FPN layers into 1 giant "layer"
   * Resize all spatial dimensions of FPN layers to match (default using fasterrcnn_reshape_transform is pool layer dimensions)
   * Conduct PCA as EigenCAM would on all channels in this layer
     
* The model itself (FasterRCNN + FPNs) is designed to create FPNs for the purpose of object detection. However, this should not matter much, as I am simply using the backbone and not propogating the model forward past FPNs
* The model uses pretrained weights, which is something to address in future improvements. Thus, it will have training bias on the images it has been trained on. However, FasterCNN + FPN trains (pre-trained weights) are trained on the same COCO dataset which SALICON images are derived from.
* Ground truth saliency maps apply a gaussian blur of σ~19. I do the same to initial EigenCAM output, and in the notebook, it will be referred to as "smoothed CAM"
* Standard correlation metric (used in papers/benchmarks) inflate correlation because 0's are being processed as not null values, but 0's itself. This renders intersection masking that standard libraries have useless. Instead, I apply a cc_mask, which is correlation based on union mask, but 0's are counted as nulls instead of 0
______
______
## Results
Top 25 most and least correlated CAM generated maps are provided. Additionally, I conduct a depeer dive and take a more detailed look into larger spatial dimensions for the top 25 least correlated images. 

For a more detailed look at the results including image outputs, click below to navigate to it:

[Click here to go to results](./results)

To summarize:
* Results generally reflect model backbone training bias (ImageNet datasets)
* Saliency maps on single object, non-occluded images tend to be highly correlated to ground truths.
* Maps on high texture and multiple different object classes tend to be very negatively correlated with ground truths.
* The standard text bias is not present in CAMs, but is present in ground truths.
* Human bias (particularly human faces) is particularly evident in ground truths (as expected), but not really the case in CAMs. Interestingly, in cat and dog images, both human and CAM show a bias towards faces, suggesting model training bias.
______
______
## Acknowledgements
* Jacob Gil for his popular tutorial on CAM for object detection with FasterRCNN. Implementation of this project is based heavily this framework, with some adjustments. Link the notebook below:

[https://github.com/jacobgil/pytorch-grad-cam/blob/master/tutorials/EigenCAM%20for%20YOLO5.ipynb](https://github.com/jacobgil/pytorch-grad-cam/blob/master/tutorials/EigenCAM%20for%20YOLO5.ipynb)](https://github.com/jacobgil/pytorch-grad-cam/blob/master/tutorials/Class%20Activation%20Maps%20for%20Object%20Detection%20With%20Faster%20RCNN.ipynb)

* SALICON dataset providers
* Jacob Gil again for his popular grad-cam Python library. While documentation is not the most intuitive, the library itself is incredibly flexible and powerful. Link to library below:

[https://github.com/jacobgil/pytorch-grad-cam/blob/master/pytorch_grad_cam/eigen_cam.py](https://github.com/jacobgil/pytorch-grad-cam/blob/master/pytorch_grad_cam/eigen_cam.py)
______
______
## References/Sources
* Muhammad, M. B., & Yeasin, M. (2020). Eigen-CAM: Class Activation Map using Principal Components. arXiv. https://arxiv.org/abs/2008.00299

* Simonyan, K., Vedaldi, A., & Zisserman, A. (2013). Deep inside convolutional networks: Visualising image classification models and saliency maps. arXiv. https://arxiv.org/abs/1312.6034

* Girshick, R. (2015). Fast R-CNN. arXiv. https://arxiv.org/abs/1504.08083

* Ren, S., He, K., Girshick, R., & Sun, J. (2016). Faster R-CNN: Towards real-time object detection with region proposal networks (arXiv preprint arXiv:1506.01497). https://arxiv.org/abs/1506.01497

* Lin, T.-Y., Dollár, P., Girshick, R., He, K., Hariharan, B., & Belongie, S. (2016). Feature Pyramid Networks for Object Detection. arXiv preprint arXiv:1612.03144. https://arxiv.org/abs/1612.03144

* Shibuya, N. (2022, August 21). FPN: Feature Pyramid Network (2016). Naoki Shibuya. https://naokishibuya.github.io/blog/2022-08-21-fpn-2016/

* Molnar, C. (2025). Interpretable Machine Learning: A Guide for Making Black Box Models Explainable (3rd ed.). Retrieved from https://christophm.github.io/interpretable-ml-book/
______
______
## License
This project is purely for academic purposes as part of CAP6415 coursework requirement.
