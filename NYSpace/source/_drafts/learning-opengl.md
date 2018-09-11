title: OpenGL Notes
date: 2018/3/21
update: 2018/5/7
comments: true
mathjax: true
tags: 
- OpenGL

categories: 
- OpenGL

---

## About OpenGL for OS X
OpenGL 是一个开放的、跨平台的拥有广泛工业支持的图形处理标准。OpenGL 通过提供一个成熟、文档完善的，支持现在和未来硬件加速的抽象概念的图形处理管道，令写实时 2D 或者 3D 的图形应用非常容易。

![](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/OpenGL-MacProgGuide/art/cpu_gpu.jpg)

## OpenGL on the Mac Platform

### OpenGL Concepts

#### OpenGL Implements a Client-Server Model
OpenGL 使用一个客户端-服务端模型，如图 1-2 所示。当你的应用调用一个 OpenGL 服务端的是时候，它跟一个 OpenGL 客户端进行通话。该客户端向一个 OpenGL 服务端传递绘画指令。客户端、服务端以及在他们之间的通讯渠道，对每一个 OpenGL 的实现而言都是特定的。例如，服务端和客户端能够在不同的计算机上，或者他们能够在同一台计算机的不同进程中。

![](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/OpenGL-MacProgGuide/art/client_server_model.jpg)

客户端-服务端模型，允许图形工作量在客户端和在服务端之间被分开。例如，所有 Macintosh 计算机搭载了专属于图形处理的硬件，使运行图形处理的并行计算最优化。图 1-3 展示了一组 CPU 和 GPU 的通用布局。通过这样的硬件配置，OpenGL 客户端在 CPU 上执行，而 OpenGL 服务端在 GPU 上执行。

![](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/OpenGL-MacProgGuide/art/macintosh_hardware_architecture.jpg)
#### OpenGL Commands Can Be Executed Asynchronously

OpenGL 客户端-服务端模型的好处是，客户端能够在命令完成执行之前将控制权返回给应用程序。一个 OpenGL 客户端可能也会缓冲或延迟 OpenGL 命令的执行。如果 OpenGL 在控制权返回给应用程序之前，要求所有命令完成，那么，CPU 或是 GPU 将要被闲置等待，为其他的提供数据，从而导致降低性能表现。

一些 OpenGL 命令含蓄或明确的要求客户端等待，直到一些或所有之前提交的命令已经完成。OpenGL 应用应当被设计去减少客户端-服务端同步的频率。详见 《OpenGL Application Design Strategies》以获得怎样设计你 OpenGL 应用的更多信息。


#### OpenGL Commands Are Executed In Order
OpenGL 确保命令按 OpenGL 收到的顺序执行。

#### OpenGL Copies Client Data at Call-Time

当一个应用调用一个 OpenGL 的函数时，OpenGL 的客户端在将控制权返回给应用程序之前，会拷贝任何一个在参数中提供的数据。例如，如果在应用的内存里存储的，一个向量数据数组的参数点，OpenGL 必须在返回之前拷贝他们。因此一个应用是自由改变他自己拥有的内存的，无需顾忌他对 OpenGL 的调用。

那些客户端拷贝的数据，在传递给服务端之前，经常会被重新格式化。向服务端拷贝、修改以及传递参数增加了调用 OpenGL 的开支。应用应该被设计为最小拷贝开支。

#### OpenGL Relies on Platform-Specific Libraries For Critical Functionality

OpenGL 提供了一套丰富的跨平台绘图命令，但并不定义它和操作系统的图形子系统交互的函数。而是，OpenGL 要求每个实现去定义一个接口用来创建渲染上下文（context），并且将它们和图形子系统相关联。一个渲染上下文保存所有在 OpenGL 状态机（OpenGL state machine）中存储的数据。允许多个上下文，允许被一个应用改变在一个机器中的状态，而没有影响其他上下文。

OpenGL 和图形处理子系统相关联，通常意味着允许 OpenGL 内容渲染到一个特定的窗口。当内容和窗口相关联时，实现都要创建允许 OpenGL 渲染和显示图片所需的任何资源。

### OpenGL in OS X

OpenGL 在 OS X 中，使用通用的 OpenGL 框架和插件驱动实现了 OpenGL 客户端-服务端模型。框架和驱动联合一起，用来实现客户端 OpenGL 端口，正如图 1-4 所示。专用的图形处理硬件支持了服务端。尽管这是通用场景，Apple 也在 CPU 上提供了全部的软件渲染实现。

