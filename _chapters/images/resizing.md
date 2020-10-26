---
title: Resizing
keywords: (insert comma-separated keywords here)
order: 11 # Lecture number for 2020
---

You can start your notes here before you start diving into specific topics under each heading. This is a useful place to define the topic of the day and lay out the structure of your lecture notes. You'll edit the Markdown file, in this case template.md, which will automatically convert to HTML and serve this web page you're reading! 

The table of contents can link to each section so long as you match the names right (see comments in template.md for more elaboration on this!). This Markdown to HTML mapping doesn't like periods in the section titles and won't link them from the table of contents, so use dashes instead if you need to.

- [First Big Topic](#first-big-topic)
	- [Subtopic 1-1](#subtopic-1-1)
	- [Subtopic 1-2](#subtopic-1-2)
	- [Subtopic 1-3](#subtopic-1-3)
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




<a name='seam-carving-algorithm'></a>
## Seam Carving Algorithm
This should give you the primary tools to develop your notes. Check out the [markdown quick reference](https://wordpress.com/support/markdown-quick-reference/) for any further Markdown functionality that you may find useful, and reach out to the teaching team on Piazza if you have any questions about how to create your lecture notes

<a name='overview'></a>
### Overview 

<a name='Implementation'></a>
### Implementation

<a name='Results'></a>
### Results

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
If we use a naive approach of iteratively computing the lowest energy seam and adding it to our input image, we will run into the pitfall of selecting the same seam at each iteration. As such, when expanding an image's width by $k$ pixels, the lowest energy seam in the image will be duplicated repeatedly $k$ times, giving us undesirable results as depicted below:

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam_runtime.png">
  <div class="figcaption">The task in Image Classification is to predict a single label (or a distribution over labels as shown here to indicate our confidence) for a given image. Images are 3-dimensional arrays of integers from 0 to 255, of size Width x Height x 3. The 3 represents the three color channels Red, Green, Blue.</div>
</div>

Note that we have many columns containing repeated pixels in the image above. This effect happens because the lowest energy seam is always the same after we replicate that seam again in the image.

#### Solution
A solution to this pitfall is to calculate as many seams as we need to insert into the image, so that we only duplicate each seam once. To expand an image by $k$ pixels, we compute the $k$ seams with the lowest energy at once, and duplicate each of them. Doing so gives us much more desirable results as depicted below:

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam_runtime.png">
  <div class="figcaption">The task in Image Classification is to predict a single label (or a distribution over labels as shown here to indicate our confidence) for a given image. Images are 3-dimensional arrays of integers from 0 to 255, of size Width x Height x 3. The 3 represents the three color channels Red, Green, Blue.</div>
</div>

This method produces a far higher quality expanded image compared to the naive approach. Furthermore, compared to scaling, our content-aware resizing produces much more please results.

#### Combined Insert and Remove
Another way of using this algorithm is combining insertion and removal of seams. This is very desirable to resize dimensions of an image different. For example, given the input image on the left, we can insert vertical seams to make the image wider, and remove horizontal seams to make the image shorter in height. Our content-aware resizing once again produces far more pleasing results than simple scaling.

<div class="fig figcenter fighighlight">
  <img src="{{ site.baseurl }}/assets/images/seam_runtime.png">
  <div class="figcaption">The task in Image Classification is to predict a single label (or a distribution over labels as shown here to indicate our confidence) for a given image. Images are 3-dimensional arrays of integers from 0 to 255, of size Width x Height x 3. The 3 represents the three color channels Red, Green, Blue.</div>
</div>

<a name='subtopic-3-4'></a>
### Multi-size image resizing

<a name='subtopic-3-5'></a>
### Object removal

<a name='subtopic-3-6'></a>
### Improvement: Forward energy

<a name='subtopic-3-7'></a>
### Video seam carving

