# Quartz 2D 概述

Quartz 2D 是一个二维绘图引擎，可在 iOS 环境中访问，也可从内核以外的所有 Mac OS X 应用程序环境中访问。你可以使用 Quartz 2D 应用程序编程接口（API）来访问基于路径的绘图，绘制透明度，阴影，绘制阴影，透明层，颜色管理，消除锯齿渲染，PDF 文档生成和 PDF 元数据访问等功能。Quartz 2D 尽可能利用图形硬件的强大功能。

在 Mac OS X 中，Quartz 2D 可以与所有其他图形和成像技术一起使用 - Core Image，Core Video，OpenGL 和 QuickTime。可以使用 QuickTime 函数 GraphicsImportCreateCGImage 从 QuickTime 图形导入器在 Quartz 中创建图像。有关详细信息，请参阅 QuickTime Framework Reference。Moving Data Between Quartz 2D and Core Image in Mac OS X 描述了如何向 Core Image 提供图像，Core Image 是一个支持图像处理的框架。

同样，在 iOS 中，Quartz 2D 可与所有可用的图形和动画技术配合使用，例如 Core Animation，OpenGL ES 和 UIKit 类。

## 页面

Quartz 2D 使用 painter’s model 进行成像。在 painter’s model 中，每个连续的绘图操作都将一层“绘画”应用于输出“画布”，通常称为页面。可以通过在其他绘图操作中覆盖更多绘制来修改页面上的绘制。除非覆盖更多绘制，否则无法修改页面上绘制的对象。此模型允许你从少量强大的基元构造极其复杂的图像。

图1-1显示了 painter’s model 的工作原理。要在图的顶部获取图像，首先绘制左侧的形状，然后绘制实心形状。固体形状覆盖第一个形状，遮挡除第一个形状的周边之外的所有形状。形状在图的底部以相反的顺序绘制，首先绘制实心形状。如你所见，在 painter’s model 中，绘制顺序很重要。

图 1-1 The painter’s model

![](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Art/painters_model.gif)

页面可以是真正的纸张（如果输出设备是打印机）; 它可能是一张虚拟纸（如果输出设备是 PDF 文件）; 它甚至可能是位图图像。页面的确切性质取决于你使用的特定图形上下文。

## 绘图目的地：图形上下文

图形上下文是一种不透明的数据类型（CGContextRef），它封装了 Quartz 用于将图像绘制到输出设备的信息，例如 PDF 文件，位图或显示器上的窗口。图形上下文中的信息包括图形绘制参数和页面上绘制的设备特定表示。Quartz 中的所有对象都被绘制到图形上下文或由图形上下文包含。

你可以将图形上下文视为绘图目标，如图1-2所示。使用 Quartz 绘制时，所有特定于设备的特征都包含在你使用的特定类型的图形上下文中。换句话说，你可以通过为相同的 Quartz 绘图例程序列提供不同的图形上下文，将相同的图像绘制到不同的设备。你无需执行任何特定于设备的计算; Quartz 为你做到了。

图 1-2 Quartz 绘图目的地

![](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Art/draw_destinations.gif)

这些图形上下文可供你的应用程序使用：

- 位图图形上下文允许你将 RGB 颜色，CMYK 颜色或灰度绘制到位图中。位图是像素的矩形阵列（或光栅），每个像素表示图像中的一个点。位图图像也称为采样图像。请参阅 Creating a Bitmap Graphics Context。
- PDF 图形上下文允许你创建 PDF 文件。在 PDF 文件中，你的绘图将保留为一系列命令。PDF 文件和位图之间存在一些显着差异：
    - 与位图不同，PDF 文件可能包含多个页面。
    - 当你从另一台设备上的 PDF 文件中绘制页面时，生成的图像会针对该设备的显示特性进行优化。
    - PDF 文件本质上是分辨率无关的 - 绘制它们的大小可以无限增加或减少，而不会牺牲图像细节。用户感知的位图图像质量与要查看位图的分辨率相关联。

    请参阅 Creating a PDF Graphics Context。
- 窗口图形上下文是可用于绘制到窗口中的图形上下文。请注意，由于 Quartz 2D 是图形引擎而不是窗口管理系统，因此你可以使用其中一个应用程序框架来获取窗口的图形上下文。有关详细信息，有关详细信息，请参阅 Creating a Window Graphics Context in Mac OS X。
- 图层上下文（CGLayerRef）是与另一个图形上下文关联的屏幕外绘图目标。它旨在将图层绘制到创建它的图形上下文时获得最佳性能。与位图图形上下文相比，图层上下文可以是屏幕外绘制的更好选择。请参阅 Core Graphics Layer Drawing。
- 如果要在 Mac OS X 中进行打印，可以将内容发送到由打印框架管理的 PostScript 图形上下文。有关详细信息，请参阅 Obtaining a Graphics Context for Printing。

## Quartz 2D 不透明数据类型

除了图形上下文之外，Quartz 2D API 还定义了各种不透明数据类型。由于 API 是 Core Graphics 框架的一部分，因此数据类型和对其进行操作的例程使用 CG 前缀。

