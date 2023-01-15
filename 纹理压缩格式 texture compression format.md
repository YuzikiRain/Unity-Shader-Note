## Android

对于LDR的RGB或RGBA纹理

| 压缩格式 | 渲染后端               | 备注                                                         | 机型                                                         |
| :------- | ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ASTC     | OpenGL ES 3.1 / Vulkan | 大部分机型都支持                                             | Qualcomm GPUs since Adreno 4xx / Snapdragon 415 (2015)<br />ARM GPUs since Mali T624 (2012)<br />NVIDIA GPUs since Tegra K1 (2014)<br />PowerVR GPUs since GX6250 (2014) |
| **ETC2** | OpenGL ES 3.0 / Vulkan | **推荐**，2022年几乎所有安卓设备都支持了，因此不必再考虑ETC  | Android 4.3（API 级别 18）及更高版本<br />IOS7以上版本       |
| ETC      | OpenGL ES 2.0          | 不推荐，仅支持RGB，而A通道信息需要额外贴图来保存，因此需要特殊的shader进行支持（采样2张纹理） | Android 2.2（API 级别 8）及更高版本                          |

对于HDR纹理，ASTC HDR是唯一支持的压缩格式。

ASTC HDR requires Vulkan or [GL_KHR_texture_compression_astc_hdr](https://opengles.gpuinfo.org/listreports.php?extension=GL_KHR_texture_compression_astc_hdr) support. ASTC is the most flexible format. If a device does not support ASTC HDR the texture is decompressed at runtime to RGB9e5 or RGBA Half, depending on alpha channel usage.

## IOS tvOS

| 压缩格式   | 备注                                                         | 机型             |
| ---------- | ------------------------------------------------------------ | ---------------- |
| ASTC       | **推荐**，此格式允许您在粒度级别上在纹理质量和大小之间进行选择eight bits/pixel (4x4 block size) down to 0.89 bits/pixel (12x12 block size) | A8 芯片 （2014） |
| ETC / ETC2 | 可支持额外的Crunch压缩                                       | A7芯片（2013）   |
| PVRTC      | 可用于更旧的设备                                             | 更旧设备         |

## Desktop



参考：

- [Unity - Manual: Recommended, default, and supported texture formats, by platform (unity3d.com)](https://docs.unity3d.com/Manual/class-TextureImporterOverride.html)
- [Unity - Manual: Requirements and compatibility (unity3d.com)](https://docs.unity3d.com/Manual/android-requirements-and-compatibility.html#texture-compression)