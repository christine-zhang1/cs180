# Image Warping and Mosaicing

## Shoot Pictures
For the mosaics, I took pictures in Souvenir Coffee on College Avenue, in my living room, and in front of the physics building. I shot these with my iPhone 11 Pro.

Souvenir Coffee:
<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/img1_half.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"> </p>
    </div>

    <div style="text-align: center;">
        <img src="images/img2_half.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"> </p>
    </div>
</div>

My living room:
<div style="display: grid; grid-template-columns: repeat(3, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/room1.png" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"> </p>
    </div>

    <div style="text-align: center;">
        <img src="images/room2.png" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>
    <div style="text-align: center;">
        <img src="images/room3.png" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>
</div>

My kitchen:
<div style="display: grid; grid-template-columns: repeat(3, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/kitchen1.png" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"> </p>
    </div>

    <div style="text-align: center;">
        <img src="images/kitchen2.png" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>
    <div style="text-align: center;">
        <img src="images/kitchen3.png" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>
</div>

## Recover Homographies
I defined correspondences in pairs of images. Let's call the points in the source image $(x_i, y_i)$ and the corresponding points in the target image $(x_i', y_i')$. Here are the correspondence points for the photos taken in Souvenir Coffee.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/im1_pts.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>

    <div style="text-align: center;">
        <img src="images/im2_pts.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>
</div>

We want to recover the 3x3 transformation matrix in this equation relating $(x, y)$ to $(x', y')$.

$$
\begin{pmatrix}
a & b & c \\
d & e & f \\
g & h & 1
\end{pmatrix}
\begin{pmatrix}
x \\
y \\
1
\end{pmatrix}
=
\begin{pmatrix}
wx' \\
wy' \\
w
\end{pmatrix}
$$

Expanding the matrix multiplication into a system of equations, we get
$$
ax + by + c = wx' \\
dx + ey + f = wy' \\
gx + hy + 1 = w
$$

Grouping the terms with $x'$ and $y'$, we get
$$
ax + by + c = (gx + hy + 1)x' \\
dx + ey + f = (gx + hy + 1)y'
$$

We can then isolate $x'$ and $y'$:
$$
ax + by + c - gx'x - hy'x' = x' \\
dx + ey + f - gx'y' - hy'y' = y'
$$

This gives the matrix equation:
$$
\begin{pmatrix}
x & y & 1 & 0 & 0 & 0 & -xx' & -yx' \\
0 & 0 & 0 & x & y & 1 & -xy' & -yy'
\end{pmatrix}
\begin{pmatrix}
a \\
b \\
c \\
d \\
e \\
f \\
g \\
h
\end{pmatrix}
=
\begin{pmatrix}
x' \\
y'
\end{pmatrix}
$$

We can create this matrix equation for all of the $(x_i, y_i)$ and corresponding $(x_i', y_i')$. We can then stack all of these equations together (in matrix form) to form an overconstrained system of equations that we can approximately solve using least squares. We get a length 8 vector as the solution, and we can then stack a 1 at the bottom and then reshape this into a 3x3 matrix. We call this matrix our homography H.

## Warp Images
I computed the homography matrix H according to the previous part, and I warped my source image towards the target image using the homography. This transforms the source image's points such that they align with the corresponding points in the target image. For this part, I dynamically calculated the size of the bounding box of the new image based on the image warp.

## Rectify Images
With the image warping function, we can "rectify" images that have a known rectangle in them, even if the rectangle is not directly facing the camera in the initial photo. We can choose four corners of the rectangle in the image, and we can then compute a homography between that rectangle's points and an actual rectangle with points that we define ourselves (e.g. `[(200, 200), (300, 200), (300, 400), (200, 400)]`). This warps the image so that the rectangle in the image is now facing the camera.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="font-weight: bold;">Original image with rectification points</div>
    <div style="font-weight: bold;">Warped image</div>

    <div style="text-align: center;">
        <img src="images/desk_pts.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>

    <div style="text-align: center;">
        <img src="images/desk_warped.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>

    <div style="text-align: center;">
        <img src="images/fridge_pts.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>

    <div style="text-align: center;">
        <img src="images/fridge_warped.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>
</div>

## Blend Images into a Mosaic
I created a caricature of my face by calculating extrapolated correspondence points and warping my face to the triangulation formed by those points. I computed `p + 1.4(q - p)` for the extrapolated correspondence points, where `p` denotes the correspondence points on my face and `q` denotes the correspondence points on the mean face of the FEI dataset.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/cz_warp_to_avg_neutral_extra_1_4.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">My caricature using mean neutral face</p>
    </div>

    <div style="text-align: center;">
        <img src="images/cz_warp_to_avg_smiling_extra_1_4.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">My caricature using mean smiling face</p>
    </div>
</div>

The top left of my face at the hairline has some artifacts. This is likely because the point labeling on the FEI dataset does not have any points on the forehead or hairline, so it is not warped as cleanly as the rest of the face.

## Bells and Whistles: Principal Component Analysis
I took each `m x n` image and represented it as a vector of length `m * n` (for the FEI dataset, this was `300 * 250`). The entire dataset can then be represented as a `k x (m * n)` matrix, where `k` is the number of images in the dataset. I performed PCA on this matrix using `sklearn.decomposition.PCA` to obtain the eigenfaces for the FEI dataset.

Here are the eigenfaces in order of decreasing eigenvalue for the neutral and smiling faces.

<div style="padding: 20px; max-width: 2400px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/eigenfaces_neutral.png" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">Neutral eigenfaces</p>
    </div>
</div>

<div style="padding: 20px; max-width: 2400px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/eigenfaces_smiling.png" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;">Smiling eigenfaces</p>
    </div>
</div>

I used the eigenface basis to generate some new face images. I multiplied each of the first 32 eigenfaces by a random weight in the range `(-1, 1)` and normalized the result to obtain a new face.

Here are some newly generated neutral faces.

<div style="display: grid; grid-template-columns: repeat(3, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/pca_rand_face_neutral2.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>

    <div style="text-align: center;">
        <img src="images/pca_rand_face_neutral4.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>

    <div style="text-align: center;">
        <img src="images/pca_rand_face_neutral5.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>
</div>

Some of the results were pretty discolored due to the random weights that were chosen. Here are some randomly generated neutral faces that don't look as good.

<div style="display: grid; grid-template-columns: repeat(3, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/pca_rand_face_neutral3.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>

    <div style="text-align: center;">
        <img src="images/pca_rand_face_neutral6.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>

    <div style="text-align: center;">
        <img src="images/pca_rand_face_neutral7.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>
</div>

Here are some newly generated smiling faces.

<div style="display: grid; grid-template-columns: repeat(3, 1fr); grid-gap: 10px; padding: 20px; max-width: 1200px; margin: auto; align-items: center; justify-items: center;">

    <div style="text-align: center;">
        <img src="images/pca_rand_face_smiling2.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>

    <div style="text-align: center;">
        <img src="images/pca_rand_face_smiling4.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>

    <div style="text-align: center;">
        <img src="images/pca_rand_face_smiling6.jpg" alt="img" style="width: 100%; height: auto; display: block;">
        <p style="margin-top: 5px; font-size: 14px; font-weight: bold; color: #333;"></p>
    </div>
</div>

In these images, we can see that the mouth area is a bit blurry. I think this is because the smiling faces have more variation in where the mouth is compared to the neutral faces, so the first 32 components of the PCA basis are unable to capture this information as accurately for the smiling faces. Otherwise, the images look fairly realistic.
