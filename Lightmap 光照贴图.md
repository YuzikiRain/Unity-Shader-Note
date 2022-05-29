

## 编码方案

如果需要，Unity 项目可以使用两种技术将烘焙的光强度范围编码为低动态范围纹理：

- **RGBM 编码**。RGBM 编码在 **RGB** 通道中存储颜色，在 Alpha 通道中存储乘数 （**M**）。RGBM光照贴图的范围在线性空间中从0到34.49（5^2.2），在伽马空间中从0到5。
- **双低动态范围 （dLDR） 编码**。dLDR编码在移动平台上使用，只需将[0，2]的范围映射到[0，1]。高于值 2 的烘焙光强度将被clamp。解码值的计算方法是，使用伽玛空间时将光照贴图纹理中的值乘以 2，使用线性空间时乘以 4.59482（2^2.2）。某些平台将光照贴图存储为 dLDR，因为它们的硬件压缩在使用 RGBM 时会产生外观不佳的伪影。

使用线性色彩空间时，光照贴图纹理将标记为 sRGB，最终值由**着色**（采样和解码后）将在线性色彩空间中。使用 Gamma 色彩空间时，最终值将位于 Gamma 色彩空间中。

**注**： 使用编码时，存储在光照贴图（GPU 纹理内存）中的值始终位于 Gamma 色彩空间中。

*UnityCG.cginc* 着色器包含文件中的**解码光照贴图**着色器函数处理从着色器中的光照贴图纹理中读取值后的光照贴图值的解码。

### 项目质量等级设置

Menu->ProjectSettings->Player->OtherSettings->Lightmap Encoding

以下是编码方案及其列表**纹理压缩**每个目标平台的格式：

| **Target platform**        | **Encoding**      | **Compression - size (bits per pixel)**                    |
| :------------------------- | :---------------- | :--------------------------------------------------------- |
| Standalone(PC, Mac, Linux) | RGBM / HDR        | DXT5 / BC6H - 8 bpp                                        |
| Xbox One                   | RGBM / HDR        | DXT5 / BC6H - 8 bpp                                        |
| PlayStation4               | RGBM / HDR        | DXT5 / BC6H - 8 bpp                                        |
| **WebGL**  1.0 / 2.0       | RGBM              | DXT5 - 8 bpp                                               |
| **iOS**                    | dLDR / RGBM / HDR | PVRTC RGB - 4 bpp / ETC2 RGBA - 8 bpp / RGB9E5 - 32 bpp    |
| tvOS                       | dLDR / RGBM / HDR | ASTC - 3.56 bpp / ASTC - 3.56 bpp / RGB9E5 - 32 bpp        |
| Android*                   | dLDR / RGBM / HDR | ETC1 RGB - 4 bpp / ETC2 RGBA - 8 bpp / ASTC HDR - 3.56 bpp |

### 不同编码方式的光照贴图采样处理

```glsl
half4 unity_Lightmap_HDR;

inline half3 DecodeLightmap( fixed4 color )
{
    return DecodeLightmap( color, unity_Lightmap_HDR );
}

// decodeInstructions is a internal constant value unity_Lightmap_HDR
inline half3 DecodeLightmap( fixed4 color, half4 decodeInstructions)
{
#if defined(UNITY_LIGHTMAP_DLDR_ENCODING)
    return DecodeLightmapDoubleLDR(color, decodeInstructions);
#elif defined(UNITY_LIGHTMAP_RGBM_ENCODING)
    return DecodeLightmapRGBM(color, decodeInstructions);
#else //defined(UNITY_LIGHTMAP_FULL_HDR)
    return color.rgb;
#endif
}
```

#### UNITY_LIGHTMAP_FULL_HDR

当启用标准HDR的时直接返回颜色rgb不需要做额外处理

**unity_Lightmap_HDR = (1.0, 1.0, 0.0, 0.0)**

#### UNITY_LIGHTMAP_DLDR_ENCODING

```glsl
// Decodes doubleLDR encoded lightmaps.
inline half3 DecodeLightmapDoubleLDR( fixed4 color, half4 decodeInstructions)
{
    // decodeInstructions.x contains 2.0 when gamma color space is used or pow(2.0, 2.2) = 4.59 when linear color space is used on mobile platforms
    return decodeInstructions.x * color.rgb;
}
```

**移动平台**

- 伽马空间：**unity_Lightmap_HDR = (2.0, 1.0, 0.0, 0.0)**
- 线性空间：**unity_Lightmap_HDR = (GammaToLinearSpace(2.0), 1.0, 0.0, 0.0) = (pow(2.0, 2.2), 1.0, 0.0, 0.0)**

桌面平台不使用dLDR

#### UNITY_LIGHTMAP_RGBM_ENCODING