![](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/OpenGL-MacProgGuide/art/cpu_gpu.jpg)

OS X 支持可以包含多个不同显示器的显示空间，每个显示器都由具有不同功能的不同显卡驱动。 另外，多个 OpenGL 渲染器可以驱动每个图形卡。 为了适应这种多功能性，OpenGL for OS X 被分割为定义良好的图层：窗口系统层，框架层和驱动程序层，如图1-5所示。 此分段允许插入到窗口系统层和框架层的接口。 插件接口在软件和硬件配置方面提供了灵活性，而不违反 OpenGL 标准。

![](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/OpenGL-MacProgGuide/art/opengl_layers.jpg)

窗口系统层是应用程序用来创建 OpenGL 渲染上下文并将它们与 OS X 窗口系统相关联的 OS X 特定层。 NSOpenGL 类和 Core OpenGL（CGL）API还为 OpenGL 在该上下文中的运行方式提供了一些额外的控制。 有关更多信息，请参阅特定于 OS X 的 OpenGL API。 最后，该图层还包含OpenGL库-GL，GLU和GLUT。 （有关详细信息，请参阅 Apple 实现的 OpenGL 库。）

常见的 OpenGL 框架层是图形硬件的软件接口。 这一层包含 Apple 实现的 OpenGL 规范。

驱动程序层包含可选的 GLD 插件接口和一个或多个 GLD 插件驱动程序，这些驱动程序可能具有不同的软件和硬件支持功能。 GLD 插件接口支持第三方插件驱动程序，允许第三方硬件供应商提供经过优化的驱动程序，以充分利用其图形硬件。

### Accessing OpenGL Within Your Application

你应用调用的编程接口最终为两类 —— 那些 Macintosh 平台特定的，和那些被 OpenGL 工作组定义的。那些 Apple 特定编程接口是 Cocoa 应用用来与 OS X 窗体系统通讯的。这些 API 并不创建 OpenGL 内容，他们管理内容，直接将其会知道一个目的地，并且控制渲染操作的多方面。你的应用调用 OpenGL API 以创建内容。OpenGL 惯例接受顶点（vertex）、像素（pixel）以及纹理（texture）数据，并且组合这些数据去创建图像。最终的图像驻留在帧缓冲区中，该帧缓冲区通过窗口系统特定的API呈现给用户。

> 帧缓冲区 framebuffer：缓冲区就是一块内存区，它用在输入输出设备和 CPU 之间，用来缓冲数据。它使得低速的输入输出设备和高速的CPU能够协调工作，避免低速的输入输出设备占用 CPU，解放出 CPU，使其能够高效率工作。

> 1、Buffer（缓冲区）是系统两端处理速度平衡（从长时间尺度上看）时使用的。它的引入是为了减小短期内突发I/O的影响，起到流量整形的作用。比如生产者——消费者问题，他们产生和消耗资源的速度大体接近，加一个buffer可以抵消掉资源刚产生/消耗时的突然变化。2、Cache（缓存）则是系统两端处理速度不匹配时的一种折衷策略。因为CPU和memory之间的速度差异越来越大，所以人们充分利用数据的局部性（locality）特征，通过使用存储系统分级（memory hierarchy）的策略来减小这种差异带来的影响。3、假定以后存储器访问变得跟CPU做计算一样快，cache就可以消失，但是buffer依然存在。比如从网络上下载东西，瞬时速率可能会有较大变化，但从长期来看却是稳定的，这样就能通过引入一个buffer使得OS接收数据的速率更稳定，进一步减少对磁盘的伤害。4、TLB（Translation Lookaside Buffer，翻译后备缓冲器）名字起错了，其实它是一个cache.

![](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/OpenGL-MacProgGuide/art/opengl_architecture.jpg)

### Terminology
There are a number of terms that you’ll want to understand so that you can write code effectively using OpenGL: renderer, renderer attributes, buffer attributes, pixel format objects, rendering contexts, drawable objects, and virtual screens. As an OpenGL programmer, some of these may seem familiar to you. However, understanding the Apple-specific nuances of these terms will help you get the most out of OpenGL on the Macintosh platform.

