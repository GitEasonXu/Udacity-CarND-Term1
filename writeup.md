# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

## **1. Reflection**
### My pipeline consisted of 5 steps. Specific can see the following introduction：
### Pipeline 主要由5步构成，具体可看以下介绍：
* ## Step 1: Convert RGB image to grayscale

<figure>
<img src="https://github.com/GitEasonXu/Udacity-CarND-Term1/blob/master/image/gray.png" width="380" alt="Gray Image" />
</figure>

`cv2.cvtColor(img, cv2.COLOR_RGB2GRAY)`

* ## Step 2: Canny Edge Detection
Before performing edge detection on a picture, we should first use a Gaussian filter function to sort the pictures so that noise can be filtered.

在对图片进行边缘检测之前，首先我们应该使用高斯滤波都图片进行整理，这样可以过滤噪声。

<figure>
<img src="https://github.com/GitEasonXu/Udacity-CarND-Term1/blob/master/image/edges.png" width="380" alt="Edge Image" />
</figure>

`blur_img = gaussian_blur(gray_img, kernel_size=3)             #高斯滤波`  
`edges = canny(blur_img,low_threshold=50, high_threshold=150) #canny边缘检测`

* ## Step 3: Extracting the region of interest
Through the above two steps, we can already get the edge picture, but there are many regions in the picture that we are not interested. How can we extract the regions of interest?

通过上面两步，我们已经可以获取到边缘图片，但是图片中有很多区域并不是我们感兴趣的区域，那么如何将我们感兴趣区域提取出来呢？

<figure>
<img src="https://github.com/GitEasonXu/Udacity-CarND-Term1/blob/master/image/region.png" width="380" alt="Region Image" />
</figure>

`vertices = np.array([[[0,image_y],[460,320],[500,320],[image_x,image_y]]], dtype=np.int32) `
**The `vertices` parameter is about the region of interest.** 

`region_img = region_of_interest(edges, vertices)  #提取区域图片`  
The function `region_of_interest` return the interesting region of image.  
```
mask = np.zeros_like(img)  
    #defining a 3 channel or 1 channel color to fill the mask with depending on the input image
    if len(img.shape) > 2:
        channel_count = img.shape[2]  # i.e. 3 or 4 depending on your image
        ignore_mask_color = (255,) * channel_count
    else:
        ignore_mask_color = 255
    #filling pixels inside the polygon defined by "vertices" with the fill color    
    cv2.fillPoly(mask, vertices, ignore_mask_color)
```        
将mask的vertices区域填充为255
```
    #returning the image only where mask pixels are nonzero
    masked_image = cv2.bitwise_and(img, mask)
```
img和mask按位相与,因为mask兴趣区域为255,其它区域为0,所以可以将img的非兴趣区域变为0(黑色),兴趣区域不变。

* ## Step 4: Hough Transform to Find Lane Lines

<figure>
<img src="https://github.com/GitEasonXu/Udacity-CarND-Term1/blob/master/image/line.png" width="380" alt="Line Image" />
</figure> 

```
line_img = hough_lines(region_img, rho, theta, threshold, min_line_len, max_line_gap)
```
通过霍夫变换将直线连接起来

* ## Step 5: Combining the line and image

<figure>
<img src="https://github.com/GitEasonXu/Udacity-CarND-Term1/blob/master/image/result.png" width="380" alt="Result Image" />
</figure>

```
result_img = weighted_img(line_img, image)
```

## **2. Identify potential shortcomings with your current pipeline**

- 视频流进行处理时,有一些线检测错误;
- 兴趣区域需要进行设定,不能实现自适应;


## **3. Suggest possible improvements to your pipeline**

- 线检测错误还是由于`draw_lines`函数实现方法过于简单,应该可以使用中位数、平均值对异常值进行排除。可以按照小步幅间距的拟合直线。
