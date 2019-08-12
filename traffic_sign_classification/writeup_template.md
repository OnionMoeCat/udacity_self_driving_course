# **Traffic Sign Recognition** 

## Writeup

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Build a Traffic Sign Recognition Project**

The goals / steps of this project are the following:
* Load the data set (see below for links to the project data set)
* Explore, summarize and visualize the data set
* Design, train and test a model architecture
* Use the model to make predictions on new images
* Analyze the softmax probabilities of the new images
* Summarize the results with a written report


[//]: # (Image References)

[image1]: ./histo.png "Visualization"
[image2]: ./grayscale.png "Grayscaling"
[image3]: ./transformed.png "Augmented"
[image4]: ./modi_histo.png "Histo after augmentation"
[image5]: ./layers.png "Layers"
[image6]: ./images.png "Traffic Signs"
[image7]: ./classifications.png "Predictions"

## Rubric Points
### Here I will consider the [rubric points](https://review.udacity.com/#!/rubrics/481/view) individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. You can submit your writeup as markdown or pdf. You can use this template as a guide for writing the report. The submission includes the project code.

You're reading it! and here is a link to my [project code](https://github.com/udacity/CarND-Traffic-Sign-Classifier-Project/blob/master/Traffic_Sign_Classifier.ipynb)

### Data Set Summary & Exploration

#### 1. Provide a basic summary of the data set. In the code, the analysis should be done using python, numpy and/or pandas methods rather than hardcoding results manually.

I used the pandas library to calculate summary statistics of the traffic
signs data set:

- Number of training examples = 34799
- Number of valid examples= 4410
- Number of testing examples = 12630
- Image data shape = (32, 32, 3)
- Number of classes = 43

#### 2. Include an exploratory visualization of the dataset.

Here is an exploratory visualization of the data set. It is a bar chart showing how the data ...
![alt text][image1]

### Design and Test a Model Architecture

#### 1. Describe how you preprocessed the image data. What techniques were chosen and why did you choose these techniques? Consider including images showing the output of each preprocessing technique. Pre-processing refers to techniques such as converting to grayscale, normalization, etc. (OPTIONAL: As described in the "Stand Out Suggestions" part of the rubric, if you generated additional data for training, describe why you decided to generate additional data, how you generated the data, and provide example images of the additional data. Then describe the characteristics of the augmented training set like number of images in the set, number of images for each class, etc.)

My dataset preprocessing consisted of: Converting to grayscale - This worked well as described in their traffic sign classification article. It also helps to reduce training time, which was nice when a GPU wasn't available. Normalizing the data to the range (-1,1) - The reason is finding a singular learning rate is much easier for normalized data.

Rumor has it that data augmentation is the single best method to increase accuracy of the model. Because several classes in the data have far fewer samples than others the model will tend to be biased toward those classes with more samples. I implemented augmentation by creating copies of each sample for a class (sometimes several copies) in order to boost the number of samples for the class to 800 (if that class didn't already have at least 800 samples). Each copy is fed into a "jitter" pipeline that randomly translates, scales, warps adjusts the image.

Then, I used skilearn kit to split the training set into 80%:20%, where 80% as the training set and 20% as the validation set. With more training data and validation data, the model's parameter estimates have smaller variance. With less testing data, tne model's performance statistic will have greater variance. Less variance in both cases are good for the model's training.

The parameters used to generate jittered images: angle_deg: [-1.5, 1.5] scale: [0.9375, 1.0675] offset_pixel: [-2, 2]

The grayscale image:
![alt text][image2]

The augmented image:
![alt text][image3]

The new dataset: 
train set size:  37184
valid set size:  9296

![alt text][image4]


#### 2. Describe what your final model architecture looks like including model type, layers, layer sizes, connectivity, etc.) Consider including a diagram and/or table describing the final model.

My final model consisted of the following layers:

| Layer         		|     Description	        					| 
|:---------------------:|:---------------------------------------------:| 
| Input         		| 32x32x3 RGB image   							| 
| Convolution 5x5     	| 1x1 stride, valid padding, outputs 28x28x6 	|
| RELU					|												|
| Max pooling	      	| 2x2 stride,  outputs 14x14x16  				|
| Convolution 5x5	    | 1x1 stride, valid padding, outputs 10x10x16   |
| RELU           		|             									|
| Max pooling			| 2x2 stride, outputs 5x5x16   					|
| Convolution 5x5       | 1x1 stride, valid padding, outputs 1x1x400 	|
| RELU					|												|
| flatten 				| outputs 1x1x400								|
| concat 				| concat the two 1x1x400 to 1x1x800				|
| dropout 				| probability 0.5               				|
| Fully connected       | input 800, output 43               			|

![alt text][image5]



#### 3. Describe how you trained your model. The discussion can include the type of optimizer, the batch size, number of epochs and any hyperparameters such as learning rate.

I used the Adam optimizer (already implemented in the LeNet lab). The final settings used were:

batch size: 128 epochs: 50 learning rate: 0.0009 mu: 0 sigma: 0.1 dropout keep probability: 0.5

#### 4. Describe the approach taken for finding a solution and getting the validation set accuracy to be at least 0.93. Include in the discussion the results on the training, validation and test sets and where in the code these were calculated. Your approach may have been an iterative process, in which case, outline the steps you took to get to the final solution and why you chose those steps. Perhaps your solution involved an already well known implementation or architecture. In this case, discuss why you think the architecture is suitable for the current problem.

My final model results were:
* training set accuracy of 0.999
* validation set accuracy of 0.954
* test set accuracy of 0.941

If a well known architecture was chosen:
* What architecture was chosen: The model described in the LeCun's traffic sign recognition paper. 
* Why did you believe it would be relevant to the traffic sign application: Because the paper's result is really good. 
* How does the final model's accuracy on the training, validation and test set provide evidence that the model is working well: The accuracy of training set (0.999), validation set (0.954), test set (0.941). 

My approach was a little of both. Like I mentioned earlier, I started with pre-defined architectures (Sermanet/LeCun model) and almost all of the tweaking from there was a process of trial and error.
 

### Test a Model on New Images

#### 1. Choose five German traffic signs found on the web and provide them in the report. For each image, discuss what quality or qualities might be difficult to classify.

Here are 8 German traffic signs that I found on the web:

![alt text][image6]

Nothing in particular sticks out that I think would make classification difficult. My images appear to be more easily distinguishable than quite a few images from the original dataset. I noticed that my images tend to be quite a bit brighter and might occupy a different range in the color space, possibly a range that the model was not trained on. In addition, the GTSRB dataset states that the images "contain a border of 10 % around the actual traffic sign (at least 5 pixels) to allow for edge-based approaches" and the images that I used do not all include such a border. This could be another source of confusion for the model.

#### 2. Discuss the model's predictions on these new traffic signs and compare the results to predicting on the test set. At a minimum, discuss what the predictions were, the accuracy on these new predictions, and compare the accuracy to the accuracy on the test set (OPTIONAL: Discuss the results in more detail as described in the "Stand Out Suggestions" part of the rubric).

Here are the results of the prediction: every picture is predicted as category 18 "General caution"

Answer: The model does not work as well as in the test dataset. The accuracy is 1 out of 8: 12.5%, which is much worse than 94.1% on the test set. The model tends to prefer the picture to be from category 18. I would like some pointers on the model's behaviour.

#### 3. Describe how certain the model is when predicting on each of the five new images by looking at the softmax probabilities for each prediction. Provide the top 5 softmax probabilities for each image along with the sign type of each probability. (OPTIONAL: as described in the "Stand Out Suggestions" part of the rubric, visualizations can also be provided such as bar charts)

The top three softmax probabilities for the predictions on the German traffic sign images found on the web. 

![alt text][image7]


### (Optional) Visualizing the Neural Network (See Step 4 of the Ipython notebook for more details)
#### 1. Discuss the visual output of your trained network's feature maps. What characteristics did the neural network use to make classifications?


