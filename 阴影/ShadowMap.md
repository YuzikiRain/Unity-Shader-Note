## 1.使用Camera渲染表示深度的纹理

**这一步主要是以光源方向来渲染深度纹理，为了简便，创建了和光源方向相同的相机并设置为正交模式（对应平行光），使用相机的变换和视锥体来创建vp矩阵而不需要手动创建。**

创建用于显示物体的默认Camera，命名为MainCamera。

创建用于绘制ShadowMap的Camera（命名为lightCamera）和DirectionalLight，两者的位置、旋转一直保持一致。

设置lightCamera的TargetTexture为指定RenderTexture（变量名为shadowRenderTexture），设置cullingmask为需要投射阴影的物体layer。

**注意！由于我们直接将NDC坐标的z分量作为深度，近平面为1，远平面为0，因此需要设置shadowRenderTexture的颜色从默认的纯白色改为纯黑色**（表示无穷远，即还未有任何在DirectionalLight的裁剪范围内的物体投影到shadowRenderTexture上），可以将lightCamera的ClearFlags改为SolidColor，BackgroundColor改为Color.black。如果不设置为纯黑色，则该RenderTexture会表示投影的地方有一定深度（投影物体提供），其他地方有最小深度（像是已经有物体在近平面上的）

调用`camera.RenderWithShader(shader, "RenderType");`使用特定的shader来渲染相机（通过cullingmask等筛选后的）物体到**shadowRenderTexture**上。

``` c#
using UnityEngine;

public class ShdowTest : MonoBehaviour
{
    // Start is called before the first frame update
    Camera m_renderCam;
    RenderTexture rt;
    void Start()
    {
        m_renderCam = transform.Find("Camera").GetComponent<Camera>();
        m_renderCam.backgroundColor = Color.black;
        rt = new RenderTexture(512, 512, 24);
        m_renderCam.targetTexture = rt;
        m_renderCam.SetReplacementShader(Shader.Find("Hidden/ShadowMap"), "RenderType");

        Shader.SetGlobalTexture("_depthTexture", rt);
    }

    // Update is called once per frame
    void Update()
    {
        Matrix4x4 vp = GL.GetGPUProjectionMatrix(m_renderCam.projectionMatrix, false) * m_renderCam.worldToCameraMatrix;
        Matrix4x4 sm = new Matrix4x4();
        // 设置矩阵的缩放和平移，其他保持默认值0
        sm.m00 = 0.5f;
        sm.m11 = 0.5f;
        sm.m22 = 0.5f;
        sm.m33 = 1f;
        sm.m03 = 0.5f;
        sm.m13 = 0.5f;
        sm.m23 = 0.5f;

        // 把顶点从世界空间变换到光源（而不是相机的）的裁剪空间的vp矩阵
        Shader.SetGlobalMatrix("_lightVPMatrix", vp);
        // 将范围从[-1,1]映射到[0,1]的变换矩阵
        Shader.SetGlobalMatrix("_ZeroOneMatrix", sm);
    }
}

```

``` glsl
Shader "Hidden/ShadowMap"
{
    SubShader
    {
        Tags { "RenderType"="Opaque" }

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 positionOS : POSITION;
            };

            struct v2f
            {
                float4 positionCS : SV_POSITION;
            };

            v2f vert (appdata v)
            {
                v2f o;
                o.positionCS = UnityObjectToClipPos(v.positionOS);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                // SV_POSITION会自动进行透视除法，这里的positionCS.z就是深度值
                float depth = i.positionCS.z;
                // 将深度存储到4个分量中，可存储的精度更高
                fixed4 color = EncodeFloatRGBA(depth);
                return color;
            }
            ENDCG
        }
    }
}
```

## 2.比较深度

在渲染接收阴影的物体时，将（转换到相机裁剪空间下的）片元深度与深度图中的深度进行比较。

在渲染接收阴影的物体时，顶点着色器中要求返回相机对应的裁剪空间坐标（使用`SV_POSITION`），除此之外还需要返回光源对应的裁剪空间坐标，因此需要传递一个额外的插值器`positionCameraCS`。

``` glsl
struct appdata
{
    float4 positionOS : POSITION;
    float2 uv : TEXCOORD0;
};

struct v2f
{
    float4 positionCameraCS : SV_POSITION;
    float2 uv : TEXCOORD0;
    float4 positionLightCS: TEXCOORD1;
};

sampler2D _MainTex;
float4 _MainTex_ST;
sampler2D _depthTexture;
//变换到灯光摄像机的 vp矩阵 从外部传入
float4x4  _lightVPMatrix;
// 将范围从[-1,1]映射到[0,1]
float4x4  _smMatrix;

v2f vert (appdata v)
{
    v2f o;
    o.positionCameraCS = UnityObjectToClipPos(v.positionOS);
    o.uv = TRANSFORM_TEX(v.uv,_MainTex);
    //把顶点从世界空间变换到光源（而不是相机的）的裁剪空间
    o.positionLightCS = mul(_lightVPMatrix, mul(unity_ObjectToWorld,v.positionOS));
    return o;
}
```

由于光源对应的裁剪空间坐标没有使用`SV_POSITION`标记，所以不会在顶点着色器之后的光栅化阶段自动进行透视除法，所以需要手动进行。

``` glsl
fixed4 frag(v2f i) : SV_Target
{
    // 手动做透视除法
    i.positionLightCS /= i.positionLightCS.w;
    // 片元在光源空间的NDC坐标范围是[-1,1]
    float depthLightSpace = i.positionLightCS.z;
    // 需要将xy范围从[-1,1]映射到[0,1]，作为采样深度贴图的uv
    i.positionLightCS = mul(_ZeroOneMatrix, i.positionLightCS);
    // sample the texture
    fixed4 col = tex2D(_MainTex, i.uv);
    //对深度图在屏幕空间下进行采样
    fixed4 dcol = tex2D(_depthTexture, i.positionLightCS.xy);
    //把取得的值进行解码，获取深度图EncodeFloatRGBA前的值
    // shadowmap：近平面为1，远平面为0
    float depthInDepthTexture = DecodeFloatRGBA(dcol);

    //深度值小于深度图中采样的值，说明在阴影中，对颜色进行处理
    if (depthLightSpace < depthInDepthTexture)
    {
        col = float4(0.5, 0.5, 0.5, 1);
    }
    return  col;
}
```

