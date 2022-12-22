## diffuse 漫反射

### lambert 兰伯特模型

遵循兰伯特定律，反射光线的强度与表面法线和光源方向之间的夹角的余弦值成正比，即两个方向向量的点积

$$c_{diffuse} = c_{light} · m_{diffuse}·max(0,n·l)$$

漫反射率（diffuse）的颜色被称为反照率（albedo），它描述了多少红、绿、蓝色通道被漫反射（其余被吸收）

### half-lambert 半兰伯特光照模型

兰伯特模型的缺点是，由于使用了saturate或max操作防止点积小于0，没有光照的地方（点积为负数）是完全黑暗的，没有任何明暗变化，因此这些纯黑色的片元就使得这部分模型看起来像是一个平面一样，失去了细节表现

valve公司在开发《半条命》时提出了一种在原兰伯特模型的基础上进行简单修改的版本，被称为半兰伯特光照模型

和原兰伯特模型不同的是，没有直接使用saturate或max操作将点积的负值结果粗暴地变为0，而是将结果范围从$[-1,1]$映射到$[0,1]$，这样一来背光面的点积结果就不会被映射到同一个0值处，而是映射到不同的值，这样就使得背光面也会有一定的明暗变化了

半兰伯特模型没有物理依据，仅仅是一种视觉加强的技巧

$$c_{diffuse}=(c_{light}\cdot m_{diffuse}(\alpha(n\cdot l)+\beta))$$

大多数情况下，α和β的值都是0.5，即
$$c_{diffuse}=(c_{light}\cdot m_{diffuse}(0.5(n\cdot l)+0.5))$$

### gouraud-shading 高洛德着色-逐顶点着色

在每个顶点上计算光照，然后在片元内进行线性插值

对于非线性变化的光照计算，比如高光反射，会出现明显的棱角/块状光照

## Specular 镜面反射

镜面反射跟漫反射不同，它描述的是光线被物体表面反射后直接进入人眼的部分，而忽略那些看不到的部分

因此需要物体表面（vertex.position）到摄像机（cameraPosition）的视角方向

### Phong模型

使用了反射方向$r$和视角方向$v$

$$c_{specular} = (c_{light}\cdot m_{specular})max(0,v\cdot r)^{m_{gloss}}$$

其中可以通过表面法线方向$n$和**表面到光源方向$l$**计算反射方向，《Unity Shader入门精要》里也是假设$l$为表面到光源方向

$$r = 2(n\cdot l)n - l$$

如果$l$表示光源到表面的方向向量，则公式应为

$$r = l-2(n\cdot l)n $$

注意，$l$加上$l$在$n$上的投影的两倍就是反射向量，计算投影时要注意$l$和$n$的方向

``` glsl
float3 reflectDirection =normalize(reflect(-light.direction, i.normalWS));
float3 specular = _SpecularTint * light.color * pow(saturate(dot(viewDirection, reflectDirection)), _Smoothness * 100);
```

#### 推导过程

以$l$表示光源到表面的方向向量为例，**$l$在$n$上的投影向量**方向与$n$刚好是相反方向的，$r$应等于$l$加上两倍的**$l$在$n$上的投影向量的相反向量**

设$l$在$n$上的投影为$p$，$l$和$n$的夹角为$\theta$，$n$的单位向量为$\frac{n}{|n|}$，则反射向量为

$$r=l+2(-p)$$

投影公式：

$$p=\frac{n}{|n|}|l|cos\theta$$

点积公式：

$$n\cdot l = |n||l|cos\theta$$

带入即可得到$$r=l-2n * (n\cdot l) / |n|^2$$

如果$n$已经是归一化后的单位向量，则上述式子可简化为$$r=l-2n * (n\cdot l) $$

Unity的reflect函数如下：

``` glsl
float3 reflect( float3 i, float3 n)
{
  return i - 2.0 * n * dot(n,i);
}
```

推断这里要求传入的是归一化的单位向量$n$

### Blinn模型

Blinn模型中引入了一个新的矢量h
$$h = \frac{v+l}{|v+l|}$$
Blinn模型计算高光反射的公式如下
$$c_{specular} = (c_{light}\cdot m_{specular})max(0,n\cdot h)^{m_{gloss}}$$

```glsl
float3 halfVector = normalize(light.direction + viewDirection);
float3 specular = specularTint * light.color * pow(saturate(dot(i.normalWS, halfVector)), _Smoothness * 100);
```
