This code is the implementation of "Two Souls in an Adversarial Image: Towards Universal
Adversarial Example Detection using Multi-view Inconsistency". Section 5.1 of the paper provides detail about Datasets and Training setup. Section 5.2, explains threat models and attack method used for generating Adversarial Samples. Section 4.2, provides algorithm for View Generation. Section 4.3, gives details about Predictors and Detection Algorithm. Section 5.3, includes the detection result.

Following are the requirements and step by step instructions to reproduce the results reported in the paper:

## Requirements:
- CUDA Enabled GPU hardware
- python == 3.6.9
- tensorflow-gpu==1.15.0
- tensorflow_probability == 0.7
- robustness (https://github.com/MadryLab/robustness)
- cleverhans == 3.1.0


## Data Preparation:
After preprocessing(this step) data will be stored in "data/\<dataset\>" folder as pickle file format.
1. For CIFAR10 dataset(https://www.cs.toronto.edu/~kriz/cifar.html), no pre-processing is required and pickle files can be downloaded directly into "data/cifar" folder.

2. Download ImageNet data (https://image-net.org/index.php) directly in "Raw_data/ImageNet" folder. We have used ILSVRC2012_img_train dateset. To buid Custom Restricted ImageNet data, necessary files are already provided in "Raw_data/ImageNet/imagenet_info". Run following Code to build and save custom imagenet dataset in "data/imagenet".
  
      ```python prepare_imagenet.py``` 
  

3. Download GTSRB data (https://benchmark.ini.rub.de/gtsrb_dataset.html) in "Raw_data/GTSRB" folder and run following code to convert it into pickle files.
      
      ```python prepare_GTSRB_data.py```




## Training:
* The value of \<dataset\> should be passed as cifar, imagenet or gtsrb.
* To avoid Classifier and Pixel-CNN training, download already trained models from https://drive.google.com/drive/folders/1fBOMF3WD4Uam5zOKM8jCXVkUZVL9tTh8?usp=sharing and store them in respective folders.

  #### 1. Classifier:
  ```ruby
     - python Classifier/train.py -d cifar -c 10
  ```
   Trained model will be saved in "Classifier/\<dataset\>_Model/ResNet50_ckpt/" folder.

  #### 2. Pixel-CNN
  ```ruby
     - python pixel-cnn/train.py -d cifar
  ```
    Trained model will be saved in "pixel-cnn/\<dataset\>\_Model/" folder.

   #### 3. GMM
    To train GMM model, first it requires representation vector of all training/validation data. After first code it will be saved in "data/\<datase \>/Representation" folder. Next code will train Class conditional GMM model and it will be saved in "GMM/GMM_Models/\<dataset\>_GMM" folder.

    ```ruby
      - python GMM/prepare_representation.py -d <dataset> -f clean
      - python GMM/train_GMM.py -d <dataset>
     ```
  

## Adversarial Samples:
Input: Test data and trained classifier model.
  
Output: Adversarial examples (PGD, FGSM, MIM, CW, DeepFool and WhiteBox) will be saved in "data/<dataset\>/Adversarial/" folder.To attack all samples give num_samples = -1.
  ```ruby
  - python Attacks/attack_Linf.py -d cifar -n 2000
  - python Attacks/attack_L2.py -d cifar -n 2000
  - python WhiteBox_attack.py -d cifar -ns 2000
  ```

## View Generation:
For precise analysis, it's important that benign test files should only contain samples that are correctly classified. Following code will filter those benign samples and save them in "data/\<dataset\>/test_c" file. Next code will generate views for all data files and save them in "data/\<dataset\>/Generated/" folder.
  ```ruby
   - python Classifier/test.py - d cifar
   - python pixel-cnn/generate_views.py -d cifar -n 1000
   ```
In view generation number of samples value can not exceed adversarial samples generated by worst attack method. To generate views for all samples give num_samples = -1. Though view generation is a time consuming step. So either keep num_samples value low or expect this code to finish in 2-3 days. The approximate detection rate that is quite close to True detection rate, can be obtained using num_samples = 1000 for cifar/gtsrb. 
     
## Detection:
Step 1: Feature/Descriptor extraction using generated views. Output of the first code will save files in data/<dataset>/Final_Descriptors.
  
Step 2: Training and Evaluation for Adversarial detector. Output of the second code will be AUC Score for all attack methods.
   ```ruby 
    - python Final_Features.py -d cifar
    - python  Adversarial_Detector.py -d cifar
  ```
