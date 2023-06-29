# Instructions to run nnUNet models for pLGG 

IMPORTANT: This code is designed to run on T2 scans only. 

## Setting up the environment

(I am assuming that this is being run on a Linux machine)
Make sure you have conda installed. The run this command:
```bash
conda env create -f environment.yml
conda activate bwh_plgg
```
Note: there may be some dependency issues with pytorch/cuda versions etc, please try to fix those first. This work was done before the release of pytorch 2.0, and I have not tested the backwards compatability, so try to use pytorch version 1.13 (this should be done by using the environment.yml to create the environment).

## Preprocessing

The preprocessing is done using file _mri_preprocess_3d.py_

To use this program, you need to modify the code to enter the path to the scans (T2W_dir) and where you want to save the output (output_path).
It is assumed that your scans and ground truth masks are in the same folder and that it if a scan name is <scan>.nii.gz, the corresponding mask is named <scan>_mask.nii.gz. If you dont have ground truth segmentations, comment out all the lines regarding them. 
  
The output should be fully preprocessed scans in <output_path>/nnunet/imagesTs. These scans will be saved with the nnunet naming convention (<scan>_0000.nii.gz for the scan). Masks will be located in <output_path>/nnunet/labelsTs (name will be <scan>.nii.gz for the mask).
  
This step may take a couple of hours to run depending on how many scans are in your folder.

## Installing nnUNet version 1

This model is trained using the base nnUNet (version 1) code which can be found here: https://github.com/MIC-DKFZ/nnUNet/tree/nnunetv1
Please refer there with any questions regarding model architecture etc.

The command to get the right code is: (you should clone the nnUNet repo inside this repo folder for the sake of organization)
```bash
git clone -b nnunetv1 git@github.com:MIC-DKFZ/nnUNet.git
cd nnUNet
pip install -e .
```
Note we are installing the branch "nnunetv1" from the repo. If you just clone from the repo and dont checkout this branch, youll be using the incorrect version of the nnUNet code.
  
## Pretrained Models

The pretrained models can be downloaded at the following Drive link (if it isnt working for some reason contact me at aidancolmboyd@gmail.com):
https://drive.google.com/file/d/1cbi3p9IoKWjKR-pl3yXde6ISx4hZy2DB/view?usp=sharing
Unzip this file, the unzipped folder should be named nnUNet_trained_models. Dont change the locations of any of the files inside.
If there is some error referring to a zip bomb (I think its something to do with number of files?) then just restart the command like this:
```bash
export UNZIP_DISABLE_ZIPBOMB_DETECTION=TRUE; unzip nnUNet_trained_models.zip
```
  
Make sure to run the following command to make sure the model picks up on where these models are:
```bash
export RESULTS_FOLDER="<path to the downloaded files>/nnUNet_trained_models/"
```
To be safe, make sure to reload the .bashrc by running source /home/<user>/.bashrc
  

## Running inference
  
Inference can be run using the following command:
```python
nnUNet_predict -i <path to images>/nnunet/imagesTs/ -o <where we want to save the predictions> -t <task number> -m 3d_fullres --save_npz
```
Where <task number> should be set as follows: 
  -- 901 for adult models
  -- 888 for models trained from scratch on peds data
  -- 889 for models fine tuned on peds data (using adult models as starting point)
  -- 871 (best) for models where we fine tune, then freeze the encoder and continue training
Output masks will be named <scan>.nii.gz in the folder you specified in th -o parameter.
