材质包含了一些绘制属性如TintColor等，对于使用相同shader的物体，如果想要不同的表现，可以使用多个材质来设置这些绘制属性，但这样一来如果需要更多不同表现就需要更多材质，而且无法实现一些动态的根据条件设置的绘制属性。这时候应该使用`MaterialPropertyBlock`

### 使用

```csharp
var block = new MaterialPropertyBlock();
var renderer = GetComponent<Renderer>();
block.SetColor("_Color", baseColor);
renderer.SetPropertyBlock(block);
```

### 清除

```csharp
renderer.SetPropertyBlock();
```

### 优化

#### propertyID

使用propertyID而不是字符串

```csharp
int baseColorId = Shader.PropertyToID("_BaseColor");

void Start()
{
    var block = new MaterialPropertyBlock();
    var renderer = GetComponent<Renderer>();
    block.SetColor(baseColorId, baseColor);
    renderer.SetPropertyBlock(block);
}
```

#### GPU Instancing

使用MaterialPropertyBlock而不是直接修改material的属性，这样不会隐式地创建新的material实例（**但不会减少DrawCall，每个sharedMaterial一个DrawCall**）。而MaterialPropertyBlock原本就是为了GPU Instancing存在的

默认情况下，Unity 仅在每个实例化绘制调用中批处理具有不同Transform[变换](https://docs.unity3d.com/Manual/class-Transform.html)的 GameObjects实例。要为您的实例化游戏对象添加更多变化，请修改您的着色器以添加每个实例的属性，例如材质颜色。

请注意，在正常情况下（不使用实例着色器，或者`_Color`不是每个实例的属性），由于 MaterialPropertyBlock 中的值不同，绘制调用批处理会中断

为了避免该情况，可以开启GPU Instancing，将需要实例化的属性比如`_Color`添加到实例化属性中

``` glsl
UNITY_INSTANCING_BUFFER_START(Props)
    UNITY_DEFINE_INSTANCED_PROP(fixed4, _Color)
UNITY_INSTANCING_BUFFER_END(Props)
```

这样一来使用MaterialPropertyBlock可以达到修改材质属性的目的而且也不会中断合批

参考：https://docs.unity3d.com/Manual/GPUInstancing.html

### MaterialPropertyBlock不适用于SRP

使用SRP而不是built-in管线时，如果有需要修改材质属性的需求，不能用MaterialPropertyBlock，而是直接通过 `GetComponent<MeshRenderer>().material.SetFloat("_Cutoff", 0.555f);`来修改材质实例，虽然会生成多个材质实例，但是SRP Batcher会能对这些材质进行合批处理
**`MaterialPropertyBlock` + `GPU Instacning`的效率优于`SRP Batcher`**

https://www.ronja-tutorials.com/post/048-material-property-blocks/
