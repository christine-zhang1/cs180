# Fun with Filters and Frequencies

## Part 1: Fun with Filters
### Part 1.1: Finite Difference Operator
We use the finite difference operators as defined below:
```
D_x = np.array([[1, -1]])
D_y = np.array([[1], [-1]])
```
Convolving these with the camera image using `scipy.signal.convolve2d` with `mode='same'` and `boundary='symm'`, we get the following images. For the binarized image, we used a threshold of 0.2.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/part1/dx_deriv_norm.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">Partial x derivative </p>
    </div>

    <div style="text-align: center;">
        <img src="images/part1/dy_deriv_norm.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">Partial y derivative </p>
    </div>

    <div style="text-align: center;">
        <img src="images/part1/grad_mag.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">Gradient magnitude </p>
    </div>

    <div style="text-align: center;">
        <img src="images/part1/grad_mag_binary_thresh02.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">Binarized gradient magnitude with threshold 0.2 </p>
    </div>
</div>

### Part 1.2: Derivative of Gaussian (DoG) Filter
We blur the image by convolving it with a 2D Gaussian filter of kernel size 7 and standard deviation 1. Then, we repeat the procedure from part 1.1 on the blurred cameraman image.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/part1/gaussian_grad_mag.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">Gradient magnitude </p>
    </div>

    <div style="text-align: center;">
        <img src="images/part1/gaussian_grad_mag_binary_thresh01.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">Binarized gradient magnitude with threshold 0.1 </p>
    </div>
</div>

Comparing this to the results in part 1.1, we see that these images are much less noisy, and they capture the information about the edges in the image more cleanly and clearly. The gradient magnitude images look much smoother after we blur the image initially. This is because the Gaussian filter is a low pass filter, so it removes the high frequency components of the image, eliminating noise and causing edge detection to be more accurate.

We check that we get the same results by convolving the gaussian with `D_x` and `D_y` first.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/part1/gaussian_filters.png" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>
</div>

We convolve these "derivative of Gaussian" filters with our original image (unblurred) to get the images below.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/part1/gaussian_grad_mag_dog.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">Gradient magnitude </p>
    </div>

    <div style="text-align: center;">
        <img src="images/part1/gaussian_grad_mag_binary_thresh01_dog.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">Binarized gradient magnitude with threshold 0.1 </p>
    </div>
</div>

The images are the same as the images we got after blurring the image and then applying `D_x` and `D_y`, so these two techniques have the same effect.

## Part 2: Fun with Frequencies
### Part 2.1: Image "Sharpening"

### Part 2.2: Hybrid Images

### Part 2.3: Gaussian and Laplacian Stacks

### Part 2.4: Multiresolution Blending

## Approach
First, we separate the color channels by computing 1/3 of the total height of the original image, and we splice the image into 3 parts of equal height. To align the channels, we align both the red and green channels to the blue channel.

The alignment procedure uses an image pyramid, which allows the algorithm to process large images. We use the sum of squared differences (SSD) as the metric to compare alignment between channels — a lower SSD means the channels are more aligned. The procedure is as follows:

* If either dimension of the image size is less than 256, then exhaustively search a 15x15 set of possible displacements for the best alignment. Specifically, search in a range of [-15, 15] for both the x and y dimensions. Return the displacement vector `(i, j)` that gives the best SSD alignment.
* Else, rescale the image to half the height and half the width, and run the procedure again. After getting the result from the previous recursive case, make appropriate adjustments:
    * Multiply the result by 2 to account for scaling up the image size.
    * Search in a 2x2 set of possible displacements for the best alignment around the result*2 from the previous recursive case.
    * Return the displacement vector `(i, j)` that gives the best SSD alignment.

When calculating the best alignment, we cut off 10% of the image height off from the top and bottom, and 10% of the image width off from the left and right. This helped prevent differences in the channels' borders from affecting the alignment. However, my output images are using the uncropped channels; the alignment displacement vector should be the same regardless.

We ran this procedure for aligning red to blue, and then for aligning green to blue. We then shifted the red and green channels by their respective best displacement vectors to align all 3 channels with each other.

