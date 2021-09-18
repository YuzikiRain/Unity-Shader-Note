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

