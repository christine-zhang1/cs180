# Images of the Russian Empire: Colorizing the Prokudin-Gorskii photo collection

## Project Overview

The goal of this project is to take the digitized Prokudin-Gorskii glass plate images and, using image processing techniques, automatically produce a color image with as few visual artifacts as possible. In order to do this, we extract the three color channel images, place them on top of each other, and align them so that they form a single RGB color image.

## Approach
First, we separate the color channels by computing 1/3 of the total height of the original image, and we splice the image into 3 parts of equal height. To align the channels, we align both the red and green channels to the blue channel.

The alignment procedure uses an image pyramid, which allows the algorithm to process large images. We use the sum of squared differences (SSD) as the metric to compare alignment between channels — a lower SSD means the channels are more aligned. The procedure is as follows:

* If either dimension of the image size is less than 256, then exhaustively search a 16x15 set of possible displacements for the best alignment. Specifically, search in a range of [-16, 15] for both the x and y dimensions. Return the displacement vector (i, j) that gives the best SSD alignment.
* Else, rescale the image to half the height and half the width, and run the procedure again. After getting the result from the previous recursive case, make appropriate adjustments:
    * Multiply the result by 2 to account for scaling up the image size
    * Search in a 2x2 set of possible displacements for the best alignment around the result*2 from the previous case.
    * Return the displacement vector (i, j) that gives the best SSD alignment.

Here are some images generated with this approach:

<div style="display: grid; grid-template-columns: repeat(3, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">
    <!-- Image 1 -->
    <div style="text-align: center;">
        <img src="images/output_base/cathedral.jpg" alt="cathedral.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">cathedral <br>
        green align (2, 5) <br>
        red align (3, 12) </p>
    </div>

    <!-- Image 2 -->
    <div style="text-align: center;">
        <img src="images/output_base/monastery.jpg" alt="monastery.jpg" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">monastery: <br>
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
</div>


<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

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

    <!-- Repeat for other images -->
</div>

## Bells and Whistles

### Structural Similarity
As we can see in `emir.tif` and `church.tif` above, SSD does not perform well in certain situations. We ran alignment using `skimage.metrics.structural_similarity` (SSIM) instead, and it produced better results. SSIM uses a sliding window approach to compare structural similarity between local parts of the image as opposed to just performing one comparison over all pixels in the image.

Here are corrected images for `emir.tif` and `church.tif` using structural similarity.

IMAGES GO HERE

### Automatic Border Cropping
We implemented automatic border cropping to eliminate some of the black and white edges as well as residual alignment artifacts on the borders of images. 

The idea behind our border cropping algorithm is that pixels across all RGB channels should be fairly similar if they are actually part of the image, i.e. each channel in the image should agree with the others to some reasonable extent. To implement this, we computed the total absolute difference between all RBG channels. Then, for each row and column, we checked whether 70% of the pixels had a total absolute difference less than 1. If yes, then we kept that row/column, and if not, then we cropped it out.

We also cropped out black and white borders on our images. For each row and column, we checked if the mean of the RGB values was both greater than 0.08 and less than 0.94, since pixels with mean values outside of this range were either black or white. 


<div style="display: grid; grid-template-columns: auto repeat(3, 1fr); grid-template-rows: auto repeat(5, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">
    <!-- Empty top-left cell -->
    <div></div>

    <!-- Column Labels -->
    <div style="font-weight: bold;">Col 1</div>
    <div style="font-weight: bold;">Col 2</div>
    <div style="font-weight: bold;">Col 3</div>

    <!-- Row 1 -->
    <div style="font-weight: bold;">Row 1</div>
    <div><img src="image1.jpg" alt="Image 1" style="width: 100%; height: auto; display: block; border: 2px solid red;"></div>
    <img src="image2.jpg" alt="Image 2" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image3.jpg" alt="Image 3" style="width: 100%; height: auto; display: block; border-radius: 5px;">

    <!-- Row 2 -->
    <div style="font-weight: bold;">Row 2</div>
    <div><img src="image4.jpg" alt="Image 4" style="width: 100%; height: auto; display: block; border: 2px solid red;"></div>
    <img src="image5.jpg" alt="Image 5" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image6.jpg" alt="Image 6" style="width: 100%; height: auto; display: block; border-radius: 5px;">

    <!-- Row 3 -->
    <div style="font-weight: bold;">Row 3</div>
    <div><img src="image4.jpg" alt="Image 4" style="width: 100%; height: auto; display: block; border: 2px solid red;"></div>
    <img src="image8.jpg" alt="Image 8" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image9.jpg" alt="Image 9" style="width: 100%; height: auto; display: block; border-radius: 5px;">

    <!-- Row 4 -->
    <div style="font-weight: bold;">Row 4</div>
    <div><img src="image4.jpg" alt="Image 4" style="width: 100%; height: auto; display: block; border: 2px solid red;"></div>
    <img src="image11.jpg" alt="Image 11" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image12.jpg" alt="Image 12" style="width: 100%; height: auto; display: block; border-radius: 5px;">

    <!-- Row 5 -->
    <div style="font-weight: bold;">Row 5</div>
    <div><img src="image4.jpg" alt="Image 4" style="width: 100%; height: auto; display: block; border: 2px solid red;"></div>
    <img src="image14.jpg" alt="Image 14" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image15.jpg" alt="Image 15" style="width: 100%; height: auto; display: block; border-radius: 5px;">
</div>