Here are some images generated with this approach.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/output_base/cathedral.jpg" alt="cathedral.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">cathedral <br>
        green align (2, 5) <br>
        red align (3, 12) </p>
    </div>

    <!-- Image 2 -->
    <div style="text-align: center;">
        <img src="images/output_base/monastery.jpg" alt="monastery.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">monastery <br>
        green align (2, -3) <br>
        red align (2, 3) </p>
    </div>

    <!-- Image 3 -->
    <div style="text-align: center;">
        <img src="images/output_base/tobolsk.jpg" alt="tobolsk.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">tobolsk <br>
        green align (3, 3) <br>
        red align (3, 6) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/harvesters.jpg" alt="harvesters.tif" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">harvesters <br>
        green align (16, 59) <br>
        red align (13, 124) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/icon.jpg" alt="icon.tif" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">icon <br>
        green align (17, 41) <br>
        red align (23, 90) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/lady.jpg" alt="lady.tif" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">lady <br>
        green align (8, 56) <br>
        red align (11, 116) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/melons.jpg" alt="melons.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">melons <br>
        green align (11, 82) <br>
        red align (13, 178) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/onion_church.jpg" alt="onion_church.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">onion_church <br>
        green align (27, 51) <br>
        red align (36, 108) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/sculpture.jpg" alt="sculpture.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">sculpture <br>
        green align (-11, 33) <br>
        red align (-27, 140) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/self_portrait.jpg" alt="self_portrait.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">self_portrait <br>
        green align (29, 79) <br>
        red align (37, 176) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/three_generations.jpg" alt="three_generations.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">three_generations <br>
        green align (14, 53) <br>
        red align (11, 112) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/train.jpg" alt="train.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">train <br>
        green align (6, 43) <br>
        red align (32, 87) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/church_ssd.jpg" alt="church.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">church <br>
        green align (4, 25) <br>
        red align (267, 64) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/emir_ssd.jpg" alt="emir.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">emir <br>
        green align (24, 49) <br>
        red align (57, 103) </p>
    </div>
</div>

## Bells and Whistles

### Structural Similarity
As we can see in `emir.tif` and `church.tif` above, SSD does not perform well in certain situations. We ran the alignment procedure using `skimage.metrics.structural_similarity` (SSIM) instead, and it produced better results. SSIM uses a sliding window approach to compare structural similarity between local parts of the image as opposed to just performing one comparison over all pixels in the image.

Here are corrected images for `emir.tif` and `church.tif` using structural similarity.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/output_base/church.jpg" alt="church.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">church <br>
        green align (4, 25) <br>
        red align (-4, 58) </p>
    </div>

    <div style="text-align: center;">
        <img src="images/output_base/emir.jpg" alt="emir.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">emir <br>
        green align (23, 50) <br>
        red align (40, 105) </p>
    </div>
</div>

SSIM might be more suited for these images because it doesn't operate on the RGB color space. In these images, one channel might be dominating over the other channels (e.g. blue in Emir's jacket), which might be throwing off the SSD metric.

### Automatic Border Cropping
We implemented automatic border cropping to eliminate some of the black and white edges as well as residual alignment artifacts on the borders of images. 

The idea behind our border cropping algorithm is that pixels across all RGB channels should be fairly similar if they are actually part of the image, i.e. each channel in the image should agree with the others to some reasonable extent. To implement this, we computed the total absolute difference between all RBG channels. Then, for each row and column, we checked whether 70% of the pixels had a total absolute difference less than 1. If yes, then we kept that row/column, and if not, then we cropped it out.

We also cropped out black and white borders on our images. For each row and column, we checked if the mean of the RGB values was both greater than 0.08 and less than 0.94, since pixels with mean values outside of this range were either (sufficiently) black or white. If the mean of a row/column did not lie in this range, then we cropped out that row/column.

Here are a few original and cropped images. The images have red borders to show white edges that are cropped out by the automatic cropping algorithm.

