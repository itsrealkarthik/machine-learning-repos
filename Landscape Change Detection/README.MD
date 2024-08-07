Landscape change detection
<b> Change detection due to human activities. <b>  <br>
<h1>Aim: </h1> Using satellite imagery, create an automated system for detecting change related only to human activities from satellite imagery. i.e. Develop AI/ML based model for change detection of only man-made objects like vehicles, buildings, roads, aircraft etc. from remote sensing images Data: Sentinel-2, LISS-4 <br>
 
<ul>Points to cover: 
<li>I. Objectives   </li>
<li>II. Data set analysis </li>

<li>III. Model Training and code snippets</li>
<li>IV.  Images</li>
<li>V. Result (accuracy etc)</li>
<li>VI. Gap and limitations</li>
</ul>
I.Objectives <br>
Importance: <br>
Monitoring deforestation: This project can be used to monitor deforestation by detecting changes in the amount of forest cover over time.  <br>
Urban Planning and Development: monitor changes in building structures, road networks, and land use in cities. 
Disaster Response and Management:<br>
In the event of natural disasters, such as earthquakes, floods, or wildfires, the system can quickly identify      changes in the affected areas<br>
Transportation and Infrastructure: Transportation agencies can use the system to monitor traffic, road conditions, and infrastructure changes.<br>
 Defense and Security:<br>
 The system can be used for national security and defense purposes by monitoring changes in sensitive                   areas, identifying the movement of military equipment, and detecting unauthorized construction near critical installations.<br>
            Steps To follow:<br>
1. Data Collection and Preprocessing:<br>
Acquire satellite imagery data (Sentinel-2 and LISS-4) from relevant sources.
Preprocess the data, including:cropping,resizing, Geometric correction. cloud removal.<br>
2. Data Labeling:<br>
Manually or semi-automatically label the data for training and validation.
Annotate images with information about the changes, specifying object types (e.g., buildings, roads).<br>
4. Model Selection and Training:<br>
Train the model using the labeled data.<br>
5. Model Integration:<br>
Develop an application or system for automated change detection.
Integrate the trained model into the system. Example  .h5 file integrated with flutter UI.
Ensure scalability and real-time processing capabilities.<br>
6. Testing and Validation:<br>
Test the system with unseen satellite imagery data.<br>
7. User Interface (UI) :<br>
Design a dashboard for inputting data and viewing change detection results.<br>


III.Model Training and code snippets  <br>
Technology and concepts used: <br>
 Keras : high-level neural networks API. Used for classification task with images as inputs, given the convolutional layers and the final dense layer with softmax activation. Each neuron applies regression over it and passes to  next layer.<br>
Softmax  Regression: Binary classification applied to multi class classification. <br>
Model We used: Slding Window approach<br>
Models we can use later: YoloV5<br>
Libraries used :<br>
import cv2
import os
import glob
import random
import numpy as np
from tqdm import tqdm

from patchify import patchify
from PIL import Image
import matplotlib.pyplot as plt
import tensorflow as tf

Data preprocessing:
WORKING_DIR = r'data'
NUMBER_OF_IMAGES = 2000 #You can put maximum of 5598 i.e. equal to `len(TRAIN_IMG)`
IMG_WIDTH = 1024
IMG_HEIGHT = 1024
PATCH_SIZE = 256
IMG_CHANNEL = 3
IOU_THRESHOLD = 0.7


# Patches per image
NUMBER_OF_PATCHES_PER_IMG = int((IMG_WIDTH / PATCH_SIZE) * (IMG_WIDTH / PATCH_SIZE))
TOTAL_PATCHES = NUMBER_OF_IMAGES * NUMBER_OF_PATCHES_PER_IMG
TRAIN_IMG = sorted(glob.glob(WORKING_DIR + '/images/*.png'))
TRAIN_MASK = sorted(glob.glob(WORKING_DIR + '/targets/*.png'))
image_dataset = []
for i, img in tqdm(enumerate(TRAIN_IMG)):
    if i== NUMBER_OF_IMAGES:
        break
    img_before_patch = cv2.imread(img) / 255
    img_patches = patchify(img_before_patch, (PATCH_SIZE, PATCH_SIZE, 3), step=PATCH_SIZE)
    img_patches = img_patches.reshape((NUMBER_OF_PATCHES_PER_IMG, PATCH_SIZE, PATCH_SIZE, 3))
    image_dataset.append(img_patches)
2000it [01:06, 30.17it/s]
image_dataset = np.array(image_dataset).reshape((NUMBER_OF_IMAGES * NUMBER_OF_PATCHES_PER_IMG, PATCH_SIZE, PATCH_SIZE, IMG_CHANNEL))
image_dataset.shape


Steps in data preprocessing<br>

NUMBER_OF_IMAGES: This sets the number of images to use for dataset creation. It appears that you are limiting the dataset to the first 2000 images from the available images in the directory.<br>
IMG_WIDTH and IMG_HEIGHT: These variables define the width and height of the images.<br>
PATCH_SIZE: The size of patches into which each image will be divided for processing.<br>
IMG_CHANNEL: This defines the number of image channels (usually 3 for RGB color images).<br>
IOU_THRESHOLD: The Intersection over Union (IoU) threshold, which is likely used for evaluating the accuracy of the segmentation model.<br>
Calculating Patch Details:<br>

