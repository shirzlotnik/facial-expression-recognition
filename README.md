# EmotionDetection

## Install Dataset
install dataset from this [kaggle](https://www.kaggle.com/lxyuan0420/facial-expression-recognition-using-cnn/data) project -> 
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
Index | emotion | pixels | Usage
------------ | ------------- | ------------- | -------------
0 | 0 | 0 80 82 72 58 58 60 63 54 58 60 48 ... | Training
1 | 0 | 151 150 147 155 148 133 111 140 170... | Training
2 | 2 | 231 212 156 164 174 138 161 173 182... | Training
3 | 4 | 24 32 36 30 32 23 19 20 30 41 21 22... | Training
4 | 6 | 4 0 0 0 0 0 0 0 0 0 0 0 3 15 23 28 ... | Training