有一些你将想理解的术语，从而你能够有效的用 OpenGL 编写代码：渲染、渲染属性、缓冲属性、像素格式化对象、渲染上下文、可绘制对象，以及虚拟屏幕。作为一个 OpenGL 程序员，这其中的一些对你来说看似熟悉。然而，理解 Apple 特定的术语的细微差别，将帮助你在 OpenGL 之外的 Macintosh 平台上获得最多。

#### Renderer
A renderer is the combination of the hardware and software that OpenGL uses to execute OpenGL commands. The characteristics of the final image depend on the capabilities of the graphics hardware associated with the renderer and the device used to display the image. OS X supports graphics accelerator cards with varying capabilities, as well as a software renderer. It is possible for multiple renderers, each with different capabilities or features, to drive a single set of graphics hardware. To learn how to determine the exact features of a renderer, see Determining the OpenGL Capabilities Supported by the Renderer.

渲染器是 OpenGL 用来执行 OpenGL 命令的硬件和软件的组合。 最终图像的特征取决于与渲染器和用于显示图像的设备相关的图形硬件的功能。 OS X 支持具有不同功能的图形加速卡以及软件渲染器。 对于具有不同功能或特性的多个渲染器来说，可以驱动一组图形硬件。 要了解如何确定渲染器的确切功能，请参阅确定渲染器支持的 OpenGL 功能。

#### Renderer and Buffer Attributes
Your application uses renderer and buffer attributes to communicate renderer and buffer requirements to OpenGL. The Apple implementation of OpenGL dynamically selects the best renderer for the current rendering task and does so transparently to your application. If your application has very specific rendering requirements and wants to control renderer selection, it can do so by supplying the appropriate renderer attributes. Buffer attributes describe such things as color and depth buffer sizes, and whether the data is stereoscopic or monoscopic.

你应用使用渲染器和缓冲属性，来将渲染器和缓冲区要求传达给 OpenGL。Apple 的 OpenGL 实现动态为当前渲染任务选择最佳渲染器，并且这样做对你的应用是透明的。如果你的的应用有非常特定的渲染要求，并且想要控制渲染器选择，可以通过申请恰当的渲染属性能够这样做。缓冲属性描述了这样的的事物，如颜色和深度缓冲大小，以及数据是立体的还是单视平面的。

Renderer and buffer attributes are represented by constants defined in the Apple-specific OpenGL APIs. OpenGL uses the attributes you supply to perform the setup work needed prior to drawing content. Drawing to a Window or View provides a simple example that shows how to use renderer and buffer attributes. Choosing Renderer and Buffer Attributes explains how to choose renderer and buffer attributes to achieve specific rendering goals.

渲染器和缓冲属性是由在 Apple 特定 OpenGL API 中定义的常量表示的。OpenGL 使用您提供的属性来执行绘制内容之前所需的设置工作。向一个窗口绘图或视图提供一个简单的例子，展示怎样用渲染器和缓冲属性。选择渲染器和缓冲属性说明了怎样选择渲染器和缓冲属性来获得特定的渲染目标。

#### Pixel Format Objects
A pixel format describes the format for pixel data storage in memory. The description includes the number and order of components as well as their names (typically red, blue, green and alpha). It also includes other information, such as whether a pixel contains stencil and depth values. A pixel format object is an opaque data structure that holds a pixel format along with a list of renderers and display devices that satisfy the requirements specified by an application.

像素格式描述了像素数据在内存中存储的格式。描述包括组件的数量和顺序，以及它们的名字（通常是红、蓝、绿以及 Alpha 值）。它同样包括他们的信息，例如，像素是否包含图案模版和深度值。像素格式对象是一个不可见数据结构，保存了一个满足应用特定的需求的包含一个渲染器列表和显示设备像素的格式。

Each of the Apple-specific OpenGL APIs defines a pixel format data type and accessor routines that you can use to obtain the information referenced by this object. See Virtual Screens for more information on renderer and display devices.

每一个 Apple 特定的 OpenGL API 定义了一个像素格式的数据类型和访问器例程，你能够用来获得与这个对象相关的信息。详见虚拟屏幕来获得更多关于渲染和显示设备的信息。

#### OpenGL Profiles
OpenGL profiles are new in OS X 10.7. An OpenGL profile is a renderer attribute used to request a specific version of the OpenGL specification. When your application provides an OpenGL profile as part of its renderer attributes, it only receives renderers that provide the complete feature set promised by that profile. The render can implement a different version of the OpenGL so long as the version it supplies to your application provides the same functionality that your application requested.