Quartz 2D 根据应用程序操作的不透明数据类型创建对象，以实现特定的绘图输出。图1-3显示了将绘图操作应用于 Quartz 2D 提供的三个对象时可以实现的各种结果。例如：

- 你可以通过创建 PDF 页面对象，将旋转操作应用于图形上下文，并要求 Quartz 2D 将页面绘制到图形上下文来旋转和显示 PDF 页面。
- 你可以通过创建 pattern 对象，定义构成 pattern 的形状以及设置 Quartz 2D 以在绘制图形上下文时将 pattern 用作绘图来绘制图案。
- 你可以通过创建着色对象来填充具有轴向或径向着色的区域，提供确定着色中每个点的颜色的函数，然后要求 Quartz 2D 将着色用作填充颜色。

图 1-3 不透明数据类型是 Quartz 2D 中绘制图元的基础

![](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Art/drawing_primitives.gif)

Quartz 2D 中提供的不透明数据类型包括以下内容：

- CGPathRef, 用于矢量图形以创建你填充或描边的路径。 见 Paths。
- CGImageRef, 用于根据你提供的样本数据表示位图图像和位图图像蒙版。请参见 Bitmap Images and Image Masks。
- CGLayerRef, 用于表示可用于重复绘图（例如背景或图案）和屏幕外绘图的绘图层。请参阅 Core Graphics Layer Drawing。
- CGPatternRef, 用于重复绘图。见 Patterns。
- CGShadingRef 和 CGGradientRef, 用于绘制渐变。请参阅 Gradients。
- CGFunctionRef, 用于定义带有任意数量浮点参数的回调函数。在为着色创建渐变时使用此数据类型。请参阅 Gradients。
- CGColorRef 和 CGColorSpaceRef, 用于告知 Quartz 如何解释颜色。请参阅 Color and Color Spaces。
- CGImageSourceRef 和 CGImageDestinationRef, 用于将数据移入和移出 Quartz。请参阅 Data Management in Quartz 2D and Image I/O Programming Guide。
- CGFontRef, 用于绘制文本。见 Text。
- CGPDFDictionaryRef, CGPDFObjectRef, CGPDFPageRef, CGPDFStream, CGPDFStringRef, 和 CGPDFArrayRef, 它们提供对 PDF 元数据的访问。请参阅 PDF Document Creation, Viewing, and Transforming。
- CGPDFScannerRef 和 CGPDFContentStreamRef, 用于解析 PDF 元数据。请参阅 PDF Document Parsing。
- CGPSConverterRef, 用于将 PostScript 转换为 PDF。它在 iOS 中不可用。请参阅 PostScript Conversion。

## 图形状态

Quartz 根据当前图形状态中的参数修改绘制操作的结果。图形状态包含参数，否则这些参数将作为绘图例程的参数。绘制到图形上下文的例程会查询图形状态以确定如何呈现其结果。例如，当你调用函数来设置填充颜色时，你正在修改存储在当前图形状态中的值。当前图形状态的其他常用元素包括线宽，当前位置和文本字体大小。

图形上下文包含一个图形状态栈。当 Quartz 创建图形上下文时，栈为空。保存图形状态时，Quartz 会将当前图形状态的副本推送到栈中。当你恢复图形状态时，Quartz 会将图形状态从栈顶部弹出。弹出状态变为当前图形状态。

要保存当前图形状态，请使用函数 CGContextSaveGState 将当前图形状态的副本推送到栈。要恢复以前保存的图形状态，请使用函数 CGContextRestoreGState 将当前图形状态替换为栈顶部的图形状态。

请注意，并非当前绘图环境的所有方面都是图形状态的元素。例如，当前路径不被视为图形状态的一部分，因此在调用函数 CGContextSaveGState 时不会保存。表1-1列出了调用此函数时保存的图形状态参数。

