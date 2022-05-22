持有存储了一系列渲染命令的List

### 使用

-   添加命令到该命令缓冲区：调用实例方法如BeginSample等，添加对应的指令到该CommanderBuffer实例的渲染命令List中（不是立即执行）
    -   设置全局变量：如SetGlobalVectorArray等，效果等同于Shader.SetGlobalVectorArray
    -   RenderTexture：GetTemporaryRT、Blit等
-   [执行该缓冲区的所有命令](#执行时机)

### 执行时机

-   立即执行（bulit-in管线）：调用`Graphics.ExecuteCommanderBuffer(commanderBuffer)`
-   SRP：[ScriptableRenderContext](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.html) .ExecuteCommandBuffer，在调用[ScriptableRenderContext.ExecuteCommandBuffer](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.ExecuteCommandBuffer.html)期间，[ScriptableRenderContext](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.html)将 commandBuffer 参数注册到它自己要执行的内部命令列表中。这些命令（包括存储在自定义 commandBuffer 中的命令）的实际执行发生在[ScriptableRenderContext.Submit](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.Submit.html)期间。
    一些绘制命令无法通过CommanderBuffer而仅能通过专用方法发出，如`scriptableRenderContext.DrawSkybox(camera);`
-   特定时间点（bulit-in管线）：
    -   Camera：public void **AddCommandBuffer**([Rendering.CameraEvent](https://docs.unity3d.com/ScriptReference/Rendering.CameraEvent.html) **evt**, [Rendering.CommandBuffer](https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.html) **buffer**);
        CameraEvent可选择任意相机渲染时机，如：`BeforeForwardOpaque`、`AfterForwardAlpha`等
    -   Light：public void **AddCommandBuffer**([Rendering.LightEvent](https://docs.unity3d.com/ScriptReference/Rendering.LightEvent.html) **evt**, [Rendering.CommandBuffer](https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.html) **buffer**, [Rendering.ShadowMapPass](https://docs.unity3d.com/ScriptReference/Rendering.ShadowMapPass.html) **shadowPassMask**);

## 申请临时RenderTexture

``` csharp
CommandBuffer commandBuffer;
// 定义用于描述RenderTexture的RenderTextureDescriptor

// 从cameraData中取得相机渲染目标的参数
RenderTextureDescriptor cameraDescriptor = renderingData.cameraData.cameraTargetDescriptor;
// 自定义
RenderTextureDescriptor renderTextureDescriptor = new RenderTextureDescriptor();

// 申请临时RenderTexture

// 定义RenderTexture句柄
int renderTextureID = Shader.PropertyToID("RTPropertyName");
commandBuffer.GetTemporaryRT(renderTextureID, cameraDescriptor);
```

## Blit

`commandBuffer.Blit(source, destination, material)`

-   **会设置`_MainTex`作为Copy的Source**，不需要使用commandBuffer.SetGlobalTexture和Shader.PropertyToID来手动设置参数

-   material是可选的，如果没有则默认使用`BlitCopy`这个内置的shader（使用_MainTex）
    ~~而如果使用了material，则是`_SourceTex`而不是`_MainTex`~~

参考：[Unity - Scripting API: Rendering.CommandBuffer.Blit (unity3d.com)](https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.Blit.html)

### 参考

-   https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.html
-   https://docs.unity3d.com/Manual/srp-using-scriptable-render-context.html