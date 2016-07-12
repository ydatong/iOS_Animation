<font color=blue></font>
## I: The Layer Beneath
CALayer: 每个UIView都内部都包含一个layer对象（图层）,UIView的渲染，布局和动画，都是通过这个layer对象来管理的。但是UIView最显著的特性之一：用户交互，不是通过CALayer来管理的，因为CALayer不支持响应链，不能响应事件（但是CALayer提供了方法来判断点击事件是否在某个layer的bounds之内）

**1.为什么iOS要设计这样的UIView与CALayer的平行分层？而不是只用一层UIView来处理所有的事情？**

为了分离职责，减少重复代码。事件处理和用户交互在iOS和Mac OS上差别很大（最基本的用户交互差别是iOS是基于触摸操作，Mac OS基于鼠标和键盘的操作，这也是为什么iOS中是UIKit和UIView而Mac OS中是AppKit和NSView，它们虽然功能像似，但是实现方法不同），但是相反的是绘图，布局和动画等，在iOS和Mac OS中概念基本相同。通过将其中的逻辑抽离出来到独立的Core Animation框架中，Apple可以在iOS平台和Mac OS平台中进行代码共享。

**2.CALayer可以做什么?**

常见的:

* 绘制阴影，圆角，着色边界
* 3D变换和定位
* 非矩形边界
* 多步，非线性动画

**3.使用CALayer**
例子：


### <font color=blue>1.The Layer Tree</font>

### <font color=blue>2.背景图片</font>

CALayer有一个属性叫做：contents，在OC中是id类型，swift中是AnyObject?类型，这代表它可以是任意类型的对象。但是你会发现如果给contents赋值CGImage类型以外的对象时，得到的都是空白的结果。这是Mac OS中遗留下来的问题。之所以设计为id类型是为了在Mac OS中可以赋值CGImage或者NSImage并且都可以得到想要的结果，但是在iOS中，如果赋值UIImage得到的也是一个空白的结果。（需要补充。。）

**contentsGravity** 等。。。

除了contents可以设置背景图片以外，我们可以直接使用Core Graphics来直接绘制图形来填充到背景图片中，**-drawRect:** 方法可以在UIView的子类中重写来实现自定义绘图。只要UIView发现drawRect方法实现了，就会自动生成一个背景图片，所以如果你不需要这个背景图片的话，最好不要留一个空白的drawRect方法，因为这样会浪费内存。
**-drawRect:** 方法在view第一次显示在屏幕上时自动调用，在这个方法中用Core Graphics来绘制图形到背景图片中，图形会一直缓存在内存中直到view需要刷新（一般都是通过手动调用 -setNeedsDisplay方法来实现，但是view有一些属性的改变也会引起重绘，比如bounds）

CALayer有一个可选的delegate属性，遵循CALayerDelegate协议（非正式协议），当CALayer需要内容信息的时候，会从delegate中取获取。
当layer需要重绘的时候，CALayer会调用delegate的 

**- (void)displayLayer:(CALayerCALayer *)layer;**

方法来获取背景图片。

如果delegate没有实现上面的方法，CALayer会自动创建一个空的背景图片和绘制的图形上下文（context），作为调用delegate的

**- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;**

方法的参数。

> So now you understand the CALayerDelegate and how to use it. But unless you are creating standalone layers, you will almost never need to implement the CALayerDelegate protocol. The reason for this is that when UIView creates its backing layer, it automatically sets itself as the layer’s delegate and provides an implementation for -displayLayer: that abstracts these issues away.When using view-backing layers, you do not need to implement -displayLayer: or -drawLayer:inContext: to draw into your layer’s backing image; you can just implement the -drawRect: method of UIView in the normal fashion, and UIView takes care of everything, including automatically calling -display on the layer when it needs to be redrawn

### <font color=blue>3.Layout Geometry</font>

* UIView 三个主要的layout属性: frame, bounds, center
* CALayer 的三个layout属性： frame , bounds, position

center 和 position 都代表anchorPoint的位置

> The anchorPoint property of a layer controls how the layer’s frame is positioned relative to its position property

> when we change the anchorPoint, the position property does not change. Instead, the position remains fixed and the frame of the layer moves


### <font color=blue>4.Visual Effects</font>
>  Instead of being drawn inside the parent, the mask layer defines the part of the parent layer that is visible.The color of the mask layer is irrelevant; all that matters is its silhouette. The mask acts like a cookie cutter; the solid part of the mask layer will be “cut out” of its parent layer and kept; anything else is discarded (see Figure 4.12).If the mask layer is smaller than the parent layer, only the parts of the parent (or its sub- layers) that intersect the mask will be visible. If you are using a mask layer, anything outside of that layer is implicitly hidden.


