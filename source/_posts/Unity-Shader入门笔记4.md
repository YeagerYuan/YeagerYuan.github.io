---
title: Unity-Shader入门笔记4
date: 2025-02-05 10:51:29
tags: [unity,shader]
cover: pictures/mita01.png
​---
---

# 顶点、片元着色器

下面是一个简单的shader代码示例：

```
Shader "Unity Shader Practice/Chapter5/5-2"
{
    Properties
    {

    }
    SubShader
    {
        Pass{
            CGPROGRAM

            //pragma是预处理指令，用来向编译器传达一些特定的指令或信息
            #pragma vertex vert
        //表示顶点着色器
            #pragma fragment frag
        //表示片元着色器

           float4 vert(float4 v:POSITION) :SV_POSITION{
            return UnityObjectToClipPos(v);//将物体空间坐标转换到裁剪空间
            }

        fixed4 frag() : SV_Target{
        return fixed4(1.0,1.0,0.5,1.0);//返回一个固定颜色
        }

        ENDCG
        }
    }
    FallBack "Diffuse"
}
```

下面是显示效果：

<img src="Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B04/1738724431524.png" alt="1738724431524" style="zoom: 50%;" />

fixed4(1.0,1.0,0.5,1.0)表示的是黄色。下面简要解释一下在 Unity 中的 `fixed4`：

#### 1. 基本定义

 `fixed4` 是一种数据类型。`fixed` 一般是 11 位浮点数（范围大约是 -2.0 到 2.0，精度为 1/256） ，`fixed4` 表示包含 4 个 `fixed` 类型元素的向量。这种数据类型设计初衷是为了在资源受限的平台（如移动设备）上节省内存和计算资源，因为它比 `float4`（通常是 32 位浮点数）占用更少的空间。 

#### 2. 用途

**颜色表示**：在着色器中，颜色通常用 4 个分量（红、绿、蓝、透明度，即 RGBA）来表示。`fixed4` 非常适合用来存储和处理颜色值，例如上面的fixed4(1.0,1.0,0.5,1.0)就是黄色。



下面是一个**进阶示意**，主要定义了a2v结构体来**存放应用程序到顶点着色器的信息**以及v2f结构体来**存放顶点着色器到片元着色器的信息**。并且构造了v2f函数来可视化法线方向，并且用片元着色器函数， 通过将顶点法线转换后的颜色与 _Color 相乘，可以对最终的颜色进行调整。例如，如果将 _Color 设置为红色 (1, 0, 0, 1)，那么最终渲染的颜色将主要偏向红色，并且不同的法线方向会呈现出不同深浅的红色。 

```
// 定义着色器的名称，在 Unity 材质面板中可以通过这个名称找到该着色器
Shader "Unity Shader Practice / Chapter5 / 5Sample"
{
    // 属性块，用于定义在 Unity 材质面板中可调整的属性
    Properties
    {
        // _Color 是属性名，"Color" 是在材质面板中显示的名称，Color 表示属性类型为颜色
        // (1,1,1,1) 是该颜色属性的默认值，代表白色
        _Color("Color", Color) = (1,1,1,1)
    }
        SubShader
    {
        Pass
        {
            CGPROGRAM

            // 告诉编译器 vert 函数是顶点着色器函数
            #pragma vertex vert 
            // 告诉编译器 frag 函数是片元着色器函数
            #pragma fragment frag

            // 声明一个全局的 uniform 变量 _Color，用于接收 Properties 块中定义的颜色属性
            uniform fixed4 _Color;

    // 定义从应用程序到顶点着色器的数据结构
    struct a2v {
        // 物体空间中的顶点位置
        float4 vertex:POSITION;
        // 物体空间中的顶点法线
        float3 normal:NORMAL;
        // 第一组纹理坐标
        float4 texcoord:TEXCOORD0;
    };

    // 定义从顶点着色器到片元着色器的数据结构
    struct v2f {
        // 裁剪空间中的顶点位置
        float4 pos:SV_POSITION;
        // COLOR0 语义可以用于传递颜色信息，这里用于传递处理后的颜色
        fixed3 color : COLOR0;
    };

    // 顶点着色器函数
    v2f vert(a2v v) {
        v2f o;
        // 将物体空间的顶点坐标转换为裁剪空间的坐标
        o.pos = UnityObjectToClipPos(v.vertex);
        // 将顶点法线从 [-1, 1] 范围映射到 [0, 1] 范围，用于可视化法线方向
        o.color = v.normal * 0.5 + fixed3(0.5, 0.5, 0.5);
        return o;
    }

    // 片元着色器函数
    fixed4 frag(v2f i) :SV_Target{
        // 从 v2f 结构体中获取传递过来的颜色
        fixed3 o = i.color;
    // 将传递过来的颜色与属性 _Color 的 RGB 分量相乘
    o *= _Color.rgb;
    // 返回最终的像素颜色，透明度设置为 1.0（不透明）
    return fixed4(o, 1.0);
}
// 结束 CG 代码块
ENDCG
}
    }
        // 当当前 SubShader 无法在当前硬件上运行时，使用 "Diffuse" 着色器作为替代
    FallBack "Diffuse"
}
```

