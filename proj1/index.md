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

# Image Grid Example

Here is a 3x5 image grid with labeled rows and columns:

<div style="display: grid; grid-template-columns: auto repeat(3, 1fr); grid-template-rows: auto repeat(5, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">
    <!-- Empty top-left cell -->
    <div></div>

    <!-- Column Labels -->
    <div style="font-weight: bold;">Col 1</div>
    <div style="font-weight: bold;">Col 2</div>
    <div style="font-weight: bold;">Col 3</div>

    <!-- Row 1 -->
    <div style="font-weight: bold;">Row 1</div>
    <img src="image1.jpg" alt="Image 1" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image2.jpg" alt="Image 2" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image3.jpg" alt="Image 3" style="width: 100%; height: auto; display: block; border-radius: 5px;">

    <!-- Row 2 -->
    <div style="font-weight: bold;">Row 2</div>
    <img src="image4.jpg" alt="Image 4" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image5.jpg" alt="Image 5" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image6.jpg" alt="Image 6" style="width: 100%; height: auto; display: block; border-radius: 5px;">

    <!-- Row 3 -->
    <div style="font-weight: bold;">Row 3</div>
    <img src="image7.jpg" alt="Image 7" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image8.jpg" alt="Image 8" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image9.jpg" alt="Image 9" style="width: 100%; height: auto; display: block; border-radius: 5px;">

    <!-- Row 4 -->
    <div style="font-weight: bold;">Row 4</div>
    <img src="image10.jpg" alt="Image 10" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image11.jpg" alt="Image 11" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image12.jpg" alt="Image 12" style="width: 100%; height: auto; display: block; border-radius: 5px;">

    <!-- Row 5 -->
    <div style="font-weight: bold;">Row 5</div>
    <img src="image13.jpg" alt="Image 13" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image14.jpg" alt="Image 14" style="width: 100%; height: auto; display: block; border-radius: 5px;">
    <img src="image15.jpg" alt="Image 15" style="width: 100%; height: auto; display: block; border-radius: 5px;">
</div>
