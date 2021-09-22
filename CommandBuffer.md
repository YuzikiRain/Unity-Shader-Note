持有存储了一系列渲染命令的List

### 使用

-   添加命令到该命令缓冲区：调用实例方法如、BeginSample等，添加对应的指令到该CommanderBuffer实例的渲染命令List中（不是立即执行）
    -   设置全局变量：如SetGlobalVectorArray等，效果等同于Shader.SetGlobalVectorArray
    -   RenderTexture：GetTemporaryRT、Blit等
-   [执行该缓冲区的所有命令](#执行时机)

### 执行时机

-   立即执行（bulit-in管线）：调用`Graphics.ExecuteCommanderBuffer(commanderBuffer)`
-   SRP：[ScriptableRenderContext](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.html) .ExecuteCommandBuffer，在调用[ScriptableRenderContext.ExecuteCommandBuffer](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.ExecuteCommandBuffer.html)期间，[ScriptableRenderContext](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.html)将 commandBuffer 参数注册到它自己要执行的内部命令列表中。这些命令（包括存储在自定义 commandBuffer 中的命令）的实际执行发生在[ScriptableRenderContext.Submit](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.Submit.html)期间。
    一些绘制命令无法通过CommanderBuffer而仅能通过专用方法发出，如`scriptableRenderContext.DrawSkybox(camera);`
-   特定时间点：
    -   Camera：public void **AddCommandBuffer**([Rendering.CameraEvent](https://docs.unity3d.com/ScriptReference/Rendering.CameraEvent.html) **evt**, [Rendering.CommandBuffer](https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.html) **buffer**);
        CameraEvent可选择任意相机渲染时机，如：`BeforeForwardOpaque`、`AfterForwardAlpha`等
    -   Light：public void **AddCommandBuffer**([Rendering.LightEvent](https://docs.unity3d.com/ScriptReference/Rendering.LightEvent.html) **evt**, [Rendering.CommandBuffer](https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.html) **buffer**, [Rendering.ShadowMapPass](https://docs.unity3d.com/ScriptReference/Rendering.ShadowMapPass.html) **shadowPassMask**);



### 参考

-   https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.html
-   https://docs.unity3d.com/Manual/srp-using-scriptable-render-context.html