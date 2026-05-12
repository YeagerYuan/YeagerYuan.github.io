---
title: 用UIManager全局管理你的UI
date: 2026-02-27 15:57:04
tags: UGUI学习笔记
---

## 回顾之前的面板基类

首先我们先回顾一下之前我们是怎么写UGUI的UI脚本的。

我们会先写一个面板基类，会定义一些公共方法，其他所有的面板脚本都会继承它，代码如下：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/// <summary>
/// 面板基类 所有面板 都会继承它 方便我们的使用 节约代码量
/// </summary>
public abstract class BasePanel<T> : MonoBehaviour where T : class
{
    private static T instance;

    //公共属性，用于访问单例化实例
    public static T Instance => instance;//

    protected virtual void Awake()
    {
        instance = this as T;
    }

    // Start is called before the first frame update
    void Start()
    {
        Init();
    }

    public abstract void Init();

    //主要用于初始化监听等

    public virtual void ShowMe()
    {
        Debug.Log("调用了");
        this.gameObject.SetActive(true);
    }

    public virtual void HideMe()
    {
        this.gameObject.SetActive(false);
    }
}

```

以防遗忘，这里回顾一下此处用到的C#知识点：

### 1. 泛型与泛型约束 (Generics & Constraints)

- **知识点：** `BasePanel` 中的 `<T>` 是泛型。它允许你编写一个类或方法，推迟指定具体的数据类型，直到你实际使用它时。
- **知识点：** `where T : class` 是**泛型约束**。它告诉编译器：“这个 `T` 不能是随便什么类型（比如不能是 `int` 或 `float` 这种值类型），它**必须是一个引用类型（类）**。”
- **作用：** 这使得基类具有极强的**通用性**。比如你创建一个 `LoginPanel : BasePanel`，此时基类里的 `T` 就自动变成了 `LoginPanel`。

### 2. 抽象类与抽象方法 (Abstract Class & Methods)

- **知识点：** `public abstract class BasePanel` 中的 `abstract` 关键字定义了一个**抽象类**。抽象类就像是“半成品图纸”，它**不能被直接实例化**（你不能 `new BasePanel()` 或者把它直接挂在物体上）。
- **知识点：** `public abstract void Init();` 是**抽象方法**。它只有声明没有方法体 `{}`。
- **作用：** 这是一种“强制契约”。**任何继承 `BasePanel` 的子类面板**（比如背包面板、设置面板），都**必须**实现 `Init()` 方法，确保每个面板都有自己的初始化逻辑。

### 3. 虚方法 (Virtual Methods)

- **知识点：** `protected virtual void Awake()` 和 `public virtual void ShowMe()` 中的 `virtual` 关键字定义了**虚方法**。
- **区别于抽象方法：** 虚方法**有具体的实现逻辑**（比如 `ShowMe` 里写了打印日志和激活物体）。
- **作用：** 它允许子类根据需要进行**重写（Override）**。如果子类在显示时除了激活物体还需要播放一个动画，就可以重写 `ShowMe` 方法；如果不需要特殊处理，**直接沿用父类的逻辑即可**。

### 4. 静态成员与单例模式 (Static Members & Singleton Pattern)

- **知识点：** `private static T instance;` 定义了一个静态变量。静态变量不属于某个具体的对象实例，而是属于类本身。无论你创建了多少个对象，静态变量在内存中永远只有一份。

- **设计模式应用：** 这几行代码实现了一个简易的**单例模式（Singleton）**：

  ```
  private static T instance;
  public static T Instance => instance;
  ```

  通过把 `instance` 变成静态的，外部脚本就可以直接通过 `具体面板类名.Instance` 来全局访问这个面板（例如 `LoginPanel.Instance.ShowMe()`），免去了频繁使用 `GameObject.Find` 查找物体的性能消耗。

### 5. C# 语法糖：表达式主体定义 (Expression-bodied Members)

- **知识点：** `public static T Instance => instance;` 中使用了 `=>` 操作符。

- **作用：** 这是 C# 6.0 引入的语法糖，用于简化**只读属性**的写法。它等价于传统的：

  ```
  public static T Instance 
  { 
      get { return instance; } 
  }
  ```

### 6. 安全的类型转换 (Safe Type Casting)

- **知识点：** `instance = this as T;` 中的 `as` 关键字。
- **作用：** 它是 C# 中用于引用类型转换的安全方式。如果是强转 `(T)this`，一旦转换失败会直接抛出异常导致程序崩溃；而使用 `as`，如果 `this` 不能转换为 `T`，只会默默返回 `null`，程序更为安全。

## 继承之前面板基类的具体面板脚本

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;
using UnityEngine.SceneManagement;

// 自动添加 CanvasGroup 组件，防止你忘记加
[RequireComponent(typeof(CanvasGroup))] 
public class StartPanel : BasePanel<StartPanel>
{
    [Header("UI组件")]
    public Button btnStart;
    public Button btnPackage;
    public Button btnShop;
    public Button btnExit;

    [Header("外部引用")]
    public GameModeManager gameModeManager;

    [Header("动画设置")]
    public float fadeDuration = 0.5f; // 淡出需要几秒
    private CanvasGroup canvasGroup;  // 用来控制透明度

    // 建议在 Awake 中获取组件，比 Init 更早
    protected override void Awake()
    {
        base.Awake(); // 保持父类逻辑
        canvasGroup = GetComponent<CanvasGroup>();
    }

    public override void Init()
    {
        btnStart.onClick.AddListener(() =>
        {
            Debug.Log("开始游戏啦");
            
            // 1. 开始淡出（HideMe 会触发协程）
            HideMe(); 

            // 2. 切换游戏模式（相机开始飞）
            if (gameModeManager != null)
            {
                gameModeManager.EnterTPCMode(); 
            }
            else
            {
                Debug.LogError("StartPanel: 忘记在 Inspector 里拖拽 GameModeManager 啦！");
            }
        });

        btnPackage.onClick.AddListener(() =>
        {
            BaokuPanel.Instance.ShowMe();
        });

        btnShop.onClick.AddListener(() =>
        {
            ShopPanel.Instance.ShowMe();
        });

        btnExit.onClick.AddListener(() =>
        {
            Application.Quit();
        });
    }

    // 重写 ShowMe：确保每次显示时，面板是不透明的
    public override void ShowMe()
    {
        base.ShowMe();
        if (canvasGroup == null) canvasGroup = GetComponent<CanvasGroup>();
        
        canvasGroup.alpha = 1f; // 设为完全不透明
        canvasGroup.blocksRaycasts = true; // 允许点击
        canvasGroup.interactable = true;
    }

    // 重写 HideMe：不直接关掉物体，而是先播动画
    public override void HideMe()
    {
        // 开启协程进行淡出
        StartCoroutine(FadeOutEffect());
    }

    // 淡出协程逻辑
    IEnumerator FadeOutEffect()
    {
        // 1. 先禁止点击，防止淡出过程中玩家乱点
        canvasGroup.blocksRaycasts = false; 
        
        float timer = 0f;
        float startAlpha = canvasGroup.alpha;

        while (timer < fadeDuration)
        {
            timer += Time.deltaTime;
            // 插值计算：从 startAlpha 变到 0
            canvasGroup.alpha = Mathf.Lerp(startAlpha, 0f, timer / fadeDuration);
            yield return null; // 等待下一帧
        }

        // 2. 确保完全透明
        canvasGroup.alpha = 0f;

        // 3. 动画播完后，再真正调用父类的隐藏（SetActive false）
        base.HideMe(); 
    }
}
```

