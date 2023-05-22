# c#脚本

``` c#
using UnityEngine;

[RequireComponent(typeof(SpriteRenderer))]
public class SpriteSequenceAnimaton : MonoBehaviour
{
    public float fps = 30f;
    public bool isLoop = false;
    public float delay = 0f;
    public float duration = 5f;
    private float elapsedTime = 0f;
    private int currentFrame = 0;
    [SerializeField] [HideInInspector] private SpriteRenderer _renderer;

    private int _TimeStampStartID = Shader.PropertyToID("_TimeStampStart");
    private int _TimeStampEndID = Shader.PropertyToID("_TimeStampEnd");
    private int _IsLoopID = Shader.PropertyToID("_IsLoop");
    private int _FPSID = Shader.PropertyToID("_FPS");

    private MaterialPropertyBlock _materialPropertyBlock;

    private void Start()
    {
        _materialPropertyBlock = new MaterialPropertyBlock();
        Play();
    }

    public void Play()
    {
        _materialPropertyBlock.SetFloat(_TimeStampStartID, Time.time + delay);
        _materialPropertyBlock.SetFloat(_TimeStampEndID, Time.time + delay + duration);
        _materialPropertyBlock.SetFloat(_IsLoopID, isLoop ? 1f : 0f);
        _materialPropertyBlock.SetFloat(_FPSID, fps);
        _renderer.SetPropertyBlock(_materialPropertyBlock);
    }

#if UNITY_EDITOR

    private void OnValidate()
    {
        _renderer = GetComponent<SpriteRenderer>();
        _renderer.drawMode = SpriteDrawMode.Sliced;
    }

#endif
}
```

# shader

``` glsl
Shader "P33/Particles/SequenceAnimation"
{
    Properties
    {
        [PerRendererData] [MainTexture] _MainTex ("_MainTex (RGBA)", 2D) = "white" {}
        _Row("行数", Float) = 0
        _Column("列数", Float) = 0
        _FPS("帧率", Float) = 0
        _TimeStampStart("开始时间戳", Float) = 0
        _TimeStampEnd("结束时间戳", Float) = 0
        [Toggle]_IsLoop("是否循环播放", Float) = 0
    }

    SubShader
    {
        Tags
        {
            "RenderType"="Transparent"
        }

        Blend SrcAlpha OneMinusSrcAlpha
        ZWrite Off

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_instancing
            #include "UnityCG.cginc"

            struct appdata
            {
                half4 vertex : POSITION;
                half2 uv : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct v2f
            {
                half4 vertex : SV_POSITION;
                half2 uv : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            sampler2D _MainTex;

            UNITY_INSTANCING_BUFFER_START(UnityPerMaterial)
            UNITY_DEFINE_INSTANCED_PROP(half, _Row)
            UNITY_DEFINE_INSTANCED_PROP(half, _Column)
            UNITY_DEFINE_INSTANCED_PROP(half, _FPS)
            UNITY_DEFINE_INSTANCED_PROP(half, _TimeStampStart)
            UNITY_DEFINE_INSTANCED_PROP(half, _TimeStampEnd)
            UNITY_DEFINE_INSTANCED_PROP(half, _IsLoop)
            UNITY_INSTANCING_BUFFER_END(UnityPerMaterial)

            v2f vert(appdata v)
            {
                v2f o;

                UNITY_SETUP_INSTANCE_ID(v);
                UNITY_TRANSFER_INSTANCE_ID(v, o);

                o.vertex = UnityObjectToClipPos(v.vertex.xyz);

                half timeStampStart = UNITY_ACCESS_INSTANCED_PROP(UnityPerMaterial, _TimeStampStart);
                half elapsedTime = _Time.y - timeStampStart;
                half timeStampEnd = UNITY_ACCESS_INSTANCED_PROP(UnityPerMaterial, _TimeStampEnd);
                half isLoop = UNITY_ACCESS_INSTANCED_PROP(UnityPerMaterial, _IsLoop);
                // 超过结束时间戳，则不显示
                if (isLoop == 0 && _Time.y > timeStampEnd)
                {
                    o.vertex = half4(11, 11, 11, 1);
                    o.uv = half2(0, 0);
                    return o;
                }

                //【帧序列uv】--------------
                // o.uv0 = v.uv0 * _SequenceTex_ST.xy + _SequenceTex_ST.zw; //必须写在最前面，这样帧序列才能计算正确
                o.uv = v.uv;
                //【进行到第几帧】
                half fps = UNITY_ACCESS_INSTANCED_PROP(UnityPerMaterial, _FPS);
                half column = UNITY_ACCESS_INSTANCED_PROP(UnityPerMaterial, _Column);
                half row = UNITY_ACCESS_INSTANCED_PROP(UnityPerMaterial, _Row);
                int sequId = floor(elapsedTime * fps % (column * row)); //定义第几帧(图)
                //【定义行列步长】
                half stepU = 1.0 / column; //行步长
                half stepV = 1.0 / row; //列步长
                //【当前帧位置，第几行，第几列】
                int idV = floor(sequId / column); //第几行
                int idU = fmod(sequId, column); //第几列
                //【设定其实位置，左上角】
                o.uv *= half2(stepU, stepV); //uv缩放,当前定位：左下角
                o.uv += half2(0.0, 1 - stepV); //uv向上偏移，当前定位: 左上角
                //【uv滚动】
                o.uv += half2(stepU * idU, -stepV * idV); //uv从左上角到右下角运动
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                UNITY_SETUP_INSTANCE_ID(i);
                return tex2D(_MainTex, i.uv);
            }
            ENDCG
        }
    }
}
```

