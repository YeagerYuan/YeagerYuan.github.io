---
title: Csharp学习笔记-2
date: 2025-01-18 16:47:32
tags: CSharp
cover: pictures/codingSakura.png
---

# C# 基本语法

学过C语言的同学对于C#的语法应该是比较熟悉的了。下面就以菜鸟教程中的一段代码作为示例。

```c#
using System;//命名空间指令，告诉编译器在代码中希望使用System命名空间中的类、接口、枚举和其他类型
namespace RectangleApplication
{
    class Rectangle
    {
        // 成员变量
        double length;//一个double类型的变量，名为length，下同
        double width;
        public void Acceptdetails()
        {
            length = 5.5;    
            width = 3.5;
        }//调用该方法，给变量赋值（C#中严格意义上只有方法Method，没有函数）
        public double GetArea()
        {
            return length * width;
        }//有返回值的方法
        public void Display()
        {
            Console.WriteLine("Length: {0}", length);
            Console.WriteLine("Width: {0}", width);
            Console.WriteLine("Area: {0}", GetArea());
        }//调用该方法，打印数据
    }
    

    class ExecuteRectangle
    {
        static void Main(string[] args)
        {
            Rectangle r = new Rectangle();//创建Rectangle类的对象r
            r.Acceptdetails();
            r.Display();
            Console.ReadLine();//等待用户输入，防止程序运行后窗口立即关闭
        }
    }//定义一个ExecuteRectangle类，其中包含了一个Main的方法，该方法在外部调用，可以创建一个矩形

}
```

