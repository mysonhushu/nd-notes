### Abstract

Most computer graphics pictures have been computed all at once, so that the rendering program takes care of all
computations relating to the overlap of objects. 

大多数计算机图形图片都一次计算所有的，以便渲染程序处理与对象重叠相关的所有计算。

There are several applications, however, where elements must be rendered separately, relying on eompositing techniques 
for the anti-aliased accumulation of the full image. 

但是，有几个应用程序, 必须单独渲染元素，依靠假设技术(eompositing techniques) 为反别名积累的完整图片。

This paper presents the case for four-channel pictures, demonstrating that a matte component can be computed similarly to the color channels. 

本文介绍了四通道图片的案例，表明哑光组件(matte component)的计算方式与颜色通道(color channels)类似。

The paper discusses guidelines for the generation of elements and the arithmetic for their arbitrary compositing. 

本文讨论了元素生成指南及其任意组合的算术。


CR Categories and Subject Descriptors: 
CR 类别和主题描述符：

1.3.3 [Computer Graphics]: Picture/Image Generations --Display algorithms; 
图片/图像生成 -- 显示算法：

1.3.4 [Computer Graphics]: Graphics Utilities -- Software support; 
图形实用程序 -- 软件支持

1.4.1 [Image Processing]: Digitization -- Sampling. 
数字化 -- 取样。

General Terms: Algorithms 

一般术语：算法

Additional Key Words and Phrases: compositing, matte channel, matte algebra, visible surface algorithms, graphics systems

其他关键词语和短语：合成、哑光通道、哑光代数、可见表面算法、图形系统

### 1. Introduction 

Increasingly, we find that a complex three dimensional scene cannot be fully rendered by a single program. 

我们越来越发现，复杂的三维场景无法完全由单个程序呈现。

The wealth of literature on rendering polygons and curved surfaces, handling the special cases of fractals and spheres
and quadrics and triangles, 

关于渲染多边形和弯曲表面、处理分形和球体、四角形和三角形的特殊情况的大量文献，

implementing refinements for texture mapping and bump mapping, 

实施纹理映射和凹凸映射的改进，

noting speed-ups on the basis of coherence or depth complexity in the scene, suggests that multiple programs are necessary. 

注意到基于场景的连贯性或深度复杂性的加速，表明需要多个程序。

===

In fact, reliance on a single program for rendering an entire scene is a poor strategy for minimizing the cost of small modeling errors.

事实上，依靠单个程序来渲染整个场景是将小建模错误成本降至最低的糟糕策略。

 
 Experience has taught us to break down large bodies of source code into separate modules in order to save compilation time. 

经验教会我们将大量源代码分解成单独的模块，以节省编译时间。
 
 An error in one routine forces only the recompilation of its module and the relatively quick reloading of the entire program.
 
 一个常规程序中的错误只会强制其模块的重新编译和整个程序的相对快速重新加载。
  
  Similarly, small errors in coloration or design in one object should not force the "recompilation" of an entire image. 
 
  同样，一个对象的着色或设计小错误不应强制"重新编译"整个图像。

===

Separating the image into elements which can be independently rendered saves enormous time.

将图像分离成可以独立呈现的元素可节省大量时间。
 
 Each element has an associated matte, coverage information which designates the shape of the element. 
 
 每个元素都有一个相关的哑光(matte)，覆盖信息(coverage information)，指定元素的形状。
 
 The eompositing of those elements makes use of the mattes to accumulate the final image. 

 这些元素的吸收利用哑光(mattes)来累积最终图像。
 

===


 The compositing methodology must not induce aliasing in the image;
 
 合成方法不得在图像中诱发别名：
  
  soft edges of the elements must be honored in computing the final image. 

在计算最终图像时，必须重视元素的边缘柔化(soft edges)。
 
 Features should be provided to exploit the full associativity of the compositing process;
 
 应提供特性，利用合成过程的充分关联性：
  
  this affords flexibility, for example, for the accumulation of several foreground elements into an aggregate foreground which can be examined over different backgrounds. 
  
  这提供了灵活性, 例如，将几个前景元素累积到一个聚合前景中，可以根据不同的背景进行校验。
  
  The compositor should provide facilities for arbitrary dissolves and fades of elements during an animated sequence. 
 
  合成器应在动画序列中为元素的任意溶解和褪色提供方法。
  
  
  
  
  
  
  
  
