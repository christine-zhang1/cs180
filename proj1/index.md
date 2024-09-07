# Images of the Russian Empire: Colorizing the Prokudin-Gorskii photo collection

## Project Overview

The goal of this project is to take the digitized Prokudin-Gorskii glass plate images and, using image processing techniques, automatically produce a color image with as few visual artifacts as possible. In order to do this, we extract the three color channel images, place them on top of each other, and align them so that they form a single RGB color image.

# Approach
First, we separate the color channels by computing 1/3 of the total height of the original image and splice the image into 3 parts of equal height. To align the channels, we align both the red and green channels to the blue channel.

The alignment procedure uses an image pyramid, which allows the algorithm to process large images. We use the sum of squared differences (SSD) as the metric to compare alignment between channels — a lower SSD means the channels are more aligned.