<div style="display: grid; grid-template-columns: auto repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">
    <!-- Empty top-left cell -->
    <div></div>

    <!-- Column Labels -->
    <div style="font-weight: bold;">Original Aligned</div>
    <div style="font-weight: bold;">Cropped</div>

    <div style="font-weight: bold;">lady</div>
    <div><img src="images/output_base/lady.jpg" alt="Image 1" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/lady.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">

    <div style="font-weight: bold;">melons</div>
    <div><img src="images/output_base/melons.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/melons.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">

    <div style="font-weight: bold;">onion_church</div>
    <div><img src="images/output_base/onion_church.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/onion_church.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">

    <div style="font-weight: bold;">self_portrait</div>
    <div><img src="images/output_base/self_portrait.jpg" alt="Image 1" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/self_portrait.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
</div>

### Automatic Contrast
After cropping, we implemented automatic contrast. We used `skimage.exposure.equalize_adapthist`, which is an algorithm for local contrast enhancement that uses histograms computed over different tile regions of the image. This algorithm allows local details to be enhanced even in regions that are darker or lighter than most of the image.

Here are a few images shown before and after contrast.

<div style="display: grid; grid-template-columns: auto repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">
    <!-- Empty top-left cell -->
    <div></div>

    <!-- Column Labels -->
    <div style="font-weight: bold;">Before</div>
    <div style="font-weight: bold;">After</div>

    <div style="font-weight: bold;">cathedral</div>
    <img src="images/output_cropped/cathedral.jpg" alt="Image 2" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">
    <img src="images/output_contrasted/cathedral.jpg" alt="Image 3" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">harvesters</div>
    <img src="images/output_cropped/harvesters.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px">
    <img src="images/output_contrasted/harvesters.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">self_portrait</div>
    <div><img src="images/output_cropped/self_portrait.jpg" alt="Image 1" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;"></div>
    <img src="images/output_contrasted/self_portrait.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">
</div>

### Appendix

Complete results are shown below. The first column contains the original aligned images, the second column contains the cropped images, and the third column contains the cropped images after contrast. The images in the first two columns have red borders to show white edges that are cropped out by the automatic cropping algorithm.

<div style="display: grid; grid-template-columns: auto repeat(3, 1fr); grid-gap: 10px; padding: 20px; max-width: 1600px; margin: auto; align-items: center; justify-items: center;">
    <!-- Empty top-left cell -->
    <div></div>

    <!-- Column Labels -->
    <div style="font-weight: bold;">Original Aligned</div>
    <div style="font-weight: bold;">Cropped</div>
    <div style="font-weight: bold;">Contrasted</div>

    <div style="font-weight: bold;">cathedral</div>
    <div><img src="images/output_base/cathedral.jpg" alt="Image 1" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/cathedral.jpg" alt="Image 2" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/cathedral.jpg" alt="Image 3" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">church</div>
    <div><img src="images/output_base/church.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/church.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/church.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">emir</div>
    <div><img src="images/output_base/emir.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/emir.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/emir.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">harvesters</div>
    <div><img src="images/output_base/harvesters.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/harvesters.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/harvesters.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">icon</div>
    <div><img src="images/output_base/icon.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/icon.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/icon.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">lady</div>
    <div><img src="images/output_base/lady.jpg" alt="Image 1" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/lady.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/lady.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">melons</div>
    <div><img src="images/output_base/melons.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/melons.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/melons.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">monastery</div>
    <div><img src="images/output_base/monastery.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/monastery.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/monastery.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">onion_church</div>
    <div><img src="images/output_base/onion_church.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/onion_church.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/onion_church.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">sculpture</div>
    <div><img src="images/output_base/sculpture.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/sculpture.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/sculpture.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">self_portrait</div>
    <div><img src="images/output_base/self_portrait.jpg" alt="Image 1" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/self_portrait.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/self_portrait.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">three_generations</div>
    <div><img src="images/output_base/three_generations.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/three_generations.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/three_generations.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">tobolsk</div>
    <div><img src="images/output_base/tobolsk.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/tobolsk.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/tobolsk.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">

    <div style="font-weight: bold;">train</div>
    <div><img src="images/output_base/train.jpg" alt="Image 4" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;"></div>
    <img src="images/output_cropped/train.jpg" alt="Image 5" style="width: 100%; height: auto; max-height: 400px; display: block; border: 2px solid red;">
    <img src="images/output_contrasted/train.jpg" alt="Image 6" style="width: 100%; height: auto; max-height: 400px; display: block; border-radius: 5px;">
</div>