OpenGL 配置文件在 OS X 10.7 中是新增的。OpenGL 配置文件是渲染器属性用于请求一个 OpenGL 规格的特定版本。当你的应用提供一个作为他渲染器属性一部分的 OpenGL 配置，他只接收被该配置允诺的提供了完全功能的集合。渲染可以实现不同版本的 OpenGL ，只要它提供给应用程序的版本提供了与应用程序请求相同的功能即可

#### Rendering Contexts
A rendering context, or simply context, contains OpenGL state information and objects for your application. State variables include such things as drawing color, the viewing and projection transformations, lighting characteristics, and material properties. State variables are set per context. When your application creates OpenGL objects (for example, textures), these are also associated with the rendering context.

渲染上下文，或者说简单上下文，包含 OpenGL 状态信息和用于应用对象。状态变量包括诸如绘图颜色、视图和投影转换、照明特性以及材质属性等内容。状态变量在每个上下文被设置。当你的应用创建 OpenGL 对象（如，纹理），这些都与渲染上下文相关。

Although your application can maintain more than one context, only one context can be the current context in a thread. The current context is the rendering context that receives OpenGL commands issued by your application.

尽管你的应用可以维护多个上下文，但只有一个上下文能够在一个线程中成为当前上下文。当前上下文是接收你应用发出 OpenGL 命令的渲染上下文。

#### Drawable Objects
A drawable object refers to an object allocated by the windowing system that can serve as an OpenGL framebuffer. A drawable object is the destination for OpenGL drawing operations. The behavior of drawable objects is not part of the OpenGL specification, but is defined by the OS X windowing system.

可绘制对象与窗体系统创建的能够作为 OpenGL 帧缓冲的对象。可绘图对象是 OpenGL 绘制操作的目标。可绘制对象的行为并不是 OpenGL 的规范，而是被 OS X 窗体系统定义的。

A drawable object can be any of the following: a Cocoa view, offscreen memory, a full-screen graphics device, or a pixel buffer.

可绘制对象可以是以下任何一种：Cocoa视图，离线内存，全屏图形设备或像素缓冲区。

> Note: A pixel buffer (pbuffer) is an OpenGL buffer designed for hardware-accelerated offscreen drawing and as a source for texturing. An application can render an image into a pixel buffer and then use the pixel buffer as a texture for other OpenGL commands. Although pixel buffers are supported on Apple’s implementation of OpenGL, Apple recommends you use framebuffer objects instead. See Drawing Offscreen for more information on offscreen rendering.

> 注意：

Before OpenGL can draw to a drawable object, the object must be attached to a rendering context. The characteristics of the drawable object narrow the selection of hardware and software specified by the rendering context. Apple’s OpenGL automatically allocates buffers, creates surfaces, and specifies which renderer is the current renderer.

The logical flow of data from an application through OpenGL to a drawable object is shown in Figure 1-7. The application issues OpenGL commands that are sent to the current rendering context. The current context, which contains state information, constrains how the commands are interpreted by the appropriate renderer. The renderer converts the OpenGL primitives to an image in the framebuffer. (See also Running an OpenGL Program in OS X .)

### Running an OpenGL Program in OS X
![](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/OpenGL-MacProgGuide/art/opengl_pipeline.jpg)

Per-vertex operations include such things as applying transformation matrices to add perspective or to clip, and applying lighting effects. Per-pixel operations include such things as color conversion and applying blur and distortion effects. Pixels destined for textures are sent to texture assembly, where OpenGL stores textures until it needs to apply them onto an object.

OpenGL rasterizes the processed vertex and pixel data, meaning that the data are converged to create fragments. A fragment encapsulates all the values for a pixel, including color, depth, and sometimes texture values. These values are used during antialiasing and any other calculations needed to fill shapes and to connect vertices.

Per-fragment operations include applying environment effects, depth and stencil testing, and performing other operations such as blending and dithering. Some operations—such as hidden-surface removal—end the processing of a fragment. OpenGL draws fully processed fragments into the appropriate location in the framebuffer.

The dashed arrows in Figure 1-12 indicate reading pixel data back from the framebuffer. They represent operations performed by OpenGL functions such as glReadPixels, glCopyPixels, and glCopyTexImage2D.

