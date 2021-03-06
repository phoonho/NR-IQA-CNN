# A No Reference Image Quality Assessment System

This project implements a no-reference image quality assessment convolutional neural network (CNN) using the deep learning framework Caffe. This project was done as part of Special Problem Research project carried out @ [OLIVES, Department of Electrical and Computer Engineering, Georgia Tech.](https://ghassanalregib.com/)
It essentially implements the paper cited in the citations section below.

## Overview
The basic directory structure of the codebase is similar to that of Caffe repo because it was initially forked from the official Caffe repo but I some modifications to this Caffe along with the implementation of SVR Loss layer and varying momentum modification in the SGD optimizer in Caffe. Hence, the modified Caffe which is present in this project should only be used with this code.
Just cloning from the above Github link gives you the entire code. Build the code by running **make -j8** in the code’s root directory. On a machine on which Caffe is properly setup, the make command should build everything required on the Caffe side.

## Detailed Instructions
You'll need *MATLAB* to run the helper scripts (.m files) metioned below. Once Caffe is properly built by running the make command as mentioned earlier, follow the below steps:
1.	There is a directory in the code’s root directory called ‘Training data preparation scripts’ (without quotes). This directory contains MATLAB scripts which are used for data preparation for training. Provided you have downloaded LIVE dataset somewhere, first of all run **prep_training_data.m.** The script will prompt you to navigate to the distortion directory in live dataset. Once you select the directory using the dialog generated by running the script, the script will start pre-processing the data and will output local contrast normalized image patches under a directory called prepared_data.
2.	In your LIVE dataset directory, create a file named dmos_local.mat containing dmos scores for the images in that distortion directory. A single mat array is provided by LIVE and the index range for every distortion is also provided in LIVE. You just need to sample out those ranges for every distortion. This is needed only once for a fresh LIVE installation.
3.	Run **prep_scores_data.m**. It will also create dialogues which are self-explanatory. Just navigate to the subdirectories in LIVE dataset installation for which it asks for. It creates a folder called mappings. And under that folder , it creates training and testing splits according to the protocol specified in the paper i.e. 80% of the reference images and their distorted versions for training and the remainder reference images and their distorted versions for testing. In this script there is a variable called *trainValIters*. It’s value is 100 by default which means it will generate 100 train/test splits so that the model can be trained and tested 100 times in order to see the generalizing capability of the model. According to this protocol, each time the training starts from scratch. You can change it if you want to change the number of train-test iterations.
4.	Copy the contents of prepared_data and the mappings folder itself to *<Code_Root_Directory>/data/live/<Distortion_Type>/*
5.	Repeat step 1 and step 3 for all distortion types you want to experiment with.
6.	From code root directory run:
**examples/IQA_dataset/create_databases.sh <iter_start> <iter_end> <distortion_type>**
For example, for 100 train-test iterations and gblur distortion type you should run:
**examples/IQA_dataset/create_databases.sh 1 100 gblur**
This will create LMDB’s for the training and testing
7.	Now run the following command from the code root directory:
**models/IQA_CNN/create_ms_descriptions.sh <no_of_iterations> <distortion_type>**
For example, for 100 train-test iterations using fastfading distortion, you should run:
**models/IQA_CNN/create_ms_descriptions.sh 100 fastfading**
This will create solver and network description files for the 100 training-testing iterations.
8.	Finally run this from the code’s root directory:
**train_IQA_CNN_gpu_0.sh <iter_start> <iter_end> <distortion_type>**
For example, for 100 train-test iterations using fastfading distortion, you should run:
**train_IQA_CNN_gpu_0.sh 1 100 fastfading**
If there are multiple GPU’s on a machine then running train_IQA_CNN_gpu_1.sh will train using GPU 1.  Since all train-test iterations are independent, you can take advantage of multiple GPU’s by running training concurrently on both of them at the same time. For example, for 2 GPU’s you can launch these two commands in parallel from 2 different terminals
**train_IQA_CNN_gpu_0.sh 1 50 fastfading**
**train_IQA_CNN_gpu_1.sh 51 100 fastfading**
In this way first 50 models are trained using GPU 0 and the next 50 models using GPU 1 simultaneously.
9.	Once training is complete, run from code’s root directory
**test_IQA_CNN.sh <iter_start> <iter_end> <distortion_type>**
10.	Finally run the MATLAB script **compute_LCC_SROCC.m** after **test_IQA_CNN.sh** finishes execution. This script will report the maximum, mean and median LCC and SROCC from all the models trained as a result of multiple train-test iterations (usually 100). It also saves the results in a text file called out.txt created under *output* folder in the code’s root directory.

## Important Notes
*	If you face permission issues while running any of the above scripts then run them with sudo.
*	On a multi-GPU machine if the second GPU is installed and is setup properly but Caffe is not able to see it then again run the scripts with *sudo*.
*	When running any script with sudo you may need to set LD_LIBRARY_PATH like this:
**sudo LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH ./train_IQA_CNN_gpu_1.sh …..**
*	In order to generate visualizations of learned kernels, you need to print out the raw kernel blob (no need to change row-major/column major format) to some file by modifying corresponding C++ code in Caffe and rebuilding the code. After that run the script **vis_kernels.m** to read the raw dump file and output visualizations. This script is also present under Training data preparation scripts directory. The modifications required in the C++ code for this are not committed in the repo because we are not always printing out the weights except when we want to visualize. So when you want to visualize, make those changes, (these are minor changes like prints/logs), generate visualizations and then revert the code so that printing doesn’t slow down training/testing in future.
*	To train and test for all distortions you need to create another folder called *all* in LIVE dataset and just copy in the content from all distortion folders there and make sure you create it’s **dmos_local.mat** appropriately. You also need to generate its **info.txt** and the script **generate_all_info_txt.m** provided under *Training data preparation scripts* directory can help you generate that.
*	After that follow the same steps you do for any other individual distortion by specifying **all** as the distortion type. Basically, **all** becomes another distortion type having data from all types of distortions in the dataset. This keeps the scripts simple and consistent.


## Dataset used
The network is trained and tested using LIVE database. The framework developed here can train the CNN for all five distortion types present in the dataset. Training can be done both for a single distortion type and for all distortion types combined as described above.

## License and Citations

If you find this work useful in your projects or research then please cite this project (https://github.com/Adnan1011/NR-IQA-CNN)

NR IQA CNN paper:
    
    @article{The IEEE Conference on Computer Vision and Pattern Recognition (CVPR), June 2014, pp.1733–1740,
      Author = {Le Kang, Peng Ye, Yi Li and David Doermann},
      Conference = {CVPR 2014},
      Title = {Convolutional Neural Networks for No-Reference Image Quality Assessment},
      Year = {2014}
    }

This repository was initially forked from Caffe repository.
Caffe is released under the [BSD 2-Clause license](https://github.com/BVLC/caffe/blob/master/LICENSE).

    @article{jia2014caffe,
      Author = {Jia, Yangqing and Shelhamer, Evan and Donahue, Jeff and Karayev, Sergey and Long, Jonathan and Girshick, Ross and Guadarrama, Sergio and Darrell, Trevor},
      Journal = {arXiv preprint arXiv:1408.5093},
      Title = {Caffe: Convolutional Architecture for Fast Feature Embedding},
      Year = {2014}
    }
