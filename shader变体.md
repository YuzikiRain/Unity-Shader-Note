## 静态分支

着色器编译器在编译时（使用预处理器常量和宏）计算条件代码，永远只会执行其中一个分支

## 动态分支

两条分支都可能执行

动态分支可能着色器在 GPU 上的运行速度会更慢，尤其是在满足以下任一条件的情况下：

- 着色器在功能较差的 GPU 上运行。
- 条件代码具有“非对称分支”，其中一个分支的代码比另一个分支长或更复杂。

一些观点：

（“if-else”语句的编译方式通常是每次都执行两个分支，然后使用条件移动指令选择正确的结果，而没有任何分支 from Michal_ (https://forum.unity.com/members/michal_.751202/) https://forum.unity.com/threads/about-branching-and-min-max.498266/）

## shader变体

使用关键字定义，Unity 将为着色器代码分支的每个可能组合（包括构建中材质未使用的组合）构建着色器变体。这意味着您可以在运行时启用和禁用关键字，但它也可能大大增加构建时间、文件大小、加载时间和内存使用量。

#### 优化

-   构建时剥离
-   将常用的shader添加到ProjectSettings->Graphics->Preloaded Shaders
    Preloaded Shaders 列表列出的是常用着色器。此处列出的着色器变体将在应用程序的整个生命周期内加载到内存中。
-   着色器变体集合
    -   对于包含大量变体的 ShaderVariantCollections 资源，这可能会占用大量内存。为避免这种情况，应以较小的粒度创建 ShaderVariantCollection 资源并从脚本进行加载。
    -   （按场景需求）手动加载
        为每个场景记录使用过的着色器变体，将它们保存到单独的 ShaderVariantCollections 资源中，并在场景启动时加载它们。

https://docs.unity3d.com/cn/2019.4/Manual/OptimizingShaderLoadTime.html

#### ShaderVariantCollection 着色器变体集合

用于预热变体

https://docs.unity3d.com/Manual/shader-variant-collections.html

## keyword 关键字

  - 使用 multi_compile 或 shader_feature 预编译指令来声明keyword，编译产生Shader的变体（variant）
  - 在脚本里用Material.EnableKeyword或Shader.EnableKeyword来控制运行时具体使用变体A还是变体B；
  - 请注意Keyword的数量和变体的数量之间的关系，并可能由此导致的性能开销，比如声明#pragma multi_compile A B和#pragma multi_compile D E 这样的两组Keyword会产生 2x2=4 个Shader变体，但若声明10组这样的keyword，则该Shader会产生1024个变体（每组keyword有2个，即2个分支，10组即产生2^10个分支）；

  ```hlsl
#pragma multi_compile A B
//OR #pragma shader_feature A B

//-----------------------A模块----------------------
#if A
  return fixed4(1,1,1,1); 
#endif 
//---------------------------------------------------

//-----------------------B模块-----------------------
#if B
  return fixed4(0,0,0,1); 
#endif
//---------------------------------------------------
  ```

  - 这个Shader会被编译成两个变体：一是只包含A模块代码的变体A；二是只包含B模块代码的变体B；
  - 指定的第一个关键字是默认生效的，即默认使用变体A；

  ##### multi_compile 和 shader_feature 的区别

  如果使用shader_feature，build时没有用到的变体会被删除，不会打出来。也就是说，在build以后环境里，运行代码Material.EnableKeyword("B")可能不起作用，因为没有Material在使用变体B，所以变体B没有被build出来，运行时也找不到变体B。
  如果想解决这个问题，可以采取以下办法中的其中一种：

  - 使用multi_complie 代替 shader_feature，multi_complie 会把所有变体build出来；
  - 把这个Shader加入“always included shaders”中 (Project Settings -> Graphic)；
  - 创造一个使用变体B的Material，强行说明变体B有用；

##### __ 双下划线

  ``` hlsl
#pragma multi_compile __ A
//OR #pragma shader_feature __ A

//-----------------------A模块----------------------
#if A
  return fixed4(1,1,1,1); 
#endif 
//---------------------------------------------------

return fixed4(0,0,0,1);
  ```

  - 此方式相比#pragma multi_compile A B 的方式，我们可以减少使用一个Keyword。
  - 此方式仍会编译成两个变体：一是不包含A模块代码的变体非A；二是包含A模块代码的变体A；
  - 默认为 __ ，即变体非A生效。

  ##### 全局关键字

  ``` 
#pragma multi_compile A B
//OR #pragma shader_feature A B
  ```

  - **全局性主要表现为可以通过```Shader.EnableKeyword```或```CommandBuffer.EnableShaderKeyword```对所有具有这种关键字的shader的材质进行调整**
  - 全局最多只能声明256个这样的Keyword；

  ##### 局部关键字

  仅在Shader内部（对应材质）起效

  ``` hlsl
#pragma multi_compile_local __ A
//OR #pragma shader_feature_local __ A

//-----------------------A模块----------------------
#if A
  return fixed4(1,1,1,1); 
#endif 
//---------------------------------------------------

return fixed4(0,0,0,1);
  ```

##### C#脚本动态设置keyword关键字

  ``` csharp
// 局部：对特定Material设置关键字
CoreUtils.SetKeyword(material, "_SPECULAR_SETUP", isSpecularWorkFlow);
public static void SetKeyword(Material material, string keyword, bool state)
{
    if (state)
    {
        material.EnableKeyword(keyword);
    }
    else
    {
        material.DisableKeyword(keyword);
    }
}
// 设置作用于全局的关键字
Shader.EnableKeyword
CommandBuffer.EnableShaderKeyword
  ```

##### 与#define 和 [Toggle]_Condition("condition", Float) = 1 的区别

  - 动态地设置关键字，相当于动态地设置了宏，即```if (condidionA)#define A```，而这个条件可以外部传入动态设置，#define最多只能做到```#if B #define C```而已，条件没法动态改变
  - 虽然属性面板上使用```[Toggle]_Specular_Workflow("Specular Workflow", Float) = 0```，就可以使用```if(_Specular)```来分支，但宏可以动态地决定哪些代码被编译，比如符合条件时才定义某些变量。if则无法做到，不论是否符合条件都必须声明某个“所有条件下都共用的变量”，再用所谓的条件去分支。况且也无法应对“条件A下变量A表示A意义，条件B下则表示B意义，代码执行逻辑不变”的情况
  - ``` #if A ```等同于```#if defined(A)```

## 着色器预热

## 关键字筛选

[Unity - Scripting API: FilterAttribute (unity3d.com)](https://docs.unity3d.com/ScriptReference/ShaderKeywordFilter.FilterAttribute.html)

## 参考

-   [Unity - Manual: Declaring and using shader keywords in HLSL (unity3d.com)](https://docs.unity3d.com/Manual/SL-MultipleProgramVariants.html)
-   [Unity - Manual: Branching in shaders (unity3d.com)](https://docs.unity3d.com/Manual/shader-branching.html)
-   [Unity - Manual: Conditionals in shaders (unity3d.com)](https://docs.unity3d.com/Manual/shader-conditionals.html)
-   [Unity - 手册：分支、变体和关键字 (unity3d.com)](https://docs.unity3d.com/Manual/shader-variants-and-keywords.html)
-   [让我们好好聊聊Unity Shader中的multi_complie 李成蹊 知乎](https://zhuanlan.zhihu.com/p/77043332)