顶点操作包括诸如应用变换矩阵来添加透视图或剪辑以及应用光照效果等。每像素操作包括颜色转换和应用模糊和失真效果等。用于纹理的像素将被发送到纹理程序集，其中OpenGL存储纹理，直到它需要将它们应用到对象上。

OpenGL 栅格化处理过的顶点和像素数据，这意味着数据会聚以创建片段。一个片段封装了一个像素的所有值，包括颜色，深度，有时还包括纹理值。这些值在抗锯齿和任何其他需要填充形状和连接顶点的计算中使用。

每片段操作包括应用环境效果，深度和模板测试以及执行其他操作，如混合和抖动。某些操作（例如隐藏表面移除）会终止对片段的处理。 OpenGL将完全处理的片段绘制到帧缓冲区中的适当位置。

图1-12中的虚线箭头表示从帧缓冲区读回像素数据。它们表示由OpenGL函数执行的操作，例如glReadPixels，glCopyPixels和glCopyTexImage2D。



## OpenGL Pipeline
- 大家都知道程序的主函数都在CPU上执行，图形的渲染在GPU上执行，GPU亦可进行通用编程，但这样的程序也需要在CPU上执行代码来操控GPU。
![](https://images0.cnblogs.com/blog/528205/201411/231515339536381.png)
- 将各个数据传输带宽值增加一倍基本就是目前最好PC的水平，不要惊讶于显存的带宽竟然是内存的5倍以上，因为显存的位宽要大，而且显存直接焊接在显卡上不像内存条有插槽，所以频率也可以高一些，但显存的延时一般不如内存低。PCIe的带宽大约是内存速度的三分之一。这个层次上性能优化的主要思路是：减少程序对PCI传输带宽的占用，增加主程序（CPU）以及着色器（GPU）访问存储器的局部性（增加缓存命中率或更多使用寄存器）。

- GPU主要由显存（Device Memory）和SMs（流多处理器，Stream Multiprocessors）组成。

- **着色器程序在GPU上执行**，OpenGL主程序在CPU上执行，主程序（CPU）向显存输入顶点等数据，启动渲染过程，并对渲染过程进行控制。

- OpenGL Context（上下文）。在调用任何OpenGL函数之前，必须已经创建了GL Context，GL Context存储了OpenGL的状态变量以及其他渲染有关的信息。我们都知道OpenGL是个状态机，有很多状态变量，是个标准的过程式操作过程，改变状态会影响后续所有操作，这和面向对象的解耦原则不符，毕竟渲染本身就是个复杂的过程。OpenGL采用Client-Server模型来解释OpenGL程序，即Server存储GL Context（可能不止一个），Client提出渲染请求，Server给予响应，一般Server和Client都在我们的PC上，但Server和Client也可以是通过网络连接（即将上面说的PCIe总线换成了网络）。

- 双缓冲是一种常用的防止画面撕裂的技术，即调用OpenGL函数进行渲染的结果都写入“back” buffer，待所有渲染完成调用SwapBuffers函数.

- 在最简单的情况下，帧缓冲区只有一个，这时帧缓冲区的读取和刷新都都会有比较大的效率问题。为了解决效率问题，显示系统通常会引入两个缓冲区，即双缓冲机制。在这种情况下，GPU 会预先渲染好一帧放入一个缓冲区内，让视频控制器读取，当下一帧渲染好后，GPU 会直接把视频控制器的指针指向第二个缓冲器。如此一来效率会有很大的提升。

- 只考虑顶点、几何、片断着色器，管线总结为：顶点数据（Vertices） > 顶点着色器（Vertex Shader） > 图元装配（Assembly） > 几何着色器（Geometry Shader） > 光栅化（Rasterization） > 片断着色器（Fragment Shader） > 逐片断处理（Per-Fragment Operations） > 帧缓冲（FrameBuffer）。再经过双缓冲的交换（SwapBuffer），渲染内容就显示到了屏幕上。

- Rasterisation (or rasterization) is the task of taking an image described in a vector graphics format (shapes) and converting it into a raster image (pixels or dots) for output on a video display or printer, or for storage in a bitmap file format. It refers to both rasterisation of models and 2D rendering primitives such as polygons, line segments, etc.

- 光栅化有两个任务：1.确定图元包含哪些由整数坐标确定的“小方块”（和屏幕像素对应，现在还不能叫片断，光栅化完成后才能叫片断），2.确定这些小方块的Depth值和Color值（从图片顶点的Depth和Color插值得到），这些颜色后来可能被其他如纹理操作修改。

- 之所以说纹理复杂，在于纹理坐标的计算上，每个片断要找到一个纹理坐标以索引纹理像素，这个计算看似简单，但出问题时将产生意想不到的效果，如下图

- 图中每个框的内部，将分为 顶点处理、图元装配裁剪等（加“等”是包括装配后的其他操作）、光栅化、逐片断处理 四个部分，顶点处理包括固定管线的顶点坐标变换、光照（也即逐顶点光照）等；图元装配裁剪等包括图元装配、裁剪、透视除法、视口变换等；光栅化包括点线光栅化、多边形填充、纹理（Texture）、雾（Fog）等；逐片断处理包括各种测试（Scissor, Alpha, Stencil, Depth Test）、混合（Blending）等。

### 总结
今天在看管线的相关内容，OpenGL Pipeline 应该是一个处理 OpenGL 渲染数据（Frame Buffer）的工作流，也是 OpenGL 库最根本的设计思想。在这个 work flow 中，最重要的应该是顶点处理（Vertex Shader）、图元处理（Geometry Shader）、光栅化（Rasterisation）和逐片段处理（Per-Fragment Operations）。那也就是，我们为了用 OpenGL 实现所期望的的图形图像效果，必须要编写代码并按照 OpenGL 规范组装最初的数据，这些数据丢给 OpenGL，并通过 OpenGL Pipline 的处理吐出 Frame Buffer 这种可以是驱动层理解的数据，最终驱动层操控底层 Hardware 进行屏幕渲染

工程师写的程序想要调用任何 OpenGL 的函数之前，必须先创建 OpenGL Context，Context 存储 OpenGL 的状态变量和渲染有关的信息。倘若，此时创建的 Context 并非当前屏幕的 Context，那么当 CPU （OpenGL Client）传递给 GPU （OpenGL Server）渲染数据，GPU 执行渲染命令的时候，GPU 就必须从当前 Context 切换到工程师所创建的 Context 。而这个过程中， CPU （OpenGL Client）拷贝的数据，在传递给 GPU （OpenGL Server）之前，经常会被重新格式化，所以向 GPU （OpenGL Server）拷贝、修改以及传递参数增加了调用 OpenGL 的开支。这整个过程就是离平渲染，和离平渲染的弊端。

## Drawing to a Window or View

### Window and Viewport


### Hello Triangle

In OpenGL everything is in 3D space, but the screen and window are a 2D array of pixels so a large part of OpenGL's work is about transforming all 3D coordinates to 2D pixels that fit on your screen. The process of transforming 3D coordinates to 2D coordinates is managed by the graphics pipeline of OpenGL. The graphics pipeline can be divided into two large parts: the first transforms your 3D coordinates into 2D coordinates and the second part transforms the 2D coordinates into actual colored pixels. In this tutorial we'll briefly discuss the graphics pipeline and how we can use it to our advantage to create some fancy pixels.

> There is a difference between a 2D coordinate and a pixel. A 2D coordinate is a very precise representation of where a point is in 2D space, while a 2D pixel is an approximation of that point limited by the resolution of your screen/window.

The graphics pipeline takes as input a set of 3D coordinates and transforms these to colored 2D pixels on your screen. The graphics pipeline can be divided into several steps where each step requires the output of the previous step as its input. All of these steps are highly specialized (they have one specific function) and can easily be executed in parallel. **Because of their parallel nature most graphics cards of today have thousands of small processing cores to quickly process your data within the graphics pipeline by running small programs on the GPU for each step of the pipeline. These small programs are called shaders.**

**Some of these shaders are configurable by the developer which allows us to write our own shaders to replace the existing default shaders.** This gives us much more fine-grained control over specific parts of the pipeline and because they run on the GPU, they can also save us valuable CPU time. Shaders are written in the OpenGL Shading Language (GLSL) and we'll delve more into that in the next tutorial.

Below you'll find an abstract representation of all the stages of the graphics pipeline. Note that the blue sections represent sections where we can inject our own shaders.(可编程管线)

![](https://learnopengl.com/img/getting-started/pipeline.png)

As you can see the graphics pipeline contains a large number of sections that each handle one specific part of converting your vertex data to a fully rendered pixel. We will briefly explain each part of the pipeline in a simplified way to give you a good overview of how the pipeline operates.

As input to the graphics pipeline we pass in a list of three 3D coordinates that should form a triangle in **an array here called Vertex Data; this vertex data is a collection of vertices**. A vertex is basically a collection of data per 3D coordinate. **This vertex's data is represented using vertex attributes that can contain any data we'd like but for simplicity's sake let's assume that each vertex consists of just a 3D position and some color value.**

With the vertex data defined we'd like to send it as input to the first process of the graphics pipeline: the vertex shader. **This is done by creating memory on the GPU** where we store the vertex data, configure how OpenGL should interpret the memory and specify how to send the data to the graphics card. The vertex shader then processes as much vertices as we tell it to from its memory.

**We manage this memory via so called vertex buffer objects (VBO) that can store a large number of vertices in the GPU's memory.** The advantage of using those buffer objects is that we can send large batches of data all at once to the graphics card without having to send data a vertex a time. Sending data to the graphics card from the CPU is relatively slow, so wherever we can we try to send as much data as possible at once. Once the data is in the graphics card's memory the vertex shader has almost instant access to the vertices making it extremely fast


glBindBuffer(GL_ARRAY_BUFFER, VBO);  
From that point on any buffer calls we make (**on the GL_ARRAY_BUFFER target**) will be used to configure the currently bound buffer, which is VBO. Then we can make a call to glBufferData function that copies the previously defined vertex data into the buffer's memory:


#### Vertex shader
The vertex shader is one of the shaders that are **programmable** by people like us. Modern OpenGL requires that we at least set up a vertex and fragment shader if we want to do some rendering so we will briefly introduce shaders and configure two very simple shaders for drawing our first triangle. In the next tutorial we'll discuss shaders in more detail.

The first thing we need to do is write the vertex shader in the shader language GLSL (OpenGL Shading Language) and then compile this shader so we can use it in our application. Below you'll find the source code of a very basic vertex shader in GLSL:

```
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

As you can see, GLSL looks similar to C. Each shader begins with a declaration of its version. Since OpenGL 3.3 and higher the version numbers of GLSL match the version of OpenGL (GLSL version 420 corresponds to OpenGL version 4.2 for example). We also explicitly mention we're using core profile functionality.

**Next we declare all the input vertex attributes in the vertex shader with the *in* keyword.** Right now **we only care about position data so we only need a *single vertex attribute*.** GLSL has a vector datatype that contains 1 to 4 floats based on its postfix digit. Since each vertex has a 3D coordinate we create a vec3 input variable with the name aPos. **We also specifically set the location of the input variable via layout (location = 0)** and you'll later see that why we're going to need that location.


This statement appears in the vertex shader. **It declares that a vertex specific attribute which is a vector of 3 floats will be known as 'Position' in the shader.** 'Vertex specific' means that for every invocation of the shader in the GPU the value of a new vertex from the buffer will be supplied. **The first section of the statement, layout (location = 0), creates the binding between the attribute name and attribute in the buffer**. This is required for cases where our vertex contains several attributes (position, normal, texture coordinates, etc). We have to let the compiler know which attribute in the vertex in the buffer must be mapped to the declared attribute in the shader. There are two ways to do this. We can either set it explicitly as we do here (to zero). In that case we can use a hard coded value in our application (which we did with the first parameter to the call to glVertexAttributePointer). Or we can leave it out (and simply declare 'in vec3 Position' in the shader) and then query the location from the application at runtime using glGetAttribLocation. In that case we will need to supply the returned value to glVertexAttributePointer instead of using the hard coded value. We choose the simply way here but for more complex applications it better to let the compiler determine the attribute indices and query them during runtime. This makes it easier integrating shaders from multiple sources without adapting them to your buffer layout.

> Vector
> In graphics programming we use the mathematical concept of a vector quite often, since it neatly represents positions/directions in any space and has useful mathematical properties. A vector in GLSL has a maximum size of 4 and each of its values can be retrieved via vec.x, vec.y, vec.z and vec.w respectively where each of them represents a coordinate in space. Note that the vec.w component is not used as a position in space (we're dealing with 3D, not 4D) but is used for something called perspective division. We'll discuss vectors in much greater depth in a later tutorial.

**To set the output of the vertex shader we have to assign the position data to the predefined gl_Position variable which is a vec4 behind the scenes.** At the end of the main function, whatever we set gl_Position to will be used as the output of the vertex shader. Since our input is a vector of size 3 we have to cast this to a vector of size 4. We can do this by inserting the vec3 values inside the constructor of vec4 and set its w component to 1.0f (we will explain why in a later tutorial).

The current vertex shader is probably the most simple vertex shader we can imagine because we did no processing whatsoever on the input data and simply forwarded it to the shader's output. In real applications the input data is usually not already in normalized device coordinates so we first have to transform the input data to coordinates that fall within OpenGL's visible region

#### Compiling a shader
We wrote the source code for the vertex shader (stored in a C string), but in order for OpenGL to use the shader it has to dynamically compile it at **run-time** from its source code.

The first thing we need to do is create a shader object, again referenced by an ID. So we store the vertex shader as an unsigned int and create the shader with glCreateShader:

#### Fragment shader
The fragment shader is the second and final shader we're going to create for rendering a triangle. The fragment shader is all about calculating the color output of your pixels. To keep things simple the fragment shader will always output an orange-ish color.

> Colors in computer graphics are represented as an array of 4 values: the red, green, blue and alpha (opacity) component, commonly abbreviated to RGBA. When defining a color in OpenGL or GLSL we set the strength of each component to a value between 0.0 and 1.0. If, for example, we would set red to 1.0f and green to 1.0f we would get a mixture of both colors and get the color yellow. Given those 3 color components we can generate over 16 million different colors!

**The fragment shader only requires one output variable** and that is a vector of size 4 that defines the final color output that we should calculate ourselves. We can declare output values with the out keyword, that we here promptly named FragColor. Next we simply assign a vec4 to the color output as an orange color with an alpha value of 1.0 (1.0 being completely opaque).

The process for compiling a fragment shader is similar to the vertex shader, although this time we use the GL_FRAGMENT_SHADER constant as the shader type:


#### Shader program
A shader program object is the final linked version of multiple shaders combined. To use the recently compiled shaders we have to link them to a shader program object and then activate this shader program when rendering objects. The activated shader program's shaders will be used when we issue render calls.

When linking the shaders into a program it links the outputs of each shader to the inputs of the next shader. This is also where you'll get linking errors if your outputs and inputs do not match.

Creating a program object is easy:


#### Linking Vertex Attributes
The vertex shader allows us to specify any input we want in the form of vertex attributes and while this allows for great flexibility, it does mean we have to manually specify what part of our input data goes to which vertex attribute in the vertex shader. This means we have to specify how OpenGL should interpret the vertex data before rendering.

Our vertex buffer data is formatted as follows:

- The first parameter specifies which vertex attribute we want to configure. **Remember that we specified the location of the position vertex attribute in the vertex shader with layout (location = 0).** This sets the location of the vertex attribute to 0 and since we want to pass data to this vertex attribute, we pass in 0.
- The next argument specifies the size of the vertex attribute. The vertex attribute is a vec3 so it is composed of 3 values.
- The third argument specifies the type of the data which is GL_FLOAT (a vec* in GLSL consists of floating point values).
The next argument specifies if we want the data to be normalized. If we set this to GL_TRUE all the data that has a value not between 0 (or -1 for signed data) and 1 will be mapped to those values. We leave this at GL_FALSE.
- The fifth argument is known as the stride and tells us the space between consecutive vertex attribute sets. Since the next set of position data is located exactly 3 times the size of a float away we specify that value as the stride. Note that since we know that the array is tightly packed (there is no space between the next vertex attribute value) we could've also specified the stride as 0 to let OpenGL determine the stride (this only works when values are tightly packed). Whenever we have more vertex attributes we have to carefully define the spacing between each vertex attribute but we'll get to see more examples of that later on.
- The last parameter is of type void* and thus requires that weird cast. This is the offset of where the position data begins in the buffer. Since the position data is at the start of the data array this value is just 0. We will explore this parameter in more detail later on


## Reference
- [OpenGL Programming Guide for Mac](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/OpenGL-MacProgGuide/)

- [OpenGL管线（用经典管线代说着色器内部）](http://www.cnblogs.com/liangliangh/p/4116164.html)

- [Learn OpenGL](https://learnopengl.com/Getting-started/Hello-Triangle)

- [OpenGL Tutorial](http://www.opengl-tutorial.org/beginners-tutorials/tutorial-1-opening-a-window/)

- [OpenGL ES Programming Guide](https://developer.apple.com/library/content/documentation/3DDrawing/Conceptual/OpenGLES_ProgrammingGuide/)