在这个面板脚本中，我们重写了方法，在继承了父类功能的基础上添加了自己的逻辑， 例如`Awake` 和 `ShowMe` 里面的 `base.Awake()` 和 `base.ShowMe()`。`base` 代表父类。这表示：“我要先执行父类原有的逻辑（比如初始化单例、激活物体），然后再执行我自己的独有逻辑（获取 CanvasGroup 设置透明度）。”这是面向对象编程中复用代码的经典操作。

而今天，我们换一种方法来写我们的UI脚本。在前面的方法中，我们采用的是多单例模式，每个子面板虽然都继承了面板基类，但是都是独立的单例。如果我想关闭A面板，打开B面板，那么我还要再A面板中调用面板B。如果我想一下关闭所有面板，则根本做不到。现在，我们将另外设置一个UIManager脚本来统一管理所有的脚本。这样做的好处有：

1.能保证解耦性，一个面板不需要再认识另外一个面板，而只需要调用UIManager就可以操控另一个面板。

2.统一的资源调度，通过使用字典记载所有面板的打开关闭情况。

## 新的面板基类

下面是面板基类和增加的重点部分分析：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

/// <summary>
/// 面板基类 所有面板 都会继承它 方便我们的使用 节约代码量
/// </summary>
public abstract class BasePanel : MonoBehaviour
{

    //专门用于控制面板透明度的组件
    private CanvasGroup canvasGroup;

    //淡入淡出的速度
    private float alphaSpeed = 10f;

    //是否是显示状态
    private bool isShow = false;

    //面板隐藏后需要做的事情
    private UnityAction hideCallBack = null;

