# SAM-guide


## steps to run pretrained aging model

1. clone SAM repo.
```bash
git clone git@github.com:yuval-alaluf/SAM.git
```

2. download pretrained model: https://github.com/yuval-alaluf/SAM/tree/master?tab=readme-ov-file#pretrained-models

You might encounter issues with gdown because sometimes multiple people are trying to download the same model simultaneously.
```bash
mkdir pretrained_models
pip install gdown
gdown "https://drive.google.com/u/0/uc?id=1XyumF6_fdAxFmxpFcmPf-q84LU_22EMC&export=download" -O pretrained_models/sam_ffhq_aging.pt
```

3. require ninja lib.

```bash
wget https://github.com/ninja-build/ninja/releases/download/v1.8.2/ninja-linux.zip
sudo unzip ninja-linux.zip -d /usr/local/bin/
sudo update-alternatives --install /usr/bin/ninja ninja /usr/local/bin/ninja 1 --force
```

4. download shape_predictor_68_face_landmarks.dat / dlib / numpy / pillow library 

```bash
wget "https://github.com/italojs/facial-landmarks-recognition/raw/master/shape_predictor_68_face_landmarks.dat"
pip install dlib
pip install numpy
pip install pillow
```

5. Algin Face. 

This step is required if you are using a FEI image or any other image that is not 1024x1024

First create a test directory in SAM folder.

```bash
mkdir SAM/test_data
```

Second align the face and put the image in SAM test directory.

```bash
python3 alignface.py \
--image_path test_images/1-11.jpg \
--landmark_path shape_predictor_68_face_landmarks.dat \
--output_path SAM/test_data
```

6. generate aging GANs images

First check if cuda is enabled or not.
```python
import torch
assert torch.cuda.is_available() == True
```

Second create output dir where the script will output the generated images.
```bash
mkdir SAM/output_exp
```

Third and final step is to get results.

```bash
cd SAM && python3 scripts/inference.py \
--exp_dir=output_exp \
--checkpoint_path=../pretrained_models/sam_ffhq_aging.pt \
--data_path=test_data \
--test_batch_size=4 \
--test_workers=4 \
--couple_outputs --target_age=0,10,20,30,40,50,60,70,80
```

Additional notes to consider:
* Adding the flag --couple_outputs will save an additional image containing the input and output images side-by-side in the sub-directory inference_coupled. Otherwise, only the output image is saved to the sub-directory inference_results.
* The flag --exp_dir is the path where the images generated from the model will be saved.
* The flag --data_path is path to input images.
* The flag --checkpoint_path is path to pSp model checkpoint.
* The flag --target_age is the argument that can be set to generate multiple ages ranging from 0 to 100.
* The --test_batch_size is the number of images you pass through the network together and the --test_workers is a parameter used by the dataloader (you don't need to worry about this one too much).
