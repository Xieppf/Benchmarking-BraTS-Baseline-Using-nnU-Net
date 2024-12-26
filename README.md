# Benchmarking-BraTS-Baseline-Using-nnU-Net

First use pip install nnunetv2 to install the nnunetv2
I download the BraTS2021 dataset from https://aistudio.baidu.com/datasetdetail/182344
I selected the first fifteen data as the training set and the last ten data as the test set. 
Run Dataset137_BraTS21.py in the terminal to generate a training dataset that meets the requirements of nnunet
nnUNet_raw/
├── Dataset001_BrainTumour
  ├── dataset.json
  ├── imagesTr
  └── labelsTr
and the test dataset also use Dataset137_BraTS21.py, I changed the file name myself and add it to the nnUNet_raw/, it now has the structure
nnUNet_raw/
├── Dataset001_BrainTumour
  ├── dataset.json
  ├── imagesTr
  ├── imagesTs
  ├── labelsTs
  └── labelsTr
here is my dataset: https://drive.google.com/drive/folders/1zosHCChzDGDybqZnzmv1Prcb0szhwNhD?usp=drive_link
I use colab to do this assignment, the gpu is A100.
and I use !nnUNetv2_plan_and_preprocess -d 137 --verify_dataset_integrity to run fingerprint extraction, experiment planning and preprocessing.
I use !nnUNetv2_train Dataset137_BraTS2021 3d_fullres 0  -tr nnUNetTrainer_10epochs (0,1,2,3,4),!nnUNetv2_train Dataset137_BraTS2021 2d 0  -tr nnUNetTrainer_10epochs (0,1,2,3,4).So nnU-Net trains all configurations in a 5-fold cross-validation over the training cases. Because the default epoch is 1000 ，I use -tr nnUNetTrainer_10epochs to limit the epoch to 10.