    protected virtual void Awake()
    {
        canvasGroup = this.GetComponent<CanvasGroup>();
        if (canvasGroup == null)
        {
            canvasGroup = this.gameObject.AddComponent<CanvasGroup>();
        }
    }

    // Start is called before the first frame update
    protected virtual void Start()
    {
        Init();
    }

/// <summary>
/// 注册控件事件的方法，所有的子面板都要去注册一些控件事件
/// 所以写成抽象方法，让子类必须去实现
/// </summary>
    public abstract void Init();

    //主要用于初始化监听等

    public virtual void ShowMe()
    {
        canvasGroup.alpha = 0;
        isShow = true;
    }

    public virtual void HideMe(UnityAction callback)
    {
        canvasGroup.alpha = 1;
        isShow = false;

        hideCallBack = callback;

    }

    void Update()
    {
        //显示状态，不为1，不断加到1
        if(isShow && canvasGroup.alpha != 1){
            canvasGroup.alpha += alphaSpeed * Time.deltaTime;
            if(canvasGroup.alpha >= 1){
                canvasGroup.alpha = 1;
            }
        }else if(!isShow && canvasGroup.alpha != 0){
            canvasGroup.alpha -= alphaSpeed * Time.deltaTime;
            if(canvasGroup.alpha <= 0){
                canvasGroup.alpha = 0;
                // 先暂存回调
                 UnityAction tempAction = hideCallBack;
        
                // 1. 无论回调里做什么，先切断引用，防止循环
                 hideCallBack = null; 
        
                // 2. 再执行回调
                tempAction?.Invoke();
            }
        }
    }
}
```

### 1.加入了淡入淡出效果

我们在这个面板基类中加入了淡入淡出效果，这意味着**以后所有继承这个基类的面板，天生就自带了平滑的淡入淡出效果**，极大节约了代码量。

### 2. 委托与回调函数 (Delegates & Callbacks)

- **知识点：** `private UnityAction hideCallBack = null;` 和 `public virtual void HideMe(UnityAction callback)`。
- **概念说明：** `UnityAction` 是 Unity 封装好的一种委托（Delegate），本质上它就是一个**“可以存储方法的变量”**。
- **作用：** 这里的 `callback` 就是**回调函数**。你可以把它想象成“留电话号码”。当你调用 `HideMe` 时，你不仅告诉面板“隐藏自己”，还塞给它一张纸条（callback）：“等你完全变透明后，**记得照着纸条上的指示做下一步**（比如打开另一个面板）。”

### 3. 安全的事件执行防重入机制 (Safe Event Invocation)

```
UnityAction tempAction = hideCallBack;
hideCallBack = null; 
tempAction?.Invoke();
```

- **知识点：防循环调用（防重入）**。如果直接写 `hideCallBack?.Invoke();`，假设在这个回调函数里，别人又调用了当前面板的方法或者触发了其他修改 `hideCallBack` 的逻辑，很容易产生死循环或数据错乱。
- **逻辑拆解：**
  1. 先用一个临时变量 `tempAction` 把委托存起来。
  2. **立刻**把原有的 `hideCallBack` 清空（切断引用）。
  3. 最后再执行临时变量里的逻辑。这样即使回调里又发生了什么奇葩操作，也不会影响当前面板的状态了。
- **语法糖 `?.`（Null 条件运算符）：** `tempAction?.Invoke()` 的意思是，如果 `tempAction` 不是空的，就执行它；如果是空的，就什么也不做。这比写 `if (tempAction != null) { tempAction(); }` 优雅得多。

## 增加的UIManager

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class UIManager
{
    private static UIManager instance = new UIManager();//单例模式

    public static UIManager Instance => instance;   //单例模式


/// <summary>
/// 用于存储显示着的面板，每显示一个面板，就会存入这个字典
/// 隐藏面板时，直接获取字典中的对应面板，进行隐藏
/// </summary>
    private Dictionary<string,BasePanel> panelDic = new Dictionary<string,BasePanel>();

    //场景中的canvas，用于设置为面板的父对象
    private Transform canvasTransform;

    public UIManager(){
        //加载canvas对象，并设置为父对象
        GameObject canvasObj = GameObject.Instantiate(Resources.Load<GameObject>("UI/Canvas"));
        //设置为父对象
        canvasTransform = canvasObj.transform;
        //设置为DontDestroyOnLoad，确保在场景切换时不会被销毁
        GameObject.DontDestroyOnLoad(canvasObj);
    }

    //显示面板
    public T show<T>() where T : BasePanel
    {
        //获取面板的类型名
        string panelName = typeof(T).Name;
        //如果字典中已经存在对应的面板，则直接返回
        if(panelDic.ContainsKey(panelName))
        {
            return panelDic[panelName] as T;
        }
         //如果字典中没有对应的面板，则加载一个新面板
         GameObject panelObj = GameObject.Instantiate(Resources.Load<GameObject>("UI/" + panelName));
         //如果加载失败，则抛出异常
         if(panelObj == null)
         {
            Debug.LogError("加载面板失败: " + panelName);
            return null;
         }
         //放到canvasTransform下，并且设置为false，确保位置不会被重置
         panelObj.transform.SetParent(canvasTransform,false);

         //指向面板上显示逻辑，并且应该保存起来
         T panel = panelObj.GetComponent<T>();

         //把这个面板脚本储存到字典中
         panelDic.Add(panelName,panel);
         
         //调用自己的显示逻辑
         panel.ShowMe();

         //返回面板
         return panel;

    }
    /// <summary>
    /// 隐藏面板
    /// </summary>
    /// <typeparam name="T">面板的类型</typeparam>
    /// <param name="isFade">是否需要淡入淡出</param>
    public void HidePanel<T>(bool isFade = false) where T : BasePanel
    {
        //获取面板的类型名
        string panelName = typeof(T).Name;
        if(isFade)//看看是否需要淡入淡出
        {
            panelDic[panelName].HideMe(() =>
            {
                //删除面板
                GameObject.Destroy(panelDic[panelName].gameObject);
                //删除字典中的对应面板
                panelDic.Remove(panelName);
            });
        }else{
            //删除面板
            GameObject.Destroy(panelDic[panelName].gameObject);
            //删除字典中的对应面板
            panelDic.Remove(panelName);
        }
    }

    //得到面板
    public T GetPanel<T>() where T : BasePanel
    {
        //获取面板的类型名
        string panelName = typeof(T).Name;
        
        if(panelDic.ContainsKey(panelName))
        {
            return panelDic[panelName] as T;
        }
        //如果字典中没有对应的面板，则抛出异常
        Debug.LogError("获取面板失败: " + panelName);
        return null;
    }

}
```

