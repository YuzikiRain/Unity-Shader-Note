Unity在加载纹理时直接从内存上传到显存，而不会等到纹理被相机看见时再上传。MipMap所有level都会被上传。

参考连接：

- [Unity中纹理与Mipmap的进一步研究 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/550818548)
- [Optimizing Graphics in Unity - Unity Learn](https://learn.unity.com/tutorial/optimizing-graphics-in-unity#5c7f8528edbc2a002053b5ad)

# MipMap Streaming

Mipmap 本身是会多消耗 1/3 的内存的（多了低级别的 mipmap 图），不过我们是可以决定纹理 Upload 给 GPU 的最高 mipmap level。我们通过引擎动态控制纹理的最高 mipmap level，反而可以有效的控制纹理的内存用量，这就是 Unity 引擎的 Texture Streaming 机制。**基于 Texture Streaming，纹理的内存总量是固定的，把不重要的纹理换出成高 level 的 mipmap 就可以减少纹理的内存占用。**当然如果重新切换到 mipmap0，可能会有纹理加载的过程，不过这个是引擎内部实现的，上层开发者是无感知的。我们看到很多 3D 游戏图片会有从模糊到清晰的过程，有可能就是 Texture Streaming 在起作用。

参考连接：

- [GPU 渲染管线和硬件架构浅谈 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2016951)