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