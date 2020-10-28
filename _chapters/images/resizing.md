---
title: Resizing
keywords: (insert comma-separated keywords here)
order: 11 # Lecture number for 2020
---

In today's world, we use devices of various shapes and sizes to consume information and display images and videos. When we go from a laptop to a mobile phone, we see that the image size is significantly smaller and its pixel information is changed. This topic introduces the techniques used for resizing images while accounting for important content and limiting artifacts that may be caused by rescaling.

- [Image Retargeting](#image-retargeting-overview)
- [Seam Carving Algorithm](#seam-carving-algorithm)
	- [Overview](#overview)
	- [Implementation using Dynamic Programming](#implementation)
	- [Results](#results)
- [Seam carving - Extensions](#topic3)
	- [Improving running time](#subtopic-3-1)
	- [Extension to both dimensions](#subtopic-3-2)
	- [Image expansion](#subtopic-3-3)
	- [Multi-size image resizing](#subtopic-3-4)
	- [Object removal](#subtopic-3-5)
	- [Improvement: Forward energy](#subtopic-3-6)
	- [Video seam carving](#subtopic-3-7)

[//]: # (This is how you can make a comment that won't appear in the web page! It might be visible on some machines/browsers so use this only for development.)

[//]: # (Notice in the table of contents that [First Big Topic] matches #first-big-topic, except for all lowercase and spaces are replaced with dashes. This is important so that the table of contents links properly to the sections)

[//]: # (Leave this line here, but you can replace the name field with anything! It's used in the HTML structure of the page but isn't visible to users)

<a name='image-retargeting-overview'></a>
## Image Retargeting and Resizing Overview
### Problem Statement
Input an image of size $n \times m$ and return an image of new size $n' \times m'$ which will be a good representative of original image. 
Upon retargeting, the expectations are:
1. Output image adheres to device geometric constraints
2. Output image preserves important content and structures
3. Output image limits artifacts

### Importance Measures
1. A function, $S : p \rightarrow [0, 1]$, measures whether a pixel is important to an image.
2. More sophisticated techniques to measure which parts of an image are important to human perception. These include attention models, eye tracking (gaze studies), and face detectors.

### General Retargeting framework
Step 1: Define an energy function $E(I)$. This could be perceived as the interest, importance or saliency.

Step 2: Use one or more operators to change the image $I$.

<div class="fig figcenter fighighlight">
    <img src="{{ site.baseurl }}/assets/images/overview-framework-example.png">
  <div class="figcaption">In this example. we have chosen to retarget using cropping</div>
</div>


<a name='seam-carving-algorithm'></a>
## Seam Carving Algorithm
<a name='overview'></a>
### Overview

**Motivation**

We have already seen how methods such as rescaling or cropping images without paying attention to the content of the image often result in suboptimal results, such as introducing artifacts and losing important content and geometric structures in the image. In this section, we will introduce the Seam carving algorithm, a powerful algorithm that will perform content-aware resizing of images to different sizes. Seam carving will help us prevent a lot of the issues that result from other content-unaware retargetting or resizing methods. The Seam carving algorithm performs content-aware resizing of images by selectively - and intelligently - adding or removing pixels from the image, while paying careful attention to the content in the image. 

**Key Ideas Behind Seam Carving**

Before diving into the algorithm, let us first discuss some key ideas behind the seam-carving algorithm. 

For more clarity, we will be using an example image to guide us and help us understand the key ideas behind the seam-carving algorithm. 

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/cCSCQ4Y.png">
  <div class="figcaption">Example Image with gradient-based energy image</div>
</div>

Take for instance a situation in which we have to **summarize**, or reduce, an image of size $m \times n$ to a new smaller size $m \times n'$ where $n' < n$. In this case, our Seam carving algorithm will need to remove $m \times (n - n')$ pixels from the image. But which pixels should we remove?

**Key Idea 1: We want to remove the pixels with the least *energy*.**

The first key idea is that we will want to remove pixels from the image which are the least important to the content of the image. Intuitively, this makes sense because if we have to remove pixels from our image, we should be removing the least important pixels rather than other more important ones. This way, we will be retaining as much information in the image as possible, and avoid loss of content during our image summarization. But how do we decide which pixels are least important? We will define the least important pixels as the pixels which have the least **energy**. The **energy** of a pixel measures how much that pixel stands out from its surroundings, and thus how important the pixel is.

We can calculate the energy of a pixel through using energy functions. One such possible energy function is a gradient-based energy function, where we measure each pixel's energy as the sum of its horizontal and vertical gradients:

$$E(I) = |\frac{\partial}{\partial x} I| + |\frac{\partial}{\partial y} I|$$ 

Although other energy functions are also possible, this is a simple gradient-based energy function, which happens to work very well. To understand, why this gradient-based energy function works so well, it is important to note that pixels with lower energy as determined by this energy function will have smaller gradients. Pixels with smaller gradients will often exist in the smooth areas of the image. Since human perception is really sensitive to edges and contours in images, pixels forming edges in the image are much more important in providing geometric constraints and information in the image than pixels from smoother areas. Therefore, removing pixels from smooth areas in the image, which have lower gradients, instead of pixels from the edges, which have higher gradients, will help preserve a lot of the important content in the image. Another positive of using this gradient-based energy function is that it is quite easy to calculate, while also providing very useful results!

**Key Idea 2: We want to remove pixels by seams.**

Now that we have a guideline on what type of pixels to remove from the image, pixels with lower energy, the question stands, which $m \times (n - n')$ pixels from the original image should we remove? We will introduce several possible ways of choosing pixels to remove from the image, and highlight the flaw with each of these methods, before proceeding into a discussion of the Seam carving algorithm.

**Optimal Removal**

One possible intuitive approach might be to remove the $m \times (n - n')$ pixels in the image with the lowest energy globally across the whole image. In this case, we will simply rank all pixels in the image in order from least to most energy based on our gradient-based energy function, and remove the 
$m \times (n - n')$ pixels with the lowest energy in the image regardless of the positions of these pixels. 

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/1tNxKNY.png">
  <div class="figcaption">Result of removing lowest energy pixels from image</div>
</div>

However, since we removed pixels without paying attention to the location their, we end up losing a lot of the geometric structure in the image. We could have removed a lot more pixels from one row or column of the image than from another row or column. As a result, our image loses a lot of its geometric constraints and becomes extremely imbalanced.

**Least-Energy Pixels Per Row Removal**

We know now that we need to also consider the positions of the pixels in the image that we are removing. One large issue with optimal removal was that the summarized image might have different amount of pixels in each row and column. To fix this, what if we tried removing the same amount of pixels from each row of the image? Since we have $m$ rows, then we could try removing the $n - n'$ pixels with the lowest energy from each row of the image. In total, we would be removing $m \times (n - n')$ pixels from the entire image, and arrive at a new summarized $m \times n'$ image.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/vUXEEBA.png">
  <div class="figcaption">Result of removing $n - n'$ pixels from each of $m$ rows of original image</div>
</div>

However, if we remove pixels from each row of the image independently, without paying attention to which pixels we removed in other rows, then we lose a lot of the vertical structure in the image. We see that columns in our new summarized image no longer make much sense. The pixels in our image begin to cave in on themselves. Pixels which shared the same column in our original image may now no longer share the same column in our new summarized image. Further, pixels from entirely different columns in our original image may now share the same column in our new summarized image. For example, in the new image, a pixel in row 1, column 5 of the original image may now be directly above a pixel in row 2, column 1 of the original image. As a result, we lose a lot of geometric structure in our image.


**Least-Energy Columns**

What if we tried removing entire columns all at once then in order to avoid this issue? By removing entire columns, we can ensure that if we remove a pixel, then the entire column is removed, and we will no longer have this issue of pixels folding in on themselves. Pixels belonging to the same column of our original image will still share the same column in our new image. We see then that by using this method we will be able to retain a lot of the vertical structure, and will help us retain more geometric consistency and constraints in our new summarized image. To do so, we can calculate the total energy of all pixels in each column. Then, since we have $m$ rows, we can remove the $n - n'$ columns with the lowest total energy of all pixels in its column. In total, we remove $m \times (n- n')$ pixels from our original image, arriving at a new $m \times n'$ summarized image.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/yqWEZ0b.png">
  <div class="figcaption">Result of removing $n - n'$ least-total-energy columns from original image</div>
</div>

This sounds like a great idea at first and we can see that this is a clear improvement from our previous methods of removing pixels. However, upon further inspection, we can see that this solution is not entirely perfect either. It introduces a lot of artifacts. For example, applying this method of removing least energy columns causes some geometries in the original image to lose its shape. For example, let's inspect the grey diagonal platform stretching across the bottom-side of our example image. The diagonal loses its shape using this method since entire columns making up the diagonal were removed without paying much attention to the diagonal's geometry. 

So far, noticeably, we have tried methods ranging from unconstrained (optimal) to more constrained (least-energy columns). However, none of these methods produce appealing results. They all have their faults. This is where the Seam carving algorithm comes in. It introduces a very important idea and method on how to remove pixels, that is to remove pixels by **seams**.

**What is a seam?**

A **seam** is defined as a connected path of pixels that stretch across the image, either from the bottom to the top of the image or from the left to the right of the image. For vertical seams that stretch from the bottom of the top of the image, there will be exactly one pixel in each row. For horizontal seams that stretch from the left to the right of the image, there will be exactly one pixel in each column. 

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/FwVjHOy.png">
  <div class="figcaption">Example of a vertical seam stretching from bottom to top of an image</div>
</div>

In mathematical notation, a vertical seam, a seam which upon removal will reduce the width of an image, can be written as:

$$s^x = \{s_i^x\}_{i=1}^m = \{(x(i), i)\}^m_{i=1}, s.t. \forall i, |x(i)-x(i-1)| \leq 1$$

Here a vertical seam is represented as a set of $m$ pixels, $\{s_i^x\}_{i=1}^m$, where $m$ is the number of rows in the image. Note that since our image has $m$ rows and our vertical seam is connected from the top to the bottom of the image, then we need $m$ pixels in our vertical seam in total, exactly one pixel for each row of the image. 

Further, each pixel $s_i^x$ in the seam can be written as its coordinate $(x(i), i)$. $x(i)$ represents the pixel's column, or x-coordinate, while $i$ represents the pixel's row, or y-coordinate. As we can see, the y-coordinate goes from $1$ to $m$ indicating that we have exactly one pixel for each row of the image. 

Note that we also have an additional constraint on the x-coordinates of the pixels in our seam, that is given by: $\forall i, \|x(i)-x(i-1)\| \leq 1$. This can be interpreted as for every $i$, or row, the x-coordinate of the pixel in the previous row $i - 1$ and the x-coordinate of the pixel in the current row $i$ must come from neighboring columns, such that the absolute difference in x-coordinate, or column, between these two pixels must be less than or equal to 1. This further ensures that our seam remains connected. 

Similarly, the mathematical definition of a horizontal seam, a seam which can be used to reduce the height of an image, can be written as: 

$$s^y = \{s_i^y\}_{i=1}^n = \{(j, y(j)\}^n_{i=1}, s.t. \forall j, |y(j)-y(j-1)| \leq 1$$

Note that here $n$ represents the number of columns in our image; we will have exactly one pixel in our horizontal seam per column in our image. We can very easily use very similar logic as above to parse this mathematical statement, and this will be left as an exercise for the reader. 

**How do we find the right seam to remove?**

Now that we know what a seam is, note that there are a lot of possible seams that traverse across an image. The number of vertical seams or horizontal seams existing in an image is much larger than the number of columns or rows, respectively, that exist in an image. How then can we find the right seam to remove?

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/392Pcxv.png">
  <div class="figcaption">Depiction of a few out of numerous seams that can exist in an image</div>
</div>

To find the right seam to remove, we can rely on ideas that we have already developed in previous sections. As a reminder, in our key idea 1 discussion, we have already highlighted that we want to remove pixels with the lowest energy from our image, because these pixels are less important to our image than pixels with higher energy. Further, we can borrow the idea of removing the lowest cumulative energy column to now instead remove the lowest cumulative energy vertical seam from our image. With that, we will remove the seam which minimizes our energy function. In mathematical notation, we will be removing vertical seam $s^{*}$, where:

$$s^* = \arg\min_{s} E(s)$$

In this case, our energy function $E$, will be defined as the gradient-based energy function, we discussed in previous sections:

$$E(I) = |\frac{\partial}{\partial x} I| + |\frac{\partial}{\partial y} I|$$ 


<a name='Implementation'></a>
### Implementation Using Dynamic Programming

When our goal is to reduce the width of an image from $n$ to $n'$, we should be removing the vertical seams from our image which minimize the energy function, that is we should remove seams with the lowest cumulative energy across all pixels in the seam. We can find the seam which minimizes our energy function through use of dynamic programming. 

To define this issue in terms of dynamic programming, our goal is to find the vertical seam of minimum cost, where the cost is defined by our energy function. For dynamic programming, we need to set up a recurrence relation. To generalize our recurrence, we need to establish a way to calculate $M(i,j)$, the minimal cost of a seam that passes through some arbitrary pixel $(i,j)$. To clarify, $M(i,j)$ will be the lowest possible cost of a seam leading up to pixel $(i,j)$, consisting of the costs, or energy, of pixel $(i,j)$ and the above pixels belonging to this seam. 

Let us first assume that we have already calculated, or have a way of calculating, the energy of each individual pixel in our image, including the energy $E(i,j)$ of the arbitrary pixel $(i, j)$. A matrix representation of the energy of each individual pixel in our image is shown below.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/qfMwABj.png">
  <div class="figcaption">Energy Matrix containing energy of each individual pixel in our image</div>
</div>

Let us then assume that we have already calculated the minimal cost $M$ of the seam that passes through every pixel above, or up to, pixel $(i,j)$, such as pixels $(0,0)$, .. $(i-1,j-1)$, $(i-1,j)$, $(i-1,j+1)$, ... $(i,j-1)$.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/b4T3egj.png">
  <div class="figcaption">Dynamic Programming Matrix containing energy of each individual pixel up to $(i,j)$</div>
</div>

With these assumptions, let us now set up a recurrence to find $M(i,j)$, again the minimal cost of the seam that passes through the pixel $(i,j)$. To find $M(i,j)$, note that $M(i,j)$ can be found by adding the cost of the energy of pixel $(i,j)$, or $E(i,j)$, and the cost of the best (lowest cost) seam so far leading up to $(i,j)$.

*Sidenote: To understand why $M(i,j)$ is the sum of $E(i,j)$ and the cost of the best seam so far leading up to $(i,j)$, note the following. Since $(i,j)$ is in the seam, it must contribute to $M(i,j)$. Since we must have arrived at $(i,j)$ from a seam leading up to it, the cost of the seam above it leading up to (i,j) must contribute to $M(i,j)$.*

Let us now find these different components in order to add them together to get $M(i,j)$. Luckily, we already have the value of $E(i,j)$ as pictured in the purple energy matrix above. 

To find the cost of the seam leading up to (i,j) with minimal cost, note that to reach pixel $(i,j)$, the seam must have arrived at $(i,j)$ by traversing through one of three possible pixels from the row directly above it, $(i-1,j-1)$, $(i-1,j)$, or $(i-1, j+1)$. This is pictured in the matrix below. 

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/9nNLQ0u.png">
  <div class="figcaption">To reach $(i,j)$, the seam could have only come from one of three possible pixels.</div>
</div>

Since we are trying to minimize $M(i,j)$, then we should then take the cost of the seam (of the three possible) with the lowest cost leading up to pixel $(i,j)$. Therefore, we should take the minimum cost out of $M(i - 1, j -1)$, $M(i - 1, j)$, and $M(i - 1, j + 1)$ and add this to $E(i,j)$ to find $M(i,j)$. 

In mathematical notation, this **recursion relation** can be written as:

$$M(i,j) = E(i,j) + \min\left\{M(i-1,j-1), M(i-1,j), M(i-1,j+1)\right\}$$

We now have the invariant property, $M(i,j)$, the minimal cost of a seam that goes through pixel $(i,j)$ and the recurrence relation above.

With the invariant property and the recurrence relation now in hand, dynamic programming can now be used to solve this problem with a runtime of $O(n \cdot m)$, where $n$ is the number of columns in our image, and $m$ is the number of rows in our image.


Let's look at one example! Below we have our dynamic programming matrix $M$ (left) and our Energy Matrix $E$ (right). Let the current $i$ and $j$ we are looking at be $(1, 1)$. Following the recurrence relation defined above, we find $E(1, 1)$, which is $2$, and substitute that into $M(1, 1)$. We also find $M(i - 1, j -1)$, $M(i - 1, j)$, and $M(i - 1, j + 1)$, which are the values $5$, $8$, and $12$, respectively. 

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/KP7qqEr.png">
</div>

The minimum of those three values is $5$, so we get $M(1, 1) = 2 + 5 = 7$.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/M4xhS0f.png">
</div>

Moving onto $M(1, 2)$, we again find the value at the current $i$ and $j$ in $E$, getting $3$, and add the minimum of $8, 12, 3$ to get $M(1, 2) = 3 + 3 = 6$.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/y0vq5Lw.png">
</div>

We can continue to populate $M$ in this fashion until we reach the last index.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/K6Cuk9O.png">
</div>

We can then backtrack to find the seam path, advancing one row at a time and looking at the minimum value on the row above the current row minimum until we reach the top of the image. This can be accomplished by storing choices along the path, but not necessarily so.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/QpIXVm1.png">
  <img src="https://i.imgur.com/1JnpsEi.png">
  <img src="https://i.imgur.com/DsBHJzX.png">
  <img src="https://i.imgur.com/UhADQTG.png">
</div>

This backtracking process allows us to find the best seam to remove. Let's take a look at an example of an input image! Below, we compute both the horizontal (left) and vertical cost (right) of the image. Both of these approaches can be used to select the best seam to remove.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/yPbwdeO.png">
</div>

In summary, we can construct the following algorithm. Given the input image $im$ and the number of columns we want our output image to have, $n'$, we assume that $im$ is of size $m$ rows and $n$ columns. 

In step 1, we have a loop that repeats $n-n'$ times. This is because we have $n$ columns in the input, and we want to remove vertical seams until we get the output of $n'$ columns.

In the loop, we have to compute the Energy Matrix $E$ of the image, find the optimal seam $s$ in $E$, and then remove $s$ from $im$. At the conclusion of the loop, we return our image $im$. We can pseudocode the algorithm as follows:

```python
seam_carving(img,n_new): // size(im) = mxn
    Do (n - n_new) times:
        E := compute energy map on im
	s := find optimal seam in E
	im := remove s from im
    return im
```

Since the running time of each step in the loop ($1.1$, $1.2$, $1.3$) is $O(mn)$, the overall running time would be $O(dmn)$, where $d = (n - n')$.

Lastly, below are some examples of seam carving, scaling, cropping, and retargeting in action. As you look through the images, take some time to compare the differences between resizing by seam-carving and resizing by other methods. We can see that in many cases seam-carving produces a much better output than scaling and cropping. We lose less information and introduce less artifacts.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/hAl7jrI.jpg">
  <img src="https://i.imgur.com/EofaFzH.jpg">
  <img src="https://i.imgur.com/a2VMa2l.jpg">
</div>

<a name ='topic3'></a>
## Seam carving - Extensions

<a name='subtopic-3-1'></a>
### Improving running time
While we've seen that seam removal can be very effective, it is also very computationally intensive. We can improve the running time of the Seam carving algorithm by accounting for locality of operations. The key observation here is to note that when we remove a seam from the image, most of the energy map $E$ remains unchanged. As such, we only need to recompute the values of $E$ in the neighborhood along the seam that was removed. This can be visualized through the figure below:

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam_runtime.png">
  <div class="figcaption">After removing the seam pixels marked in red, we only recompute $E$ for the neighbourhood pixels marked in yellow. The values of $E$ remain unchanged for most of the image pixels, marked in white.</div>
</div>

Note that for an image of size $m \times n$, where we want to resize $n$ to $n'$, the new run time complexity of the optimized Seam carving algorithm becomes $O((n-n')m)$, since each update to $E$ now takes $O(m)$ time.  Note that this is an order of magnitude faster than the original run time complexity of $O((n-n')mn)$.

<a name='subtopic-3-2'></a>
### Extension to both dimensions
Consider the task of resizing an input image from size $m \times n$ to size $m' \times n'$, where $m > m'$ and $n > n'$. This leads to the question: What is the correct order of seam carving? Should we remove vertical seams first? Horizontal seams first? Or alternate between the two? The optimal ordering of horizontal and ordering seam removal is not very clear. However, we can use Dynamic Programming once again to define the search for an optimal order through the recursion relation:

$$ \mathbf{T}(r,c) = \min \begin{Bmatrix} \mathbf{T}(r-1,c) + E(s^y(I_{m-r+1 \times n-c})), \\ \mathbf{T}(r,c-1) + E(s^x(I_{m-r \times n-c+1})) \end{Bmatrix} $$

where $I_{m-r \times n-c}$ denotes an image of size $m-r \times n-c$, $E(s^y(I))$ is the cost of removing a horizontal seam and $E(s^x(I))$ is the cost of removing a vertical seam. We find the optimal order using a transport map $\mathbf{T}$ that specifies, for each desired target image size $m' \times n'$, the cost of the optimal sequence of horizontal and vertical seam removals. In other words, the entry $\mathbf{T}(r,c)$ holds the minimum cost needed to obtain an image of size $m-r \times n-c$. We compute $\mathbf{T}$ using dynamic programming. Starting at $\mathbf{T}(0,0) = 0$, we fill each entry $(r,c)$ choosing the best of two options:
- Removing a horizontal seam $s^y$ from an image of size $m-r+1 \times n-c$ where we have already removed $r-1$ rows and $c$ columns.
- Removing a vertical seam $s^x$ from an image of size $m-r \times n-c+1$ where we have already removes $r$ rows and $c-1$ columns.

We store which of the two options was picked at each step of the dynamic programming algorithm in a $m \times n$ 1-bit map. For a target image size of $m' \times n'$ where $m' = m - r$ and $n' = n - c$, we backtrack from $\mathbf{T}(r,c)$ to $\mathbf{T}(0,0)$ and apply the seam removal operation at each step corresponding to the optimal order.

For more clarity, let us now consider a simple example to better understand the recursion relation. Consider the task of resizing a $4 \times 4$ input image to size $2 \times 3$. In this case, we want to remove $r=2$ rows and $c=1$ columns. We can express the recursion relation for this simple case as follows:

$$ \mathbf{T}(2,1) = \min \begin{Bmatrix} \mathbf{T}(1,1) + E(s^y(I_{3 \times 3})), \\ \mathbf{T}(2,0) + E(s^x(I_{2 \times 4})) \end{Bmatrix} $$

Since the output image is of size $2 \times 3$, we could have arrived at this point through one of two options:
- By removing a horizontal seam from a $3 \times 3$ image, indicated as $\mathbf{T}(1,1) + E(s^y(I_{3 \times 3}))$
- By removing a vertical seam from a $2 \times 4$ image, indicated as $\mathbf{T}(2,0) + E(s^x(I_{2 \times 4}))$

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam_both_dimensions_a.png">
</div>

The recursion relation selects the lowest energy between the two possible options. Continuing, let us inspect the recursion relation for $\mathbf{T}(1,1)$, which can be written as follows:

$$ \mathbf{T}(1,1) = \min \begin{Bmatrix} \mathbf{T}(0,1) + E(s^y(I_{4 \times 3})), \\ \mathbf{T}(1,0) + E(s^x(I_{3 \times 4})) \end{Bmatrix} $$

Intuitively, since the image is of size $3 \times 3$, we could have arrived at this point through one of two options:
- By removing a horizontal seam from a $4 \times 3$ image, indicated as $\mathbf{T}(0,1) + E(s^y(I_{4 \times 3}))$
- By removing a vertical seam from a $3 \times 4$ image, indicated as $\mathbf{T}(1,0) + E(s^x(I_{3 \times 4}))$

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam_both_dimensions_b.png">
</div>

The recursion relation selects the lowest energy between the two possible options. Moreover, note that our recursion reaches the base case from both $\mathbf{T}(0,1)$ and $\mathbf{T}(1,0)$. This is because we arrive at $\mathbf{T}(0,1)$ by removing a vertical seam from the original image, and $\mathbf{T}(1,0)$ by removing a horizontal seam from the original image. Therefore, we have that

$$\mathbf{T}(0,1) = \mathbf{T}(0,0) + E(s^x(I_{4 \times 4})) = E(s^x(I_{4 \times 4}))$$
$$\mathbf{T}(1,0) = \mathbf{T}(0,0) + E(s^y(I_{4 \times 4})) = E(s^y(I_{4 \times 4}))$$
since for the base case $\mathbf{T}(0,0) = 0$.

Similarly, we can traverse the recursion tree for $\mathbf{T}(2,0)$ as shown in below:
<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam_both_dimensions_c.png">
  <div class="figcaption">In this recursion tree, we are considering the energy of removing 2 horizontal seams from the original $4 \times 4$ input image, leading to a $2 \times 4$ image.</div>
</div>

Putting it all together, the full recursion tree for our simple example is shown below. The dynamic programming algorithm selects the minimum energy between two options at each stage of our recursion:
- Removing a horizontal seam $s^y$ from an image of size $m-r+1 \times n-c$ where we have already removed $r-1$ rows and $c$ columns.
- Removing a vertical seam $s^x$ from an image of size $m-r \times n-c+1$ where we have already removes $r$ rows and $c-1$ columns.

We track the choice we make at each step of recursion. After the entire recursion tree has been explored, we propogate our choices backwards to find the optimal ordering of seam removal.

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam_both_dimensions_all.png">
  <div class="figcaption">The full recursion tree will select the lowest energy path from the input $4 \times 4$ image to the desired resized $2 \times 3$ image.</div>
</div>

<a name='subtopic-3-3'></a>
### Image expansion
Seams carving can also be used to increase the size of images. By expanding the least important areas of the image, as indicated by the seams, we can increase the dimensions of the image without distorting the main content. 

#### Pitfall
If we use a naive approach of iteratively computing the lowest energy seam and adding it to our input image, we will run into the pitfall of selecting the same seam at each iteration. As such, when expanding an image's width by $k$ pixels, the lowest energy seam in the image will be duplicated repeatedly $k$ times, producing undesirable results as depicted below:

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam-pitfall.png">
  <div class="figcaption">In the naive approach, the same seam will be inserted repeatedly. This is is because the lowest energy seam remains the same after it is replicated in the image.</div>
</div>

Note that we have many columns containing repeated pixels in the image above. This effect happens because the lowest energy seam is always the same after we replicate that seam again in the image.

#### Solution
A solution to this pitfall is to calculate as many seams as we need to insert into the image, so that we only duplicate each seam once. To expand an image by $k$ pixels, we compute the $k$ seams with the lowest energy at once, and duplicate each of them. Doing so gives us much more desirable results as depicted below:

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam-expansion.png">
  <div class="figcaption">Computing the $k$ seams with the lowest energy at once and duplicating them yields much more pleasing results. Note that with this method, an image cannot be expanded by more than a factor of $2$, since each seam is only duplicated once.</div>
</div>

This method produces a far higher quality expanded image compared to the naive approach. Furthermore, compared to scaling, our content-aware resizing produces much more pleasing results.

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam-scaling.png">
  <div class="figcaption">The content-aware resizing preserves the important areas of the image while replicating unimportant, low-energy areas. By contrast, simple scaling distorts the entire image.</div>
</div>

#### Combined Insert and Remove
Another way of using this algorithm is combining insertion and removal of seams. This is very desirable when we want to reduce one dimension while expanding the other. For example, given the input image on the left, we can insert vertical seams to make the image wider, and remove horizontal seams to make the image shorter in height. Our content-aware resizing once again produces far more pleasing results than simple scaling.

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/insert-and-remove.png">
  <div class="figcaption">Using content-aware resizing preserves the main areas of the input image while duplicating, removing the less important low-energy areas. This produces much more appealing results than simple scaling which introduces heavy distortion throughout, and whose affect is amplified in the case of uneven scaling of width and height.</div>
</div>

<a name='subtopic-3-4'></a>
### Multi-size image resizing
So far, all of our images have primarily operated in the case where we know the output size of the final image. However, what happens if we don't have this information readily available to us? In this case, we can create a new representation of these images that enable us to adapt any given image to different sizes.

First, we must compute all the seams in an image. Next, we can store these precomputed vertical and horizontal seams alongside the image content in order to later map to any changes that we need to be made. This way we can convenientl add and remove seams when the image needs to be resized.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/XMkkkFi.jpg">
</div>

<a name='subtopic-3-5'></a>
### Object removal
The use of seams can also be applied for purposes of object removal. In the below example, we may want to remove the green object from the image while leaving the red object intact.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/lF4B4Z5.jpg">
</div>

In order to accomplish this, what we can consider doing is by manipulating the energy component of the previous seams equation. As described before, our seam equation is: 

$$M(i,j) = E(i,j) + \min\left\{M(i-1,j-1), M(i-1,j), M(i-1,j+1)\right\}$$

We can accomplish the goal of object removal by assigning a very low energy value to the piels in green while assigning a very high energy value $E$ to the pixels in red. The SEAM carving will always create paths going through the green pixels, as a result, while avoiding paths with the red pixels, thus accomplishing the task of object removal. 

<a name='subtopic-3-5.5'></a>
### Limitations

One potential problem of this approach is that sometimes we cannot retain the image shape. See the image below for an example: 

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/EFb1vU7.png">
</div>

One way we can look to resolve this limitation is using a similar approach as before where we manually adjust the energy value at certain pixels. By making this energy value large, we can communicate to the algorithm the importance of retaining this image's shape.

<a name='subtopic-3-6'></a>
### Improvement: Forward energy

Next, we will discuss possible improvements to the existing algorithm. Currently, our method of approach focuses on staying away from pixels with high energy value while exploring pixels with low energy values. An example of this can be seen in the image below. 

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/lmiMLrt.jpg">
</div>

Here, as an example, the "10%" image highlights the top 10% of pixels with highest energy values. As low energy pixels are removed from the image, we can also observe the average energy increasing in the image from the remaining pixels. However, one observation is that as we remove a seam from an image, we are also creating new edges which can thereby also increase the average pixel value. See the image below as an example. 

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/DzsinY4.jpg">
</div>

As a result, one thing we could consider is, rather than removing the lowest-energy pixels, we could focus on removing pixels that cause the smallest energy to be added to the image overall. This is because we want to avoid the unappealing above image on the right. 


<a name='subtopic-3-5.5'></a>
### Approach
We can start off this procedure by looking at each of the ways in which we can remove the seams. The image below illustrates that we can remove the left seam, the middle seam, and the right seam as valid options. 

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/UCJJa1G.png" width="400">
</div>

Let us first consider the case of removing the left seam.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/fkgeEZl.png">
</div>

Once we remove the seam in red, we can calculate the new amount of energy introduced in the image as a result of removing the red pixels. We can perform a similar process for the right seam: 

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/iVavhcG.png">
</div>

For the middle seam, the calculation looks as follows:

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/rSscXsY.png" width="500">
</div>

Here, it can be noted that we only take one gradient (the two botom pixels) since we don't want to double-count. This can lead to the following new algorithm:

$$ M(i,j) = \min \cases{M(i-1,j-1) + C_L(i,j) \\ M(i-1,j) + C_U(i,j) \\ M(i-1,j+1) + C_R(i,j)}$$

We can see the improvements this new, forward-looking approach has in comparison to our previously introduced backwards-looking approach.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/51NGf0c.png" width="300">
</div>

<a name='subtopic-3-7'></a>
### Video seam carving

Lastly, we’ll discuss how to extend the seam carving algorithm to video. 

#### Challenges

This is a substantially more difficult problem than image processing, for two reasons:

1. Cardinality: There is a much greater amount of pixels in video than in image. As a helpful comparison, consider a one-minute video that has a frame rate of 30 fps (frames per second); there are 1800 frames in this video. If our algorithm is able to process one image per minute, processing this video would take 30 hours!
2. Dimensionality/algorithmic: Humans are highly sensitive to movement and motion, thus when working on video processing, we have to prioritize keeping motion smooth.

#### Naive Approach

Let’s first consider a naive approach to seam-carving video, which seam-carves frame by frame independently. This means that the algorithm applies seam-carving to a frame independent from the following frames.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/YiAjzsv.png">
  <div class="figcaption">Watch 0:00-0:36 of [this video](https://www.youtube.com/watch?v=AJtE8afwJEg) to check out the results of this process. </div>
</div>

#### Improved Approach

Notice that the result using the naive approach is very jittery. Let's improve this by moving from 2D to 3D. To do so, consider images as 2D matrices and videos as 3D volumes of image frames. Rather than computing 1D paths in images, we will compute 2D manifolds to traverse video cubes.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/dSOdbCU.png">
</div>

Watch 1:11-2:13 of [the video](https://www.youtube.com/watch?v=AJtE8afwJEg) to check out an example of video retargeting and a high-level walkthrough of the 3D approach. Specifically, we replace the 2D dynamic programming algorithm with a 3D graph cut approach. We define a grid-like graph, where the nodes are pixels, and we add a source and target virtual nodes on both sides of the cube.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/Tm0fzbP.png" width="500">
  <img src="https://i.imgur.com/S2HHEoH.png" width="500">
  <div class="figcaption">A cut in the graph defines a surface seam, and the intersection of this 2D surface with each frame defines the seam path on that frame. As such, we can guarantee that the seams are monotonic and connected.</div>
</div>

#### Object Detection and Seam Carving

Seam carving can also be combined with object detection. In the example below, we have a face detector which specifies the areas that we do not want to be removed by the seam-carving resizing algorithm. Our goal is to make the energy in those pixels very high such that the seam-carving will never prefer to go through those pixels.

<div class="fig figcenter fighighlight">
  <img src="https://i.imgur.com/PyAlrEH.png">
</div>


To learn more about object detection with seam carving, feel free to take a look at these relevant resources.

1. Seam Carving for Content-Aware Image Resizing – Avidan and Shamir 2007
2. Content-driven Video Retargeting – Wolf et al. 2007
3. Improved Seam Carving for Video Retargeting – Rubinstein et al. 2008
4. Optimized Scale-and-Stretch for Image Resizing – Wang et al. 2008
5. Summarizing Visual Data Using Bidirectional Similarity – Simakov et al. 2008
6. Multi-operator Media Retargeting – Rubinstein et al. 2009
7. Shift-Map Image Editing – Pritch et al. 2009
8. Energy-Based Image Deformation – Karni et al. 2009
9. Seam carving in Photoshop CS4: https://helpx.adobe.com/photoshop/using/content-aware-scaling.html

