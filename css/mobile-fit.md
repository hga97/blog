<!-- 移动端适配
1px 问题
UI 图完美适配
iPhoneX 适配方案
横屏适配
高清屏图片模糊问题

概念：像素、分辨率、PPI、DPI、DP、DIP、DPR、视口 -->

<!-- 设备的分辨率不一样，独立像素也不一样。使用px为单位时，不同设备尺寸不一致 -->
<!-- 使用vw尺寸，可以在设备上等比例还原设计图 -->

### 英寸

一般用英寸描述屏幕的物理大小，如电脑显示器的 17、22，手机显示器的 4.8、5.7 等使用的单位都是英寸。

需要注意，上面的尺寸都是屏幕对角线的长度：

英寸(inch,缩写为 in)在荷兰语中的本意是大拇指，一英寸就是指甲底部普通人拇指的宽度。

英寸和厘米的换算：1 英寸 = 2.54 厘米

### 分辨率

#### 像素

像素即一个小方块，它具有特定的位置和颜色

图片、电子屏幕（手机、电脑）就是由无数个具有特定颜色和特定位置的小方块拼接而成。

像素可以作为图片或电子屏幕的最小组成单位。

通常我们所说的分辨率有两种，屏幕分辨率和图像分辨率。

#### 屏幕分辨率

屏幕分辨率指一个屏幕具体由多少个像素点组成。

iPhone 14 pro 的分辨率为 2556 x 1179。这表示手机分别在垂直和水平上所具有的像素点数。

当然分辨率高不代表屏幕就清晰，屏幕的清晰程度还与尺寸有关。

#### 图像分辨率

我们通常说的图片分辨率其实是指图片含有的像素数，比如一张图片的分辨率为 800 x 400。这表示图片分别在垂直和水平上所具有的像素点数为 800 和 400。

同一尺寸的图片，分辨率越高，图片越清晰

#### PPI

PPI(Pixel Per Inch)：每英寸包括的像素数。  
PPI 可以用于描述屏幕的清晰度以及一张图片的质量。  
使用 PPI 描述图片时，PPI 越高，图片质量越高，使用 PPI 描述屏幕时，PPI 越高，屏幕越清晰。

#### DPI

DPI(Dot Per Inch)：即每英寸包括的点数。

这里的点是一个抽象的单位，它可以是屏幕像素点、图片像素点也可以是打印机的墨点。

平时你可能会看到使用 DPI 来描述图片和屏幕，这时的 DPI 应该和 PPI 是等价的，DPI 最常用的是用于描述打印机，表示打印机每英寸可以打印的点数。

### 设备独立像素

设备独立像素（device independent pixels）是操作系统定义的一种像素单位，应用程序将设备独立像素告诉操作系统，操作系统再将设备独立像素转化为设备像素，从而控制屏幕上真正的物理像素点。

为什么需要在应用程序与设备像素之间定义这么一种单位呢？为什么应用程序不应该直接使用设备像素？
例如原先在 1280×720 设备分辨率的显示屏中，显示高度为 12 个设备像素的字体，现在放到设备分辨率为 2560 ×1440 的显示屏中，如果要想得到原先的大小，则需要 24 个设备像素，如果应用程序直接使用设备像素，那么编写应用程序则将变得非常困难，需要编写应用程序逻辑：字体在一些屏幕上高度为 12 个设备像素，在另一些屏幕上却需要 24 个设备像素。

因此操作系统定义了一个单位：设备独立像素。操作系统保证：用设备独立像素定义的尺寸，不管屏幕的参数如何，都能以合适的大小显示（这也是设备独立像素名字的由来）。操作系统是如何做到的呢？对于那些像素密度高的屏幕，将多个设备像素划分为一个逻辑像素。至于将多少设备像素划分为一个逻辑像素，这由操作系统决定。

打开 chrome 的开发者工具，我们可以模拟各个手机型号的显示情况，每种型号上面会显示一个尺寸，比如 iPhone X 显示的尺寸是 375x812，实际 iPhone X 的分辨率会比这高很多，这里显示的就是设备独立像素。