下面是显示效果：

<img src="Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B04/1738727450195.png" alt="1738727450195" style="zoom:50%;" />

# 内置文件和变量：

首先打开unity编辑器安装目录

![1738727606158](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B04/1738727606158.png)

打开 CGIncludes 文件夹 ，这里的文件主要是包含着色器代码片段的文件，这些文件对于简化着色器开发、提高代码复用性起着关键作用。 有兴趣的可以自行查阅。

<img src="Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B04/1738727723627.png" alt="1738727723627" style="zoom: 67%;" />

# Unity中提供的CG/HLSL语义

 在 Unity 中，**CG（C for Graphics）**和 **HLSL（High - Level Shading Language）**是用于编写着色器的编程语言，语义（Semantics）在其中扮演着重要角色。语义是一种标识符，用于告诉 GPU 每个变量的用途和数据流向，下面详细介绍 Unity 中常用的 CG/HLSL 语义。 

### 顶点着色器输入语义

顶点着色器的输入通常来自于模型的顶点数据，以下是常见的输入语义：

- **POSITION**：用于表示顶点在物体空间中的位置，一般是 `float3` 或 `float4` 类型。

- **NORMAL**：表示顶点的法线向量，用于光照计算等，通常是 `float3` 类型。
-  **TEXCOORDn**：用于传递纹理坐标，`n` 是纹理坐标的索引，从 0 开始。可以是 `float2`、`float3` 或 `float4` 类型，取决于纹理的维度 

# Unity 中提供的 CG/HLSL 语义介绍

在 Unity 里，CG（C for Graphics）与 HLSL（High - Level Shading Language）是用于编写着色器的编程语言，而语义（Semantics）在其中有着重要作用。语义是一种标识符，它能告知 GPU 每个变量的用途以及数据流向。下面为你详细介绍 Unity 中常用的 CG/HLSL 语义。

## 顶点着色器输入语义
顶点着色器的输入一般源于模型的顶点数据，常见的输入语义如下：

### POSITION

用于表示顶点在物体空间中的位置，通常为 `float3` 或者 `float4` 类型。示例代码如下：
```glsl
struct appdata {
    float4 vertex : POSITION;
};
```

### NORMAL

表示顶点的法线向量，可用于光照计算等，通常是 `float3` 类型。示例如下：
```glsl
struct appdata {
    float3 normal : NORMAL;
};
```

### TEXCOORDn
用于传递纹理坐标，`n` 是纹理坐标的索引，从 0 开始。其类型可以是 `float2`、`float3` 或者 `float4`，具体取决于纹理的维度。示例：
```glsl
struct appdata {
    float2 texcoord : TEXCOORD0;
};
```

