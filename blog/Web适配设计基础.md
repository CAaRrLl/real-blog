## 一、像素(pixel)

>像素，通常来说就是一个个有各种颜色的小方点，作为构成影像的最小单位。它即有硬件上的概念也有非硬件上的概念，比如，在css中，1px被称为1个像素，此时这一个像素并不等同与硬件上的一个像素，它们存在一定的比例关系，这个比例的多少是由设备像素比决定的(如浏览器的window.devicePixelRatio)。另一个例子是我们的笔记本电脑，其分辨率规格为1920x1080，也就是说笔记本屏幕上宽有1920个硬件像素，高有1080个硬件像素，而当我们打开浏览器并全屏时，window.screen.width和window.screen.heigth显示结果不是1920、1080。还有，比如电脑中显示某张图片为140x140，那么指的也不是硬件像素，而是参照像素。

#### 1.硬件像素/设备像素(dp)
>硬件像素是显示屏能够显示的最小的点，通常由红、绿、蓝三个子像素构成，这三个子像素组成了一个有颜色的小方点。通常来说，电子产品的分辨率规格都是硬件像素。

#### 2.非硬件像素
>当硬件像素这个概念已经深入人心时，随着时间的推移，电脑，手机最终都出现了各种各样的分辨率，所有的设计如果再基于硬件像素，那么就会有非常大的局限性，同样分辨率的图片在不同分辨率的设备上显示可能会存在较大差异。于是，参照像素出现了，它可能等于硬件像素的n倍，不同设备上这个n可能是不同的，换句话说就是以设备的尺寸作为标准来定义参照像素的。这样一来，基于参照像素的设计在不同尺寸或者不同分辨率的设备屏幕上看起来是一样。css中的像素单位就是参照像素中的一种

#### 3.像素密度(ppi)
>像素密度表示每英寸所拥有的像素(pixel)数目，数值越高代表屏幕能以更高的密度显示图像，即图像越精细。

