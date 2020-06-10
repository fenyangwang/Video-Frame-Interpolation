# Video-Frame-Interpolation  

# Environment Setup  

# Running  

# Final Report  

## Introduction and diagram for the Video Frame Interpolation project
When we watch movies and TV shows online today, most of them are 24fps format. As most of our screens like monitors or television are 60hz or even 120hz frame rate or more, use Figure 1 as example. We will see some common artifacts on the screen if we watch these videos on our screen. The reason for the artifacts occurring is because the low frame rate video will lose moving detail during the movement.

 ![Motion Interpolation](./proposal/pic1.png)

To avoid these common artifacts we want to create a program that is powered by Video Frame Interpolation technology to convert 24fps video to 60fps video. This is a technology that aims to generate non-existent frames in-between the original frames. The usage of this technology can be used not only frame rate up-conversion but also the slow-motion video. 

Triditional video frame interpolation methods are using the estimate optical flow to predict the movement of the object between input frames and synthesizing intermediate frames. However the performance will be vary depends the quality of optical flow. Futhermore, the optical flow methods still challenging to generate high-quality frames due to large motion and occlusions. 

![Different between triditional method and kernal based method](./proposal/pic4.png)

Since the goal of this project is to produce high-quality frame between existing frames, we decide to use the kernal based method to predict the frame. This method is to estimate spatially-adaptive convolution kernels for each output pixel and convolve the kernels with the input frames to generate a new frame. Specifically, for each pixel in the interpolated frame, the method takes two receptive field patches centered at that pixel as input and estimates a convolution kernel. The difference between these two method are shown as Figure 2.

As you can see in Figure 3, the object moves from one frame to the next frame. The model use kernal to draw the missing frame and then insert it in-between these two frames. The missing frame is generated by the CNN network with the other two frames as input.

 ![Generate frame with two frame](./proposal/pic3.jpg)

For the test part, we use 60fps frames videos to train our model as shown in Figure 4. Each video in the data set is process in three set of rames:t, t+1, t+2, where t is from 1th frames to 58 frames. We use the t frame and t+2 frame as input and t+1 frame as ground truth. The output frame is used to compare with the original t+1 frame to accurately model.

 ![Compare with the original frame](./proposal/pic2.jpg)

## How it is related to Deep Learning for CV 
Traditional methods use optical flow for frame interpolation. However, in general, optical flow cannot be calculated from images with ambiguity (known as the aperture problem[7] in computer vision); additional constraints are needed to find a unique solution. Therefore, the quality of the interpolation heavily depends on the accuracy of the flow estimation. It is important to note that this work implements the work of Niklaus et al.[1]. on Adaptive Separable Convolution, which claims high-quality results on the video frame interpolation task. We design a convolutional neural network to estimate a proper convolutional kernel to synthesize each output pixel in the interpolated images. Instead of implementing optical flow-based interpolation, our method captures both the motion and interpolation coefficients, generates kernel through convolution layers, and synthesizes an intermediate video frame. 

Our neural network can be trained using widely available video data, which provides a sufficiently large training dataset. The main advantages of our method are: 
1. It achieves better transformation learning and better results; 
2. It learns models can learn on their own, while traditional video compression work requires a lot of manual design. 
However, there is a disadvantage of generating large kernel for each pixel that it requires huge amount of graphics memory.


## Steps 

There are several steps towards making the project. First, We read some related articles and look into previous works on video frame interpolation. We are currently working on bring tradition video coding algorithms into this project, and adapt them into machine learning algorithms. Then, we can decide which approach we take to prediction inter-frame images.  
Second, we need to decide which dataset we use to train and test the neural network. Since we plan to convert lower FPS videos to 60FPS or 90FPS, we need to find some native 60FPS and 90 FPS video or corresponding picture frames.  
In addition, we implement the research method of Niklaus et al. from scratch using Keras library on Ubuntu 18.04 with Anaconda. Also, we develop a demo for quantitative analysis and generate videos for class presentation.  
Finally, we run experiment on our demo against test data, or possibly previous research, such as Sepconv Slomo. Metrics including MSE, PSNR, and SSIM are used to quantify the results and evaluate the performance. A final report is conducted to summarize our experience results. 

