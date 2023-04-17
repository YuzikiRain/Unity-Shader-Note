# shader设置矩阵

[Unity - Scripting API： Material.SetMatrix (unity3d.com)](https://docs.unity3d.com/ScriptReference/Material.SetMatrix.html)

# 使用帧缓冲进行自定义透明度混合

## Using Shader framebuffer fetch

Some GPUs (most notably PowerVR-based ones on iOS) allow you to do a form of programmable blending by providing current fragment color as input to the Fragment Shader (see `EXT_shader_framebuffer_fetch` on [khronos.org](https://www.khronos.org/registry/gles/extensions/EXT/EXT_shader_framebuffer_fetch.txt)).

It is possible to write Shaders in Unity that use the framebuffer fetch functionality. To do this, use the `inout` color argument when you write a Fragment Shader in either HLSL (Microsoft’s shading language - see [msdn.microsoft.com](http://msdn.microsoft.com/)) or Cg (the shading language by Nvidia - see [nvidia.co.uk](http://www.nvidia.co.uk/)).

The example below is in Cg.

```cg
CGPROGRAM
// only compile Shader for platforms that can potentially
// do it (currently gles,gles3,metal)
#pragma only_renderers framebufferfetch

void frag (v2f i, inout half4 ocol : SV_Target)
{
    // ocol can be read (current framebuffer color)
    // and written into (will change color to that one)
    // ...
}   
ENDCG
```

[Unity - Manual: Writing shaders for different graphics APIs (unity3d.com)](https://docs.unity3d.com/Manual/SL-PlatformDifferences.html)