表 1-1 与图形状态关联的参数
![](https://github.com/yangxiaoju/Blogs/blob/master/iOS/UI/Quartz%202D%20Programming%20Guide/Quartz%202D%20%E6%A6%82%E8%BF%B0/Table%201-1.png?raw=true)

## Quartz 2D 坐标系

坐标系统（如图1-4所示）定义了用于表示要在页面上绘制的对象的位置和大小的位置范围。你可以在用户空间坐标系中指定图形的位置和大小，或者更简单地说，指定用户空间。坐标定义为浮点值。

图 1-4 Quartz 坐标系

![](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Art/quartz_coordinates.gif)

由于不同的设备具有不同的底层成像功能，因此必须以与设备无关的方式定义图形的位置和大小。例如，屏幕显示设备可能能够显示每英寸不超过96个像素，而打印机可能能够显示每英寸300个像素。如果在设备级别定义坐标系（在此示例中为96像素或300像素），则在该空间中绘制的对象无法在其他设备上重现，而不会出现可见的失真。它们看起来太大或太小。

Quartz 通过单独的坐标系统 - 用户空间 - 使用当前变换矩阵或 CTM 将其映射到输出设备 - 设备空间的坐标系来实现设备独立性。矩阵是用于有效地描述一组相关方程的数学构造。当前变换矩阵是称为仿射变换的特定类型的矩阵，其通过应用平移，旋转和缩放操作（移动，旋转和调整坐标系的大小的计算）将点从一个坐标空间映射到另一个坐标空间。

当前转换矩阵具有次要目的：它允许你转换对象的绘制方式。例如，要绘制旋转45度的框，可以在绘制框之前旋转页面的坐标系（CTM）。Quartz 使用旋转坐标系绘制到输出设备。

用户空间中的点由坐标对（x，y）表示，其中 x 表示沿水平轴（左和右）的位置，y 表示垂直轴（向上和向下）。用户坐标空间的原点是点（0,0）。原点位于页面的左下角，如图1-4所示。在 Quartz 的默认坐标系中，x 轴从页面的左侧向右侧移动时增加。当 y 轴从页面的底部向顶部移动时，y 轴的值增加。

一些技术使用与 Quartz 使用的默认坐标系不同的默认坐标系来设置其图形上下文。相对于 Quartz，这样的坐标系是一个修改过的坐标系，必须在执行某些 Quartz 绘图操作时进行补偿。最常见的修改坐标系将原点放置在上下文的左上角，并将 y 轴更改为指向页面底部。你可能会看到使用此特定坐标系的几个地方如下：

- 在 Mac OS X 中，NSView 的子类重写其 isFlipped 方法以返回 YES。
- 在 iOS 中，由 UIView 返回的绘图上下文。
- 在 iOS 中，通过调用 UIGraphicsBeginImageContextWithOptions 函数创建的绘图上下文。

UIKit 返回带有修改坐标系的 Quartz 绘图上下文的原因是 UIKit 使用不同的默认坐标约定; 它将变换应用于它创建的 Quartz 上下文，以便它们匹配其约定。如果你的应用程序想要使用相同的绘图例程来绘制 UIView 对象和 PDF 图形上下文（由 Quartz 创建并使用默认坐标系），则需要应用变换以便 PDF 图形上下文接收相同的修改坐标系。要执行此操作，请应用将原点转换为 PDF 上下文左上角的变换，并将 y 坐标缩放-1。

使用缩放变换来否定 y 坐标会改变 Quartz 绘图中的一些约定。例如，如果调用 CGContextDrawImage 将图像绘制到上下文中，则在将图像绘制到目标时，图像将被变换修改。类似地，路径绘制例程接受指定是否在默认坐标系中以顺时针或逆时针方向绘制弧的参数。如果修改了坐标系，则也会修改结果，就像图像在镜像中反射一样。在图1-5中，将相同的参数传递给 Quartz 会导致默认坐标系中的顺时针圆弧和 y 坐标被变换抵消后的逆时针圆弧。

图 1-5 修改坐标系会创建镜像图像。

![](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Art/flipped_coordinates.jpg)

你的应用程序可以调整它对已应用转换的上下文进行的任何 Quartz 调用。例如，如果要将图像或 PDF 正确绘制到图形上下文中，则应用程序可能需要临时调整图形上下文的 CTM。在 iOS 中，如果使用 UIImage 对象来包装你创建的 CGImage 对象，则无需修改 CTM。UIImage 对象自动补偿 UIKit 应用的修改坐标系。

> 重要提示：上述讨论对于了解你是否计划在 iOS 上编写直接针对 Quartz 的应用程序至关重要，但这还不够。在 iOS 3.2 及更高版本中，当 UIKit 为你的应用程序创建绘图上下文时，它还会对上下文进行其他更改以匹配默认的 UIKIt 约定。特别是，不受 CTM 影响的图案和阴影会单独调整，以使其惯例与 UIKit 的坐标系相匹配。在这种情况下，没有与 CTM 等效的机制，你的应用程序可以使用它来更改 Quartz 创建的上下文以匹配 UIKit 提供的上下文的行为; 你的应用程序必须识别它正在绘制的上下文类型并调整其行为以匹配上下文的期望。

## 内存管理：对象所有权

Quartz 使用 Core Foundation 内存管理模型，其中对象被引用计数。创建时，Core Foundation 对象的引用计数为1。你可以通过调用保留对象的函数来增加引用计数，并通过调用函数来减少引用计数以释放对象。当引用计数递减到0时，将释放该对象。 此模型允许对象安全地共享对其他对象的引用。

要记住一些简单的规则：

- 如果你创建或复制对象，则拥有它，因此你必须将其释放。也就是说，通常，如果从名称中使用“Create”或“Copy”字样的函数获取对象，则必须在完成对象后释放该对象。否则，会导致内存泄漏。
- 如果从名称中不包含“Create”或“Copy”字样的函数中获取对象，则不具有对该对象的引用，并且不得释放该对象。该对象将在未来的某个时刻由其所有者发布。
- 如果你没有对象并且需要保留它，则必须保留它并在完成后释放它。你可以使用特定于对象的 Quartz 2D 函数来保留和释放该对象。例如，如果你收到对 CGColorspace 对象的引用，则使用 CGColorSpaceRetain 和 CGColorSpaceRelease 函数根据需要保留和释放对象。你还可以使用 Core Foundation 函数 CFRetain 和 CFRelease，但必须注意不要将 NULL 传递给这些函数。