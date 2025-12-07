# CAP6415_F25_project-Saliency-Correlation
Large-scale receptive field to saliency correlation study

## Setup Instructions (PLEASE FOLLOW SEQUENTIALLY)

### (Recommended) If you DON'T want to manually process all 5000 images from scratch:
1. Download the main notebook in the repository. Link provided here for ease: [Main Notebook](https://github.com/szhou2024/CAP6415_F25_project-Saliency-Correlation/blob/main/EigenCAM_on_fastRCNN.ipynb)
2. Download backup.npy in results subfolder. Link provided here for ease: [backup](https://github.com/szhou2024/CAP6415_F25_project-Saliency-Correlation/blob/main/results/backup.npy)
3. In the notebook, change save_path variable to where you downloaded to '.../your_path/backup.npy'
4. In the notebook, set recovery_mode = True
5. Run the entire notebook

### If you DO want to manually process all 5000 images from scratch:
1. Download the main notebook in the repository. Link provided here for ease: [Main Notebook](https://github.com/szhou2024/CAP6415_F25_project-Saliency-Correlation/blob/main/EigenCAM_on_fastRCNN.ipynb)
2. Download the two datasets. First link below is the original images in SALICON validation set. Second link is the ground truth maps
   
  [Images](https://s3.amazonaws.com/salicon-dataset/2015r1/val.zip)

  [Ground Truth Saliency Maps](https://drive.google.com/a/umn.edu/uc?id=1PnO7szbdub1559LfjYHMy65EDC4VhJC8&export=download)

3. For images, extract entire val folder locally to a directory.
4. In the notebook, change path_to_images variable to that directory.
5. For ground truths, extract just the val folder locally to a directory.
6. In the notebook, change path_to_maps variable to that directory.
7. Run entire notebook

## Requirements
See requirements.txt in repository if you want to use that, but notebook has a !pip section that will install the requirements and versions.

## Usage 
Recommended way to recreate results file is to simply follow the first section of setup instructions above (uses backup.npy).
If you want to recreate the entire process, follow the second section of setup instructions above.
After you're done setting up, simply run the entire notebook.

## Preface
abc

## Abstract
abc

## Pipeline
abc

## Results
abc

## Acknowledgements
abc

## References/Sources
abc

## License
This project is purely for academic purposes as part of CAP6415 coursework requirement.
