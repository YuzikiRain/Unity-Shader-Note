### Light亮度过大，烘焙得到的光照贴图出现奇怪的颜色

- Project Settings -> Player -> Other Settings -> Rendering
    - Color Space
    - Lightmap Encoding
    - 对照 [Unity Lightmap 技术信息 整理](https://zhuanlan.zhihu.com/p/371900093) ，对应的Color Space和Lightmap Encoding，是否使用了合适的编码格式
- 光照贴图 -> Inspector -> Compression：是否被压缩
- Window -> Rendering -> Lighting -> Lightmap Parameters：是否使用了合适的配置

### 法线贴图无效

- Window -> Rendering -> Lighting
    - Directional Mode：选择 Directional（使用定向光照贴图时，Unity 将创建两张贴图而不是一张贴图。第一张图像往常一样包含照明信息，称为强度图。第二张地图称为方向图。它包含大部分烘焙光来自的方向）
    - Lightmap Resolution：尽可能大一些（与法线贴图的纹素大小有关）

### 参考

- [Unity Lightmap 技术信息 整理](https://zhuanlan.zhihu.com/p/371900093)
- [Unity渐进式光照贴图烘焙详解](https://zhuanlan.zhihu.com/p/157992819)
- https://catlikecoding.com/unity/tutorials/rendering/part-16/