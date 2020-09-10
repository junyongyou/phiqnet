# PHIQnet Implementation

TF-Keras implementation of PHIQnet as described in [Perceptual Hierarchical Networks for No-Reference Image Quality Assessment] by Junyong You and Jari Korhonen.

## Installation

1) Clone this repository.
2) Install required Python packages. The code is developed by PyCharm in Python 3.7. The requirements.txt document is generated by PyCharm, and the code should also be run in latest versions of the packages.

## Training a model
Many examples of training PHIQnet and its variants can be seen in image_quality/bin.
Argparser should be used, but the authors prefer to use disctionary with parameters being defined. It is easy to convert to take arguments.
In principle, the following parameters can be defined:
    args = {}
    args['multi_gpu'] = 0 # gpu setting, set to 1 for using multiple GPUs
    args['gpu'] = 0  # If having multiple GPUs, specify which GPU to use

    args['result_folder'] = r'..\databases\experiments' # Define result path
    args['n_quality_levels'] = 5  # Choose between 1 (MOS prediction) and 5 (distribution prediction)

    args['train_folders'] =  # Define folders containing training images
        [
        r'..\databases\train\koniq_normal',
        r'..\databases\train\koniq_small',
        r'..\databases\train\live'
        ]
    args['val_folders'] =  # Define folders containing testing images
        [
        r'..\databases\val\koniq_normal',
        r'..\databases\val\koniq_small',
        r'..\databases\val\live'
        ]
    args['koniq_mos_file'] = r'..\databases\koniq10k_images_scores.csv'  # MOS (distribution of scores) file for KonIQ database
    args['live_mos_file'] = r'..\databases\live_mos.csv'   # MOS (standard distribution of scores) file for LIVE-wild database

    args['naive_backbone'] = False  # Choose between True and False, indicating using backbone network only or neck + head as well
    args['backbone'] = 'resnet50' # Choose from ['resnet18', 'resnet50', 'resnet152', 'resnet152v2', 'vgg16', 'resnest50']
    args['weights'] = r'..\pretrained_weights\resnet50_weights_tf_dim_ordering_tf_kernels_notop.h5'  # Define the path of ImageNet pretrained weights
    args['initial_epoch'] = 0  # Define initial epoch for use in fine-tune

    args['lr_base'] = 1e-4 / 2  # Define the back learning rate in warmup and rate decay approach
    args['lr_schedule'] = True  # Choose between True and False, indicating if learning rate schedule should be used or not
    args['batch_size'] = 4  # Batch size, should choose to fit in the GPU memory
    args['epochs'] = 120  # Maximal epoch number, can set early stop in the callback or not

    args['fpn_type'] = 'fpn'  # FPN type, choose from ['fpn', 'bifpn', 'pan', 'no_fpn'], it is noted that if 'bifpn' is chosen, the image resolutions must be power of 2 otherwise shape mismatch will be thrown
    args['attention_module'] = True  # Choose between True and False, indicating if attention module should be used or not

    args['image_aug'] = True # Choose between True and False, indicating if image augmentation should be used or not

## Predict image quality using the trained model
After PHIQnet has been trained, and the weights have been stored in h5 file, it can be used to predict image quality with arbitrary sizes,

```shell
    args = {}
    args['n_quality_levels'] = 5
    args['naive_backbone'] = False
    args['backbone'] = 'resnet50'
    args['fpn_type'] = 'fpn'
    args['weights'] = 'xxxx.h5'
    model = phiq_net(n_quality_levels=args['n_quality_levels'],
                     naive_backbone=args['naive_backbone'],
                     backbone=args['backbone'],
                     fpn_type=args['fpn_type'])
    model.load_weights(args['weights'])
```
And then use ModelEvaluation to predict quality of imageset.

## Prepare datasets for model training
This work uses two publicly available databases: KonIQ-10k [KonIQ-10k: An ecologically valid database for deep learning of blind image quality assessment](https://ieeexplore.ieee.org/document/8968750) by V. Hosu, H. Lin, T. Sziranyi, and D. Saupe;
 and LIVE-wild [Massive online crowdsourced study of subjective and objective picture quality](https://ieeexplore.ieee.org/document/7327186) by D. Ghadiyaram, and A.C. Bovik

1) The two databases were merged, and then split to training and testing sets. Please see README in image_quality/databases for details.
2) Make MOS files:

    For database with score distribution available, the MOS file is like this (koniq format):
    ```
        image path, voter number of quality scale 1, voter number of quality scale 2, voter number of quality scale 3, voter number of quality scale 4, voter number of quality scale 5, MOS or Z-score
        10004473376.jpg,0,0,25,73,7,3.828571429
        10007357496.jpg,0,3,45,47,1,3.479166667
        10007903636.jpg,1,0,20,73,2,3.78125
        10009096245.jpg,0,0,21,75,13,3.926605505
    ```

    For database with standard deviation available, the MOS file is like this (live format):
    ```
        image path, standard deviation, MOS or Z-score
        t1.bmp,18.3762,63.9634
        t2.bmp,13.6514,25.3353
        t3.bmp,18.9246,48.9366
        t4.bmp,18.2414,35.8863
    ```

    The format of MOS file ('koniq' or 'live') and the format of MOS or Z-score ('mos' or 'z_score') should also be specified in image_quality/misc/imageset_handler/get_image_scores.
3) In the train script in image_quality/bin, the folders containing training and testing images are provided.
4) Pretrained ImageNet weights can be downloaded (see README in image_quality/pretrained_weights) and pointed to in the train script in image_quality/bin.

## State-of-the-art metrics
Other three metrics are also included in the repo. The original implementations of metrics are employed, if findable online. However, these metrics are less commented.
 In addition, the pathes to training, testing sets, pretrained weights, and saliency map folder should also be revised accordingly.

Koncept512 [KonIQ-10k: An ecologically valid database for deep learning of blind image quality assessment](https://github.com/subpic/koniq)

SGDNet [SGDNet: An end-to-end saliency-guided deep neural network for no-reference image quality assessment](https://github.com/ysyscool/SGDNet)

CaHDC [End-to-end blind image quality prediction with cascaded deep neural network](https://web.xidian.edu.cn/wjj/files/20190620_152557.zip)

## FAQ
* To be added