![ppi计算公式.png](http://upload-images.jianshu.io/upload_images/8568352-b0686403dc7578b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>当我们在设计手机的图片素材时，会遇到同一张图片在不同尺寸或分辨率的手机上显示大小是不同的。这是因为同一张图片是固定的物理像素，而不同的设备长和宽上物理像素的数目不同，从而导致同张图片，在手机端的长宽上占用的比例不同

![不同ppi的手机图标显示(显示的是硬件像素).png](http://upload-images.jianshu.io/upload_images/8568352-1154cbde5528bf70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 从显示情况来看，显然是十分奇怪的，同样尺寸的手机，显示的图标大小竟然不同。一般的解决方案是，dpr大(后面会提到)的设备下对图片进行放大处理，让图片的一个像素覆盖多个物理像素。但是这样的办法不可取，图片上(jpg,png,gif)的一个最小单位成为位图像素，是不可再分割的。若想一个位图像素覆盖多个物理像素，那么就得就近取色进行填充，势必会产生模糊。所以很有必要准备多个分辨率的图片，对于ppi较小的手机使用分辨率更小的图标。

#### 4.设备独立像素(dips:device-independent pixels)
>设备独立像素实际上也就是非硬件像素的一种，是由操作系统控制的一个虚拟的像素。如在web中开发中的css像素，无论是PC端还是安卓端的，在浏览器中的得到的screen.width/height都是设备独立像素。设备独立像素与硬件像素(设备像素)存在一定比例关系，这个比例关系就是上文提到的设备像素比(devicePixelRatio)。它们的关系可以用公式表示为：

![设备像素比计算.png](http://upload-images.jianshu.io/upload_images/8568352-0e0a03825fa959d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 5.如何确定设备像素比(dpr)
> 当确定了设备像素比，就确定了设备独立像素。在pc端中操作系统和浏览器都能够改变设备像素比，只不过操作系统改变设备像素比后应用于全局，浏览器改变设备像素比应用于浏览器，无论PC端和手机端都会有一个默认的合理的设备像素比的大小，一般情况下用户不会做更改的。

1.PC端的设备像素比
> window系统都可以通过改变屏幕分辨率改变设备像素比

![win10改变设备像素比.png](http://upload-images.jianshu.io/upload_images/8568352-ff7e970ed7941649.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 在Chrome控制台中，查看到电脑当前的设备像素比是1.25，即在某个方向上1.25个设备像素用1个css像素来表示

![默认设备像素比.png](http://upload-images.jianshu.io/upload_images/8568352-b0ad1da281afa12b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 如果此时进行页面放大为125%，设备像素比就变为1.56，此时某个方向上1.56个设备像素用1个css像素来表示

![页面放大时的设备像素比.png](http://upload-images.jianshu.io/upload_images/8568352-3e0353c6658e9327.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>将页面缩小为80%时，设备像素比就变为1.0，此时某个方向上1个设备像素用1个css像素来表示

![页面缩小时的设备像素比.png](http://upload-images.jianshu.io/upload_images/8568352-48259410de9811b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.手机端的设备像素比
> iphone5的设备独立像素为320x568，设备像素为640x1136

![iphone5的设备像素比.png](http://upload-images.jianshu.io/upload_images/8568352-a4e3651577514c88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> Galaxy S5的设备独立像素为360x640，设备像素为1080x1920

![Galaxy S5.png](http://upload-images.jianshu.io/upload_images/8568352-e3934303124703df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 从上面两个设备来看，即便分辨率相差较大，但是因为设备独立像素的合理设置，渲染的内容比例确是差不多的，也就是说1个css像素宽度占这两个手机宽度的比例是差不多的。

**[各个手机的设备像素比参照](https://bjango.com/articles/min-device-pixel-ratio/)**

#### 6.三个视口(非硬件像素)
1.布局视口(documnent.documentElement.clientWidth/clientHeight)
>PC端的布局视口一般与浏览器可见窗口宽度一致，而手机端中布局视口却不是一致的，其布局视口远远大于其设备独立像素，并有可能还比设备像素大，一般在768px~1024px，这是因为PC端浏览器的页面设计宽度一般都很大，为了在手机端也能较好的显示，就将手机端的布局视口设计的较大，手机端可以通过放大缩小来观看网站的全貌。但是布局视口的大小并不是理想的手机视口大小，当网站是专门为手机设计时，手机应该用回属于自己的理想视口。
PC端浏览器放大时，css像素视觉上会变大，但是其数值却不变，所以布局视口变小，同理浏览器缩小时，布局视口变大；
手机端却不一样，浏览器的放大缩小都不会改变布局视口的宽度。

2.理想视口(window.screen.height/width)
>PC端与手机端的理想视口是固定的，无论怎么缩放大小都不会发生变化，与设备的屏幕宽度相一致。说白了就是设备独立像素。下方的代码能够将手机端的布局视口设置为理想视口，当我们专门为手机端设计页面布局时，都会用到。
``<meta name="viewport" content="width=device-width，initial-scale">``
每一个名/值对都是一个给浏览器发号命令的指令。它们被逗号分隔，共有6个
1、width:设置布局视口的宽度为特定的值
2、initial-scale:设置页面的初始缩放程度和布局视口的宽度
3、minimum-scale:设置了最小缩放程度(用户可缩小的程度)
4、maximum-scale:设置了最大缩放程度(用户可放大的程度)
5、user-scalable:是否阻止用户进行缩放
6、height:设置布局视口的高度(未被实现)

3.视觉视口(window.innerHeight/innerWidth)
>等价于用户看到的区域内的css像素的数量。PC端的视觉视口一般等于布局视口，浏览器缩放时，无论是手机还是PC端，大小都会发生变化。浏览器缩小时，用户看到的css像素多了，就是变大，反之则变小。

#### 7.css的长度px、em、rem、%
      px：绝对单位，css的最小单位
      em: 相对单位，继承父节点字体的大小，相当于“倍”，如果自身定义了font-size按自身来计算，整个页面内1em不是一个固定的值
     rem: 相对单位，继承根节点html字体大小，相当于“倍”，相当于“倍”，css3属性，整个页面1rem是一个固定的值
       %: 相对单位，不同与vw和vh(后文说)，相对与什么就具体情况而定，较复杂，若对于block的高度、宽度，一般相对于父元素

#### 8.适配众多分辨率的电脑或手机
方案1：响应式设计
>响应式设计就是PC和手机端公用一套dom结构，通过@media为主要手段，对不同分辨率(设备独立像素)的设备使用不同的css样式。

方案2：rem自适应设计
>不使用@media，且以比例布局为主。只要确定了某台设备的页面布局，以这台设备作为标准，其他不同分辨率、不同大小但是同比例的设备以此设备的大小做缩放。如阿里团队开源的一个库：[flexible.js](https://github.com/amfe/lib-flexible)。原理是通过js控制html的根字体的大小，在dom元素的宽高单位(除文本用px)都使用rem，并动态地改变设备像素比和字体大小来实现不同设备的统一。

下面是rem自适应的一个例子
比如我们的设计稿宽是640px(css)，以宽度为640px的设备为基准，其html字体大小为64px，1rem=64px，此时在该设备设计一块大小为160px的div，160px=2.5rem。所以通过改变html字体的大小，让其他设备同样使用2.5rem

| 设备宽度  | html字体大小 | html字体实际输出 | 设计稿缩放大小 | 设计稿实际输出 |
|:-------------:|:-----------------:|:------------------------:|:---------------------:|:----------------------:|
| 320px | 32px | 1rem | 80px | 2.5rem |
| 360px |    36px   | 1rem | 90px | 2.5rem |
| 414px | 41.4px | 1rem | 103.5px | 2.5rem |
| 640px |  64px | 1rem | 160px | 2.5rem |

方案3：vw自适应方案
>vw、vh是基于Viewport视窗的长度单位，分别是Viewport宽和高，而Viewport是指浏览器的可视区域，即视觉视口(window.innerHeight/innerWidth)。有如下公式：
``1vw = window.innerWidth * 1%``
``1vh = window.innerHeigth * 1%``
所以根据vw就可以对不同分辨率、dpr等不同的终端设备，就能够以原生css的方式使用vw、vh来实现flexible的自适应设计。
![vw、vh、vmin和vmax.png](http://upload-images.jianshu.io/upload_images/8568352-92849342ddd5645b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


***
##### 参考文章：
* [DPI、PPI、DP、PX 的详细计算方法及算法来源是什么？](https://www.zhihu.com/question/21220154)
* [CSS布局基础之一设备像素，设备独立像素，设备像素比，css像素之间的关系](http://www.cnblogs.com/samwu/p/5341056.html)
* [设备像素，设备独立像素，CSS像素](http://yunkus.com/physical-pixel-device-independent-pixels/)
* [移动web开发之视口](https://www.cnblogs.com/xiaohuochai/p/5496995.html)
* [再聊移动端页面的适配](https://www.w3cplus.com/css/vw-for-layout.html)
* [使用Flexible实现手淘H5页面的终端适配](https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html)
* [rem自适应布局](http://caibaojian.com/flexible-js.html)
