渲染大量的物体，会产生大量draw call（主要是每个的SetShaderPass的开销）如果这些物体的mesh和material相同，则应该考虑使用GPU Instancing

GPU Instancing通过一次为具有相同网格的多个对象发出单个绘制调用来工作。CPU 收集所有每个对象的Transform（默认就支持）和材质属性（需要在shader中编写代码以支持），并将它们放入发送到 GPU 的数组中。然后 GPU 遍历所有条目并按照提供的顺序呈现它们。此外只要该数组没有变化（Transform、材质属性发生变化），那么只需要创建一次该数组的开销，如果发生变化，则每次变化都有一次开销（一般情况下是正优化）。

### 如何开启

使用相同的mesh和material，对于

-   一般物体：需要material使用支持了GPU Instancing的shader（如Standard shader），或者自定义支持GPU Instancing的shader
-   动态绘制物体的mesh：使用调用[Graphics.DrawMeshInstanced](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstanced.html)和[Graphics.DrawMeshInstancedIndirect](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html)从脚本执行 GPU 实例化

#### 支持实例化的属性

-   物体的Transform属性：默认支持
-   Material属性（Properties暴露出来的变量）：需要在shader中编写代码以支持，**shader中不仅要声明这些变量，还要声明对应的Property**，然后在Mono脚本中通过MaterialPropertyBlock组件来修改想要支持实例化的Material属性。

#### 限制

支持MeshRenderer、SpriteRenderer等。

#### 自定义支持GPU Instancing的shader

##### Built-in管线

```glsl
Shader "SimplestInstancedShader"
{
    Properties
    {
        _Color ("Color", Color) = (1, 1, 1, 1)
    }

    SubShader
    {
        Tags { "RenderType"="Opaque" }

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_instancing
            // include以下文件，它定义了之后用到的宏替换
            // built-in管线
            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                // 会被宏替换为uint instanceID; 或 uint instanceID : SV_InstanceID;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct v2f
            {
                float4 vertex : SV_POSITION;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            UNITY_INSTANCING_BUFFER_START(InstanceProperties)
                UNITY_DEFINE_INSTANCED_PROP(float4, _Color)
            UNITY_INSTANCING_BUFFER_END(InstanceProperties)

            v2f vert(appdata v)
            {
                v2f o;
                UNITY_SETUP_INSTANCE_ID(v);
                UNITY_TRANSFER_INSTANCE_ID(v, o);
                o.vertex = UnityObjectToClipPos(v.vertex);
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                UNITY_SETUP_INSTANCE_ID(i);
				
                float4 color = UNITY_ACCESS_INSTANCED_PROP(InstanceProperties, _Color);
                return color;
            }
            ENDCG
        }
    }
}
```

##### URP管线

``` glsl
// 此着色器将在网格上绘制纹理。
Shader "Example/ParticleGPUInstancing"
{
    Properties
    {
        [MainTexture] _MainTex("Base Map", 2D) = "white"
    }

    SubShader
    {
        Tags
        {
            "RenderType" = "Transparent" "RenderPipeline" = "UniversalPipeline"
        }

        Blend SrcAlpha OneMinusSrcAlpha
        ZWrite Off

        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_instancing

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/UnityInstancing.hlsl"

            struct Attributes
            {
                float4 positionOS : POSITION;
                // uv 变量包含给定顶点的纹理上的
                // UV 坐标。
                float2 uv : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct Varyings
            {
                float4 positionHCS : SV_POSITION;
                // uv 变量包含给定顶点的纹理上的
                // UV 坐标。
                float2 uv : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };
                
UNITY_INSTANCING_BUFFER_START(MyProps)
                UNITY_DEFINE_INSTANCED_PROP(float4, _Color)
            UNITY_INSTANCING_BUFFER_END(MyProps)

            // 此宏将 _MainTex 声明为 Texture2D 对象。
            TEXTURE2D(_MainTex);
            // This macro declares the sampler for the _MainTex texture.
            SAMPLER(sampler_MainTex);

            float4 _MainTex_ST;

            Varyings vert(Attributes IN)
            {
                Varyings OUT;
                UNITY_SETUP_INSTANCE_ID(IN);
                UNITY_TRANSFER_INSTANCE_ID(IN, OUT);
                OUT.positionHCS = TransformObjectToHClip(IN.positionOS.xyz);
                // TRANSFORM_TEX 宏执行平铺和偏移
                // 变换。
                OUT.uv = TRANSFORM_TEX(IN.uv, _MainTex);
                return OUT;
            }

            half4 frag(Varyings IN) : SV_Target
            {
                UNITY_SETUP_INSTANCE_ID(IN);
                // SAMPLE_TEXTURE2D 宏使用给定的采样器对纹理进行
                // 采样。
                half4 color = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, IN.uv);
                return color;
            }
            ENDHLSL
        }
    }
}
```

