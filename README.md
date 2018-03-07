
## Vehicle Detection Project
### Johannes Paas

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/YOLO2.JPG
[image2]: ./output_images/single_image.JPG

[video1]: ./output_videos/project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

You're reading it!

### YOLO detection and classification CNN

Instead of the proposed HOG and SVN method, I used a convolutional deep neural network to do the object detection and classification. a convolution works similar to sliding window search as it can be seen as a filter that slides across the image and looks for trained features. The associated research papers also state, that it is much more performant and robust than methods like e.g. HOG and SVN's. Further reading on the network can be done [here](https://arxiv.org/pdf/1506.02640.pdf).

The image below shows the structure of YOLO2. The input is firstly resized, so multiple input resolutions can be handled. Then multiple convolutional layers follow, that basically are filters of increasingly higher levels.

![alt_text][image1]

Weights for a trained network are available online, so I could save me the time to train myself. The network puts out solid detections of cars after loading the weights by initializing the network as follows:

```
options = {
    'model': 'cfg/yolo.cfg',
    'load': 'bin/yolo.weights',
    'threshold': 0.3,
    'gpu': 0.8
}

tfnet = TFNet(options)
```
To replicate the outputs, go to [darkflow](https://github.com/thtrieu/darkflow), install the network by following the instructions and download the weights for 608x608 images. Create the `bin` folder in the `darkflow` directory and put the weights inside.

The network puts out different classes, so first I filter for the class `car`. The output then contains the top-left and bottom-right corner of the bounding box. Below is an output image of the single image pipeline. You can see the different (yet in this case similar) colors of the bounding boxes, which indicate different detections, but more on that in the **Video** section

![alt_text][image2]

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

Here's a [link to my video result](./output_videos/project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The output of the network could alredy be used to draw the bounding boxes on the image, since it is pretty stable, which was my first result. Then I started to look for a way to track the vehicles, since for planning it is important to know, where the vehicle will go in the next steps. After trying to implement my own kalman filter, eventually for performance reasons I imported a kalman filter class.

The kalman filter provides big advantages over for example taking the mean over the last three detections and adding a new one if it falls inside a circle of defined radius around the current mean. In addition to the observable states (x and y coordinates and size of the detection), the kalman filter can estimate the velocity in x and y over time. This enables it to *predict* the exact (noisy) position, in which the tracked object is expected in a new frame. This *prediction* can then be *updated* with the current measurement. This finally leads to an *estimate* of the objects states.

By transforming the top-left and bottom-right inputs into center-point, area `(w\*h)` and aspect ratio `(w/h)`, those states can be estimated. This leads to more stable detections and smoother transitions. 

the project video the colors of the different bounding boxes indicate different track-ID's. I just iterate through a set of colors.



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

This project was a lot of fun and ganve me some really cool insights. After getting the network to run on my GPU the results were pretty good. My first try to implement a tracker with individual kalman filters was not very efficient though. Using the imported class for that really gave me a lot of insights on how to initialize, purge and administrate the tracking lists.

I am very happy with my result and am thinking about putting it into one of our cars at my job to see how it performs on live data! Also I want to experiment with differnet CNN's and track multiple classes for example trucks, cars, road signs, pedestrians, etc.

To improve performance the tracking could be enhanced, so detections are updated by pure prediction, even without measurements. This could help to redetect the white car as the same color (ID), when the black car has passed.