===

  Several highly successful rendering algorithms have worked by reducing their environments to pieces that can be combined in a 2 1/2 dimensional manner, and then overlaying them either front-to-back or back-to-front [3].
  
 几个非常成功的渲染算法通过将其环境简化为可以以 2 1/2 维方式组合的片段，然后将它们从前到后或从后到前 [3] 叠加在一起。
 
  
  
  Whitted and Weimar's graphics test-bed [6] and Crow's image generation environment [2] are both designed to deal with heterogenously rendered elements. 
 
  惠特德和魏玛的图形测试台(graphics test-bed) [6] 和 Crow 的图像生成环境 (image generation environment)[2] 都旨在处理不同类地渲染的元素。
  
  Whitted and Weimar's system reduces all objects to horizontal spans which are eomposited using a Warnock-like algorithm. 
 
  Whitted 和 Weimar 的系统将所有物体缩小到水平跨度，并使用类似沃诺克的算法合成(Warnock-like algorithm)。
  

  In Crow's system a supervisory process decides the order in which to combine images created by independent special-purpose rendering processes. 
  
 在 Crow 的系统中，监督过程决定合并独立特殊用途渲染过程创建的图像的顺序。
  
  The imaging system of Warnock and Wyatt [5] incorporates 1-bit mattes. 
  
  
  The Hanna-Barbera cartoon animation system [4] incorporates soft-edge mattes, representing the opacity information in a less convenient manner than that proposed here. 
  
 沃诺克和怀亚特[5]的成像系统包含 1-bit 哑光(mattes)。
  
  
  The present paper presents guidelines for rendering elements and introduces the algebra for compositing. 
 
  本文介绍了渲染元素的指南，并介绍了合成代数。
  
### 2. The Alpha Channel

A separate component is needed to retain the matte information, the extent of coverage of an element at a pixel. 

需要单独的组件来保留哑光(matte)信息，即像素上元素的覆盖范围。

In a full color rendering of an element, the RGB components retain only the color. In order to place the element over an arbitrary background, a mixing factor is required at every pixel to control the linear interpolation of foreground and background colors. 

在元素的全彩色渲染中，RGB 组件仅保留颜色。为了将元素置于任意背景上，需要每个像素的混合因子来控制前景和背景颜色的线性插值。


In general, there is no way to encode this component as part of the color information.

 一般来说，无法将此组件编码为颜色信息的一部分。
 
 For anti-aliasing purposes, this mixing factor needs to be of comparable resolution to the color channels. 

 为了防别名目的，此混合因子需要具有与颜色通道可比的分辨率。
 
 Let us call this an alpha channel, and let us treat an alpha of 0 to indicate no coverage, 1 to mean full coverage, with fractions corresponding to partial coverage. 
 
让我们称之为阿尔法通道(alpha channel)，让我们把 alpha=0 表示没有覆盖，alpha=1 表示全覆盖，alpha如果是分数分数,表示部分覆盖。
 
 ====
 
 In an environment where the compositing of elements is required, we see the need for an alpha channel as an integral part of all pictures.
 
 在需要元素组合的环境中，我们认为需要将 alpha 通道作为所有图片的组成部分。
  
  Because mattes are naturally computed along with the picture, a separate alpha component in the frame buffer is appropriate. 
 
  由于哑光与图片一起自然计算，因此帧缓冲区中的单独的 alpha 组件是适当的。
  
  Off-line storage of alpha information along with color works conveniently into run-length encoding schemes because the alpha information tends to abide by the same runs. 
  
  离线存储阿尔法信息以及颜色，方便地进入运行长度编码方案，因为阿尔法信息倾向于遵守相同的运行。
  
