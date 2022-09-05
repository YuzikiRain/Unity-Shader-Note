## 黑边

![image-20220531221945164](https://fastly.jsdelivr.net/gh/YuzikiRain/ImageBed/img/202205312219679.png)

假设有2个像素，左边这个为**(0,0,0,0)完全透明的黑色**，右边这个为**(1,0,0,1)完全不透明的红色**，filter为双线性采样，采样点在2个像素正中间，那么最终采样结果为$(0,0,0,0) * 0.5 + (1,0,0,1) * 0.5 = (0.5, 0,0,0.5)$，半透明的橘黄色，这是反直觉的，完全透明的黑色像素竟然因为距离采样点比较近，为线性插值提供了一半的权重！

这是发生在为纹理采样的阶段，最终进行alpha blend时，使用的就是刚才线性插值得到的半透明橘黄色像素点$(0.5,0,0,0.5)$

我们认为更合理的是，线性插值得到像素应该还要考虑到alpha值的贡献，即刚才的完全透明的黑色应该贡献度为0

## straightAlpha 与 premultiAlpha

### premultiAlpha

将RGB值预先乘以alpha值并存储，比如原本是(1, 0, 0, 0.5)，则变为(0.5, 0, 0, 0.5)，此时无论是因为非point filter的纹理过滤方式还是mipmap，都不会出现上述问题，

alpha blend时使用的混合公式为`blend One OneMinusSrcAlpha`

### straightAlpha

如果是非point filter的纹理过滤方式或启用了mipmap，则会出现黑边问题（对于半透明图像，透明到不透明的边界部分的点为$(0,0,0,0)$，一些图像软件在预览时，采样了周围的这些像素并为插值提供了一定权重）

alpha blend时使用的混合公式为`blend SrcAlpha OneMinusSrcAlpha`

可通过Bleed的方式解决

## Alpha Bleeding

如果一个像素的周围有完全不透明的像素，则将周围这些像素的RGB值修改为该非全透明像素的RGB值，这样在采样时即使是非point filter的纹理过滤方式或是mipmap，插值都不会影响这些像素

对应Unity引擎中的Texture导入设置勾选`Is Alpha Transparency`

如果是完全不透明（没有alpha通道）或者仅用单通道表示透明度，或者用图像存储数据，则不需要进行Bleed