### <font color=blue>5.Transforms</font>
##### CGAffineTransform(3\*3的矩阵) 代表的是 <font color=red>*二维平面</font>* 的 旋转(rotation)，缩放(scale)，平移(translation)
##### 仿射变换(Affine) 代表的是平面之内变化之前平行的线，变化之后依然平行

### <font color=blue>6.Specialized Layers</font>
* #### CAShapeLayer

CAShapeLayer绘制的是矢量图形，而不是位图。

优点：速度快，内存效率高（不用像CALayer一样创建一个backing 
image），不会被layer的bounds切边，没有像素

**单独设置layer每个角的圆角**

`
//define path parametersCGRect rect = CGRectMake(50, 50, 100, 100); CGSize radii = CGSizeMake(20, 20); UIRectCorner corners = UIRectCornerTopRight |UIRectCornerBottomRight | UIRectCornerBottomLeft;//create pathUIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:rect byRoundingCorners:corners cornerRadii:radii];
`
> If we want to clip the view’s contents to this shape, we can use our CAShapeLayer as the mask property of the view’s backing layer instead of adding it as a sublayer.

* #### CATextLayer

渲染速度比UILabel快，支持富文本

**retina屏幕下可能显示有问题:**

`
textLayer.contentsScale = [UIScreen mainScreen].scale;
`

> We could subclass UILabel and override its methods to display the text in a CATextLayer that we’ve added as a sublayer, but we’d still have the redundant empty backing image created by the presence of UILabel -drawRect: method. And because CALayer doesn’t support autoresizing or autolayout, a sublayer wouldn’t track the size of the view bounds automatically, so we would need to manually update the sublayer bounds every time the view is resized.
What we really want is a UILabel subclass that actually uses a CATextLayer as its backing layer, then it would automatically resize with the view and there would be no redundant backing image to worry about.

> UIView is backed by an instance of CALayer. That layer is automatically created and managed by the view, so how can we substitute a different layer type? We can’t replace the layer once it has been created, but if we subclass UIView, we can override the +layerClass method to return a different layer subclass at creation time. UIView calls the +layerClass method during its initialization, and uses the class it returns to create its backing layer.

> in general, using +layerClass to create views backed by different layer types is a clean and reusable way to utilize CALayer subclasses in your apps.


* #### CATransformLayer
* #### CAGradientLayer
* #### CAReplicatorLayer
* #### CAScrollLayer
* #### CATiledLayer
* #### CAEmitterLayer
* #### CAEAGLLayer
* #### AVPlayerLayer

***

**创建仿射变换矩阵**

`
CGAffineTransformMakeRotation(CGFloat angle) CGAffineTransformMakeScale(CGFloat sx, CGFloat sy) CGAffineTransformMakeTranslation(CGFloat tx, CGFloat ty)
`

**在已经存在的矩阵上创建新的矩阵**
`CGAffineTransformRotate(CGAffineTransform t, CGFloat angle) CGAffineTransformScale(CGAffineTransform t, CGFloat sx, CGFloat sy) CGAffineTransformTranslate(CGAffineTransform t, CGFloat tx, CGFloat ty)
`

** 单位矩阵（identity matrix）**

`CGAffineTransformIdentity`
**两个矩阵结合**
`
CGAffineTransformConcat(CGAffineTransform t1, CGAffineTransform t2)`
### 3D Transforms (4*4的矩阵)
创建:

`
CATransform3DMakeRotation(CGFloat angle, CGFloat x, CGFloat y, CGFloat z)`

` CATransform3DMakeScale(CGFloat sx, CGFloat sy, CGFloat sz) `

` CATransform3DMakeTranslation(Gloat tx, CGFloat ty, CGFloat tz)
`
## II: Setting Things In Motion### <font color=blue>1.隐式动画</font>
> When you change a property, how does Core Animation determine the type and duration of the animation that it will perform? The duration of the animation is specified by the settings for the current transaction, and the animation type is controlled by layer actions.

> Transactions are the mechanism that Core Animation uses to encapsulate a particular set of property animations. Any animatable layer properties that are changed within a given transaction will not change immediately, but instead will begin to animate to their new value as soon as that transaction is committed.### <font color=blue>2.显示动画</font>### <font color=blue>3.Layer Time</font>### <font color=blue>4.Easing</font>### <font color=blue>5.Timer-based Animation</font>