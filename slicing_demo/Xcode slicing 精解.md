# Xcode slicing 精解

## slicing 和图片拉伸
从 Xcode 5 开始就支持 image slicing 功能，这个功能非常强大，从一定程度上解决了不规则图片的拉伸问题。然而许多同学并不知道这个功能，或者在对这个功能理解得并不透彻，导致使用中出现这样那样的问题。

简单说，这个功能是用来拉伸图片的，是这两个方法的可视化版本：

1. stretchableImageWithLeftCapWidth: topCapHeight: 方法（已抛弃）或者 
2. resizableImageWithCapInsets: resizingMode: 方法

打个比方，美工给你这样一张气泡图：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/1.png?raw=true" width="240"/>

这个图要怎么用？因为聊天气泡是会跟随文字内容的多少变化的，比如这些聊天气泡：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/2.png?raw=true" width="240"/>

这些聊天气泡的背景图片使用的都是美工给的图片，不过根据聊天内容的多少进行了拉伸处理。由于美工给的气泡图片不是规则图片（四方形），那怎么拉伸就成了难题。比如图片中某些部位可以拉伸，有的部位保持原样。

这就是上面两个函数的由来。这两个函数就是用于进行“不规则”图片的拉伸处理的。它们都会对图片的某些部位进行“保护”，在这些保护的部位，图片是不拉伸的，而不保护的部位则是可拉伸的。这个“受保护”的部位，一般位于图片的4个角。比如美工给的气泡图片，我们可以将它的4个角保护起来，位于这四个角的像素不进行拉伸，而除此之外的像素可以拉伸。

还是以上一张气泡图为例子吧，看看我们怎样用 slicing 功能对它进行拉伸。

## 简单操作

将这张图片拖到 assets 中。注意 slicing 只支持对处于 Assets 中的图片。打开 Assets，选中这张图片，你会看到右下角有一个 show slicing 按钮：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/3.png?raw=true" width="160"/>

点击它，图片上出现 start slicing 按钮：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/4.png?raw=true" width="340"/>

点击这颗按钮，又出现 3 个图标按钮：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/5.png?raw=true" width="300"/>

这 3 个选项分别表示：左右拉伸、上下左右拉伸、上下拉伸。

对于我们的气泡来说，应该使用上下左右拉伸。因为聊天内容既会让气泡变长，有会让气泡变高。如果是图片只会变长，那么你可以选择左右拉伸，这样切图的时候不需要考虑上下拉伸情况，切图时按左中右切就可以了。反之，如果图片只会变高，那么可以选择上下拉伸，切图时按上中下切就可以了。

后两者的切法相对简单，我们不讨论这种情况。

选择“上下左右拉伸”，你会看到图片上多了几条虚线，把图分成了 9 个区域：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/6.png?raw=true" width="300"/>

这几条虚线分别是 3 条竖线和 3 条横线。注意，默认情况下左边 2 条竖线和上边 2 条横线会叠在一起，不注意会看不出来。

先保持这个样子不变，因为默认 Xcode 会对图片进行一个“智能”的切图，在大部分情况下，这种切分发就是我们需要的。

打开 Main.storyboard，拖入大小不同的几张 image view，将 image 属性设置为这张气泡图。运行 app，你会看到图片根据图片的 frame 完美地进行了拉伸：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/7.png?raw=true" width="240"/>

这其中除了点了几次按钮之外，我们什么也没做，这也太神奇了吧？

## 了解 slicing 

slicing 到底做了些什么？再来看 Xcode 的 slicing 界面。

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/6.png?raw=true" width="300"/>

这其中有一些区域被灰色蒙版区域所覆盖。这些区域实际上是会被丢弃的。也就是说美工给的图最终会被 Xcode 切除一部分（灰色部分），变成这个样子：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/8.png?raw=true" width="480"/>