## Proposed Framework
We develop a deep neural network based on the Adaptive Convolution Network of Niklaus et al. As illustrated in Figure ?, the convolution layers take in two receptive fields, R1 and R2, from two consecutive frames respectively.  
![Proposed Framework](./proposal/framework.png)
The convolutional model is trained to output a kernal K, which is used with the corresponding patches, P1 and P2,  centered in receptive fields, to compute the output pixel of the interpolated frame I_hat. The formula for computing the output pixel is shown as below:   

![Formula for computing the interpolation pixel I_hat(x, y)](./proposal/formula0.png)

## Dataset
We train and test our model using 720p video images from Middlebury dataset as Niklaus et al. and DAIN did. It allows us to do similar experiment or even comparison if possible. If more data are needed for training, we download as many 720p video as needed via YouTube.  

## Training Environment
We train and test on Ubuntu 18.04 with Anaconda. We use one Quatro RTX 6000 and two RTX 2070 Super to train our model. Because all the video cards we use are RTX series, we can enable the RTX 16-bit optimization which makes our training faster. For the model itself, We use Keras to built our model. And for the training method, we use blocks and patches to train the model. Also, we use mini batch to reduce the memory cost and improve training efficiency. Three framesets are used for taining at a time, which have 9 frames in total. Since we use blocks and patches, for every pixel on frame, we have the bolcks and patches for it. So the block memory and patch memory come to about 7.3 GB and 2 GB per frame, 22 GB and 6 GB for each set.

## Training and testing dataset 
We use the triplet dataset from the Vimeo90K dataset to train our model. Vimeo90K is a large-scale, high-quality video dataset with 89,800 video clips downloaded from vimeo.com. The triplet dataset extracted from 15K selected video clips from Vimeo-90K. The triplet dataset has 73,171 triplets for training, where each triplet is a 3-frame sequence with a fixed resolution of 448 x 256 pixels. We train our network to predict the middle frame of each triplet. We also found Depth-Aware Video Frame Interpolation use this dataset as well. For the training data, we use 2/3 on training and 1/3 on validation.

For the testing data, we use the triplet dataset and HEVC dataset. For the HEVC dataset, we use BlowingBubbles and BasketballDrill. Each part has 500 frames and we use 250 of these 500 frames for testing.

## Training method/configuration 
To train our neural network, we initialized our neural network parameters using Xavier initialization method and use the AdaMax to optimize the proposed network. Xavier initialization is a method to ensure variance of both input and output to be the same. For AdaMax optimizer parameters, we set beta1 to 0.9, beta2 to 0.999, and learning rate to 0.001. We use 128 for the Mini-batch size to minimize the loss function. The maximum epochs number is set to 1000.

We also use EarlyStopping to increase our training speed. We monitor validation loss with 1.0 for Min_delta and 10 for the patience. The model save the best one to hdf5 file when early stopping is triggered.

## Experimental Results and Analysis
**Training Result**

During our training it takes about 150 seconds to complete one epoch. And the CNN model we finally get contains 15,842,114 parameters totally.

| #Parameters (million) | Runtime per Epoch (seconds) |
| --------------------- | --------------------------- |
| 15.8                  | 150                         |

When we train our model with the images of the same scene, the training loss continues to decrease within 20 epochs. After a following climbing halted around epoch #40, the training loss decreases again with experience and eventually keeps at a point of stability after epoch #100. The validation loss follows the trend of training loss, but a gap remains between these two curves.

Regarding the accuracy, both training and validation have the similar ascending trend although there is a stagnancy and small drop between epochs #20 and #50 before they eventually climb to a stable higher level.

<img src=".\results\loss_on_same_scene.png" alt="image-loss" style="zoom:90%;" /> <img src=".\results\acc_on_same_scene.png" alt="image-acc" style="zoom: 90%;" />

