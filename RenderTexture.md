## RenderTexture

渲染纹理是可以渲染到的纹理。

它们可用于实现基于图像的渲染效果、动态阴影、投影仪、反射或监控摄像机。

渲染纹理的一个典型用法是将它们设置为 Camera （[Camera.targetTexture](https://docs.unity3d.com/ScriptReference/Camera-targetTexture.html)） 的“目标纹理”属性，这将使相机渲染成纹理，而不是渲染到屏幕上。

请记住，渲染纹理内容可能会在某些事件中“丢失”，例如加载新关卡，系统进入屏幕保护程序模式，进入和退出全屏等。发生这种情况时，您现有的渲染纹理将再次变为“尚未创建”，您可以使用[IsCreated函数进行](https://docs.unity3d.com/ScriptReference/RenderTexture.IsCreated.html)检查。

与其他“本机引擎对象”类型一样，重要的是要注意任何渲染纹理的生存期，并在将它们与 Release 函数一起使用完毕后[释放](https://docs.unity3d.com/ScriptReference/RenderTexture.Release.html)它们，因为它们不会像普通托管类型那样被垃圾回收。

渲染纹理仅在 GPU 上具有数据表示形式，您需要使用 [Texture2D.ReadPixel](https://docs.unity3d.com/ScriptReference/Texture2D.ReadPixels.html) 将其内容传输到 CPU 内存。

新创建的渲染纹理的初始内容未定义。在某些平台/ API上，内容将默认为黑色，但用户不应依赖此。

参考：[Unity - Scripting API： RenderTexture (unity3d.com)](https://docs.unity3d.com/ScriptReference/RenderTexture.html)

## RenderTextureDescriptor

此结构包含创建呈现纹理所需的所有信息。可以复制、缓存和重用它，以便轻松创建所有共享相同属性的呈现纹理。避免使用默认构造函数，因为它不会使用建议的值初始化某些标志。

参考：[Unity - Scripting API： RenderTextureDescriptor (unity3d.com)](https://docs.unity3d.com/ScriptReference/RenderTextureDescriptor.html)

## RenderTextureFormat

请注意，当前平台或 GPU 可能不支持特定的渲染纹理格式。使用 [SystemInfo.SupportsRenderTextureFormat](https://docs.unity3d.com/ScriptReference/SystemInfo.SupportsRenderTextureFormat.html) 在使用前进行检查。

参考：[Unity - Scripting API: RenderTextureFormat (unity3d.com)](https://docs.unity3d.com/ScriptReference/RenderTextureFormat.html)

## FrameBuffer



参考：[Unity3d中渲染到RenderTexture的原理，几种方式以及一些问题_leonwei的博客-CSDN博客_unity的rendertexture](https://blog.csdn.net/leonwei/article/details/54972653)