### COLOR
用于传递顶点颜色，通常是 `fixed4` 类型。示例：
```glsl
struct appdata {
    fixed4 color : COLOR;
};
```

## 顶点着色器输出和片元着色器输入语义
顶点着色器的输出会作为片元着色器的输入，相关的常用语义如下：

### SV_POSITION

表示顶点在裁剪空间中的位置，顶点着色器**必须输出该值**，是 `float4` 类型。示例：
```glsl
struct v2f {
    float4 pos : SV_POSITION;
};
```

### COLOR0、COLOR1 等

用于传递颜色数据，可用于存储光照计算结果等，通常是 `fixed4` 或者 `float4` 类型。示例：
```glsl
struct v2f {
    fixed4 color : COLOR0;
};
```

### TEXCOORDn
同样用于传递纹理坐标，和顶点着色器输入的 `TEXCOORDn` 相对应，可在片元着色器中进行纹理采样。示例：
```glsl
struct v2f {
    float2 uv : TEXCOORD0;
};
```

## 片元着色器输出语义

### SV_Target
表示最终输出的颜色值，用于渲染到屏幕或者渲染目标上，通常是 `fixed4` 或者 `float4` 类型。示例：
```glsl
fixed4 frag(v2f i) : SV_Target {
    return fixed4(1.0, 0.0, 0.0, 1.0); // 输出红色
}
```

## 其他特殊语义

### SV_Depth

在某些情形下，可用于手动控制片元的深度值。若使用该语义，片元着色器需要输出一个 `float` 类型的值，代表该片元的深度。示例：
```glsl
float frag(v2f i) : SV_Depth {
    return i.depth; // 自定义深度值
}
```

## 语义的作用
- **数据传递**：借助语义，不同阶段的着色器能够明确每个变量的含义与用途，进而正确地进行数据传递和处理。
- **GPU 理解**：GPU 可利用语义来理解代码的意图，合理地分配资源并执行计算，保证着色器能够正确运行。

语义是 Unity 中 CG/HLSL 着色器编程的重要组成部分，它能让着色器代码清晰地表达数据的流向和用途，方便开发者编写高效、正确的着色器程序。 

# Shader Debug

 在 Unity 中对 Shader 进行调试是一个相对复杂但有多种方法可用的过程 。

## 使用Visual Studio

### 1. 使用断点调试（部分支持）

#### 设置断点

打开包含 Shader 代码的 `.shader` 文件，在想要调试的代码行旁边单击设置断点。例如，在片元着色器的关键计算代码行设置断点：
```glsl
fixed4 frag(v2f i) : SV_Target {
    // 假设在这里设置断点
    fixed4 color = someCalculation(i); 
    return color;
}
```

#### 附加到 Unity 进程
在 Visual Studio 中，选择 `Debug` -> `Attach to Process`。在弹出的“Attach to Process”窗口中，从进程列表中选择 Unity 编辑器进程（若在编辑器中调试）或 Unity 游戏进程（若调试打包后的游戏），然后点击 `Attach` 按钮。

#### 触发断点
在 Unity 中运行场景，当执行到设置断点的代码行时，程序会暂停，此时可查看变量的值、单步执行代码等。但需注意，Shader 代码的断点调试支持有限，部分功能可能无法正常使用。

### 2. 输出调试信息到 Visual Studio 控制台
#### 使用 `#pragma enable_d3d11_debug_symbols`（仅适用于 Windows DirectX 11）
在 Shader 代码的 `CGPROGRAM` 或 `HLSLPROGRAM` 块中添加 `#pragma enable_d3d11_debug_symbols` 指令，这样可在 Visual Studio 的输出窗口中查看一些调试信息：
```glsl
CGPROGRAM
#pragma vertex vert
#pragma fragment frag
#pragma enable_d3d11_debug_symbols

// 顶点和片元着色器代码
ENDCG
```

#### 通过颜色输出调试

