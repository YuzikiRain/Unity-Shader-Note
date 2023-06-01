# c#脚本

``` c#
using UnityEngine;

#if UNITY_EDITOR
[ExecuteAlways]
#endif
[RequireComponent(typeof(SpriteRenderer))]
public class SpriteSequenceAnimaton : MonoBehaviour
{
    [ColorUsageAttribute(true, true)] public Color color = Color.white;

    public int row = 0;
    public int column = 0;
    public bool isRandomStart = false;
    public bool isLoop = false;
    public float fps = 30f;
    public float delay = 0f;
    public float duration = 5f;

    [SerializeField] [HideInInspector] private SpriteRenderer _renderer;

    private int _ColorID = Shader.PropertyToID("_Color");
    private int RowID = Shader.PropertyToID("_Row");
    private int ColumnID = Shader.PropertyToID("_Column");
    private int _TimeStampStartID = Shader.PropertyToID("_TimeStampStart");
    private int _TimeStampEndID = Shader.PropertyToID("_TimeStampEnd");
    private int _IsLoopID = Shader.PropertyToID("_IsLoop");
    private int _FPSID = Shader.PropertyToID("_FPS");

    private MaterialPropertyBlock _materialPropertyBlock;

    private void OnEnable()
    {
        _materialPropertyBlock = new MaterialPropertyBlock();
        Play();
    }

    public void Play()
    {
        _materialPropertyBlock.SetColor(_ColorID, color);
        _materialPropertyBlock.SetFloat(RowID, row);
        _materialPropertyBlock.SetFloat(ColumnID, column);
        _materialPropertyBlock.SetFloat(_FPSID, fps);
        // 如果勾选随机开始isRandomStart，则增加随机开始时间戳延迟[0,一轮播放时长]
        float period = isRandomStart ? Random.Range(0f, column * row / fps) : 0f;
        _materialPropertyBlock.SetFloat(_TimeStampStartID, Time.timeSinceLevelLoad + delay + period);
        _materialPropertyBlock.SetFloat(_TimeStampEndID, Time.time + delay + duration);
        _materialPropertyBlock.SetFloat(_IsLoopID, isLoop ? 1f : 0f);
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
Shader "Particles/SequenceAnimation"
{
    Properties
    {
        [PerRendererData][MainTexture] _MainTex ("_MainTex (RGBA)", 2D) = "white" {}
        [PerRendererData][HDR]_Color ("Color", Color) = (1,1,1,1)
        [PerRendererData]_Row("行数", Float) = 0
        [PerRendererData]_Column("列数", Float) = 0
        [PerRendererData]_FPS("帧率", Float) = 0
        [PerRendererData]_TimeStampStart("开始时间戳", Float) = 0
        [PerRendererData]_TimeStampEnd("结束时间戳", Float) = 0
        [PerRendererData][Toggle]_IsLoop("是否循环播放", Float) = 0
    }

    SubShader
    {
        Tags
        {
            "RenderType" = "Transparent" "RenderPipeline" = "UniversalPipeline" "Queue" = "Transparent"
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
                half4 color : TEXCOORD1;
                UNITY_VERTEX_INPUT_INSTANCE_ID // use this to access instanced properties in the fragment shader.
            };

            sampler2D _MainTex;

            UNITY_INSTANCING_BUFFER_START(UnityPerMaterial)
            UNITY_DEFINE_INSTANCED_PROP(half4, _Color)
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

                half4 color = UNITY_ACCESS_INSTANCED_PROP(UnityPerMaterial, _Color);
                o.color = color;
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                UNITY_SETUP_INSTANCE_ID(i);
                return tex2D(_MainTex, i.uv) * i.color;
            }
            ENDCG
        }
    }
}
```

# 使用手册

## 运行时

- 添加SpriteSequenceAnimaton组件。
- 修改大小，注意不要修改Scale，而是应该SpriteRenderer里的Size。

- 为SpriteRenderer组件附加使用了"P33/Particles/SequenceAnimation"shader的材质。

- 设置SpriteSequenceAnimation组件中的属性来控制帧动画播放。

## 编辑器下预览

勾选IsLoop，取消激活SpriteSequenceAnimation组件上，再激活该组件即进行播放。（每次修改组件上的属性后，都要重新播放）

到Scene视图下，按住鼠标右键或中键，即可持续预览。

# 字段说明

| 属性          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| fps           | 帧率，单位：帧/秒。                                          |
| isLoop        | 是否循环播放。                                               |
| delay         | 延迟播放时间，单位：秒。                                     |
| duration      | 持续播放时间，单位：秒。在持续时间内会循环播放直到时间结束。 |
| isRandomStart | 开始时的帧是随机的。                                         |

# 性能测试

测试环境：

- 后处理开启bloom，其他后处理、抗锯齿等额外效果均关闭。
- 小米10（骁龙865），目标帧率45。
- shader：半透明队列。

| 组件                 | 粒子数量 |
| -------------------- | -------- |
| ParticleSystem序列帧 | 2200     |
| 高性能帧动画组件     | 9500     |
