---
title: Unity-Shader入门笔记2
date: 2025-01-24 18:38:38
tags: [shader,unity]
cover: pictures/mita0.png
---

# Shader 是什么

Shader的中文意思是 着色器是一种用于描述如何渲染图形和计算图形外观的程序主要用于控制图形的颜色、光照、纹理和其他视觉效果。

着色器通常由着色器语言编写，这些着色器语言提供了指令和语法用于编写描述光照、纹理映射、阴影、反射等图形外观的代码。

**Shader就是着色器，是用于编写图形表现效果的程序代码。**

为了和前面通用的Shader 语义进行区分，我们把Unity 中的Shader文件统称为**Unity Shader**。这是因为，Unity Shader和我们之前提及的渲染管线的Shader有很大不同。

# Shader 和渲染管线的关系

Shader和渲染管线的关系是密不可分的，渲染管线(流水线)的基本概念是将数据分阶段的变为屏幕图像的过程
而Shader开发就是针对其中**某些阶段的自定义开发**从而**决定**图形图像最终呈现到屏幕上的表现效果。

主要针对的是几何阶段中的**顶点着色器**小阶段和光栅化阶段中的**片元着色器**小阶段。

![1737731392848](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737731392848.png)

![1737731427662](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737731427662.png)

## 如何学习Shader开发

学习Shader开发，我们必须要学习的基本知识有：
		1.数学相关知识
		2.语法相关知识
		3.着色器开发相关知识

在Unity Shader开发过程中，我们需要学习Unity中的**ShaderLab**语法和着色开发的**CG**语言。在Unity中，所有的Unity Shader 都是使用 ShaderLab来编写的。ShaderLab是Unity 提供的编写Unity Shader的一种说明性语言。它使用了一些嵌套在花括号内部的**语义(syntax)**来描述一个Unity Shader 文件的结构。这些结构包含了许多渲染所需的数据，例如Properties 语句块中定义了着色器所需的各种属性，这些属性将会出现在材质面板中。

![1737731996254](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737731996254.png)





# Material

在Unity 中我们需要配合使用材质(Material)和Unity Shader才能达到需要的效果。一个最常见的流程是： 

(1)创建一个材质； 

(2)创建一个Unity Shader,并把它赋给上一步中创建的材质： 

(3)把材质赋给要渲染的对象； 

(4)在材质面板中调整Unity Shader的属性，以得到满意的效果。

下面是Unity Shader的导入设置面板。

![1737732586808](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737732586808.png)

![1737780661854](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737780661854.png)

![1737780867047](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737780867047.png)

#  Shader示例

下面是一个 **ShaderLab** 编写的自定义Shader 。

![1737780837136](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737780837136.png)

下面是一般形式：

```shaderlab
Shader"ShaderName"{
	Properties
	{
	// 属性
	}
	SubShader {
// 显卡A使用的子着色器
	}
	SubShader {
// 显卡B使用的子着色器
	}
Fallback "Diffuse"
//Fallback 是一个用于指定当某个Shader无法使用时，自动回退到其他Shader的机制。它通常用于确保在不支持某些高级图形特性的设备上，仍然能够渲染对象而不会导致错误或崩溃。
}

```

Unity 在背后会根据使用的平台来把这些结构编译成**真正的代码和 Shader 文件**，而开发者只需要和Unity Shader打交道即可。

## Properties

材质和 Unity Shader的桥梁：Properties。

Properties语义块中包含了一系列属性(property)，这些属性将会出现在材质面板中，斜杠来组织在材质面板中的位置 。

Properties 语义块的定义通常如下： 

```
Properties{

Name("display name",PropertyType)= DefaultValue 

Name("display name",PropertyType)- DefaultValue 

// 更多属性 

}
```

开发者们声明这些属性是为了在材质面板中能够方便地调整各种材质属性。如果我们需要在Shader 中访问它们，就需要使用每个属性的名字(Name)。在Unity 中，这些属性的名字通常由一个下划线开始。显示的名称(display name)则是出现在材质面板上的名字。我们需要为每个属性指定它的类型(PropertyType)。除此之外，我们还需要为每 个属性指定一个默认值，在我们第一次把该Unity Shader 赋给某个材质时，材质面板上显示的就是这些默认值。

![1737781808834](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737781808834.png)

## SubShader

每一个Unity Shader文件可以包含多个SubShader语义块，但最少要有一个。当Unity 需要加载这个Unity Shader时，Unity会扫描所有的SubShader语义块，然后选择第一个能够在目标平台运行的SubShader。如果都不支持的话，Unity 就会使用Fallback语义指定的Unity Shader。Unity 提供这种语义的原因在于，不同的显卡具有不同的能力。例如，一些旧的显卡仅能支 持一定数目的操作指令，而一些更高级的显卡可以支持更多的指令数，那么我们希望在旧的显卡 上使用计算复杂度较低的着色器，而在高级的显卡上使用计算复杂度较高的着色器，以便提供更出色的画面。

![1737781848687](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737781848687.png)

![1737781918447](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737781918447.png)

上述标签仅可以在SubShader中声明，而不可以在Pass块中声明。Pass块虽然也可以定义标签，但这些标签是不同于SubShader的标签类型。

Pass 语义块包含的语义如下：

```
Pass {
[Name]
[Tags]
[RenderSetup]
// Other code
}
```

Pass同样可以设置标签，但它的标签不同于SubShader的标签。这些标签也是用于告诉渲染引擎我们希望怎样来渲染该物体。

![1737782439567](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737782439567.png)



## Fallback

紧跟在各个SubShader语义块后面的，可以是一个Fallback 指令。它用于告诉Unity，“如果上面所有的SubShader在这块显卡上都不能运行，那么就使用这个最低级的Shader吧!”

Fallback 是一个用于指定当某个Shader无法使用时，自动回退到其他Shader的机制。它通常用于确保在不支持某些高级图形特性的设备上，仍然能够渲染对象而不会导致错误或崩溃。

# Unity Shader 的形式

最重要的任务还是指定各种着色器所需的代码。这些着色器代码可以写在SubShader语义块中(表面着色器的做法),也可以写在Pass 语义块中(顶 点/片元着色器和固定函数着色器的做法)。

## 表面着色器

表面着色器(Surface Shader)是Unity自己创造的一种着色器代码类型。它需要的代码量很少，Unity 在背后做了很多工作，但渲染的代价比较大。它在本质上和下面要讲到的顶点/片元着色器是一样的。当给 Unity 提供一个表面着色器的时候，它在背后仍旧把它转换成对应的顶点/片元着色器。

![1737784331576](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737784331576.png)

GPROGRAM和ENDCG之间的代码是使用CG/HLSL 编写的，也就是说，我们需要把CG/HLSL语言嵌套在 ShaderLab语言中。值得注意的是，这里的CG/HLSL是Unity 经封装后提供的，它的语法和标准的CG/HLSL 语法几乎一样，但还是有细微的不同，例如有些原生的函数 和用法Unity并没有提供支持。

## 顶点/片元着色器

在Unity中我们可以使用CG/HLSL语言来编写顶点/片元着色器(Vertex/Fragment Shader)。它们更加复杂，但灵活性也更高。一个非常简单的顶点/片元着色器示例代码如下：

![1737784414078](Unity-Shader%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B02/1737784414078.png)

