
This notebook is based on Chapter 6 Interpreting an Image: Classification


In classification, the response variable is categorical or discrete, meaning it has a finite number of categories. Examples include predicting whether an email is spam or not, classifying
images into different objects.

On the other hand, regression predicts continuous variables, where the target variable can take on any value within a range. Examples include predicting a person’s
income, the price of a house, or the temperature.

In remote sensing, image classification is an attempt to categorize all pixels
in an image into a finite number of labeled land cover and/or land use classes

Supervised classification uses a training dataset with known labels and rep-
resenting the spectral characteristics of each land cover class of interest to
“supervise” the classification. The overall approach of a supervised classification
in Earth Engine is summarized as follows:

1. Get a scene.
2. Collect training data.
3. Select and train a classifier using the training data.
4. Classify the image using the selected classifier.