Figure XX: (a) The training loss and validation loss show the training set maybe is small relative to the validation dataset.

​                   (b) The training accuracy and validation accuracy shows a fair good performance



When we train our model with the images of different scenes, the training loss continues to decrease within the epochs in which the images are from same scenes. But the loss will jump sharply when the epoch switches to the images from different scenes. Such a phenomena is called catastrophic forgetting because the change of the training data is so significant that the model has to forget previous experience to fit the new data. However, the validation loss has a pretty much smooth trace without sharp jumping during its descending. 

The training accuracy steps down in the first about 100 epochs but quickly rebounds and stays at a much higher level. Though the validation accuracy steadily climbs to the plateau and gets stable after epoch #110.

<img src=".\results\train_loss_on_diff_scene.png" alt="img" style="zoom:40%;" /> <img src=".\results\val_loss_on_diff_scene.png" alt="img" style="zoom: 40%;" />



<img src=".\results\acc_on_diff_scene.png" alt="image-20200606135119012" style="zoom: 90%;" />

Figure XX: (a) The training loss has sharp jump while the training data change significantly. It indicates that the training set maybe is small relative to the validation dataset.             

​                   (b) The validation loss shows a relative smooth descending trend.

​                   (c) Both of the training accuracy and validation accuracy reach to a good performance while the training accuracy performs poorly at the early stage.



**Testing Result**

We test our model with Vimeo triplet sets and HEVC data. The table below shows the metrics of MSE, PSNR and SSIM we got when testing with 100 Vimeo triplet sets which are respectively from the same scene and different scenes.

| 100 Vimeo Triplet Sets | MSE     | PSNR    | SSIM   |
| ---------------------- | ------- | ------- | ------ |
| **Same Scene**         | 51.3136 | 18.0673 | 0.6097 |
| **Different Scenes**   | 36.9612 | 21.7903 | 0.7966 |

The images below show the interpolated frames generated by our method. The upper right image is the interpolated frame based on the frames from the same scene and the upper left is the ground truth for it. The lower right one is the interpolated frame based on the frames from different scene and the lower left is the ground truth for it. As shown in the metric table above and the visual comparison below, the interpolation based on images of different scene gives a better outcome.

<img src=".\results\origin_same_scene.png" alt="img" style="zoom:90%;" />    <img src=".\results\interpolated_same_scene.png" alt="img" style="zoom:90%;" />

Figure XX  (a)  ground truth and the interpolated frame based on images from the same scene

<img src=".\results\origin_diff_scene.png" alt="img" style="zoom:90%;" />  <img src=".\results\interpolated_diff_scene.png" alt="img" style="zoom:90%;" />

​               (b) ground truth and the interpolated frame based on images from different scene

## Discussion  
### Issues we could avoid  
As mentioned previously in training, intensive RAM consumption is a huge challenge to our training process, since each consecutive images set (3 frames in total) could take 9GB RAM memory. Therefore, our method only takes in 3 sets of images at a time, and switches to new images sets in the next training section. However, this could lead to an issue called Catastrophic Forgetting, also known as Catastrophic Interference, in Machine Learning. It generally happens when new data is feeded to train the model, while the data differs significantly from previous one. The issue arises because as the CNN model is trained by more data, the model will learn new data quickly and fit them into model faster than before. In the meantime, the model may forget what it has learnt before.

Unfortunately, we could avoid this issue at beginning by simply pre-processing the training data. Intead of input 3 sets of images at each time, we could fetch slices of images from different images sets (from different scenes), so the input data could be diversified. 

### Possible Improvement  
- More training on data
  In this project, we perform 7 rounds of training; for each round, 3 sets of images from different scenes are used. In future research, more scenes are needed to apply model training.
  We set patience as 10. That means the training will terminate only if there is no improvement in the monitor performance measure within the 10 epochs. This might not be the most ideal model since it might go up or down from one epoch to the next. What we really care about is that the general trend should be improving, so higher patience value will boost the accuracy of the model which can be conducted in future improvement.

