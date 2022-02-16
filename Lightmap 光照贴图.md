### Light亮度过大，烘焙得到的光照贴图出现奇怪的颜色

- Project Settings -> Player -> Other Settings -> Rendering
    - Color Space
    - Lightmap Encoding
    - 对照 [Unity Lightmap 技术信息 整理](https://zhuanlan.zhihu.com/p/371900093) ，对应的Color Space和Lightmap Encoding，是否使用了合适的编码格式
- 光照贴图 -> Inspector -> Compression：是否被压缩
- Window -> Rendering -> Lighting -> Lightmap Parameters：是否使用了合适的配置
### 烘焙光和实时光的效果看起来不一样
以默认的URP的`lit shader`为准，
如果是自定义的shader，参考`lit shader`在采样烘焙贴图后进行了哪些处理

### 法线贴图无效

- Window -> Rendering -> Lighting
    - Directional Mode：选择 Directional（使用定向光照贴图时，Unity 将创建两张贴图而不是一张贴图。第一张图像往常一样包含照明信息，称为强度图。第二张地图称为方向图。它包含大部分烘焙光来自的方向）
    - Lightmap Resolution：尽可能大一些（与法线贴图的纹素大小有关）

### 烘焙建议

杭-偶尔不帅 14:10:21
@后知后觉 分 大中小 3类 烘焙方法不同

杭-偶尔不帅 14:10:40
小就直接烘焙 distanceshadowmask 模式

杭-偶尔不帅 14:11:25
中就 烘焙室外lightmap 但不烘焙shadowmask+室内烘焙体积探针

杭-偶尔不帅 14:11:39
大就 室外都不烘焙 +室内烘焙体积探针

杭-偶尔不帅 14:12:02
中和大 都采用 静态shadowmap 流式加载解决远处阴影

杭-偶尔不帅 14:12:25
因为摆放决定动态物体阴影的 lightprobe 烘焙太慢 摆放的维护太麻烦

![image-20220118150827762](https://cdn.jsdelivr.net/gh/YuzikiRain/ImageBed/img/image-20220118150827762.png)

远处(50米)的阴影 就算优化到这么高精度静态shadowmap 效果也不好吧? 每200米 4096

from 偶尔不帅

### 参考

- [Unity Lightmap 技术信息 整理](https://zhuanlan.zhihu.com/p/371900093)
- [Unity渐进式光照贴图烘焙详解](https://zhuanlan.zhihu.com/p/157992819)
- https://catlikecoding.com/unity/tutorials/rendering/part-16/