NUMBER_OF_PATCHES_PER_IMG: This calculates the number of patches that can be extracted from each image. It divides the image into patches of size PATCH_SIZE with a step of PATCH_SIZE.<br>
TOTAL_PATCHES: This variable calculates the total number of patches considering all images in the dataset.<br>
TRAIN_IMG and TRAIN_MASK: These variables store file paths to the image and corresponding target mask files. They are obtained by using the glob module to search for files in the specified directories.<br>
Inside the loop, each image is read using OpenCV (cv2.imread) and normalized by dividing by 255 to scale pixel values to the range [0, 1].<br><br>
Patching Images:<br>
patchify is a custom or imported function that divides each image into non-overlapping patches of size (PATCH_SIZE, PATCH_SIZE, 3) with a step equal to PATCH_SIZE. The function may be a wrapper around slicing operations.<br>
Reshaping Patches:<br>
The patches are reshaped into a 4D array. img_patches is reshaped into a shape that represents (NUMBER_OF_PATCHES_PER_IMG, PATCH_SIZE, PATCH_SIZE, IMG_CHANNEL), and these patches are appended to image_dataset.<br>
Final Dataset Shape: <br>
After processing all the images, image_dataset is transformed into a NumPy array, resulting in a dataset with the shape (TOTAL_PATCHES, PATCH_SIZE, PATCH_SIZE, IMG_CHANNEL).<br>
TQDM Progress Bar:<br>
The tqdm library is used to display a progress bar while processing the images. It shows the progress of the loop as it processes each image.<br>

#Model training<br>
def unet_model(IMG_WIDTH, IMG_HIGHT, IMG_CHANNELS):
    inputs = tf.keras.layers.Input((IMG_WIDTH, IMG_HIGHT, IMG_CHANNELS))


    # Converted inputs to floating
    #s = tf.keras.layers.Lambda(lambda x: x / 255)(inputs)
    #Contraction path
    c1 = tf.keras.layers.Conv2D(16, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(inputs)
    c1 = tf.keras.layers.Dropout(0.1)(c1)
    c1 = tf.keras.layers.Conv2D(16, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(c1)
    p1 = tf.keras.layers.MaxPooling2D((2, 2))(c1)


    c2 = tf.keras.layers.Conv2D(32, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(p1)
    c2 = tf.keras.layers.Dropout(0.1)(c2)
    c2 = tf.keras.layers.Conv2D(32, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(c2)
    p2 = tf.keras.layers.MaxPooling2D((2, 2))(c2)


    c3 = tf.keras.layers.Conv2D(64, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(p2)
    c3 = tf.keras.layers.Dropout(0.2)(c3)
    c3 = tf.keras.layers.Conv2D(64, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(c3)
    p3 = tf.keras.layers.MaxPooling2D((2, 2))(c3)


    c4 = tf.keras.layers.Conv2D(128, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(p3)
    c4 = tf.keras.layers.Dropout(0.2)(c4)
    c4 = tf.keras.layers.Conv2D(128, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(c4)
    p4 = tf.keras.layers.MaxPooling2D(pool_size=(2, 2))(c4)


    c5 = tf.keras.layers.Conv2D(256, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(p4)
    c5 = tf.keras.layers.Dropout(0.3)(c5)
    c5 = tf.keras.layers.Conv2D(256, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(c5)


    #Expansive path
    u6 = tf.keras.layers.Conv2DTranspose(128, (2, 2), strides=(2, 2), padding='same')(c5)
    u6 = tf.keras.layers.concatenate([u6, c4])
    c6 = tf.keras.layers.Conv2D(128, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(u6)
    c6 = tf.keras.layers.Dropout(0.2)(c6)
    c6 = tf.keras.layers.Conv2D(128, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(c6)


    u7 = tf.keras.layers.Conv2DTranspose(64, (2, 2), strides=(2, 2), padding='same')(c6)
    u7 = tf.keras.layers.concatenate([u7, c3])
    c7 = tf.keras.layers.Conv2D(64, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(u7)
    c7 = tf.keras.layers.Dropout(0.2)(c7)
    c7 = tf.keras.layers.Conv2D(64, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(c7)


    u8 = tf.keras.layers.Conv2DTranspose(32, (2, 2), strides=(2, 2), padding='same')(c7)
    u8 = tf.keras.layers.concatenate([u8, c2])
    c8 = tf.keras.layers.Conv2D(32, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(u8)
    c8 = tf.keras.layers.Dropout(0.1)(c8)
    c8 = tf.keras.layers.Conv2D(32, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(c8)


    u9 = tf.keras.layers.Conv2DTranspose(16, (2, 2), strides=(2, 2), padding='same')(c8)
    u9 = tf.keras.layers.concatenate([u9, c1], axis=3)
    c9 = tf.keras.layers.Conv2D(16, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(u9)
    c9 = tf.keras.layers.Dropout(0.1)(c9)
    c9 = tf.keras.layers.Conv2D(16, (3, 3), activation='relu', kernel_initializer='he_normal', padding='same')(c9)


    outputs = tf.keras.layers.Conv2D(1, (1, 1), activation='sigmoid')(c9)


    model = tf.keras.Model(inputs=[inputs], outputs=[outputs])
    model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
   
    return model
model = unet_model(PATCH_SIZE, PATCH_SIZE, IMG_CHANNEL)