**需要光照贴图的a分量**

```glsl
// Decodes HDR textures
// handles dLDR, RGBM formats
// Called by DecodeLightmap when UNITY_NO_RGBM is not defined.
inline half3 DecodeLightmapRGBM (half4 data, half4 decodeInstructions)
{
    // If Linear mode is not supported we can skip exponent part
    #if defined(UNITY_COLORSPACE_GAMMA)
        # if defined(UNITY_FORCE_LINEAR_READ_FOR_RGBM)
            return (decodeInstructions.x * data.a) * sqrt(data.rgb);
        # else
            return (decodeInstructions.x * data.a) * data.rgb;
        # endif
    #else
        return (decodeInstructions.x * pow(data.a, decodeInstructions.y)) * data.rgb;
    #endif
}
```

**移动平台**

- 伽马空间：**unity_Lightmap_HDR = (5.0, 1.0, 0.0, 0.0)**，y 分量不使用
- 线性空间：**unity_Lightmap_HDR = (powf(5.0, 2.2), 2.2, 0.0, 0.0)** decodeInstructions.y = 2.2 也就是当前使用的Gamma值

### 安卓Android平台下的贴图设置

| 质量等级                  | RGB Compressed ETC 4 bits                                    | RGB Compressed ETC2 4 bits                                   | RGBA Compressed ETC2 8 bits | RGB(A) Compressed ASTC 4x4 block | RGB(A) Compressed ASTC HDR 6x6 block                         |
| :------------------------ | :----------------------------------------------------------- | ------------------------------------------------------------ | --------------------------- | -------------------------------- | ------------------------------------------------------------ |
| Low  **(dLDR encoded)**   | **默认**                                                     |                                                              |                             |                                  |                                                              |
| Medium **(RGBM encoded)** | **不可用**，因为RGBM编码下走`DecodeLightmapRGBM`分支，需要alpha通道 | **不可用**，因为RGBM编码下走`DecodeLightmapRGBM`分支，需要alpha通道 | **默认**                    |                                  | 不支持                                                       |
| High **（不压缩）**       |                                                              |                                                              |                             |                                  | **默认**，但需要硬件支持 Android: requires [GL_KHR_texture_compression_astc_hdr](https://opengles.gpuinfo.org/listreports.php?extension=GL_KHR_texture_compression_astc_hdr) extension. iOS: requires A13 or later chip (2019). When not supported, the texture is decompressed to RGB9E5 format, losing the alpha channel. |

参考：

- [Unity3D ShaderLab 之 DecodeLightmap解读 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/35096536)
- [Unity Lightmap 技术信息 整理](https://zhuanlan.zhihu.com/p/371900093) 
- [Unity - Manual: Lightmaps: Technical information (unity3d.com)](https://docs.unity3d.com/Manual/Lightmaps-TechnicalInformation.html)
- [Unity - Manual: Recommended, default, and supported texture formats, by platform (unity3d.com)](https://docs.unity3d.com/Manual/class-TextureImporterOverride.html)

## Light亮度过大，烘焙得到的光照贴图出现奇怪的颜色

- Project Settings -> Player -> Other Settings -> Rendering
    - Color Space
    - Lightmap Encoding
    - 对照 [Unity Lightmap 技术信息 整理](https://zhuanlan.zhihu.com/p/371900093) ，对应的Color Space和Lightmap Encoding，是否使用了合适的编码格式
- 光照贴图 -> Inspector -> Compression：是否被压缩
- Window -> Rendering -> Lighting -> Lightmap Parameters：是否使用了合适的配置

## 烘焙光和实时光的效果看起来不一样

以默认的URP的`lit shader`为准，
如果是自定义的shader，参考`lit shader`在采样烘焙贴图后进行了哪些处理

## 法线贴图无效

- Window -> Rendering -> Lighting
    - Directional Mode：选择 Directional（使用定向光照贴图时，Unity 将创建两张贴图而不是一张贴图。第一张图像往常一样包含照明信息，称为强度图。第二张地图称为方向图。它包含大部分烘焙光来自的方向）
    - Lightmap Resolution：尽可能大一些（与法线贴图的纹素大小有关）

## 烘焙建议

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

![image-20220118150827762](https://fastly.jsdelivr.net/gh/YuzikiRain/ImageBed/img/image-20220118150827762.png)

远处(50米)的阴影 就算优化到这么高精度静态shadowmap 效果也不好吧? 每200米 4096

from 偶尔不帅

## 参考

- [Unity Lightmap 技术信息 整理](https://zhuanlan.zhihu.com/p/371900093)
- [Unity渐进式光照贴图烘焙详解](https://zhuanlan.zhihu.com/p/157992819)
- https://catlikecoding.com/unity/tutorials/rendering/part-16/