- Xavier initialization
  The variance of the activation value in training decreases layer by layer, which causes the gradient in backpropagation to also decrease layer by layer. To solve the disappearance of the gradient, it is necessary to avoid the reduction of the variance of the activation value. The ideal situation is that the output value of each layer maintains a Gaussian distribution. The basic idea of Xavier initialization is to keep the variance of the input and output consistent, so as to avoid all output values tending to zero.

### Convolution Layers
In our model, we use receptive block with size 79*79*6 as input. We use convolution layers and down-sampling layer to process the receptive block. The activation function we use is ReLu and we also use batch normalization to ensure the variance for both input node and output node is the same, which will help us to train our model better and faster. The first convolution layers will process the data to 32 x 73 x 73 size with 7 x 7 filter size and 1 x 1 stride size to get eigenvalues of the data. After the convolution layers, we use filter size 2 x 2 and stride size 2 x 2 for the down-sampling layer to reduce the dimensionality of the data. Repeat the step above to get output with 256 x 4 x 4. In order to get the final output with 41 x 82 x 1 x 1 size, we will proceed to reduce dimensionality to 1 x 1 by applying one convolution layers with 4 x 4 filter size and 1 x 1 stride size on it. After that, we use fully connected convolution layers with 1 x 1 filter size and 1 x 1 stride size to finally get our result kernel size 41 x 82 x 1 x 1.   

#### Niklaus's later research
Niklaus et al. have proposed a new method in their later research to optimize the memmory consumption. As shown in Figure ?, first, instead of generating a 2-D kernel for each pixel, their new network produces four 1-D kernels for each pixel.   

![New framework in Video Frame Interpolation via Adaptive Separable Convolution](./proposal/niklaus_framework2.png)   

This method is improved by reducing the dimension of the kernel, from one 2-dimension matrix K to two 1-dimension vectors, kh and kv, where they can be used to estimate K. Similar to Formular ?, kh and kv could be re-written as k1,h, k2,h and k1,v,  k2,v respectively. 
Since we can rewrite the original formular to calculate I(x, y) to Formular ?, the formular to estimate kernal K is Formular ?. In conclusion, the space complexity of each kernel will be reduced from O(n^2) to O(2n).

  
![Re-written Formular](./proposal/formula1.png)  
![Kernel Estimation](./proposal/formula3.png)  

## Reference  
- Ans, B., & Rousset, S. (1997). Avoiding catastrophic forgetting by coupling two reverberating neural networks. Comptes Rendus de l'Académie des Sciences-Series III-Sciences de la Vie, 320(12), 989-997.  
- Bao, W., Lai, W. S., Ma, C., Zhang, X., Gao, Z., & Yang, M. H. (2019). Depth-aware video frame interpolation. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (pp. 3703-3712).
- Cote, R. (2016, February 22). Motion Interpolation On TVs: Soap Opera Effect. Retrieved April 26, 2020, from https://www.rtings.com/tv/tests/motion/motion-interpolation-soap-opera-effect.
- Epson Frame Interpolation. (n.d.). Retrieved April 26, 2020, from https://files.support.epson.com/docid/cpd5/cpd52094/source/adjustments/tasks/frame_interpolation.html.
- Florian Raudies (2013) Optic flow. Scholarpedia, 8(7):30724. Retrieved May 10, 2020, from http://www.scholarpedia.org/article/Optic_flow
- Niklaus, S., Mai, L., & Liu, F. (2017). Video frame interpolation via adaptive convolution. In IEEE Conference on Computer Vision and Pattern Recognition (pp. 670-679).
- Niklaus, S., Mai, L., & Liu, F. (2017). Video frame interpolation via adaptive separable convolution. In IEEE International Conference on Computer Vision (pp. 261-270).
- What Is The Soap Opera Effect? - Everything You Need To Know. (2019, June 10). Retrieved April 26, 2020, from https://www.displayninja.com/what-is-the-soap-opera-effect/.


