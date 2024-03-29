# Template-Matching_play
玩陽春Template Matching，sliding window方法

medium: https://yellowgirl3614.medium.com/template-matching-sliding-window-90f3941e3c4f

```python
import os
import cv2
import ipyplot
import numpy as np
from matplotlib import pyplot as plt
from matplotlib.image import imread
import pandas as pd
```


```python
image_path = "img2.png"
template_path = "temp.png"
```


```python
img = plt.imread(image_path)
plt.imshow(img)
```




    <matplotlib.image.AxesImage at 0x2359129b608>




    
![png](image/output_2_1.png)
    



```python
temp_img = plt.imread(template_path)
plt.imshow(temp_img)
```




    <matplotlib.image.AxesImage at 0x235912eaa48>




    
![png](image/output_3_1.png)
    



```python
print(img.shape)
print(temp_img.shape)
```

    (250, 250, 4)
    (70, 71, 4)
    

### BGRA
RGBA是代表R、G、Blue + Alpha的色彩空間

![wiki](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a7/Hue_alpha.png/220px-Hue_alpha.png)

**alpha通道一般用作不透明度參數。**
如果一個像素的alpha通道數值為0%，那它就是完全透明的（也就是看不見的）
而數值為100%則意味著一個完全不透明的像素（傳統的數位圖像）。
在0%和100%之間的值則使得像素可以透過背景顯示出來，就像透過玻璃（半透明性），這種效果是簡單的二元透明性（透明或不透明）做不到的。它使數碼合成變得容易。alpha通道值可以用百分比、整數或者像RGB參數那樣用0到1的實數表示。

有時它也被寫成ARGB（像RGBA一樣，但是第一個數據是alpha），是Macromedia的產品使用的術語。
比如，0x80FFFF00是50%透明的黃色，因為所有的參數都在0到255的範圍內表示。0x80是128，大約是255的一半。

#### PNG是一種使用RGBA的圖像格式。

from: https://zh.wikipedia.org/wiki/RGBA


```python
img = cv2.cvtColor(img, cv2.COLOR_BGRA2BGR)
temp_img = cv2.cvtColor(temp_img, cv2.COLOR_BGRA2BGR)
```


```python
print(img.shape)
print(temp_img.shape)
```

    (250, 250, 3)
    (70, 71, 3)
    

### 做slideing window並做cosine similarity 


```python
def cos_simiarity(a1, a2):
    cos_sim = np.dot(a1, a2) / (np.linalg.norm(a1) * np.linalg.norm(a2))
    return cos_sim
```


```python
w = temp_img.shape[1]
h = temp_img.shape[0]

img_size = img.shape[0]

w_times = img_size-w+1
h_times = img_size-h+1

for ht in range(h_times):
    for t in range(w_times):
        part_img = np.matrix.flatten(img[ht:ht+h, t:t+w])
        part_temp = np.matrix.flatten(temp_img)
        
        cos = cos_simiarity(part_img, part_temp)
        # print(cos)
        if cos >= 0.999:
            plt.imshow(img[ht:ht+h, t:t+w])
            print('[{w_from}:{w_to}, {h_from}:{h_to}]'.format(w_from = str(t), w_to = str(t+w),
                                                             h_from = str(ht), h_to = str(ht+h)))
```

    [92:163, 95:165]
    


    
![png](image/output_10_1.png)
    



```python
img_copy = np.copy(img)
cv2.rectangle(img_copy, pt1=(92,95), pt2=(164, 166), color=(255, 0, 0), thickness=2)
plt.imshow(img_copy)
plt.show()
```

    Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers).
    


    
![png](image/output_11_1.png)
    


### 用cosine similarity去找相似效果不佳，他會去找到總和相近的，但位置可能差很多，例如:
![img](https://imgur.com/VSpYfdk.png)

使用cossimi_match的結果


```python
def cossimi_match(a1, a2):
    cos_sim = np.dot(a1, a2) / (np.linalg.norm(a1) * np.linalg.norm(a2))
    if cos_sim > 0.96:
        print(cos_sim)
        return True
    else:
        return False
```


```python
def template_match2(a1, a2):
    sim = np.sum(a1-a2)
    if sim < 2.6 and sim > 0:
        print(sim)
        return True
    else
        return False
```


```python
image2_path = "img.png"
img2 = plt.imread(image2_path)
img2 = cv2.cvtColor(img2, cv2.COLOR_BGRA2BGR)
print(img2.shape)
plt.imshow(img2)
```

    (209, 209, 3)
    




    <matplotlib.image.AxesImage at 0x2359178a308>




    
![png](image/output_15_2.png)
    



```python
w = temp_img.shape[1]
h = temp_img.shape[0]

img_size_w = img2.shape[1]
img_size_h = img2.shape[0]

w_times = img_size_w-w+1
h_times = img_size_h-h+1
```


```python
for ht in range(h_times):
    for t in range(w_times):
        part_img = np.matrix.flatten(img2[ht:ht+h, t:t+w])
        part_temp = np.matrix.flatten(temp_img)
        
        cos = template_match2(part_img, part_temp)
        # print(cos)
        if cos:
            # plt.subplot(10, 5, i)
            plt.imshow(img2[ht:ht+h, t:t+w])
            print('[{w_from}:{w_to}, {h_from}:{h_to}]'.format(w_from = str(t), w_to = str(t+w),
                                                             h_from = str(ht), h_to = str(ht+h)))
            i+=1
        plt.show()
```

    2.5137634
    [73:144, 77:147]
    


    
![png](image/output_17_1.png)
    


### 使用平方差


```python
def template_match_meansqare(a1, a2):
    sim = np.sum(np.square(a1)-np.square(a2))
    if sim < 1.0 and sim > 0:
        print(sim)
        return True
    else:
        return False
```


```python
for ht in range(h_times):
    for t in range(w_times):
        part_img = np.matrix.flatten(img2[ht:ht+h, t:t+w])
        part_temp = np.matrix.flatten(temp_img)
        
        cos = template_match_meansqare(part_img, part_temp)
        # print(cos)
        if cos:
            # plt.subplot(10, 5, i)
            plt.imshow(img2[ht:ht+h, t:t+w])
            print('[{w_from}:{w_to}, {h_from}:{h_to}]'.format(w_from = str(t), w_to = str(t+w),
                                                             h_from = str(ht), h_to = str(ht+h)))
            i+=1
        plt.show()
```

    0.7698059
    [63:134, 71:141]
    


    
![png](image/output_20_1.png)
    



```python
img_copy2 = np.copy(img2)
cv2.rectangle(img_copy2, pt1=(63,71), pt2=(134, 141), color=(255, 0, 0), thickness=2)
plt.imshow(img_copy2)
plt.show()
```

    Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers).
    


    
![png](image/output_21_1.png)
    

### References:
1. 特徵匹配 (Template Matching): https://medium.com/@bkwtwn/%E7%89%B9%E5%BE%B5%E5%8C%B9%E9%85%8D-template-matching-f2de49998dcc