===

  What is the meaning of the quadruple (r,g,b,a) at a pixel?
  
  像素的💀部分(r, g, b, a) 是什么意思？
   
  How do we express that a pixel is half covered by a full red object? 
  
  我们如何表达像素被全红色物体覆盖的一半？
  
  One obvious suggestion is to assign (1,0,0,.5} to that pixel: the .5 indicates the coverage and the (1,0,0) is the color.
  
  一个显而易见的建议是将（1, 0, 0, .5）分配给该像素：.5表示覆盖范围，而（1，0，0）表示颜色。
  
   
   There are a few reasons to dismiss this proposal, the most severe being that all compositing operations will involve multiplying the 1 in the red channel by the .5 in the alpha channel to compute the red contribution of this object at this pixel.
    
   有几个理由驳回这一建议，最严重的是，所有合成操作将涉及将红色通道中的 1 乘以 alpha 通道中的 0.5，以计算此像素中此对象的红色贡献。
    
   The desire to avoid this multiplication points up a better solution, storing the pre-multiplied value in the color component,
   
   避免这种乘法的期望为更好的解决方案指出了更好的解决方案，将预乘值(pre-multiplied value)存储在颜色组件(color component)中，
    
   so that (.5,0,0,.5) will indicate a full red object half covering a pixel. 
 
  这样（0.5，0，0，。5）将指示一个覆盖一个像素的全红色物体的一半。
  
 ====
 
 The quadruple (r,g,b,a) indicates that the pixel is a covered by the color (r/a, g/a, b/a). 
 
 这个四个部分(r，g，b，a) 表示像素是颜色覆盖的 (r/a，g/a，b/a) 。
 
 A quadruple where the alpha component is less than a color component indicates a color outside the [0,1] interval, which is somewhat unusual. 

 alpha 组件小于颜色组件的四个部分表示 [0，1] 区间之外的颜色，这有些不同寻常。
 
 We will see later that luminescent objects can be usefully represented in this way.
 
 我们稍后将看到，发光物体可以以这种方式有用地表示。
  
  For the representation of normal objects, an alpha of 0 at a pixel generally forces the color components to be 0. 
  
  对于正常对象的表示，像素为 0 的 alpha 通常强制颜色组件为 0。
  
  Thus the RGB channels record the true colors where alpha is 1, linearly  darkened colors for fractional alphas along edges, and black where alpha is 0.
  
  因此，RGB 通道记录阿尔法为 1 的真彩色，沿边缘的分数阿尔法的线性变暗颜色，以及 alpha 为 0 的黑色。
    
 Silhouette edges of RGBA elements thus exhibit their anti-aliased nature when viewed on an RGB monitor.
 
  因此，当在 RGB 监视器上查看时，RGBA 元素的剪影边缘会表现出其反别名性质。
 
It is important to distinguish between two key pixel representations:

区分两个关键像素表示很重要：
                                                                           black ~- (0,0,0,1);
                                                                           clear-~ (0,0,0,0).
                                                                           
The former pixel is an opaque black; the latter pixel is transparent. 

前像素是不透明的黑色：后一个像素是透明的。


### 3. RGB Pictures

If we survey the variety of elements which contribute to a complex animation, we find many complete background images which have an alpha of 1 everywhere.

 如果我们调查促成复杂动画的各种元素，我们会发现许多完整的背景图像，其阿尔法为 1 无处不在。
 
 Among foreground elements, we find that the color components roll off in step with the alpha channel, leaving large areas of transparency.
 
 在前景元素中，我们发现颜色组件与 alpha 通道同步滚动，留下大量区域的透明。
 
  
 Mattes, colorless stencils used for controlling the compositing of other elements, have 0 in their RGB components. 
 
 用于控制其他元素组合的无色模具哑光(mattes)在其 RGB 组件中具有 0。
 
 
 Off-line storage of RGBA pictures should therefore provide the natural data compression for handling the RGB pixels of backgrounds, RGBA pixels of foregrounds, and A pixels of mattes. 
 
 因此，RGBA 图片的离线存储应提供自然数据压缩，用于处理背景(backgrounds)的 RGB 像素、前景(foregrounds)的 RGBA 像素和一个哑光(mattes)的像素。
 
