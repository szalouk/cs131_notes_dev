---
title: Resizing
keywords: (insert comma-separated keywords here)
order: 11 # Lecture number for 2020
---

You can start your notes here before you start diving into specific topics under each heading. This is a useful place to define the topic of the day and lay out the structure of your lecture notes. You'll edit the Markdown file, in this case template.md, which will automatically convert to HTML and serve this web page you're reading! 

The table of contents can link to each section so long as you match the names right (see comments in template.md for more elaboration on this!). This Markdown to HTML mapping doesn't like periods in the section titles and won't link them from the table of contents, so use dashes instead if you need to.

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
<a name='Topic 1'></a>

## First Big Topic
	
Here you can start to talk about the first topic of your notes. You can bold text like **this**, or italicize text like *this*. If you want to make a numbered list it's as easy as
1.  
2. 
3. 

- Bullet
- points
- are
- similar 

For a more detailed cheatsheet on the most important functionality of Markdown, check out this link https://wordpress.com/support/markdown-quick-reference/, which you can format in Markdown with your own [link title](https://wordpress.com/support/markdown-quick-reference/)


<a name='Subtopic 1-1'></a>
### Subtopic 1-1
You might want to include images in your notes, since Computer Vision as a field is blessed with tons of cool visualizations. Here's an example from the CS 231N notes page we included as a reference for you:

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/examples/classify.png">
  <div class="figcaption">Put your informative caption here! If you really want to mess around with the classes in this div container then feel free, but inserting images just like this should work great!</div>
</div>

<a name='Subtopic 1-2'></a>
### Subtopic 1-2
Sometimes you might want to insert some code snippets into your notes. As an example, here's a snippet of python code taken from the CS 231N notes:
```python
Xtr, Ytr, Xte, Yte = load_CIFAR10('data/cifar10/') # a magic function we provide
# flatten out all images to be one-dimensional
Xtr_rows = Xtr.reshape(Xtr.shape[0], 32 * 32 * 3) # Xtr_rows becomes 50000 x 3072
Xte_rows = Xte.reshape(Xte.shape[0], 32 * 32 * 3) # Xte_rows becomes 10000 x 3072
```

<a name='Subtopic 1-3'></a>
### Subtopic 1-3
Sometimes you might want to write some mathematical equations, and LaTeX is a great tool for that! You can write an inline equation like this \\( a^2 = b^2 \\), or you can display an equation on its own line like this! \\[ a^2 = b^2 + c^2 \\]

You can also apply LaTeX syntax to label your equations and refer to them later! Here's the equation:

$$ \begin{equation} \label{your_label} a^2 = b^2 + c^2 + d^2 + e^2 \end{equation} $$

and here's a linked reference to it: \eqref{your_label}. For now, this configuration likes the \\"\\$\\$ equation stuff ... \\$\\$\\" syntax to have an empty line above and below it, but it displays the same anyway.