这段代码里涉及了非常多架构级别的 C# 和 Unity 知识点，我们来逐一拆解：

### 1. 纯 C# 单例模式 (饿汉式单例)

- **知识点：** `private static UIManager instance = new UIManager();` 和 `public static UIManager Instance => instance;`
- **不同之处：** 注意到这个类**没有继承 `MonoBehaviour`** 吗？这意味着它不能被挂载到游戏物体上。它是一个纯 C# 类。
- **作用：** 当程序第一次用到 `UIManager` 时，它就会自动 `new` 出一个唯一的实例。这种写法非常安全，自带线程安全特性，且不需要像 MonoBehaviour 单例那样去场景里找物体。

### 2. 泛型与反射机制的巧妙结合 (Generics & Reflection)

- **知识点：** `typeof(T).Name`
- **作用：** 这是一种极其优雅的**“约定优于配置”**的设计！
  - 当你调用 `UIManager.Instance.show()` 时，`typeof(T).Name` 会自动把类名转换成字符串 `"LoginPanel"`。
  - 然后它利用这个字符串去 `Resources/UI/` 文件夹下找名字一模一样的 Prefab（预制体）。
  - **好处：** 你永远不需要手动去配置“哪个脚本对应哪个预制体”，只要保证**脚本名字和预制体名字完全一致**，系统就能自动加载。

### 3. 字典：高效的缓存管理 (Dictionary Data Structure)

- **知识点：** `Dictionary panelDic`
- **作用：** 字典通过“键值对（Key-Value）”来存储数据。这里用面板的名字作 Key，面板脚本的引用作 Value。
  - 当你想隐藏面板时，不需要用耗性能的 `GameObject.Find` 去场景里找，直接通过 `panelDic["LoginPanel"]` 就能瞬间拿到它。时间复杂度是 O(1)，极其高效。

### 4. 完美闭环：回调函数实战 (Callback Implementation

- **知识点：** 在 `HidePanel` 中传入 Lambda 表达式。

  ```
  panelDic[panelName].HideMe(() =>
  {
      GameObject.Destroy(panelDic[panelName].gameObject);
      panelDic.Remove(panelName);
  });
  ```

- **原理解析：** 这就是前面的 `BasePanel` 里那个 `callback` 的真面目！UIManager 告诉面板：“你先去播你的淡出动画（`HideMe`），我这里写好了一个匿名函数（销毁物体+清理字典），交给你带着。等你动画播完彻底黑掉的那一帧，你帮我调用一下这段逻辑。” 这样就完美实现了**“先平滑淡出，再彻底销毁”**的视觉体验。