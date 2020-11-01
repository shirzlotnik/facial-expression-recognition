# EmotionDetection

```python

```

## Install Dataset
install dataset from this [kaggle](https://www.kaggle.com/lxyuan0420/facial-expression-recognition-using-cnn/data) project   
fer2013.csv


## Installation

```bash
pip install numpy
pip install keras
pip install tensorflow
pip install pandas
pip install matplotlib
pip install seaborn
```

## Import Libraries
```python
import numpy as np # linear algebra
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)

from keras.utils import to_categorical
from keras.callbacks import EarlyStopping
from keras.models import Sequential
from keras.layers import Dense, Dropout, Activation, Flatten
from keras.layers import Conv2D, MaxPooling2D, BatchNormalization
from keras.losses import categorical_crossentropy
from sklearn.metrics import accuracy_score
from keras.optimizers import Adam
from keras.regularizers import l2
from keras.preprocessing.image import ImageDataGenerator
from sklearn.metrics import classification_report, confusion_matrix
import matplotlib.pyplot as plt
import seaborn as sns
```

Than it will print this if everything work fine  
Using TensorFlow backend.  

## Dataset Overview
You need to make sure you know the location of the file in directory
```python
file_path = '/Users/shirzlotnik/emotion_dataset/fer2013.csv' # file path in the computer

data = pd.read_csv(file_path)
# check data shape
print(data.shape)
# preview first 5 row of data
print(data.head(5))
```
(35887, 3)

Index | emotion | pixels | Usage
------------ | ------------- | ------------- | -------------
0 | 0 | 0 80 82 72 58 58 60 63 54 58 60 48 ... | Training
1 | 0 | 151 150 147 155 148 133 111 140 170... | Training
2 | 2 | 231 212 156 164 174 138 161 173 182... | Training
3 | 4 | 24 32 36 30 32 23 19 20 30 41 21 22... | Training
4 | 6 | 4 0 0 0 0 0 0 0 0 0 0 0 3 15 23 28 ... | Training

```python
# check usage values
# 80% training, 10% validation and 10% test
print(data.Usage.value_counts())
```
Training       28709  
PublicTest      3589  
PrivateTest     3589  
Name: Usage, dtype: int64  


```python
 # check target labels
emotion_map = {0: 'Angry', 1: 'Digust', 2: 'Fear', 3: 'Happy', 4: 'Sad', 5: 'Surprise', 6: 'Neutral'}
emotion_counts = data['emotion'].value_counts(sort=False).reset_index()
emotion_counts.columns = ['emotion', 'number']
emotion_counts['emotion'] = emotion_counts['emotion'].map(emotion_map)
print(emotion_counts)
```
 A | emotion | number  
------------ | ------------- | ------------- 
0 | Angry | 4953  
1 | Digust | 547  
2 | Fear | 5121  
3 | Happy | 8989  
4 | Sad | 6077  
5 | Surprise | 4002
6 | Neutral | 6198

```python
# Plotting a bar graph of the class distributions
plt.figure(figsize=(6,4))
sns.barplot(emotion_counts.emotion, emotion_counts.number)
plt.title('Class distribution')
plt.ylabel('Number', fontsize=12)
plt.xlabel('Emotions', fontsize=12)
plt.show()
```
![class distribution](https://github.com/shirzlotnik/EmotionDetection/blob/main/class_distribution.png?raw=true)



## plot some images

```python
def row2image(row):
    pixels, emotion = row['pixels'], emotion_map[row['emotion']]
    img = np.array(pixels.split())
    img = img.reshape(48,48)
    image = np.zeros((48,48,3))
    image[:,:,0] = img
    image[:,:,1] = img
    image[:,:,2] = img
    return np.array([image.astype(np.uint8), emotion])

plt.figure(0, figsize=(12,6))
for i in range(1,8):
    face = data[data['emotion'] == i-1].iloc[0]
    img = row2image(face)
    plt.subplot(2,4,i)
    plt.imshow(img[0])
    plt.title(img[1])

plt.show()
```
![Images](https://github.com/shirzlotnik/EmotionDetection/blob/main/angry.png?raw=true)


## Pre-processing data

1. Splitting dataset into 3 parts: train, validation, test
2. Convert strings to lists of integers
3. Reshape to 48x48 and normalise grayscale image with 255.0
4. Perform one-hot encoding label, e.g. class 3 to [0,0,0,1,0,0,0]

```python

#split data into training, validation and test set
data_train = data[data['Usage']=='Training'].copy()
data_val   = data[data['Usage']=='PublicTest'].copy()
data_test  = data[data['Usage']=='PrivateTest'].copy()
print("train shape: {}, \nvalidation shape: {}, \ntest shape: {}".format(
        data_train.shape, data_val.shape, data_test.shape))
```

train shape: (28709, 3),   
validation shape: (3589, 3),  
test shape: (3589, 3)  

```python
# barplot class distribution of train, val and test
emotion_labels = ['Angry', 'Disgust', 'Fear', 'Happy', 'Sad', 'Surprise', 'Neutral']

def setup_axe(axe,df,title):
    df['emotion'].value_counts(sort=False).plot(ax=axe, kind='bar', rot=0)
    axe.set_xticklabels(emotion_labels)
    axe.set_xlabel("Emotions")
    axe.set_ylabel("Number")
    axe.set_title(title)
    
    # set individual bar lables using above list
    for i in axe.patches:
        # get_x pulls left or right; get_height pushes up or down
        axe.text(i.get_x()-.05, i.get_height()+120, \
                str(round((i.get_height()), 2)), fontsize=14, color='dimgrey',
                    rotation=0)

   
fig, axes = plt.subplots(1,3, figsize=(20,8), sharey=True)
setup_axe(axes[0],data_train,'train')
setup_axe(axes[1],data_val,'validation')
setup_axe(axes[2],data_test,'test')
plt.show()
```

![Charts](https://github.com/shirzlotnik/EmotionDetection/blob/main/chart1.png?raw=true)

Notice that the later two subplots share the same y-axis with the first subplot.  
The size of train, validation, test are 80%, 10% and 10%, respectively.  
The exact number of each class of these datasets are written on top of their x-axis bar.  

```python
#initilize parameters
num_classes = 7 
width, height = 48, 48
num_epochs = 50
batch_size = 64
num_features = 64
```

```python
"""
CRNO stands for Convert, Reshape, Normalize, One-hot encoding
(i) convert strings to lists of integers
(ii) reshape and normalise grayscale image with 255.0
(iii) one-hot encoding label, e.g. class 3 to [0,0,0,1,0,0,0]
"""

def CRNO(df, dataName):
    df['pixels'] = df['pixels'].apply(lambda pixel_sequence: [int(pixel) for pixel in pixel_sequence.split()])
    data_X = np.array(df['pixels'].tolist(), dtype='float32').reshape(-1,width, height,1)/255.0   
    data_Y = to_categorical(df['emotion'], num_classes)  
    print(dataName, "_X shape: {}, ", dataName, "_Y shape: {}".format(data_X.shape, data_Y.shape))
    return data_X, data_Y

    
train_X, train_Y = CRNO(data_train, "train") #training data
val_X, val_Y     = CRNO(data_val, "val") #validation data
test_X, test_Y   = CRNO(data_test, "test") #test data
```
train _X shape: {},  train _Y shape: (28709, 48, 48, 1)  
val _X shape: {},  val _Y shape: (3589, 48, 48, 1)  
test _X shape: {},  test _Y shape: (3589, 48, 48, 1)  

```python

```
