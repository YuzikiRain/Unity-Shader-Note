## Property Inspector

``` glsl
Properties {
    _MySliderValue("This is a Slider",Range(0,10)) = 2.5
    _MyColorValue("This is a Color",Color) = (1,1,1,1)
    _My2DValue("This is a 2D",2D) = "white"{}
    _MyRectValue("This is a Rect",Rect) = "white"{}
    _MyCubeValue("This is a Cube",Cube) = ""{}
    _MyFloatValue("This is a Float",Float) = 2.5
    _MyVectorValue("This is a Vector",Vector) = (1,2,3,4)
}
```

| **Type**           | **Example syntax**                                           | **Comment**                                                  |
| :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Integer**        | `_ExampleName ("Integer display name", Integer) = 1`         | This type is backed by a real integer (unlike the legacy `Int` type described below, which is backed by a float). Use this instead of Int when you want to use an integer. |
| **Int** (legacy)   | `_ExampleName ("Int display name", Int) = 1`                 | **Note:** This legacy type is backed by a float, rather than an integer. It is supported for backwards compatibility reasons only. Use the `Integer` type instead. |
| **Float**          | `_ExampleName ("Float display name", Float) = 0.5`  `_ExampleName ("Float with range", Range(0.0, 1.0)) = 0.5` | The maximum and minimum values for the range slider are inclusive. |
| **Texture2D**      | `_ExampleName ("Texture2D display name", 2D) = "" {}`  `_ExampleName ("Texture2D display name", 2D) = "red" {}` | Put the following values in the default value string to use one of Unity’s built-in textures: “white” (RGBA: 1,1,1,1), “black” (RGBA: 0,0,0,1), “gray” (RGBA: 0.5,0.5,0.5,1), “bump” (RGBA: 0.5,0.5,1,0.5) or “red” (RGBA: 1,0,0,1).  If you leave the string empty or enter an invalid value, it defaults to “gray”.  **Note:** these default textures are not visible in the Inspector. |
| **Texture2DArray** | `_ExampleName ("Texture2DArray display name", 2DArray) = "" {}` | For more information, see [Texture arrays](https://docs.unity3d.com/Manual/class-Texture2DArray.html). |
| **Texture3D**      | `_ExampleName ("Texture3D", 3D) = "" {}`                     | The default value is a “gray” (RGBA: 0.5,0.5,0.5,1) texture. |
| \*Cubemap\** **    | `_ExampleName ("Cubemap", Cube) = "" {}`                     | The default value is a “gray” (RGBA: 0.5,0.5,0.5,1) texture. |
| **CubemapArray**   | `_ExampleName ("CubemapArray", CubeArray) = "" {}`           | See [Cubemap arrays](https://docs.unity3d.com/Manual/class-CubemapArray.html). |
| **Color**          | `_ExampleName("Example color", Color) = (.25, .5, .5, 1)`    | This maps to a float4 in your shader code.  The Material Inspector displays a color picker. If you would rather edit the values as four individual floats, use the Vector type. |
| **Vector**         | `_ExampleName ("Example vector", Vector) = (.25, .5, .5, 1)` | This maps to a float4 in your shader code.  The Material Inspector displays four individual float fields. If you would rather edit the values using a color picker, use the Color type. |

参考：[Unity - Manual: ShaderLab: defining material properties (unity3d.com)](https://docs.unity3d.com/Manual/SL-Properties.html)

## Material property attributes

| **Attribute**       | **Function**                                                 |
| :------------------ | :----------------------------------------------------------- |
| `[Gamma]`           | Indicates that a float or vector property uses sRGB values, which means that it must be converted along with other sRGB values if the color space in your project requires this. For more information, see [Properties in Shader Programs](https://docs.unity3d.com/Manual/SL-PropertiesInPrograms.html). |
| `[HDR]`             | Indicates that a texture or color property uses [high dynamic range (HDR)](https://docs.unity3d.com/Manual/HDR.html) values.  For texture properties, the Unity Editor displays a warning if an LDR texture is assigned. For color properties, the Unity Editor uses the HDR color picker to edit this value. |
| `[HideInInspector]` | Tells the Unity Editor to hide this property in the Inspector. |
| `[MainTexture]`     | Sets the main texture for a Material, which you can access using [Material.mainTexture](https://docs.unity3d.com/ScriptReference/Material-mainTexture.html).  By default, Unity considers a texture with the property name name `_MainTex` as the main texture. Use this attribute if your texture has a different property name, but you want Unity to consider it the main texture.  If you use this attribute more than once, Unity uses the first property and ignores subsequent ones.  **Note:** When you set the main texture using this attribute, the texture is not visible in the Game view when you use the texture streaming debugging view mode, or a custom debug tool. |
| `[MainColor]`       | Sets the main color for a Material, which you can access using [Material.color](https://docs.unity3d.com/ScriptReference/Material-color.html).  By default, Unity considers a color with the property name name `_Color` as the main color. Use this attribute if your color has a different property name, but you want Unity to consider it the main color. If you use this attribute more than once, Unity uses the first property and ignores subsequent ones. |
| `[NoScaleOffset]`   | Tells the Unity Editor to hide tiling and offset fields for this texture property. |
| `[Normal]`          | Indicates that a texture property expects a normal map.  The Unity Editor displays a warning if you assign an incompatible texture. |
| `[PerRendererData]` | Indicates that a texture property will be coming from per-renderer data in the form of a [MaterialPropertyBlock](https://docs.unity3d.com/ScriptReference/MaterialPropertyBlock.html).  The Material inspector shows these properties as read-only. |

## MaterialPropertyDrawer

### Toggle

```glsl
// This version specifies a keyword name.
// The material property name is not important.
// When the toggle is enabled, Unity enables a shader keyword with the name "ENABLE_EXAMPLE_FEATURE".
// When the toggle is disabled, Unity disables a shader keyword with the name "ENABLE_EXAMPLE_FEATURE".
[Toggle(ENABLE_EXAMPLE_FEATURE)] _ExampleFeatureEnabled ("Enable example feature", Float) = 0// This version does not specify a keyword name.
// The material property name determines the shader keyword it affects.
// When this toggle is enabled, Unity enables a shader keyword with the name "_ANOTHER_FEATURE_ON".
// When this toggle is disabled, Unity disables a shader keyword with the name "_ANOTHER_FEATURE_ON".
[Toggle] _Another_Feature ("Enable another feature", Float) = 0// ...Later, in the HLSL code:
#pragma multi_compile __ ENABLE_EXAMPLE_FEATURE
#pragma multi_compile __ _ANOTHER_FEATURE_ON
```

### ToggleOff

```glsl
// This version specifies a keyword name.
// The material property name is not important.
// When the toggle is enabled, Unity disables a shader keyword with the name "DISABLE_EXAMPLE_FEATURE".
// When the toggle is disabled, Unity enables a shader keyword with the name "DISABLE_EXAMPLE_FEATURE".
[ToggleOff(DISABLE_EXAMPLE_FEATURE)] _ExampleFeatureEnabled ("Enable example feature", Float) = 0// This version does not specify a keyword name.
// The material property name determines the shader keyword it affects.
// When this toggle is enabled, Unity disables a shader keyword with the name "_ANOTHER_FEATURE_OFF".
// When this toggle is disabled, Unity enables a shader keyword with the name "_ANOTHER_FEATURE_OFF".
[ToggleOff] _Another_Feature ("Enable another feature", Float) = 0// ...Later, in the HLSL code:
#pragma multi_compile __ DISABLE_EXAMPLE_FEATURE
#pragma multi_compile __ _ANOTHER_FEATURE_OFF
```

### KeywordEnum

```glsl
// Display a popup with None, Add, Multiply choices.
// Each option will set _OVERLAY_NONE, _OVERLAY_ADD, _OVERLAY_MULTIPLY shader keywords.
[KeywordEnum(None, Add, Multiply)] _Overlay ("Overlay mode", Float) = 0// ...Later, in the HLSL code:
#pragma multi_compile _OVERLAY_NONE _OVERLAY_ADD _OVERLAY_MULTIPLY
```

### Enum

```glsl
// Blend mode values
[Enum(UnityEngine.Rendering.BlendMode)] _Blend ("Blend mode", Float) = 1// A subset of blend mode values, just "One" (value 1) and "SrcAlpha" (value 5).
[Enum(One,1,SrcAlpha,5)] _Blend2 ("Blend mode subset", Float) = 1
```

### 自定义MaterialPropertyDrawer

```glsl
Shader "Custom/Example"
{
    Properties
    {
        _MainTex("Base (RGB)", 2D) = "white" {}        // Display a popup with None,Add,Multiply choices,
        // and setup corresponding shader keywords.
        [KeywordEnum(None, Add, Multiply)] _Overlay("Overlay mode", Float) = 0        _OverlayTex("Overlay", 2D) = "black" {}        // Display as a toggle.
        [Toggle] _Invert("Invert color?", Float) = 0
    }    // rest of shader code...
}
```

```cs
using UnityEngine;
using UnityEditor;
using System;// The property drawer class should be placed in an editor script, inside a folder called Editor.
// Use with "[MyToggle]" before a float shader property.public class MyToggleDrawer : MaterialPropertyDrawer
{
    // Draw the property inside the given rect
    public override void OnGUI (Rect position, MaterialProperty prop, String label, MaterialEditor editor)
    {
        // Setup
        bool value = (prop.floatValue != 0.0f);        EditorGUI.BeginChangeCheck();
        EditorGUI.showMixedValue = prop.hasMixedValue;        // Show the toggle control
        value = EditorGUI.Toggle(position, label, value);        EditorGUI.showMixedValue = false;
        if (EditorGUI.EndChangeCheck())
        {
            // Set the new value if it has changed
            prop.floatValue = value ? 1.0f : 0.0f;
        }
    }
}
```

## ShaderGUI

[Unity - Manual: ShaderLab: assigning a custom editor (unity3d.com)](https://docs.unity3d.com/Manual/SL-CustomEditor.html)

## 参考

[Unity - Scripting API: MaterialPropertyDrawer (unity3d.com)](https://docs.unity3d.com/ScriptReference/MaterialPropertyDrawer.html)