在片元着色器中，将想要调试的变量值映射到颜色的某个通道，然后在 Unity 场景中观察物体颜色的变化。同时，可在 Visual Studio 中记录这些颜色变化对应的变量逻辑，辅助调试：
```glsl
fixed4 frag(v2f i) : SV_Target {
    float debugValue = someVariable;
    fixed4 debugColor = fixed4(debugValue, 0, 0, 1);
    return debugColor;
}
```

##  Unity 的 Frame Debugger

#### 使用 Frame Debugger 定位问题

在 Unity 中打开 `Window` -> `Analysis` -> `Frame Debugger`，记录一帧的渲染过程。通过 Frame Debugger 逐帧查看渲染过程中每个 Pass 的输入和输出，找出可能存在问题的 Shader 和渲染阶段。

<img src="Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B04/1738735282237.png" alt="1738735282237" style="zoom:50%;" />

![1738735381255](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B04/1738735381255.png)

# 其它问题

## float ,half与fixed

在Unity Shader及相关图形编程中，`float`、`fixed`和`half`是常用的浮点数数据类型，以下是它们的详细介绍：

### `float`

- **精度与取值范围**：`float`是32位浮点数类型，遵循IEEE 754标准。它可以表示非常大或非常小的数值，取值范围能精确到小数点后7位左右。例如，`3.1415926`可以用`float`类型精确表示到一定程度。
- **性能消耗**：在GPU上进行计算时，`float`类型的运算通常需要较多的硬件资源和时间。因为它的精度较高，所以在进行乘法、除法等运算时，硬件需要更多的步骤来处理。
- **使用场景**：适用于需要较高精度的计算，如物理模拟中的向量运算、复杂的数学模型计算等。在处理光照模型、纹理坐标计算等对精度要求较高的场景时，经常会使用`float`类型。

### `fixed`

- **精度与取值范围**：`fixed`是16位定点数类型，在Unity Shader中，它通常用于表示颜色值等对精度要求不是特别高的数据。它的取值范围一般是 -1到1之间，精度相对较低，大约能精确到小数点后4位。
- **性能消耗**：由于`fixed`是16位数据类型，在GPU上进行运算时，它所占用的硬件资源和计算时间相对较少，性能表现较好。
- **使用场景**：常用于表示颜色值，如RGB颜色分量通常可以用`fixed`类型来表示。在一些对性能要求较高，且对精度要求不是特别严格的简单图形计算中，也会使用`fixed`类型，比如简单的顶点颜色计算等。

### `half`

- **精度与取值范围**：`half`是16位浮点数类型，它的精度介于`float`和`fixed`之间。取值范围比`fixed`更广泛，能表示的精度通常能精确到小数点后5位左右。
- **性能消耗**：`half`类型在性能上介于`float`和`fixed`之间。由于它是16位浮点数，在GPU运算时，相较于`float`能节省一定的硬件资源和计算时间，但比`fixed`类型的计算复杂度要高一些。
- **使用场景**：在一些对精度有一定要求，但又希望提高性能的场景中使用。例如，在处理一些不需要`float`类型那么高精度的纹理数据、法线数据等时，可以使用`half`类型来平衡精度和性能。

三者的区别总结如下表所示：
| 数据类型 | 位数 | 精度              | 取值范围                                                     | 性能 | 典型使用场景                                     |
| -------- | ---- | ----------------- | ------------------------------------------------------------ | ---- | ------------------------------------------------ |
| `float`  | 32   | 高，约小数点后7位 | ![1738735709531](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B04/1738735709531.png) | 低   | 高精度计算，如物理模拟、复杂光照模型             |
| `fixed`  | 16   | 低，约小数点后4位 | -1到1                                                        | 高   | 颜色值表示、简单图形计算                         |
| `half`   | 16   | 中，约小数点后5位 | ![1738735738149](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B04/1738735738149.png) | 中   | 纹理数据、法线数据等对精度和性能有一定要求的场景 |

## 避免不必要的计算