### 视口

#### 布局视口

布局视口(layout viewport)：当我们以百分比来指定一个元素的大小时，它的计算值是由这个元素的包含块计算而来的。当这个元素是最顶级的元素时，它就是基于布局视口来计算的。

所以，布局视口是网页布局的基准窗口，在 PC 浏览器上，布局视口就等于当前浏览器的窗口大小（不包括 borders 、margins、滚动条）。

在移动端，布局视口被赋予一个默认值，大部分为 980px，这保证 PC 的网页可以在手机浏览器上呈现，但是非常小，用户可以手动对网页进行放大。

我们可以通过调用 document.documentElement.clientWidth / clientHeight 来获取布局视口大小。

#### 视觉视口

视觉视口(visual viewport)：用户通过屏幕真实看到的区域。

视觉视口默认等于当前浏览器的窗口大小（包括滚动条宽度）。

当用户对浏览器进行缩放时，不会改变布局视口的大小，所以页面布局是不变的，但是缩放会改变视觉视口的大小。

例如：用户将浏览器窗口放大了 200%，这时浏览器窗口中的 CSS 像素会随着视觉视口的放大而放大，这时一个 CSS 像素会跨越更多的物理像素。

所以，布局视口会限制你的 CSS 布局而视觉视口决定用户具体能看到什么。

我们可以通过调用 window.innerWidth / innerHeight 来获取视觉视口大小。

#### 理想视口

布局视口在移动端展示的效果并不是一个理想的效果，所以理想视口(ideal viewport)就诞生了：网站页面在移动端展示的理想大小。

如上图，我们在描述设备独立像素时曾使用过这张图，在浏览器调试移动端时页面上给定的像素大小就是理想视口大小，它的单位正是设备独立像素。

上面在介绍 CSS 像素时曾经提到页面的缩放系数 = CSS 像素 / 设备独立像素，实际上说页面的缩放系数 = 理想视口宽度 / 视觉视口宽度更为准确。

所以，当页面缩放比例为 100%时，CSS 像素 = 设备独立像素，理想视口 = 视觉视口。

我们可以通过调用 screen.width / height 来获取理想视口大小。

#### Meta viewport

<meta> 元素表示那些不能由其它HTML元相关元素之一表示的任何元数据信息，它可以告诉浏览器如何解析页面。

我们可以借助<meta>元素的 viewport 来帮助我们设置视口、缩放等，从而让移动端得到更好的展示效果。

```html
<meta
  name="viewport"
  content="width=device-width; initial-scale=1; maximum-scale=1; minimum-scale=1; user-scalable=no;"
/>
```

#### 移动端适配

为了在移动端让页面获得更好的显示效果，我们必须让布局视口、视觉视口都尽可能等于理想视口。

device-width 就等于理想视口的宽度，所以设置 width=device-width 就相当于让布局视口等于理想视口。

由于 initial-scale = 理想视口宽度 / 视觉视口宽度，所以我们设置 initial-scale=1;就相当于让视觉视口等于理想视口。

这时，1 个 CSS 像素就等于 1 个设备独立像素，而且我们也是基于理想视口来进行布局的，所以呈现出来的页面布局在各种设备上都能大致相似。

### 移动端设配方案

#### vh、vw 方案

vh、vw 方案即将视觉视口宽度 window.innerWidth 和视觉视口高度 window.innerHeight 等分为 100 份。

如果视觉视口为 375px，那么 1vw = 3.75px，这时 UI 给定一个元素的宽为 75px（设备独立像素），我们只需要将它设置为 75 / 3.75 = 20vw。

当然，没有一种方案是十全十美的，vw 同样有一定的缺陷：

**px 转换成 vw 不一定能完全整除，因此有一定的像素差。**

比如当容器使用 vw，margin 采用 px 时，很容易造成整体宽度超过 100vw，从而影响布局效果。当然我们也是可以避免的，例如使用 padding 代替 margin，结合 calc()函数使用等等...