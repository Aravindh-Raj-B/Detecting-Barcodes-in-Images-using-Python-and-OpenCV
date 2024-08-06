# Detecting-Barcodes-in-Images-using-Python-and-OpenCV
The goal of this repo is to demonstrate a basic implementation of barcode detection using computer vision and image processing techniques
![image](https://github.com/user-attachments/assets/15462009-6512-4057-89a2-62d5364088f9)


![image](https://github.com/user-attachments/assets/6c37e9b1-9c58-4851-86d2-20b55861df0b)


The first thing we’ll do is import the packages we’ll need. We’ll utilize NumPy for numeric processing, argparse
for parsing command line arguments, and cv2
for our OpenCV bindings.

Then we’ll setup our command line arguments. We need just a single switch here, --image
, which is the path to our image that contains a barcode that we want to detect.

Now, time for some actual image processing:
![image](https://github.com/user-attachments/assets/715c734a-b1c7-4279-bb98-4e370d8bf738)


On Lines 14 and 15 we load our image
off disk and convert it to grayscale.

Then, we use the Scharr operator (specified using ksize = -1
) to construct the gradient magnitude representation of the grayscale image in the horizontal and vertical directions on Lines 19-21.

From there, we subtract the y-gradient of the Scharr operator from the x-gradient of the Scharr operator on Lines 24 and 25. By performing this subtraction we are left with regions of the image that have high horizontal gradients and low vertical gradients.

Our gradient
representation of our original image above looks like:
![image](https://github.com/user-attachments/assets/791ec495-f9a7-448d-b0e3-8130641c95e1)


Notice how the barcoded region of the image has been detected by our gradient operations. The next steps will be to filter out the noise in the image and focus solely on the barcode region.

![image](https://github.com/user-attachments/assets/5d8e5e3d-4df7-4285-abf4-3f1d238d4769)


The first thing we’ll do is apply an average blur on Line 28 to the gradient image using a 9 x 9 kernel. This will help smooth out high frequency noise in the gradient representation of the image.

We’ll then threshold the blurred image on Line 29. Any pixel in the gradient image that is not greater than 225 is set to 0 (black). Otherwise, the pixel is set to 255 (white).

The output of the blurring and thresholding looks like this:

![image](https://github.com/user-attachments/assets/99447d76-8629-4cd4-ac3a-463ced701f72)

However, as you can see in the threshold image above, there are gaps between the vertical bars of the barcode. In order to close these gaps and make it easier for our algorithm to detect the “blob”-like region of the barcode, we’ll need to perform some basic morphological operations:
![image](https://github.com/user-attachments/assets/a47bf2c0-50bf-471d-99ec-7ff25f8e21c0)


We’ll start by constructing a rectangular kernel using the cv2.getStructuringElement
on Line 32. This kernel has a width that is larger than the height, thus allowing us to close the gaps between vertical stripes of the barcode.

We then perform our morphological operation on Line 33 by applying our kernel to our thresholded image, thus attempting to close the the gaps between the bars.

You can now see that the gaps are substantially more closed, as compared to the thresholded image above:

![image](https://github.com/user-attachments/assets/9a076b85-2cc7-4fe9-83ac-6eb9e83922d3)

Of course, now we have small blobs in the image that are not part of the actual barcode, but may interfere with our contour detection.

Let’s go ahead and try to remove these small blobs:
![image](https://github.com/user-attachments/assets/05f40cf1-966a-4927-9131-9703eb6d320b)


All we are doing here is performing 4 iterations of erosions, followed by 4 iterations of dilations. An erosion will “erode” the white pixels in the image, thus removing the small blobs, whereas a dilation will “dilate” the remaining white pixels and grow the white regions back out.

Provided that the small blobs were removed during the erosion, they will not reappear during the dilation.

After our series of erosions and dilations you can see that the small blobs have been successfully removed and we are left with the barcode region:

![image](https://github.com/user-attachments/assets/2195a303-1fd1-45c6-b77c-9c202ae904bb)

Finally, let’s find the contours of the barcoded region of the image:

![image](https://github.com/user-attachments/assets/4e02fbe5-95cd-44d8-8fc1-9efa88cf7585)


Luckily, this is the easy part. On Lines 41-44 we simply find the largest contour in the image, which if we have done our image processing steps correctly, should correspond to the barcoded region.

We then determine the minimum bounding box for the largest contour on Lines 47-49 and finally display the detected barcode on Lines 53-55.

As you can see in the following image, we have successfully detected the barcode:
Figure 6: Successfully detecting the barcode in our example image.

![image](https://github.com/user-attachments/assets/7f1dc7e3-e3de-4d1f-a6b9-c66b3a795a89)