====
 
There are some objections to computing with these RGBA pictures. 

有一些反对计算与这些RGBA图片。


Storage of the color components premultiplied by the alpha would seem to unduly quantize the color resolution, especially as alpha approaches 0.

由 alpha 预制的颜色组件的存储似乎不适当地量化了颜色分辨率，尤其是在 alpha 接近 0 时。
 
 However, because any compositing of the picture will require that multiplication anyway, storage of the product forces only a very minor loss of precision in this regard.

 但是，由于图片的任何组合都需要这种乘法，因此，存储产品只会使这方面的精度损失非常小。
 
  Color extraction, to compute in a different color space for example, becomes more difficult. 
  
 例如，在不同的颜色空间中计算颜色提取变得更加困难。
  
  
  We must recover (r/a, g/a, b/a), and once again, as alpha approaches 0, the precision falls off sharply. 
  
  我们必须恢复(r/a，g/a，b/a)，并再次，随着 alpha 接近 0，精度急剧下降。
  
  
  For our applications, this has yet to affect us.

  对于我们的应用程序，这尚未影响我们。
  
###    4. The Algebra of Compositing


Given this standard of RGBA pictures, let us examine how compositing works.
 
 We shall do this by enumerating the complete set of binary compositing operations.
  
  For each of these, we shall present a formula for computing the contribution of each of two input pictures to the output composite at each pixel. 
  
  We shall pay particular attention to the output pixels, to see that they remain pre-multiplied by their alpha. 
  
#### 4.1   Assumptions

When blending pictures together, we do not have information about overlap of coverage information within a pixel; 

all we have is an alpha value. When we consider the mixing of two pictures at a pixel, we must make some assumption about 
the interplay of the two alpha values.

In order to examine that interplay, let us first consider the overlap of two semi-transparent elements like haze,
then consider the overlap of two opaque, hard-edged elements. 

If a A and aB represent the opaqueness of semitransparent objects which fully cover the pixel, the computation is well known.
 
 
 Each object lets (l-a) of the background through, so that the background shows through only (1-aA)(1-aB) of the pixel, aA(l-a~) of the
background is blocked by object A and passed by object B;
 
 
 (1-~A)a B of the background is passed by A and blocked by B. 
 
 This leaves OlAOl B of the pixel which we can consider to be blocked by both. 
 
 If  αA and αB represent subpixel areas covered by opaque geometric objects, the overlap of objects within the pixel
 is quite arbitrary.
  
  We know that object A divides the pixel into two subpixel areas of ratio C~A:l-a A.
   
  We know that object B divides the pixel into two subpixel areas of ratio crB:l-cr13.
   
  Lacking further information, we make the following assumption: there is nothing special about the shape of the pixel; 
  
  we expect that object B will divide each of the subpixel areas inside and outside of object A into the same ratio a/3:l-a B. 
  
  The result of the assumption is the same arithmetic as with semi-transparent objects and is summarized in the following table: 
  
  差一张合成的图:
  
  The assumption is quite good for most mattes, though it can be improved if we know that the coverage seldom overlaps (adjacent segments of a continuous line) or always overlaps (repeated application of a picture).
   
   For ease in presentation throughout this paper, let us make this assumption and consider the alpha values as representing subpixel coverage of opaque objects. 
  
  ### 4.2 Compositing Operators
  
  Consider two pictures A and B. They divide each pixel into the 4 subpixel areas 
  
  差一张合成的图：
  
  listed in this table along with the choices in each area for contributing to the composite. 
  
  In the last area, for example, because both input pictures exist there, either could survive to the composite. 
  
  Alternatively, the composite could be clear in that area. 
  
  A particular binary compositing operation can be identified as a quadruple indicating the input picture which contributes to the composite in each of the four subpixel areas 0, A, B, AB of the table above. 
  
  With three choices where the pictures intersect, two where only one picture exists and one outside the two pictures, there
  are 3 X 2 X 2 X 1~---12 distinct compositing operations listed.
  
  
  in the table below. Note that pictures A and B are diagrammed as covering the pixel with triangular wedges whose overlap conforms to the assumption above. 
  
  
  Useful operators include A over/5, A in B, and A held out by B. A over B is the placement of foreground A in front of background B.
   
   A in B refers only to that part of A inside picture B.
    
   A held out by B, normally shortened to A out B, refers only to that part of A outside picture B. 
   
   For completeness, we include the less useful operators A atop B and A xor B.
    
   A atop B is the union of A in B and B out A.
    
   Thus, paper atop table includes paper where it is on top of table, and table otherwise; area beyond the edge of the table is out of the picture.
   
  A xor Bis the union of A out B and B out A. 
  
  
  
