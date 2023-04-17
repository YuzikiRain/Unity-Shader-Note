## ReverseZ

| 平台                                              | 近平面depth | 远平面depth |
| ------------------------------------------------- | ----------- | ----------- |
| DirectX 11, DirectX 12, Metal: Reversed direction | 1           | 0           |
| Other platforms: Traditional direction            | 0           | 1           |

| 平台                          | 裁剪空间范围  |
| ----------------------------- | ------------- |
| DirectX 11, DirectX 12, Metal | $ [near,0]$   |
| 类似 Direct3D 的平台          | $ [0,far]$    |
| 类似OpenGL的平台              | $[-near,far]$ |

深度 （Z） 方向在不同的着色器平台上有所不同。

**DirectX 11， DirectX 12， Metal： 反转方向**

- 深度 （Z） 缓冲区在近平面为 1.0，在远平面处减小到 0.0。
- 裁剪空间范围为 [near，0]（表示近平面处的近平面距离，在远平面处减小到 0.0）。

**其他平台：传统方向**

- 深度 （Z） 缓冲区值在近平面为 0.0，在远平面为 1.0。
- 裁剪空间取决于特定平台：
  - 在类似 Direct3D 的平台上，范围为 [0，far]（表示近平面为 0.0，在远平面增加到远平面的远平面距离）。
  - 在类似OpenGL的平台上，范围为[-near，far]（意思是减去近平面的近平面距离，增加到远平面的远平面距离）。

请注意，反转方向深度 （Z），结合浮点数**深度缓冲区**，与传统方向相比，显著提高了深度缓冲区精度。这样做的优点是 Z 坐标的冲突更少，阴影更好，尤其是在使用小近平面和大远平面时。

因此，当您从深度 （Z） 相反的平台使用着色器时：

- 已定义`UNITY_REVERSED_Z`宏。
- `_CameraDepth`纹理纹理范围为 1（近）到 0（远）。
- 裁剪空间范围在“near”（近）到 0（远）之间。

但是，以下宏和函数会自动计算出深度 （Z） 方向上的任何差异：

- `Linear01Depth(float z)`
- `LinearEyeDepth(float z)`
- `UNITY_CALC_FOG_FACTOR(coord)`

链接：[Unity - Manual: Writing shaders for different graphics APIs (unity3d.com)](https://docs.unity3d.com/Manual/SL-PlatformDifferences.html)

# 参考连接：

- 深度详解 https://www.cyanilux.com/tutorials/depth/
- SIG http://beta.unitychina.cn/talks/Siggraph2011_SpecialEffectsWithDepth_WithNotes.pdf
- [使用深度图从场景创建剖面图 - Unity 论坛](https://forum.unity.com/threads/create-a-cutaway-from-scene-using-depth-map.685927/)
- [URP | Depth 深度 - 哔哩哔哩 (bilibili.com)](https://www.bilibili.com/read/cv15915308)