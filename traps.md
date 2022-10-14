## 明显条带（banding）、模糊后处理颗粒感太强

确保Editor的打包环境为安卓，**Project Settings -> Player -> Use 32-bit Display Buffer**如果未勾选，可能有该问题，但勾选后对GPU带宽有一定影响

## 条带

### 使用了lerp

计算时使用lerp，结果是线性的，但人眼感知是非线性的，可以简单地替换为smoothstep来修复

### 使用了lerp，但没有进行伽马校正

![QQ图片20221012170353](F:\QQ图片20221012170353.jpg)

![QQ图片20221012170531](F:\QQ图片20221012170531.jpg)

``` glsl
// Unity built-in shader source. Copyright (c) 2016 Unity Technologies. MIT license (see license.txt)

Shader "UI/AlphaMask"
{
 	Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        bar ("滑动条", Range(0, 1)) = 0
        fadeRange ("淡入范围", Range(0, 1)) = 0.1
    }

    SubShader
    {
        Tags
        {
            "Queue"="Transparent"
            "IgnoreProjector"="True"
            "RenderType"="Transparent"
        }

        Stencil
        {
            Ref [_Stencil]
            Comp [_StencilComp]
            Pass [_StencilOp]
            ReadMask [_StencilReadMask]
            WriteMask [_StencilWriteMask]
        }

        Cull Off
        Lighting Off
        ZWrite Off
        ZTest [unity_GUIZTestMode]
        Blend One OneMinusSrcAlpha
        ColorMask [_ColorMask]

        Pass
        {
            Name "Default"
        CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma target 2.0

            #include "UnityCG.cginc"
            #include "UnityUI.cginc"

            struct appdata_t
            {
                float4 vertex   : POSITION;
                float4 color    : COLOR;
                float2 texcoord : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct v2f
            {
                float4 vertex   : SV_POSITION;
                fixed4 color    : COLOR;
                float2 texcoord  : TEXCOORD0;
                float4 worldPosition : TEXCOORD1;
                half4  mask : TEXCOORD2;
                half4  positionOS : TEXCOORD3;
                UNITY_VERTEX_OUTPUT_STEREO
            };

            sampler2D _MainTex;
            fixed4 _Color;
            fixed4 _TextureSampleAdd;
            float4 _ClipRect;
            float4 _MainTex_ST;
            float _UIMaskSoftnessX;
            float _UIMaskSoftnessY;
            sampler2D _MaskTexture;
            float _Progress;
            float _SpeedX;
            float _Strength;
            sampler2D _NoiseTexture;

			v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }

            float4 frag (v2f i) : SV_Target
            {
                float endU = bar;
                float fadeWidth = fadeRange;
                float startU = endU - fadeWidth;
                float currentU = i.uv.x;

                float4 color = tex2D(_MainTex, i.uv);
                float t = saturate((currentU - startU) / (endU - startU));
                color.a *= lerp(color.a, 0,  pow(t,2.2));

                return color;
}
        ENDCG
        }
    }
}

```

试了下，那个不是亮线，是因为透明度只有0~255，如果你图片拉宽一些就比较明显了，每一级alpha之间都有一个明暗间隔，因为每个数条其实都是一个alpha值，比如0，1，2，就是3个块
alpha是离散的，就算是0和1还是有区别的
最后我加了伽马矫正
color.rgb = pow(color.rgb, 1.0 / 2.2);
第一个加了，第二个没加

![QQ图片20221012170945](F:\QQ图片20221012170945.jpg)

![QQ图片20221012170948](F:\QQ图片20221012170948.jpg)

player设置是在gamma空间