**For a guide on LaTeX syntax and how to write mathematical equations and formulas with it, check out [this link](https://www.overleaf.com/learn/latex/mathematical_expressions)** 

**Here's a short guide on how to use the basics of LaTeX**
- You've seen above the syntax to start and end an equation, so now let's work on what you fill in the middle
- You can make variables and expressions **bold** in equations too: \\(\mathbf{x} + y\\)
- Superscripts and subscripts are easy: use the ^ and _ symbol and bound your super/sub script by {} if it's more than one character. For example: \\(e^{-x+10}\\)
- Greek letters are also simple, use the \ character with their written name with optional capitalization, such as alpha or Alpha. For example: \\(\alpha + \beta + \gamma + \delta + \Gamma + \Delta\\). Not all capital greek letters work like this, but you can search online for solutions if this trick fails or reach out to the CA's. In general the \ character in LaTeX is the gateway to all kinds of special characters and functionalities.
- Sums and Products are really useful in Latex. You can use both superscripts and subscripts to mark the bounds: \\(\log(\prod_{i=0}^{2n}i^2) = \sum_{i=0}^{2n}\log (i^2)\\)
- Another useful trick is to write out a matrix or a vector in LaTex. There's a lot of customization you can do with this, so check out this [page](https://www.overleaf.com/learn/latex/Matrices) for more details. Here's some examples in our Markdown environment: 


$$\begin{bmatrix}
1 & 2 & 3\\
a & b & c
\end{bmatrix}$$

$$\begin{bmatrix}
1\\
2\\
3\\
\end{bmatrix}$$

$$\begin{bmatrix}
1 & 2 & 3\\
\end{bmatrix}$$

As with the labelled equations, it makes a difference whether the lines above and below the equation are blank, so keep that in mind while debugging! 

<a name='image-retargeting-overview'></a>
## Image Retargeting and Resizing Overview
In today's world, we use devices of various shapes and sizes to consume information and display images and videos. When we go from a laptop to a mobile phone, we see that the image size is significantly smaller and its pixel information is changed. This topic introduces the techniques used for resizing images while accounting for important content and limiting artifacts that may be caused by rescaling.

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
We have already seen how methods such as rescaling or cropping images without paying attention to the content of the image often results in suboptimal results, such as introducing artifacts, removing content, violating geometric constraints, or losing important content and geometric structures in the image. In this section, we will introduce the seam-carving algorithm, a powerful algorithm that will perform content-aware resizing of images to different sizes. Seam-carving will help us prevent a lot of the issues that result from other content-unaware retargetting or resizing methods. The seam-carving algorithm performs content-aware resizing of images by selectively and intelligently adding or removing pixels from the image, all while paying careful attention to the content in the image. 

**Key Ideas Behind Seam Carving**
Before diving into the algorithm, let us first discuss some key ideas behind the seam-carving algorithm. 

For more clarity, we will be using an example image to guide us and help us understand the key ideas behind the seam-carving algorithm. 

![](https://i.imgur.com/cCSCQ4Y.png)
*Example Image with gradient-based energy image*
*(Courtesy of CS131 Lecture 11 Slides)*

Take for instance a situation in which we have to **summarize**, or reduce, an image of size m x n to a new smaller size m x n' where n' < n. In this case, our seam-carving algorithm will need to remove m x (n - n') pixels from the image. But which pixels should we remove?


<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/examples/challenges.jpeg">
  <div class="figcaption"></div>
</div>
Image of size $m x n$

**Key Idea 1: We want to remove the pixels with the least *energy*.**
The first key idea is that we will want to remove pixels from the image which are the least important to the content of the image. Intuitively, this makes sense bceause if we have to remove pixels from our image, we should be removing the least important pixels rather than other more important pixels. This way, we will be retaining as much information in the image as possible, and avoid loss of content during our image summarization. But how do we decide which pixels are least important? We will define the least important pixels as the pixels which have the least **energy**. The **energy** of a pixel measures how much that pixel stands out from its surroundings, and thus how important the pixel is.

We can calculate the energy of a pixel through using energy functions. One such possible energy function, as discussed in class, is a gradient-based energy function, where we measure each pixel's energy as the sum of its horizontal and vertical gradients:

$$E(I) = |\frac{\partial}{\partial x} I| + |\frac{\partial}{\partial y} I|$$ 

Although other energy functions are also possible, this is a simple gradient-based energy function, which happens to work very well. To understand, why this gradient-based energy function works so well, it is important to note that pixels with lower energy as determined by this energy function will have smaller gradients. Pixels with smaller gradients will often exist in the smooth areas of the image. Since human perception is really sensitive to edges and contours in images, pixels forming edges in the image are much more important in providing geometric constraints and information in the image than pixels from smoother areas. Therefore, removing pixels from smooth areas in the image, which have lower gradients, instead of pixels from the edges, which have higher gradients, will help preserve a lot of the important content in the image. Another positive to using this gradient-based energy function is because it is quite easy to calculate, while also providing very useful results!

**Key Idea 2: We want to remove pixels by seams.**
Now that we have a guideline on what type of pixels to remove from the image, pixels with lower energy, the question stands, which $m x (n - n')$ pixels from the original image should we remove? We will introduce several possible ways of choosing pixels to remove from the image, and highlight the flaw of each of these methods, before proceeding into a discussion of the seam-carving algorithm.

**Optimal Removal**
One possible intuition might be to remove the $m x (n - n')$ pixels in the image with the lowest energy globally across the whole image. In this case, we will simply rank all pixels in the image in order from least to most energy based on our gradient-based energy function, and remove the 
$m x (n - n')$ pixels with the lowest energy in the image regardless of the positions of these pixels. 

![](https://i.imgur.com/1tNxKNY.png)
*Result of removing pixels from image with optimal removal*
*(Courtesy of CS131 Lecture 11 Slides)*

However, since we removed pixels without paying attention to the location of the pixels, we end up losing a lot of the geometric structure in the image. We could have removed a lot more pixels from one row or column of the image than from another row or column. As a result, our image loses a lot of its geometric constraints and becomes extremely imbalanced.


**Least-Energy Pixels Per Row Removal**
We know now that we need to also consider the positions of the pixels in the image that we are removing. One large issue with optimal removal was that the summarized image might have different amount of pixels in each row and column. To fix this, what if we tried removing the same amount of pixels from each row of the image? Since we have $m$ rows, then we could try removing the $n - n'$ pixels with the lowest energy from each row of the image. In total, we would be removing $m x (n - n')$ pixels from the entire image, and arrive at a new summarized $m x n'$ image.

![](https://i.imgur.com/vUXEEBA.png)
*Result of removing $n - n'$ pixels from each of $m$ rows of original image*
*(Courtesy of CS131 Lecture 11 Slides)*

However, if we remove pixels from each row of the image independent of other rows, without paying attention to which pixels we removed in other rows, then we lose a lot of the vertical structure in the image. We see that columns in our new summarized image no longer make much sense. The pixels in our image begin to cave in on themselves. Pixels which shared the same column in our original image may now no longer share the same column in our new summarized image. Further, pixels from entirely different columns in our original image may now share the same column in our new summarized image. For example, in the new image, a pixel in row 1, column 5 of the original image may now be directly above a pixel in row 2, column 1 of the original image. As a result, we lose a lot of geometric structure in our image.


**Least-Energy Columns**
What if we tried removing entire columns all at once then in order to avoid this issue? By removing entire columns, we can ensure that if we remove a pixel, then the entire column is removed, and we will no longer have this issue of pixels folding in on themselves. Pixels belonging to the same column of our original image will still share the same column in our new image. We see then that by using this method we will be able to retain a lot of the vertical structure, and will help us retain more geometric consistency and constraints in our new summarized image. To do so, we can calculate the total energy of all pixels in each column. Then, since we have $m$ rows, we can remove the $n - n'$ columns with the lowest total energy of all pixels in its column. In total, we remove $m x (n- n')$ pixels from our original image, arriving at a new $m x n'$ summarized image.

![](https://i.imgur.com/yqWEZ0b.png)
*Result of removing $n - n'$ least-total-energy columns from original image*
*(Courtesy of CS131 Lecture 11 Slides)*

This sounds like a great idea at first and we can see that this is a clear improvement from our previous methods of removing pixels. However, upon further inspection, we can see that this solution is not entirely perfect either. It introduces a lot of artifacts. For example, applying this method of removing least energy columns causes some geometries in the original image to lose its shape. For example, let's inspect the grey diagonal platform stretching across our example image. The diagonal loses its shape using this method since entire columns making up the diagonal were removed without paying much attention to the diagonal's geometry. 

So far, noticeably, we have tried methods, ranging from unconstrained (optimal) to more constrained (least-energy columns). However, none of these methods produce appealing results. They all have their faults. This is where the seam carving algorithm comes in. It introduces a very important idea and method on how to remove pixels, that is to remove pixels by **seams**.

**What is a seam?**

A **seam** is defined as a connected path of pixels that stretch across the image, either from the bottom to the top of the image or from the left to the right of the image. For vertical seams that stretch from the bottom of the top of the image, only one pixel can be in each row. For horizontal seams that stretch from the left to the right of the image, only one pixel can be in each column. 

![](https://i.imgur.com/FwVjHOy.png)
Example of a vertical seam stretching from bottom to top of an image

In mathematical notation, a vertical seam, a seam which can be used to reduce the width of an image, can be written as:

$$s^x = \{s_i^x\}_{i=1}^m = \{(x(i), i)\}^m_{i=1}, s.t. \forall i, |x(i)-x(i-1)| \leq 1$$

Here a vertical seam is represented as a set of $m$ pixels, $\{s_i^x\}_{i=1}^m$, where $m$ is the number of rows in the image. Note that since our image has $m$ rows and our vertical seam is connected from the top to the bottom of the image, then we need $m$ pixels in our vertical seam in total, exactly one pixel for each row of the image. 

Further, each pixel $s_i^x$ in the seam can be written as its coordinate $(x(i), i)$. $x(i)$ represents the pixel's column, or x-coordinate, while $i$ represents the pixel's row, or y-coordinate. As we can see, the y-coordinate goes from $1$ to $m$ indicating that we have exactly one pixel for each row of the image. 

Note that we also have an additional constraint on the x-coordinates of the pixels in our seam, that is given by: $\forall i, \|x(i)-x(i-1)\| \leq 1$. This can be interpreted as for every $i$, or row, the x-coordinate of the pixel in the previous row $i - 1$ and the x-coordinate of the pixel in the current row $i$ must come from neighboring columns, such that the absolute difference in x-coordinate, or column, between these two pixels must be less than or equal to 1. This further ensures that our seam remains connected. 

Similarly, the mathematical definition of a horizontal seam, a seam which can be used to reduce the height of an image, can be written as: 

$$s^y = \{s_i^y\}_{i=1}^n = \{(j, y(j)\}^n_{i=1}, s.t. \forall j, |y(j)-y(j-1)| \leq 1$$

Note that here $n$ represents the number of columns in our image; we will have exactly one pixel in our horizontal seam per column in our image. We can very easily use very similar logic as above to parse this mathematical statement, and this will be left as an exercise for the reader. 

**How do we find the right seam to remove?**
Now that we know what a seam is, note that there are a lot of possible seams that traverse across an image. The number of vertical seams or horizontal seams existing in an image is much larger than the number of columns or rows, respectively, that exist in an image. How then can we find the right seam to remove?

![](https://i.imgur.com/392Pcxv.png)
Depiction of a few out of numerous seams that can exist in an image

To find the right seam to remove, we can rely on ideas that we have already developed in previous sections. As a reminder, in our key idea 1 discussion, we have already highlighted that we want to remove pixels with the lowest energy from our image, because these pixels are less important to our image than pixels with higher energy. Further, we can borrow the idea of removing the lowest cumulative energy column to now instead remove the lowest cumulative energy vertical seam from our image. With that, we will remove the seam which minimizes our energy function. In mathematical notation, we will be removing vertical seam $s^{*}$, where:

$$s^* = \arg\min_{s} E(s)$$

In this case, our energy function $E$, will be defined as the gradient-based energy function, we discussed in previous sections:

$$E(I) = |\frac{\partial}{\partial x} I| + |\frac{\partial}{\partial y} I|$$ 


<a name='Implementation'></a>
### Implementation Using Dynamic Programming

When our goal is to reduce the width of an image from $n$ to $n'$, we should be removing the vertical seam from our image which minimizes the energy function, that is the seam with the lowest cumulative energy across all pixels in the seam. We can find the seam which minimizes our energy function through using dynamic programming. 

To define this issue in terms of dynamic programming, our goal is to find the vertical seam of minimum cost, where the cost is defined by our energy function. For dynamic programming, we need to set up a recurrence. To generalize our recurrence, we need to establish a way to calculate $M(i,j)$, the minimal cost of a seam that passes through some arbitrary pixel $(i,j)$. For clarity, here $M(i,j)$ will be the lowest possible cost of a seam leading up to pixel $(i,j)$, consisting of the costs, or energy, of $(i,j)$ and pixels above which belong to this seam. 

Let us first assume that we have calculated, or have a way of calculating, the energy of each individual pixel in our image, including the energy $E(i,j)$ of the arbitrary pixel $(i, j)$. A matrix representation of the energy of each individual pixel in our image is shown below.

![](https://i.imgur.com/qfMwABj.png)
*Energy Matrix containing energy of each individual pixel in our image*

Now we have $E(i,j)$, the energy of the pixel $(i,j)$, we need to find $M(i,j)$, the minimal cost of the seam that passes through $(i,j)$.

Let us also assume that we have already calculated the minimal cost of the seam that passes through every pixel up until pixel $(i,j)$, that is we have already calculated the minimal cost of the seam that passes through pixels above $(i,j)$, such as for pixels $(0,0)$, .. $(i-1,j-1)$, $(i-1,j)$, $(i-1,j+1)$, ... $(i,j-1)$.

![](https://i.imgur.com/b4T3egj.png)
*Dynamic Programming Matrix containing energy of each individual pixel up to $(i,j)$*

Let us now set up a recurrence to find $M(i,j)$, again the minimal cost of the seam that passes through the pixel $(i,j)$. To find $M(i,j)$, note that $M(i,j)$ can be found by adding the cost of the energy of pixel $(i,j)$ and the cost of the best (lowest cost) seam so far leading up to $(i,j)$.

*Sidenote: To understand why $M(i,j)$ is the sum of $E(i,j)$ and the cost of the best seam so far leading up to $(i,j)$, note the following. Since $(i,j)$ is in the seam, it must contribute to $M(i,j)$. Since we must have arrived at $(i,j)$ from a seam leading up to it, the cost of a seam above it leading up to (i,j) must contribute to $M(i,j)$.*

Let us now find these different components in order to add them together to get $M(i,j)$. Luckily, we already have the value of $E(i,j)$ as pictured in the purple energy matrix above. 

Let us now find the cost of the best (lowest cost) seam so far leading up to $(i,j)$. To do so note, that to reach pixel $(i,j)$, the seam must have arrived at $(i,j)$ by traversing through one of three possible pixels from the row directly above it, $(i-1,j-1)$, $(i-1,j)$, or $(i-1, j+1)$. This is pictured in the matrix below. 

![](https://i.imgur.com/9nNLQ0u.png)
*To reach (i,j), the seam could have only come from one of three possible pixels.*

Since we are trying to minimize $M(i,j)$, then we should then take the cost of the seam (of the three possible) with the lowest cost leading up to pixel $(i,j)$. The (minimal) costs of the seams containing pixels $(i-1,j-1)$, $(i-1,j)$, and $(i-1, j+1)$ are $M(i - 1, j -1)$, $M(i - 1, j)$, and $M(i - 1, j + 1)$, respectively. Therefore, we should take the minimum cost out of $M(i - 1, j -1)$, $M(i - 1, j)$, and $M(i - 1, j + 1)$ and add this to $E(i,j)$ to find $M(i,j)$. 

In mathematical notation, this **recursion relation** can be written as:

$$M(i,j) = E(i,j) + \min\left\{M(i-1,j-1), M(i-1,j), M(i-1,j+1)\right\}$$

We now have the invariant property, $M(i,j)$, the minimal cost of a seam that goes through pixel $(i,j)$ and the recurrence relation above.

With the invariant property and the recurrence relation now in hand, dynamic programming can now be used to solve this problem with a runtime of $O(n \cdot m)$, where $n$ is the number of columns in our image, and $m$ is the number of rows in our image.


Let's look at one example!**[INSERT LECTURE 11.2, SLIDES 12 AND BEYOND]**


<a name ='topic3'></a>
## Seam carving - Extensions

<a name='subtopic-3-1'></a>
### Improving running time
While we've seen that seam removal can be very effective, it is also very computationally intensive. We can improve the running time of the Seam carving algorithm by accounting for locality of operations. The key observation here is to note that when we remove a seam from the image, most of the energy map $E$ remains unchanged. As such, we only need to recompute the values of $E$ in the neighborhood along the seam that was removed. This can be visualized through the figure below:

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam_runtime.png">
  <div class="figcaption">After removing the seam pixels marked in red, we only recompute $E$ for the neighbourhood pixels marked in yellow. The values of $E$ remain unchanged for most of the image pixels, marked in white.</div>
</div>

Note that for an image of size $m \times n$, where we want to resize $n$ to $n'$, the new run time complexity of the optimized Seam carving algorithm becomes $O((n-n')m)$, since the each update to $E$ now takes $O(m)$ time.  Note that this is an order of magnitude faster than the original run time complexity of $O((n-n')mn)$.

<a name='subtopic-3-2'></a>
### Extension to both dimensions
Consider the task of resizing an input image from size $m \times n$ to size $m' \times n'$, where $m > m'$ and $n > n'$. This leads to the question: What is the correct order of seam carving? Should we remove vertical seams first? Horizontal seams first? Or alternate between the two? The optimal ordering of horizontal and ordering seam removal is not very clear. However, we can use Dynamic Programming once again to define the search for an optimal order through the recursion relation:

$$ \mathbf{T}(r,c) = \min \begin{Bmatrix} \mathbf{T}(r-1,c) + E(s^y(I_{m-r+1 \times n-c})), \\ \mathbf{T}(r,c-1) + E(s^x(I_{m-r \times n-c+1})) \end{Bmatrix} $$

where $I_{m-r \times n-c}$ denotes an image of size $m-r \times n-c$, $E(s^y(I))$ is the cost of removing a horizontal seam and $E(s^x(I))$ is the cost of removing a vertical seam. We find the optimal order using a transport map $\mathbf{T}$ that specifies, for each desired target image size $m' \times n'$, the cost of the optimal sequence of horizontal and vertical seam removals. In other words, the entry $\mathbf{T}(r,c)$ holds the minimum cost needed to obtain an image of size $m-r \times n-c$. We compute $\mathbf{T}$ using dynamic programming. Starting at $\mathbf{T}(0,0) = 0$, we fill each entry $(r,c)$ choosing the best of two options:
- Removing a horizontal seam $s^y$ from an image of size $m-r+1 \times n-c$ where we have already removed $r-1$ rows and $c$ columns.
- Removing a vertical seam $s^x$ from an image of size $m-r \times n-c+1$ where we have already removes $r$ rows and $c-1$ columns.

We store which of the two options was picked at each step of the dynamic programming algorithm in a $m \times n$ 1-bit map. For a target image size of $m' \times n'$ where $m' = m - r$ and $n' = n - c$, we backtrack from $\mathbf{T}(r,c)$ to $\mathbf{T}(0,0)$ and apply the seam removal operation at each step corresponding to the optimal order.


<a name='subtopic-3-3'></a>
### Image expansion
Seams carving can also be used to increase the size of images. By expanding the least import areas of the image, as indicated by the seams, we can increase the dimensions of the image without distorting the main content. 

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
Another way of using this algorithm is combining insertion and removal of seams. This is very desirable to resize dimensions of an image different. For example, given the input image on the left, we can insert vertical seams to make the image wider, and remove horizontal seams to make the image shorter in height. Our content-aware resizing once again produces far more pleasing results than simple scaling.

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/insert-and-remove.png">
  <div class="figcaption">Using content-aware resizing preserves the main areas of the input image while duplicating, removing the less important low-energy areas. This produces much more appealing results than simple scaling which introduces heavy distortion throughout, and whose affect is amplified in the case of uneven scaling of width and height.</div>
</div>

<a name='subtopic-3-4'></a>
### Multi-size image resizing

<a name='subtopic-3-5'></a>
### Object removal

<a name='subtopic-3-6'></a>
### Improvement: Forward energy

<a name='subtopic-3-7'></a>
### Video seam carving