这就是 slicing (切图）一词的来由。我把图放大了一点，注意第一根横线/竖线和第二根横/竖线之间的区域，就是以图中红色标注的点为交叉点的横、竖两根 1 像素的蓝线。这些线属于“样本区域”。而四周的 4 个角的区域是“保护区域”。也就是说，当图片被拉伸时，“保护区域”原样显示，不会被拉伸，而其它“非保护区域”则会进行拉伸，拉伸的方式是用“样本区域”进行复制（复制方式有两种：平铺或拉伸）而来。“样本”区域的定义是，第一根线和第二根线之间的区域。即第一根横线和第二根横线，以及第一根竖线和第二根竖线之间的区域。

> 注意：第一根线和第二根线是不可能完全重合的，二者间距最低也会有 1 个像素的间距，因为“样本”最起码也得有一个像素，否则拉伸时就没有样本像素可以复制了。

如果图片作横向拉伸（也就是宽度超过 slicing 后的切图宽度），则第一第二两根竖线之间的区域会被复制（横向拉伸或平铺），以填充多出来的区域。如果图片作竖向拉伸（也就是高度超过 slicing 后的切图的高度），则第一第二两根横线之间的区域会被复制，以填充多出来的区域。如果图片同时向横向、竖向进行拉伸，则横向、竖向两个动作同时进行。

通过上面的了解，我们可以看到，实际上 Xcode 默认的切图操作并不完美。其实保护区域中还是部分区域可以切除的，因为它们还有一部分区域是属于规则图形（矩形），很容易通过“复制”的方式克隆出来。比如下图：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/9.png?raw=true" width="240"/>

第一根横线上的红框区域和第三根横线下面的红框区域，这些都是竖向拉伸时的保护区域，但它们其实都是规则图形（矩形），可以通过“复制”方式克隆出来，所以完全可以被切除。
所以我们可以调整三根横线的位置，将这两部分区域从“保护区域”中剔除：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/10.png?raw=true" width="240"/>

这样实际保留下的部分应该是：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/11.png?raw=true" width="200"/>

除了不好复制的“保护区域”，我们只留下了 1 个像素的“样本区域”了。


## slicing 的一般步骤

通过上面的例子，我们知道 slicing 的过程其实可以分成 3 个步骤：

1. 切掉原图中的多余部分。所谓多余部分，也就是图片中可以通过简单拉伸来复制的区域。这个区域有两个，一个是横向复制区域（第二根竖线和第三根竖线之间的区域），这些区域是一个连续矩形区域，当图片做横向拉伸时这个区域是通过横向复制得来的。第二个是竖向复制区域（第二根横线和第三根横线之间的区域），这个区域也是一个连续矩形区域，当图片作竖向拉伸时这个区域是通过竖向复制得来的。因为这些区域可以通过 resizableImageWithCapInsets 函数简单复制而来，它们是不需要保留的，它们可以在运行时根据内容填充，你想要多少 resizableImageWithCapInsets 函数就会生成多少，所以这部分是多余的，不需要放在图片中，这些像素是由代码生成。
2. 划定“样本”区域。既然需要复制，那么必须有复制的“样本”。对应地，“样本”区域也会分成两个，一个是横向复制样本，一个是竖向复制样本，它们分别在作横向复制和竖向复制中作为拷贝的样本。大部分情况下，样本区域只需要保留 1 个像素宽。
3. 设定样本区域后，“保护区域”就自动划定了。因为一个完美的可拉伸切图中，除了样本区域，就只剩下“保护区域”了（因为“多余”的部分已经在第 1 步中剔除了）。这一步不需要进行任何操作，当上一步做完，它自然就会完成。

总结，slicing 的目的是去除图中的规则部分，只留下不规则部分和 1 像素的样本即可。这样做的原因是规则部分方便复制，而不规则部分不好复制，只有保留下来原样显示了。至于不规则部分为什么不好复制，这就要问 resizableImageWithCapInsets 函数了。因为 resizableImageWithCapInsets 函数只能做简单的二方向复制（也就是只能进行矩形的像素填充），还没有“智能”到能够沿不规则路径进行复制（也就是进行复杂图形的像素填充）的程度。

## slicing 转换成 resizableImageWithCapInsets 函数

其实 slicing 也能够很方便地转换成 resizableImageWithCapInsets 函数拉伸。在 slicing 过程中，要达到完美切图，有时候必须对横竖虚线进行像素级移动，对于鼠标操作来说，进行像素移动并不方便，这时候我们可以利用属性面板中的 slicing 区域来进行：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/12.png?raw=true" width="240"/>

其中：

* Left 调整的是第一根竖线距离左边的位置。
* Right 调整的是第三根竖线距离右边的位置。
* Top 调整的是第一根横线距离上边的位置。
* Bottom 调整的是第三根横线语句下边的位置。
* Center 指定拉伸（复制）方式： Tiles 表示平铺，Streches 表示拉伸。
* Width 调整的是第一根竖线和第二根竖线之间的水平间距。
* Height 调整的是第一根横线和第二根横线之间的垂直间距。

这些参数是不是看起来很眼熟（前 5 个参数）？没错，它就是 resizableImageWithCapInsets 中两个参数的来由。

因此通过这些参数，你可以将上图中的 slicing 转换成 resizableImageWithCapInsets 函数实现：

```swift
UIImage* image = [UIImage imageNamed:@"Combined Shape3"];
    
image = [image resizableImageWithCapInsets:UIEdgeInsetsMake(57, 40, 36, 57) resizingMode:UIImageResizingModeTile];
_imageView.image = image;
```
Build & run，你会看到效果如下：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/13.png?raw=true" width="240"/>

这是什么鬼？注意看图片文件的名字了吗，代码中使用的是 Combined Shape3，但实际上这个文件在项目导航器中显示为 Combined Shape3@3x.png，这说明这是一设备相关的图形文件，而 resizableImageWithCapInsets 函数是和设备无关的。

因此你可以将文件名中的 @3x 去掉，把图片降级为设备无关图片。或者，像下面一样，将 slicing 属性面板中的参数值除以 3，改成设备无关的像素单位：

	image = [image resizableImageWithCapInsets:UIEdgeInsetsMake(19, 13.333, 12, 19) resizingMode:UIImageResizingModeTile];

运行 App，你会看到：

<img src="https://github.com/kmyhy/tips/blob/master/slicing_demo/14.png?raw=true" width="240"/>

完美！











