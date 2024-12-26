# Benchmarking-BraTS-Baseline-Using-nnU-Net

First use pip install nnunetv2 to install the nnunetv2
<br>I download the BraTS2021 dataset from https://aistudio.baidu.com/datasetdetail/182344
<br>I selected the first fifteen data as the training set and the last ten data as the test set. 
<br>Run Dataset137_BraTS21.py in the terminal to generate a training dataset that meets the requirements of nnunet
<br>  nnUNet_raw/
<br>  ├── Dataset001_BrainTumour
<br>    ├── dataset.json
<br>    ├── imagesTr
<br>    └── labelsTr
<br>and the test dataset also use Dataset137_BraTS21.py, I changed the file name myself and add it to the nnUNet_raw/, it now has the structure
<br>  nnUNet_raw/
<br>  ├── Dataset001_BrainTumour
<br>    ├── dataset.json
<br>    ├── imagesTr
<br>    ├── imagesTs
<br>    ├── labelsTs
<br>    └── labelsTr
<br>here is my dataset: https://drive.google.com/drive/folders/1zosHCChzDGDybqZnzmv1Prcb0szhwNhD?usp=drive_link
<br>I use colab to do this assignment, the gpu is A100.
<br>and I use !nnUNetv2_plan_and_preprocess -d 137 --verify_dataset_integrity to run fingerprint extraction, experiment planning and preprocessing.
<br>I use !nnUNetv2_train Dataset137_BraTS2021 3d_fullres 0  -tr nnUNetTrainer_10epochs (0,1,2,3,4),!nnUNetv2_train Dataset137_BraTS2021 2d 0  -tr nnUNetTrainer_10epochs (0,1,2,3,4).So nnU-Net trains all configurations in a 5-fold cross-validation over the training cases. Because the default epoch is 1000 ，I use -tr nnUNetTrainer_10epochs to limit the epoch to 10.
<br>Because of the ensemble strategy, all probability maps that require model predictions, rather than segmentation results, are generated by  generating npz files for each fold
```python 
<br>  !nnUNetv2_train Dataset137_BraTS2021 2d  0 -tr nnUNetTrainer_10epochs --val --npz
<br>  !nnUNetv2_train Dataset137_BraTS2021 2d  1 -tr nnUNetTrainer_10epochs --val --npz
<br>  !nnUNetv2_train Dataset137_BraTS2021 2d  2 -tr nnUNetTrainer_10epochs --val --npz
<br>  !nnUNetv2_train Dataset137_BraTS2021 2d  3 -tr nnUNetTrainer_10epochs --val --npz
<br>  !nnUNetv2_train Dataset137_BraTS2021 2d  4 -tr nnUNetTrainer_10epochs --val --npz
<br>  !nnUNetv2_train Dataset137_BraTS2021 3d_fullres  0 -tr nnUNetTrainer_10epochs --val --npz
<br>  !nnUNetv2_train Dataset137_BraTS2021 3d_fullres  1 -tr nnUNetTrainer_10epochs --val --npz
<br>  !nnUNetv2_train Dataset137_BraTS2021 3d_fullres  2 -tr nnUNetTrainer_10epochs --val --npz
<br>  !nnUNetv2_train Dataset137_BraTS2021 3d_fullres  3 -tr nnUNetTrainer_10epochs --val --npz
<br>  !nnUNetv2_train Dataset137_BraTS2021 3d_fullres  4 -tr nnUNetTrainer_10epochs --val --npz
```
<br>then nnU-Net can automatically identify the best combination 
```python 
<br>  !nnUNetv2_find_best_configuration Dataset137_BraTS2021 -c 2d  3d_fullres -tr nnUNetTrainer_10epochs -f 0 1 2 3 4
```
<br>then run inference
<br>  !nnUNetv2_predict -d Dataset137_BraTS2021 -i /content/drive/MyDrive/DATASET/nnUNet_raw/Dataset137_BraTS2021/imagesTs -o /content/drive/MyDrive/DATASET/BraTS2021_2d_predict -f  0 1 2 3 4 -tr nnUNetTrainer_10epochs -c 2d -p nnUNetPlans --save_probabilities
<br>  !nnUNetv2_predict -d Dataset137_BraTS2021 -i /content/drive/MyDrive/DATASET/nnUNet_raw/Dataset137_BraTS2021/imagesTs -o /content/drive/MyDrive/DATASET/BraTS2021_3d_fullres_predict -f  0 1 2 3 4 -tr nnUNetTrainer_10epochs -c 3d_fullres -p nnUNetPlans --save_probabilities
<br>  !nnUNetv2_ensemble -i /content/drive/MyDrive/DATASET/BraTS2021_2d_predict /content/drive/MyDrive/DATASET/BraTS2021_3d_fullres_predict -o /content/drive/MyDrive/DATASET/BRATS2021_ensemble
<br>Finally, apply the previously determined postprocessing to the (ensembled) predictions:
<br>  !nnUNetv2_apply_postprocessing -i /content/drive/MyDrive/DATASET/BRATS2021_ensemble  -o /content/drive/MyDrive/DATASET/BRATS2021_ensemble_pp -pp_pkl_file  /content/drive/MyDrive/DATASET/nnUNet_results/Dataset137_BraTS2021/nnUNetTrainer_10epochs__nnUNetPlans__2d/crossval_results_folds_0_1_2_3_4/postprocessing.pkl -np 8 -plans_json  /content/drive/MyDrive/DATASET/nnUNet_results/Dataset137_BraTS2021/nnUNetTrainer_10epochs__nnUNetPlans__2d/crossval_results_folds_0_1_2_3_4/plans.json  -dataset_json /content/drive/MyDrive/DATASET/nnUNet_results/Dataset137_BraTS2021/nnUNetTrainer_10epochs__nnUNetPlans__2d/crossval_results_folds_0_1_2_3_4/dataset.json
<br>nnU-Net's performance in comparison to the BraTS baseline,I use 
<br>  !nnUNetv2_evaluate_folder -djfile /content/drive/MyDrive/DATASET/nnUNet_raw/Dataset137_BraTS2021/dataset.json  -pfile /content/drive/MyDrive/DATASET/nnUNet_results/Dataset137_BraTS2021/nnUNetTrainer_10epochs__nnUNetPlans__2d/crossval_results_folds_0_1_2_3_4/plans.json    /content/drive/MyDrive/DATASET/nnUNet_raw/Dataset137_BraTS2021/labelsTs  /content/drive/MyDrive/DATASET/BRATS2021_ensemble_pp
<br>it He stored the results in summary.json in BRATS2021_ensemble_pp, including Dice Score, FN, FP, TN, TP, and IoU 。
<br>And the last part I calculate the Hausdorff Distance.