在 Shader 中避免不必要的计算对于提升性能至关重要，尤其是在移动设备等资源受限的环境中。以下是一些常见的避免不必要计算的方法：

### 1. 条件判断与分支优化
- **减少分支**：GPU 对于分支语句（如 `if-else`、`switch`）的处理效率较低，尤其是在不同线程执行不同分支时会导致性能下降。尽量减少复杂的分支判断，或者将分支判断提前到 CPU 进行。
```glsl
// 不好的写法
fixed4 frag(v2f i) : SV_Target {
    if (i.someCondition) {
        return fixed4(1, 0, 0, 1);
    } else {
        return fixed4(0, 1, 0, 1);
    }
}

// 优化后的写法
fixed4 frag(v2f i) : SV_Target {
    fixed4 color1 = fixed4(1, 0, 0, 1);
    fixed4 color2 = fixed4(0, 1, 0, 1);
    return lerp(color2, color1, step(0.5, i.someValue)); // 使用 lerp 和 step 函数替代 if-else
}
```
- **提前判断**：如果某些计算只在特定条件下需要执行，可以在条件不满足时提前返回结果，避免后续不必要的计算。
```glsl
fixed4 frag(v2f i) : SV_Target {
    if (i.someCondition == false) {
        return fixed4(0, 0, 0, 1);
    }
    // 执行复杂计算
    fixed4 result = someComplexCalculation(i);
    return result;
}
```

### 2. 缓存计算结果
- **避免重复计算**：如果同一个值在多个地方被使用，将其计算结果缓存起来，避免重复计算。
```glsl
fixed4 frag(v2f i) : SV_Target {
    // 计算一次
    float someValue = calculateSomeValue(i); 
    fixed4 color1 = someValue * fixed4(1, 0, 0, 1);
    fixed4 color2 = someValue * fixed4(0, 1, 0, 1);
    return color1 + color2;
}
```

### 3. 选择合适的数据类型
- **使用低精度类型**：根据实际需求选择合适的数据类型，避免使用高精度类型带来的不必要计算。例如，对于颜色值等对精度要求不高的情况，可以使用 `fixed` 类型代替 `float` 类型。
```glsl
// 使用 fixed 类型
fixed4 color : COLOR;
```

### 4. 减少纹理采样
- **合并纹理采样**：如果可能，将多个纹理采样合并为一个。例如，将多个相关的纹理信息存储在一个纹理的不同通道中，通过一次采样获取多个信息。
```glsl
// 合并前
float4 texColor1 = tex2D(_Texture1, i.uv);
float4 texColor2 = tex2D(_Texture2, i.uv);
float4 finalColor = texColor1 + texColor2;

// 合并后（假设两个纹理信息存储在一个纹理的不同通道）
float4 combinedTexColor = tex2D(_CombinedTexture, i.uv);
float4 finalColor = combinedTexColor.rg + combinedTexColor.ba;
```
- **避免不必要的采样**：确保只在需要时进行纹理采样，避免在每个像素着色器调用中都进行不必要的采样。

### 5. 常量和静态计算
- **使用常量**：如果某些值在整个渲染过程中不会改变，将其定义为常量，避免重复计算。
```glsl
const float PI = 3.1415926;
float result = someCalculation(PI);
```
- **静态计算**：对于一些可以在编译时计算的表达式，让编译器进行计算，而不是在运行时计算。例如，简单的数学运算 `2 + 3` 可以在编译时得到结果 5。

### 6. 剔除不可见部分
- **视锥体剔除**：确保只对可见的物体进行着色计算。Unity 引擎会自动进行视锥体剔除，但在自定义 Shader 中也可以通过一些技巧进一步优化。
- **背面剔除**：对于封闭的几何体，只对朝向相机的面进行计算，忽略背向相机的面。在 Shader 中可以通过设置 `Cull` 指令来实现。
```glsl
Pass {
    Cull Back // 剔除背面
    // 其他代码
}
```

通过以上方法，可以有效地在 Shader 中避免不必要的计算，提高渲染性能。 