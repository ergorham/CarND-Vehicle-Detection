## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car.png
[image1a]: ./output_images/not_car.png
[image2a]: ./output_images/hog_example1.png
[image2b]: ./output_images/hog_example2.png
[image2c]: ./output_images/hog_example3.png
[image3]: ./output_images/all_windows.png
[image4a]: ./output_images/pipelineTest1.png
[image4b]: ./output_images/pipelineTest2.png
[image4c]: ./output_images/pipelineTest3.png
[image4d]: ./output_images/pipelineTest4.png
[image4e]: ./output_images/pipelineTest5.png
[image4f]: ./output_images/pipelineTest6.png
[image5a]: ./output_images/bbox1.png
[image5b]: ./output_images/bbox2.png
[image5c]: ./output_images/bbox3.png
[image5d]: ./output_images/bbox4.png
[image5e]: ./output_images/bbox5.png
[image5f]: ./output_images/bbox6.png
[image6a]: ./output_images/heat1.png
[image6b]: ./output_images/heat2.png
[image6c]: ./output_images/heat3.png
[image6d]: ./output_images/heat4.png
[image6e]: ./output_images/heat5.png
[image6f]: ./output_images/heat6.png
[video1]: ./output_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in lines 19 through 36 of the file called `handyfunctions.py`.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]
![alt text][image1a]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2a]
![alt text][image2b]
![alt text][image2c]

#### 2. Explain how you settled on your final choice of HOG parameters.

In experimenting with parameters for HOG feature extraction, I started by determining which direction of change for each parameter provided more versus less information. For instance, increasing the number of orientations provided more bins, whereas increasing the pixels per cell decreased the resolution of image and thus reduced the information available to the algorithm.

I had used a subset of the car and not car data (1200 images each) to train the classifier. I would then gauge the performance of the classifier against the sample images to determine effectiveness of the parameters. I then would reduce the amount of information available to the classifier and determine how far I can reduce the parameters and still achieve positive results.

In the end, I settled on `orientations=11`, `pixels_per_cell=16`, `cells_per_block=2`, `hog_channels="ALL"`. When trained over all available samples, this provided the most consistent results in the least amount of train time.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using `sklearn`, as seen in my notebook `VehDetect.ipynb`, under the heading `Build and Train Classifer`. I opted for the SVM classifier in `sklearn`: `SVC` due to the parameters available for tuning. In using a grid search optimizer, I tested between `rbf` and `linear` kernels, and values of `C=0.0001` to `C=10` in decade increments.

Once the optimizer settled on `linear` and `C=0.001`, I froze those parameters for future retraining in order to save time.

I had opted to use color features as well as HOG features in order to improve the robustness of my classifier, again taking the same "minimum information to do the job" approach. This resulted in `spatial_size=(32,32)` and `histogram_bins=48`, meaning keeping a quarter of the incoming image data in spatial binning, and one fifth of the data in histogram binning.

Based on the Advanced Lane Detection project, and confirmed through empirical results, setting `color_space=YCrCb` yielded the best results from the sample images and the test video. This is likely due to the bright images and high contrast between the cars and the background.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to limit the search windows between `x=350` and `x=1280` as this is the likely range where cars would be present. This is seen within the notebook, under the heading `Sliding Windows Implementation`. I opted on five sizes of windows, `64`, `96`, `128`, `160`, and `192` pixels, as determined by examining the video and pictures for the various sizes of cars.

Each window size was given a range for searching, allowing a 70% overlap. The amount of overlap was determined empirically, maximizing positive detection in the test video while minimizing false positives.

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4a]
![alt text][image4b]
![alt text][image4c]
![alt text][image4d]
![alt text][image4e]
![alt text][image4f]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./output_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

Taking from the lessons learned from the `Advanced Lane Lines` project, I implemented a smoothing algorithm to add the heat maps over 10 frames, this results in an update rate of 400 ms. I keep a running list of the the heat maps generated and add the last ten heat maps together, thresholding the heatmap values to zero for detections of two or less.

### Here are six frames and their corresponding heatmaps:

![alt text][image5a] ![alt text][image6a]
![alt text][image5b] ![alt text][image6b]
![alt text][image5c] ![alt text][image6c]
![alt text][image5d] ![alt text][image6d]
![alt text][image5e] ![alt text][image6e]
![alt text][image5f] ![alt text][image6f]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The biggest challenge I had was processing time. Retraining each time I tuned a hyperparameter, it took several minutes to retrain the SVM. I initially would test on a subset of the data to ensure I was on the right track. At one point, the model seemed to not generalize enough so I added more examples to the dataset (GTI Right) and that corrected the false positives. 

As a result of using KTTI and GTI Right datasets for training, the model is highly biased for left-lane driving as in the video. The pipeline will not generalize well for centre or right lane driving. 

There are instances where HOG is not suited for detecting a car, particularly when a car is close up with feature contrast (for instance, only seeing front quarter panel). It's not until more of the shape of a car is visible will HOG be able to detect a car.

The sample video had very good lighting. It would be interesting to see how the algorithm performs in lower contrast settings. Equally, the system will perform poorly in inclement weather. Fusioning with sensors other than optical will be beneficial in these cases.

