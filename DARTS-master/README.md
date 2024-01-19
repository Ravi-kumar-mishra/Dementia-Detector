# DenseUnet-based Automatic Rapid brain Segmentation (DARTS)

## Paper associated with the project
[Here](https://arxiv.org/abs/1911.05567) is the paper describing the project and experiments in detail.

## Package
* The DARTS package can be installed using:
```
pip install DARTSeg
```

## Pre-trained model wts
* Download the pretrained models from [here](https://drive.google.com/file/d/1OJ0RmcALNkiU49Npm7Rez6thIKOf3gLQ/view?usp=sharing) as follows:

```
gdown https://drive.google.com/uc?id=1OJ0RmcALNkiU49Npm7Rez6thIKOf3gLQ -O saved_model_wts.zip
unzip saved_model_wts.zip
```
There are two model architectures: Dense U-Net and U-Net. Each model is trained using 2D slices extracted coronally, sagittally,or axially. The name of the model will contain the orientation and model architecture information.

## Using pre-trained models to perform complete brain segmentation

Follow these steps to perform segmentation:

```
from DARTS import Segmentation
seg_obj = Segmentation(model_wts_path='./saved_model_wts/dense_unet_saggital_finetuned.pth', model_type="dense-unet")
seg_out, seg_proba_out = seg_obj.predict(inputs="T1.mgz")
```

* The user may also execute the perform_pred.py script with the following code block to perform segmentation:

```
usage: perform_pred.py [-h] [--input_image_path INPUT_IMAGE_PATH]
                       [--segmentation_dir_path SEGMENTATION_DIR_PATH]
                       [--file_name FILE_NAME] [--model_type MODEL_TYPE]
                       [--model_wts_path MODEL_WTS_PATH] [--is_mgz]

optional arguments:
  -h, --help            show this help message and exit
  --input_image_path INPUT_IMAGE_PATH
                        Path to input image (can be of .mgz or .nii.gz
                        format)(required)
  --segmentation_dir_path SEGMENTATION_DIR_PATH
                        Directory path to save the output segmentation
                        (required)
  --file_name FILE_NAME
                        Name of the segmentation file (required)
  --model_type MODEL_TYPE
                        Model types: "dense-unet", "unet" (default: "dense-
                        unet")
  --model_wts_path MODEL_WTS_PATH
                        Path for model wts to be used, provide a model from
                        saved_model_wts/
  --is_mgz              Use this flag when image is in .mgz format

```
An example could look something like this:

```
perform_pred.py --input_image_path './../../../data_orig/199251/mri/T1.mgz' \
--segmentation_dir_path './sample_pred/' \
--file_name '199251' \
--is_mgz \
--model_wts_path './saved_model_wts/dense_unet_back2front_non_finetuned.pth' \
```

An illustration can be seen in [`predicting_segmentation_illustration.ipynb`](https://github.com/NYUMedML/DARTS/blob/master/predicting_segmentation_illustration.ipynb).

## Deep learning models for brain MR segmentation
We pretrain our Dense Unet model using the Freesurfer segmentations of 1113 subjects available in the [Human Connectome Project](https://www.humanconnectome.org/study/hcp-young-adult/document/1200-subjects-data-release) dataset and fine-tuned the model using 101 manually labeled brain scans from [Mindboggle](https://mindboggle.info/data.html) dataset.

The model is able to perform the segmentation of complete brain **within a minute** (on a machine with single GPU). The model labels 102 regions in the brain making it the first model to segment more than 100 brain regions within a minute. The details of 102 regions can be found below.

## Quantitative results on the Mindboggle held out data
The box plot compares the dice scores of different ROIs for Dense U-Net and U-Net. The Dense U-Net consistently outperforms U-Net and achieves good dice scores for most of the ROIs.

<img src="plots/compare_dice_plot_aparc_manual_fd_part_1_dn_v_unet.png" width="800" />
<img src="plots/compare_dice_plot_aparc_manual_fd_part_2_dn_v_unet.png" width="800" />

## Qualitative results on the HCP held out data
We perform an expert reader evaluation to measure and compare the proposed deep learning models' performance with Freesurfer model. We use HCP held-out test set scans for reader study. On these scans, Freesurfer results have undergone a manual quality control. We also compare the non-finetuned and fine-tuned model with Freesurfer model with manual QC. Seven regions of interest (ROIs) were selected:L/R Putamen (axial view), L/R Pallidum (axial view), L/R Caudate (axial view), L/R Thalamus (axial view), L/R Lateral Ventricles (axial view), L/R Insula (axial view) and L/R Cingulate Gyrus (sagittal view).The readers rated each example on a Likert-type scale from 1 (Poor) to 5 (Excellent).

Based on the readers' ratings, we investigate if there are statistically significant differences between the three methods using paired T-test and Wilcoxon signed rank test at 95\% significance level. The results can be seen below.
<img src="plots/reader_study_results.png" width="650" />

## Output segmentation
The output segmentation has 103 labeled segments with the last one being the **None** class. The labels of the segmentation closely resembles the aseg+aparc segmentation protocol of Freesurfer.

We exclude 4 brain regions that are not common to a normal brain: White matter and non-white matter hypointentisites, left and right frontal and temporal poles. We also excluded left and right 'unknown' segments. We also exclude left and right bankssts as there is no common definition for these segments that is widely accepted by the neuroradiology community.


The complete list of class number and the corresponding segment name can be found [here](https://github.com/NYUMedML/BrainSeg/blob/master/name_class_mapping.p) as a pickled object or [here](https://github.com/NYUMedML/BrainSeg/blob/master/FreeSurferColorLUT_modified.txt) as a .txt file.

## Sample Predictions
### Insula
Here we can clearly see that Freesurfer (FS) incorrectly predicts the right insula segment, the model trained only using FS segmentations also learns a wrong prediction. Our proposed model which is finetuned on manually annotated dataset correctly captures the region. Moreover, the segment looks biologically natural unlike FS's segmentation which is grainy, noisy and with non-smooth boundaries.
<img src="plots/rt_insula_aparc_with_man_3.png" width="800" />

### Putamen
Here again, we see that FS segmentation is of low quality but our proposed fine-tuned model performs well and produces more natural looking segmentation.
<img src="plots/Faulty_seg_Putamen.png" width="800" />

### Pallidum
FS segmentation for pallidum also of low quality, but the proposed model performs well.
<img src="plots/Faulty_seg_Pallidum.png" width="800" />

### More Predictions
Some sample predictions for [Putamen](https://github.com/NYUMedML/BrainSeg/blob/master/plots/Left-Putamen_627549_143_0_1_2.pdf), [Caudate](https://github.com/NYUMedML/BrainSeg/blob/master/plots/Right-Caudate_194443_137_0_1_2.pdf), [Hippocampus](https://github.com/NYUMedML/BrainSeg/blob/master/plots/Right-Hippocampus_894774_108_0_1_2.pdf) and [Insula](https://github.com/NYUMedML/BrainSeg/blob/master/plots/ctx-lh-insula_147030_138_0_1_2.pdf) can be seen here. In all the images, prediction 1 = Freesurfer, Prediction 2 = Non-Finetuned Dense Unet, Prediction 3 = Finetuned Dense Unet.

We demonstrate that that Freesurfer often makes errors in determining the accurate boundaries whereas the deep learning-based models have natural looking ROIs with accurate boundaries.

## Contact
If you have any questions regarding the code, please contact ark576[at]nyu.edu or raise an issue on the github repo.