### 一些宏的说明

| 添加                                                         | 简介                                                         | 备注                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| `#pragma multi_compile_instancing`                           | 使用它来指示 Unity 生成实例化变体。                          |                                                              |
| `#include "UnityCG.cginc"`                                   | 用于built-in管线，声明了GPU Instancing相关的宏               | 其中包含了`#include "UnityInstancing.cginc"`                 |
| `#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/UnityInstancing.hlsl"` | 用于URP或自定义SRP，声明了GPU Instancing相关的宏             |                                                              |
| `UNITY_VERTEX_INPUT_INSTANCE_ID`                             | 声明instanceID变量                                           | 会被宏替换为`uint instanceID`; 或` uint instanceID : SV_InstanceID;` |
| `UNITY_INSTANCING_BUFFER_START(bufferName)` / `UNITY_INSTANCING_BUFFER_END(arrayName)` | 每个 per-instance property 都必须在一个特别命名的instancing constant buffer中定义。使用这对宏来包装每个实例之间可不同的属性，而不是直接声明这些属性。`UNITY_INSTANCING_BUFFER_END`的`arrayName`对应`UNITY_ACCESS_INSTANCED_PROP`的`arrayName` | 会被宏替换为`cbuffer bufferName {}`;                         |
| `UNITY_DEFINE_INSTANCED_PROP(type, var)`                     | 定义具有类型和名称的每个instance的 Shader 属性               | 开启了GPU Instancing时，会被宏替换为`type var;`，未开启则为`static type var;` |
| `UNITY_SETUP_INSTANCE_ID(input);`                            | 从input中提取instanceID变量，然后并将其存储在全局静态变量 unity_InstanceID 中 | 会被宏替换为` { UnitySetupInstanceID(UNITY_GET_INSTANCE_ID(input)); UnitySetupCompoundMatrices(); }` |
| `UNITY_TRANSFER_INSTANCE_ID(v, o);`                          | 使用它可以将实例 ID 从输入结构复制到顶点着色器中的输出结构。仅当您需要访问片段着色器中的每个实例数据时才需要这样做。 | `#define UNITY_TRANSFER_INSTANCE_ID(input, output)   output.instanceID = UNITY_GET_INSTANCE_ID(input)` |
| `UNITY_ACCESS_INSTANCED_PROP(arrayName, var)`                | 使用它来访问在实例化常量缓冲区中声明的每个实例的 Shader 属性。它使用实例 ID 索引到实例数据数组中。该`arrayName`宏必须在一个匹配`UNITY_INSTANCING_BUFFER_END(name)`宏。 | `#define UNITY_ACCESS_INSTANCED_PROP(arr, var)   arr##Array[unity_InstanceID].var` |

### MaterialPropertyBlocks

- 不适用MaterialPropertyBlocks时，也可以对使用了同一材质（sharedMaterial）的物体进行实例化，比如大量的物体都使用了一个相同的sharedMaterial，那么只有一个DrawCall（如果没超过限制的话）
    参考：https://programmer.help/blogs/simple-understanding-of-gpu-instancing.html
- 如果想要在使用了相同的sharedMaterial的基础上，再为每个物体应用不同的材质属性，那么只能使用MaterialPropertyBlocks

### 其他说明

-   使用多个实例属性时，您无需将所有属性都填写在`MaterialPropertyBlocks`
-   如果一个实例缺少该属性，Unity 会从引用的材质中获取默认值。如果材质没有指定属性的默认值，Unity 会将值设置为 0。不要将非实例化属性放在 中`MaterialPropertyBlock`，因为这会禁用实例化。相反，为它们创建不同的材质。
-   **应用优先级：SRP Batcher > Static Batch > GPU Instancing > Dynamic Batch**
-   不支持Skin Mesh Renderer
-   如果多通道着色器有两个以上通道，则只能实例化第一个通道。这是因为 Unity 会强制为每个对象一起渲染后面的通道，从而强制更改材质。

### 参考

-   https://docs.unity3d.com/Manual/GPUInstancing.html