### 4.3 Compositing Arithmetic

For each of the compositing operations, we would like to compute the contribution of each input picture at each pixel. 

This is quite easily solved by recognizing that each input picture survives in the composite pixel only within its own matte.
 
For each input picture, we are looking for that fraction of its own matte which prevails in the output.

  By definition then, the alpha value of the composite, the total area of the pixel covered, can be computed by
adding aA times its fraction F A to aB times its fraction  FB.



The color of the composite can be computed on a component basis by adding the color of the picture A times
its fraction to the color of picture B times its fraction.


To see this, let CA, cB, and c o be some color component of pictures A, B and the composite, and let CA, CB, and
C o be the true color component before pre-multiplication by alpha. 

Then we have Now C o can be computed by averaging contributions made by C A and CB, so


but the denominator is just ao, so 



Because each of the input colors is pre-multiplied by its alpha, and we are adding contributions from nonoverlapping areas, the sum will be effectively premultiplied by the alpha value of the composite just computed.
 
 The pleasant result that the color channels are handled with the same computation as alpha can be traced back to our decision to store pre-multiplied RGBA quadruples.
  
  Thus the problem is reduced to finding a table of fractions FA and FB which indicate the extent of contribution of A and B, plugging these values into equation 1 for both the color and the alpha components. 
  
 ===========
 
 By our assumptions above, the fractions are quickly determined by examining the pixel diagram included in the table of operations. 
 
 
 Those fractions are listed in the F A and F B columns of the table. For example, in the A over B case, picture A survives everywhere while picture B survives only outside picture A, so the corresponding fractions are 1 and (1-αA). 
 
 
 Substituting into equation 1, we find 
 
 This is almost the well used linear interpolation of foreground F with background B 
 
 except that our foreground is pre-multiplied by alpha. 
 
 ### 4.4 Unary operators
 
 To assist us in dissolving and in balancing color brightness of elements contributing to a composite, it is useful to introduce a darken 
 factor ¢ and a dissolve factor 6: 
 
 darken(A,¢)=(¢r A,~gA,dPbA,Ol A)
 dlssolve(A,6)----(6rA,6gA,6bA,6OtA) . 
 
 Normally, 0<~,6_~1 although none of the theory requires it. 
 
 
 As ~ varies from 1 to 0, the element will change from normal to complete blackness. If 4>1 the element will be brightened. 
 
 As 6 goes from 1 to 0 the element will gradually fade from view. 
 
 Luminescent objects, which add color information without obscuring the background, can be handled with the introduction of a opaqueness factor w, 0<w< 1: 
 
 opaque(A,w)----(rA, gA, bA,W°t A) 
 
 
 As w varies from 1 to 0, the element will change from normal coverage over the background to no obscuration.
 
 This scaling of the alpha channel alone will cause pixel quadruples where (~ is less than a color component, indicating a representation of a color outside of the normal range. 
 
 
 This possibility forces us to clip the output composite to the [0,1] range. 
 
 
 An w of 0 will produce quadruples (r,g,b,O) which do have meaning. 
 
 The color channels, pre-multiplied by the original alpha, can be plugged into equation 1 as always.
  
  The alpha channel of 0 indicates that this pixel will obscure nothing. 
  
  In terms of our methodology for examining subpixel areas, we should understand that using the opaque operator corresponds to shrinking the matte coverage with regard to the color coverage. 
  
  
  ### 4.5. The PLUS operator 
  
  We find it useful to include one further binary compositing operator plus. The expression A plus B holds no notion of precedence in any area covered by both pictures; the components are simply added.
   
   
   This allows us to dissolve from one picture to another by specifying
   
   差一个公式：
   
   In terms of the binary operators above, plus allows both pictures to survive in the subpixel area AB. 
   
   The operator table above should be appended: 
   
   
 ### 5. Examples
 
   The operations on one and two pictures are presented as a basis for handling compositing expressions involving several pictures.
   
   A normal case involving three pictures is the compositing of a foreground picture A over a background picture B, with regard to an independent matte C. 
   
   The expression for this compositing operation is
   
   
   Using equation 1 twice, we find that the composite in this case is computed at each pixel by 
   
   
   As an example of a complex compositing expression, let us consider a subwindow of Rob Cook's picture Road to
   Point Reyes [1]. 
   
   This still frame was assembled from many elements according to the following rules: 
   
   
 ====  
   A further example demonstrates the problem of correlated mattes. 
   
   In Figure 2, we have a star field background, a planet element, fiery particles behind the planet, and fiery particles in front of the planet. 
   
   
   We wish to add the luminous fires, obscure the planet, darkened for proper balance, with the aggregate fire matte, and
   place that over the star field. 
   
   
   An expression for this eompositing is 
  
   差一个公式
   
   We must remember that our basic assumption about the division of subpixel areas by geometric objects breaks down in the face of input pictures with correlated mattes.
   
   When one picture appears twice in a compositing expression, we must take care with our computations of F A and
   F B. 
   
   Those listed in the table are correct only for uneorrelated pictures. 
   
   
   =====
   
   There are several problems to be resolved in related areas, which are open for future research.
    
   We are interested in methods for breaking arbitrary three dimensional scenes into elements separated in depth. 
   
   Such elements are equivalent to clusters, which have been a subject of discussion since the earliest attempts at hidden
   surface elimination. 
   
   We are interested in applying the compositing notions to Z-buffer algorithms, where depth information is retained at each pixel. 
   
   
   ===
   
   ###  7. References
   
   1. Cook, R. Road to Point Reyes. Computer Graphics Vol 17, No. 3 (1983), Title Page Picture.
   
   2. Crow, F. C. A More Flexible Image Generation Environment. Computer Graphics Vol. 16, No. 3 (1982), pp. 9-18.
   
   3. Newell, M. G., Newell, R. G., and Saneha, T. L.. A Solution to the Hidden Surface Problem, pp. 443- 448. Proceedings of the 1972 ACM National Conference.
   
   4. Wallace, Bruce. Merging and Transformation of Raster Images for Cartoon Animation. Computer Graphics Vol. 15, No. 3 (1981), pp. 253.262.
   
   5. Warnoek, John, and Wyatt, Douglas. A Device Independent Graphics Imaging Model for Use with Raster Devices. Computer Graphics Vol. 16, No. 3 (1982), pp. 313-319.
   
   6. Whitted, Turner, and Weimer, David. A Software Test-Bed for the Development of 3-D Raster Graphies Systems. Computer Graphics Vol. 15, No. 3 (1981), pp. 271-277. 


   =====
   
  8. Acknowledgment•
  
 The use of mattes to control the compositing of pictures is not new. 
  
 The graphics group at the New York Institute of Technology has been using this for years. 
  
 NYIT color maps were designed to encode both color and matte information; 
  
  that idea was extended in the Ampex AVA system for storing mattes with pictures.
   
   Credit should be given to Ed Catmull, Alvy Ray Smith, and Ikonas Graphics Systems for the existence of an alpha channel as an
    integral part of a frame buffer, which has paved the way for the developments presented in this paper.
    
    
 The graphics group at Lucasfilm should be credited with providing a fine test bed for working out these ideas.
   
 Furthermore, certain ideas incorporated as part of this work have their origins as idle comments within this
    group. 
    
 Thanks are also given to Rodney Stock for comments on an early draft which forced the authors to clarify the major assumptions. 
 
 
 
 
 
  
  

 
  
 






