教程来自【《迷失岛2》游戏框架开发00:项目介绍｜Unity教程】https://www.bilibili.com/video/BV1vY4y1W7YD?vd_source=28e3994bf6aa5fa346ca67b74f2c51f6，项目素材和工具可根据视频简介自行查找。

# 前置知识

## 场景层叠

之前我们的Hierarchy中只有一个Scene，要么游戏只需要一个场景就能完成所有的东西，要么切换场景时使用代码加载项目中保存的场景并切换。

这次教程将多个场景同时置入Hierarchy，通过一个无任何实体对象的Scene来控制各个处理系统和一个MainCamera。

关于两种不同的场景加载模式的区别优劣见：[Unity多场景和单场景的优劣及应用场景解析 - 易知微开发者社区 (easyv.cloud)](https://easyv.cloud/c/article/4844.html#:~:text=1.模块化开发：多场景模式将游戏划分为多个独立的场景，每个场景可独立开发和测试，有利于团队协作和项目管理。,如果需要修改某个场景，只需打开对应的场景进行操作，不会对其他场景产生影响。 2.加载与卸载：多场景模式可以实现场景的动态加载与卸载，节省内存空间和提高游戏性能。)

## 单例模式

定义和作用自不用多说，我们需要提前编写一个`Singleton<T>`类(挂载到Unity对象上，必须继承MonoBehavier)：

```C#
using UnityEngine;

/// <summary>
/// 继承MonoBehaviour的泛型单例基类
/// </summary>
public class SingletonMono<T> : MonoBehaviour where T : MonoBehaviour
{
    //记录单例对象是否存在。用于防止在OnDestroy方法中访问单例对象报错
    public static bool IsExisted { get; private set; } = false;

    private static T instance;

    public static T Instance
    {
        get
        {
            if (instance == null)
            {
                instance = FindObjectOfType<T>();
                if (instance == null)
                {
                    GameObject go = new GameObject(typeof(T).Name); // 创建游戏对象
                    instance = go.AddComponent<T>(); // 挂载脚本
                }
            }
            DontDestroyOnLoad(instance);
            IsExisted = true;
            return instance;
        }
    }

    
    // 构造方法私有化，防止外部 new 对象
    protected SingletonMono() { }

    private void OnDestroy() {
        IsExisted = false;
    }
}

```

## 协程

[Unity中协程(IEnumerator)使用方法+停止方法+协程start前需要判断其是否开启了,否则协程会不断地叠加-CSDN博客](https://blog.csdn.net/qq_40544338/article/details/115414085)

## ScriptableObject实现数据存储

[Unity进阶：ScriptableObject使用指南-CSDN博客](https://blog.csdn.net/qq_46044366/article/details/124310241)

## 事件订阅

[【Unity记录】如何优雅地在Unity中订阅与退订C#事件_unity事件取消订阅-CSDN博客](https://blog.csdn.net/mkr67n/article/details/126277253)

事件声明除了public delegate void ConditionChangeHandler(float value);这种写法以外，也可以写成类似public static event Action<ItemDetail, int> UpdateUIEvent;这种，区别在于第一种写法更复杂但是可以自定义返回类型（void时不要<>），第二种更简洁但是返回类型只能是void。

事件订阅和退订是很好的降低不同模块之间耦合度以及有效传递数据的方法。

# 场景之间的切换

![image-20240928155206782](picture/image-20240928155206782.png)

## 场景建立

对于一个场景，我们需要不同的箭头提示去往不同的场景，并使用鼠标点击进行传送。

先在Project中新建两个Scene，命名为H1和H2，不切换Scene，直接将两个Scene拖入Hierarchy中，(注意先删除两个默认创建的MainCamera对象，)实现场景的层叠。

将两个背景的素材拖入Hierarchy，重命名为H1(2)Background。

右键H1场景，选择**Unload Scene**，这样在Scene窗口中H1场景不会被加载和显示（**运行时是否被加载同样会受影响**）。

## 场景间的切换控制器

为了实现点击背景中的箭头以触发场景切换，在H2中创建一个Empty对象，命名为TeleportToH1，添加Box碰撞体，调整大小和位置并设为Trigger。

然后为这个对象添加Teleport脚本：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class Teleport : MonoBehaviour
{
    public enum Scene
    {
        SampleScene,
        H1,
        H2
    }
    public Scene sceneFrom;
    public Scene sceneToGo;
    public void TeleportToScene()
    {
        TransitionManager.Instance.Transition(sceneFrom, sceneToGo);
    }
}
```



在SampleScene下添加一个空对象，命名为TransitionManager用于控制场景切换。添加以下脚本代码：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class TransitionManager : SingletonMono<TransitionManager>
{
    public void Transition(Teleport.Scene sceneFrom, Teleport.Scene sceneToGo)
    {
        StartCoroutine(TransitionToScene(sceneFrom, sceneToGo));
    }
    private IEnumerator TransitionToScene(Teleport.Scene sceneFrom, Teleport.Scene sceneToGo)
    {
        yield return SceneManager.UnloadSceneAsync(sceneFrom.ToString());
        yield return SceneManager.LoadSceneAsync(sceneToGo.ToString(), LoadSceneMode.Additive);
        

        // 设置新场景为激活场景
        Scene newScene = SceneManager.GetSceneByName(sceneToGo.ToString());
        SceneManager.SetActiveScene(newScene);
    }
}
```

然后设置H2场景为Set Active Scene，这样H2会被高亮（暂时不知道有什么用）。

## 点击控制器（监听事件）

给刚刚添加的碰撞体更换新标签Teleport。

在Project中新添加脚本CursorManager，并且在SampleScene中添加空对象并挂载CursorManager脚本：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CursorManager : MonoBehaviour
{
    private Vector3 mouseWorldPos => Camera.main.ScreenToWorldPoint(new Vector3(Input.mousePosition.x, Input.mousePosition.y, 0));
    private bool canClick;

    private void Update()
    {
        canClick = ObjectAtMousePosition();

        if (canClick && Input.GetMouseButtonDown(0))
        {
            ClickAction(ObjectAtMousePosition().gameObject);
        }
    }

    private void ClickAction(GameObject clickObject)
    {
        // 根据GameObject的标签来判断点击的是哪个物体，便于扩展
        switch (clickObject.tag)
        {
            case "Teleport":
                var teleport = clickObject.GetComponent<Teleport>();
                teleport?.TeleportToScene();//?表示判断不为空再执行
                break;
        }
    }
        /// <summary>
        /// 检测鼠标点击范围的碰撞体
        /// </summary>
        /// <returns></returns>
    private Collider2D ObjectAtMousePosition()
    {
        return Physics2D.OverlapPoint(mouseWorldPos);
    }

}

```

对于 ` private Vector3 mouseWorldPos =>`一行，GPT是这样解释的：

*这段代码是Unity游戏引擎中C#脚本语言的一行代码，用于将鼠标在屏幕上的位置转换为游戏世界中的坐标。下面是对这行代码的解释：*

- `private Vector3 mouseWorldPos`: 这是一个私有的属性，意味着它只能在定义它的类内部被访问。
- `=>`: 这是C#中的一个特性，称为“lambda表达式”，用于创建匿名函数。
- `Camera.main`: 获取当前场景中标记为“MainCamera”的相机。
- `ScreenToWorldPoint(new Vector3(Input.mousePosition.x, Input.mousePosition.y, 0))`: 调用`Camera`类的`ScreenToWorldPoint`方法，将屏幕坐标转换为世界坐标。`Input.mousePosition`获取鼠标在屏幕上的位置，`x`和`y`是鼠标位置的屏幕坐标，`0`是屏幕坐标的Z轴值，表示在屏幕平面上。

*这行代码的意思是，创建一个名为`mouseWorldPos`的属性，每次访问这个属性时，都会计算并返回鼠标当前在游戏世界中的坐标。这个坐标可以用来检测鼠标点击的对象，或者用于其他需要鼠标世界位置的操作。*

现在可以测试了：确保H1处于Unloaded状态，H2下的TeleportToH1对象下的脚本的公开变量赋值正确。运行游戏，点击箭头，跳转到H1场景并且H2场景从Hierarchy消失，说明逻辑正确。

## 逻辑梳理

CursorManager：持续监听可点击状态和鼠标点击事件，获取被点击物体后根据标签触发不同的点击事件。

Teleport：编写必要变量和点击触发事件`TeleportToScene()`，并将该方法委托给CursorManager来监听并触发。

TransitionManager：提供场景切换的接口，供Teleport调用。

## 简单渐入渐出

给场景切换增加一点点动画，在SampleScene下新建Canvas，更改UI Scale Mode为Scale With Screen Size，设置分辨率为1080P。再新建panel，更改Image为Unity自带的Square并置为纯黑色。添加Canvas Group组件，勾去Blocks Raycasts，设置初始Alpha为0。

更改TransitionManager，添加Fade协程方法以及需要的变量：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class TransitionManager : SingletonMono<TransitionManager>
{
    public CanvasGroup fadeCanvasGroup;
    public float fadeDuration = 1f;
    private bool isFade;
    public void Transition(Teleport.Scene sceneFrom, Teleport.Scene sceneToGo)
    {
        if(!isFade)
            StartCoroutine(TransitionToScene(sceneFrom, sceneToGo));
    }
    private IEnumerator TransitionToScene(Teleport.Scene sceneFrom, Teleport.Scene sceneToGo)
    {
        yield return StartCoroutine(Fade(1));
        yield return SceneManager.UnloadSceneAsync(sceneFrom.ToString());
        yield return SceneManager.LoadSceneAsync(sceneToGo.ToString(), LoadSceneMode.Additive);
        

        // 设置新场景为激活场景
        Scene newScene = SceneManager.GetSceneByName(sceneToGo.ToString());
        SceneManager.SetActiveScene(newScene);
        yield return StartCoroutine(Fade(0)); 
    }
    /// <summary>
    /// 淡入淡出场景
    /// </summary>
    /// <param name="targetAlpha"></param>
    /// <returns></returns>
    private IEnumerator Fade(float targetAlpha)
    {
        isFade = true;
        fadeCanvasGroup.blocksRaycasts = true;

        float speed = Mathf.Abs(fadeCanvasGroup.alpha - targetAlpha) / fadeDuration;

        while(!Mathf.Approximately(fadeCanvasGroup.alpha, targetAlpha))
        {
            fadeCanvasGroup.alpha = Mathf.MoveTowards(fadeCanvasGroup.alpha, targetAlpha, speed * Time.deltaTime);
            yield return null;
        }

        fadeCanvasGroup.blocksRaycasts = false;
        isFade = false;
    }
}

```

# 背包系统

这个游戏的道具系统实际非常简单——获取道具（不管是什么），有道具就能触发特殊的点击事件。当然，道具自身的简单与否并不怎么影响背包系统和UI显示。

## 交互道具

在H2场景的桌子上放置一把钥匙，同样的添加碰撞体做触发器，设标签为Item。

新建Inventory文件夹并添加Item脚本：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Item : MonoBehaviour
{
    public ItemName itemName;

    public void ItemClicked()
    {
        InventoryManager.Instance.AddItem(itemName);
        Destroy(gameObject);
    }
}
```

在CursorManager脚本中添加新的case "Item"：

```C#
case "Item":
    var item = clickObject.GetComponent<Item>();
    item?.ItemClicked();
    break;
```

\*注：目前Item脚本代码存在问题，不管是Destroy还是SetActive，从其他场景切换回H2的时候钥匙都会随H2的加载而重新加载。

定义了一个枚举类型`ItemName`，另外编写一个脚本Ultility/Enum.cs来存储`ItemName`的定义：

```C# 
public enum ItemName
{
    None, 
    Key,
    Ticket
}
```

为了存储道具信息并能被每个Item实例引用，我们需要利用`ScriptableObject`来实现**暂时的**持久化存储。（具体原因见前置知识），由于存储的不是GameObject，我们需要编写一个实体类`ItemDatail`来定义道具的名字和相应的图片精灵。然后再存储一个`List<ItemDetail>`并定义查询接口：

```C#
using System.Collections.Generic;
using System.Collections;
using UnityEngine;
using System;

[CreateAssetMenu(fileName = "itemDetailList", menuName = "Inventory/itemDetailList")]

public class ItemDataList : ScriptableObject
{
    public List<ItemDetail> itemDetailList = new List<ItemDetail>();
    public ItemDetail GetItemDetail(ItemName itemName)
    {
        return itemDetailList.Find(x => x.itemName == itemName);
    }
}

// 使用[Serializable]特性来标记一个类，使其可以被序列化
[Serializable]
public class ItemDetail
{
    public ItemName itemName;
    public Sprite itemSprite;
}
```

特性`[CreateAssetMenu(fileName = "itemDetailList", menuName = "Inventory/itemDetailList")]`的参数解释如下：

- `fileName`：创建的资产文件的名称。在这个例子中，当你通过这个菜单项创建资产时，文件将被命名为`itemDetailList`。
- `menuName`：在Unity编辑器的“Assets”菜单下显示的路径和名称。在这个例子中，它将创建一个名为“Inventory”的新子菜单，然后在该子菜单下创建一个名为“itemDetailList”的菜单项。

在Project中根目录下新建文件夹Game Data，再在Game Data下新建文件夹Inventory，右键Create>（最顶上）Inventory>ItemDetailList，这个ItemDetailList.asset就是我们定义的资产了。

如果正确的话，点开ItemDetailList，能看到这样的一个折叠栏：

![image-20241001202916428](picture/image-20241001202916428.png)

点开收缩栏，添加Key和Ticket类型并拖入对应的精灵，这样两个道具的数据就算存储好了。

然后同样地，在SampleScene中添加道具控制器。编写InventoryManager脚本：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class InventoryManager : SingletonMono<InventoryManager>
{
    public ItemDetailList itemDetailList;
    [SerializeField]private List<ItemName> itemList = new List<ItemName>();
    public void AddItem(ItemName itemName)
    {
        if(!itemList.Contains(itemName))
        {
            itemList.Add(itemName);
            // UI对应显示
        }
    }
}
```

在Inspector中挂载好脚本并将ItemDetailList资产拖入itemDetailList属性，运行游戏，当点击钥匙时，钥匙消失，同时itemDetailList的长度为1，则逻辑正确。

## UI设计

![image-20241001222710424](picture/image-20241001222710424.png)

新建MainCanvas，添加Menu Button，调整Width和height，使用Anchor Presets调整大致位置和锚点，再调整Pos进行微调。

同理添加道具栏ItemHolder（Image），在道具栏下添加左右Button、图标显示Slot（Image）、顶部道具名栏ItemToolTip（Image）。更改Button组件的Transition属性为Sprite Swap（使用精灵变换形态），将素材中普通、按下、不可点击形态的精灵拖入相应的位置即可。

为Slot添加脚本提供控制图标显示接口：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
public class SlotUI : MonoBehaviour
{
    public Image ItemImage;
    private ItemDetail currentItem;
    private bool isSelected;

    public void SetItem(ItemDetail itemDetail)
    {
        currentItem = itemDetail;
        // 注意这里启用了Slot对象，所以即使被禁用了，调用该方法仍能让对象重新启用
        this.gameObject.SetActive(true);
        ItemImage.sprite = itemDetail.itemSprite;
        ItemImage.SetNativeSize();
    }

    public void SetEmpty()
    {
        this.gameObject.SetActive(false);
    }
}
```

**注意：ItemImage可以在Hierarchy中直接拖入Slot自己的Image组件。**



然后为ItemHolder添加控制脚本InventoryUI.cs:

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
public class InventoryUI : MonoBehaviour
{
    public Button leftButton, rightButton;

    public int currentIndex;
    public SlotUI slotUI;
    private void OnUpdateUIEvent(ItemDetail itemDetail, int index)
    {
        if (itemDetail == null)
        {
            // 防止Error用的
            slotUI.SetEmpty();
            currentIndex = -1;
            leftButton.interactable = false;
            rightButton.interactable = false;
        }
        else
        {
            currentIndex = index;
            slotUI.SetItem(itemDetail);
        }
    }
}

```

### 事件订阅

现在，我们需要重新仔细思考一下背包UI应该实现哪些功能：

* 切换背包中不同的道具并能正确触发其功能
* 当拾起新道具时，更新`InventoryManager`中存储道具名字的`itemList`，并且在道具栏中将新道具放到最后且立即切出来。

第一个功能暂时不用管，因为当前我们的场景中只有钥匙一个道具，后续我们再继续补充代码功能。要想实现拾起钥匙时道具栏立刻显示钥匙图标，最简单的方法就是在`InventoryManager`的`AddItem()`方法中当在list中添加新道具名字之后立刻调用`InventoryUI`的`SetItem()`方法。这种方法固然简单，实际应用的时候会有诸多问题：

* *万一在某一时刻我不想调用`SetItem()`方法了怎么办？*
* *如果更改的不止是Slot，一堆东西要更改怎么办？如果都添加进来，那我要更改岂不是很麻烦？*

还是一样的思想，我们希望有一个中介，从InventoryManager处接受调用，再将调用发送给SlotUI或者其他脚本。这下终于理解了怎样从m * n 问题转换为m + n问题了。

再进一步地，我们引入"订阅"机制，当一个读者（脚本）想要阅读最新的报纸（可能是一种也可能是多种），他（它）只需要向送报员（Handler）说：“每次有相应类型的最新一期的报纸（变动）的时候第一时间送过来（通知）我看看（调用订阅者的处理方法）。”然后每一次有报社（被订阅的脚本）有新的报纸（变动）就发送给送报员（Handler），送报员立即送到订阅者的家里，订阅者再马上查看（处理）新的报纸。当然了，当读者不想再看这种报纸的时候，就通知送报员一声“我不想再看这种了，以后不送了”。那么之后就算有新报纸出版，这位读者也不会再看（处理）了。这就是**事件订阅**。



首先在一个新的脚本中定义事件处理类`EventHandler`：

```C#
using System;
using UnityEngine;

public static class EventHandler
{
    // 定义事件及其规定接收参数
    public static event Action<ItemDetail, int> UpdateUIEvent;
    // 提供调用接口
    public static void CallUpdateUIEvent(ItemDetail itemDetail, int index)
    {
        // 当被调用时发送消息
        UpdateUIEvent?.Invoke(itemDetail, index);
    }
}
```

可以理解为，UpdateUIEvent是一个方法列表，这些方法都必须规定相同类型的参数，当`EventHandler`的接口被调用时，调用这个“方法列表”中的所有方法。



然后在InventoryUI.cs中添加：

```C#
// 当组件被启用（SetActive(true)）时调用
private void OnEnable()
{
    EventHandler.UpdateUIEvent += OnUpdateUIEvent;
}
// 当组件被禁用（SetActive(true)）时调用
public void OnDisable()
{
    EventHandler.UpdateUIEvent -= OnUpdateUIEvent;
}
// 事件处理方法
private void OnUpdateUIEvent(ItemDetail itemDetail, int index)
{
    if (itemDetail == null)
    {
        // 防止Error用的,如果itemDetail为空，说明道具栏为空，设置为空
        slotUI.SetEmpty();
        currentIndex = -1;
        leftButton.interactable = false;
        rightButton.interactable = false;
    }
    else
    {
        currentIndex = index;
        slotUI.SetItem(itemDetail);
    }
}
```



最后在InventoryManager中添加事件处理器的调用：

```C#
 public void AddItem(ItemName itemName)
 {
     if(!itemList.Contains(itemName))
     {
         itemList.Add(itemName);
         // UI对应显示
         EventHandler.CallUpdateUIEvent(itemDetailList.GetItemDetail(itemName), itemList.Count - 1);
     }
 }
```

这样事件订阅就完成了。



现在测试一下：确认该拖入赋值的公开变量都赋值了（尤其是Slot的ItemImage)，禁用Slot和ItemToolTip（检验Slot是否能重新被启用）。启动游戏，点击钥匙，道具栏显示钥匙图标，完成！

# 场景状态保存

先前我们提到，即使在当前场景中点击钥匙，将钥匙置入物品栏了，当从其他场景回到该场景的时候，钥匙又会出现。

现在我们要做的就是避免这种情况，引申一下，不止是钥匙这些场景道具，包括与老人的对话的状态也应该被保存，所以总结一下我们的任务就是保存场景中所有可交互事物的状态。

我们用字典<ItemName, bool>来存储道具和其状态。拆分一下，有两种情况需要保存或者更新物体状态：1. 初次加载场景时，需要将场景的所有Item（挂上标签）都添加进字典。2.当点击某一可交互物体时，更新其在字典中的状态。

而当再次加载场景时，需要通过字典中保存的道具状态复现场景。

在EventHandler中添加场景即将被关闭和场景加载后两个事件：

```C#
public static event Action BeforeSceneUnloadEvent;
public static void CallBeforeSceneUnloadEvent()
{
    BeforeSceneUnloadEvent?.Invoke();
}
public static event Action AfterSceneLoadEvent;
public static void CallAfterSceneLoadEvent()
{
    AfterSceneLoadEvent?.Invoke();
}
```

添加新的脚本ObjectManager.cs:

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ObjectManager : MonoBehaviour
{
    private Dictionary<ItemName, bool> itemAvailableDict = new Dictionary<ItemName, bool>();

    private void OnEnable()
    {
        EventHandler.BeforeSceneUnloadEvent += OnBeforeSceneUnloadEvent;
        EventHandler.AfterSceneLoadEvent += OnAfterSceneLoadEvent;
        EventHandler.UpdateUIEvent += OnUpdateUIEvent;
    }
    private void OnDisable()
    {
        EventHandler.BeforeSceneUnloadEvent -= OnBeforeSceneUnloadEvent;
        EventHandler.AfterSceneLoadEvent -= OnAfterSceneLoadEvent;
        EventHandler.UpdateUIEvent -= OnUpdateUIEvent;
    }
    /// <summary>
    /// 当场景即将被卸载时触发
    /// </summary>
    private void OnBeforeSceneUnloadEvent()
    {
        foreach(var item in FindObjectsOfType<Item>())
        {
            // 同下，但是个人认为这里只是个保险
            if (!itemAvailableDict.ContainsKey(item.itemName)){
                itemAvailableDict.Add(item.itemName, true);
            }
        }
    }
    /// <summary>
    /// 当场景被加载后触发
    /// </summary>
    private void OnAfterSceneLoadEvent()
    {
        // 查找场景中所有挂载了Item脚本的物体的Item脚本对象
        foreach (var item in FindObjectsOfType<Item>())
        {
            // 如果字典中不包含该道具（初次加载场景），添加进字典
            if (!itemAvailableDict.ContainsKey(item.itemName)){
                itemAvailableDict.Add(item.itemName, true);
            }
            // 否则根据存储的状态bool值设置对象的激活状态
            else
            {
            	item.gameObject.SetActive(itemAvailableDict[item.itemName]);
            }
        }
    }
    /// <summary>
    /// 当有道具被添加进背包时触发
    /// </summary>
    /// <param name="itemDetail"></param>
    /// <param name="index"></param>
    public void OnUpdateUIEvent(ItemDetail itemDetail, int index)
    {
        if(itemDetail != null)
        {
            itemAvailableDict[itemDetail.itemName] = false;
        }
    }
}
```

现在，我们加载H2，拾取钥匙，然后切换到H1，再回到H2，现在钥匙不会重新生成了。

我们再补充一点逻辑，不用在Editor中加载某个场景，可以直接使用TransitionManager的属性来设置游戏开始时加载的场景。

更改TransitionManager脚本如下：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class TransitionManager : SingletonMono<TransitionManager>
{
    public Teleport.Scene startScene;
    public CanvasGroup fadeCanvasGroup;
    public float fadeDuration = 1f;
    private bool isFade;
    void Start()
    {
        StartCoroutine(TransitionToScene(Teleport.Scene.None, startScene));
    }
    public void Transition(Teleport.Scene sceneFrom, Teleport.Scene sceneToGo)
    {
        if(!isFade)
            StartCoroutine(TransitionToScene(sceneFrom, sceneToGo));
    }
    private IEnumerator TransitionToScene(Teleport.Scene sceneFrom, Teleport.Scene sceneToGo)
    {
        yield return StartCoroutine(Fade(1));
        if(sceneFrom != Teleport.Scene.None) //设置了sceneFrom才卸载场景
        {
            EventHandler.CallBeforeSceneUnloadEvent();
            yield return SceneManager.UnloadSceneAsync(sceneFrom.ToString());
        }
        

        if(sceneToGo != Teleport.Scene.None){
            yield return SceneManager.LoadSceneAsync(sceneToGo.ToString(), LoadSceneMode.Additive);
            // 设置新场景为激活场景
            Scene newScene = SceneManager.GetSceneByName(sceneToGo.ToString());
            SceneManager.SetActiveScene(newScene);
            EventHandler.CallAfterSceneLoadEvent();
        }
        yield return StartCoroutine(Fade(0)); 
    }
    /// <summary>
    /// 淡入淡出场景
    /// </summary>
    /// <param name="targetAlpha"></param>
    /// <returns></returns>
    private IEnumerator Fade(float targetAlpha)
    {
        isFade = true;
        fadeCanvasGroup.blocksRaycasts = true;

        float speed = Mathf.Abs(fadeCanvasGroup.alpha - targetAlpha) / fadeDuration;

        while(!Mathf.Approximately(fadeCanvasGroup.alpha, targetAlpha))
        {
            fadeCanvasGroup.alpha = Mathf.MoveTowards(fadeCanvasGroup.alpha, targetAlpha, speed * Time.deltaTime);
            yield return null;
        }

        fadeCanvasGroup.blocksRaycasts = false;
        isFade = false;
    }
}
```

这样既可以设置TransitionManager的startScene后无游戏场景加载启动游戏，也可以将其设为None并加载一个场景后启动游戏。

# 互动对象逻辑

考虑到每个对象都有通用的需求，所以我们建一个基类，后续再让每个互动对象类继承。

创建新脚本Interactive/Interactive.cs脚本：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Interactive : MonoBehaviour
{
    public ItemName requireItem;// 需要的道具名
    public bool isDone;// 是否已经触发过

    public void CheckItem(ItemName itemName)
    {
        if (itemName == this.requireItem)
        {
            isDone = true;
            // 使用这个物品
            OnClickedAction();
        }
        return;
    }
	// virtual关键字表示该方法可以被派生类override
    protected virtual void OnClickedAction()
    {

    }
    public virtual void EmptyClicked()
    {
        Debug.Log("空点");
    }
}
```

为H2的老奶奶添加碰撞体作为Trigger，更改标签为Interactive，添加Interactive脚本。

然后在CursorManager中添加检测点击时是否携带道具的bool变量holdItem，在ClickAction()的Switch下添加：

```C#
case "Interactive":
    var interactive = clickObject.GetComponent<Interactive>();
    if(holdItem)
        interactive?.CheckItem(currentItemName);
    else
        interactive?.EmptyClicked();
    break;
```

然后我们需要更新UI逻辑——当道具栏中有道具时，点击道具栏将出现一个手的图标并且跟随鼠标移动，还要记录下当前手持的是什么道具。

在FadeCanvas下新建Image HandImage，拖入手的素材精灵，简单更改hand素材的pivot并设置Set Native Size，勾去Reycast Target属性以确保手的UI不会阻碍我们触发鼠标点击事件。在CursorManager中添加新的成员变量：

```
private ItemName currentItemName;
public RectTransform hand;
```

在EventHandler中创建一个新的事件：

```C#
public static event Action<ItemDetail, bool> ItemSelectedEvent;
public static void CallItemSelectedEvent(ItemDetail itemDetail, bool isSelected)
{
    ItemSelectedEvent?.Invoke(itemDetail, isSelected);
}
```

添加订阅和退订，当CursorManager接收到消息后，触发处理方法OnItemSelectedEvent：

```C#
private void OnEnable()
{
    EventHandler.ItemSelectedEvent += OnItemSelectedEvent;
}
private void OnDisable()
{
    EventHandler.ItemSelectedEvent -= OnItemSelectedEvent;
}


private void OnItemSelectedEvent(ItemDetail itemDetail, bool isSelected)
{
    // holdItem继承传入的isSelected的值
    holdItem = isSelected;
    if (isSelected)
    {
        // 在CursorManager中保存当前持有的道具名
        currentItemName = itemDetail.itemName;
    }
    // 设置手的启用状态同步
    hand.gameObject.SetActive(holdItem);
}
```

为ItemToolTip添加控制脚本：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class ItemToolTip : MonoBehaviour
{
    public TextMeshProUGUI itemNameText;

    public void UpdateItemName(ItemName itemName)
    {
        itemNameText.text = itemName switch
        {
            ItemName.Key => "信箱钥匙",
            ItemName.Ticket => "船票",
            _ => ""// default
        };
    }
}

```

在SlotUI脚本中添加鼠标监听：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.EventSystems;
public class SlotUI : MonoBehaviour, IPointerClickHandler, IPointerExitHandler, IPointerEnterHandler
{
    // ……
    private bool isSelected;
    public ItemToolTip toolTip;
    // ……
    public void OnPointerClick(PointerEventData eventData)
    {
        isSelected = !isSelected;
        EventHandler.CallItemSelectedEvent(currentItem, isSelected);
    }
    public void OnPointerEnter(PointerEventData eventData)
    {
        if (this.gameObject.activeInHierarchy)
        {
            toolTip.gameObject.SetActive(true);
            toolTip.UpdateItemName(currentItem.itemName);
        }
    }
    public void OnPointerExit(PointerEventData eventData)
    {
        toolTip.gameObject.SetActive(false);
    }
}

```

保存代码，返回Editor，确认已经设置Interactive、CursorManager、Slot和ItemToolTip脚本的属性值。还要**在SampleScene下新建一个UI>EventSystem对象**，如果没有添加EventSystem，切换到其他场景时SlotUI的鼠标事件不会触发。运行游戏，直接点击老奶奶会Debug.Log("空点")，拾取钥匙，鼠标进入道具栏时，道具的名字栏显示，移出时消失。点击钥匙，出现手的图片，并且能随鼠标移动而移动，再次点击道具栏，手消失，功能全部实现。

## 实现第一个互动对象

向H4中添加关闭的MailBox对象（注意预先更改两个状态的pivot避免转换时有视觉错位），标签设为Interactive。在Interactive文件夹下新建脚本MailBox.cs，MailBox类继承Interactive类。

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MailBox : Interactive
{
    private SpriteRenderer spriteRenderer;
    private BoxCollider2D boxCollider;
    public Sprite openSprite;
    private void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
        boxCollider = GetComponent<BoxCollider2D>();
    }
    // override表示重写了Interactive类的OnClickedAction方法
    protected override void OnClickedAction()
    {
        spriteRenderer.sprite = openSprite;
        boxCollider.enabled = false;
        Debug.Log("打开邮箱");
    }
}
```

添加H4和H2的Teleport并配置好参数。运行游戏，在H2中拾取钥匙并在H4中点击钥匙并再次点击邮箱，此时邮箱打开，表示逻辑正确。

然后为打开的邮箱添加ticket(如果邮箱和船票的图层不对则将其Sorting Layer更改为Back），添加Item脚本。先将图片精灵更改为Open状态下的，然后将船票一角的图片拖入MailBox的子对象，调整位置并设置碰撞体，添加Item脚本并设置为Ticket。

然后我们需要设置当邮箱处于关闭状态时，船票隐藏。除了在OnClickedAction()内添加逻辑外，当切换场景的时候也应该对邮箱的状态进行判断。所以仍然需要订阅AfterSceneLoadedEvent：

```C#
 private void OnEnable()
 {
     EventHandler.AfterSceneLoadedEvent += OnAfterSceneLoadedEvent;
 }
 private void OnDisable()
 {
     EventHandler.AfterSceneLoadedEvent -= OnAfterSceneLoadedEvent;
 }
protected override void OnClickedAction()
{
    spriteRenderer.sprite = openSprite;
    boxCollider.enabled = false;
    // 同样在这里添加
    transform.GetChild(0).gameObject.SetActive(false);
    Debug.Log("打开邮箱");
}
private void OnAfterSceneLoadedEvent()
{
    if (!isDone)
    {
        transform.GetChild(0).gameObject.SetActive(false);
    }
    else
    {
        spriteRenderer.sprite = openSprite;
        boxCollider.enabled = false;
    }
}

```

编写好脚本后确认ticket在MailBox下且标签设为Item，现在运行游戏当邮箱未被打开时船票就隐藏啦。并且船票可以被添加到物品栏中。

现在还需要解决几个问题：1. 钥匙被使用之后物品栏钥匙仍然存在。2. 在邮箱上使用钥匙过后手的UI并未消失。 3. 当打开邮箱后，切换场景再回来时，邮箱仍然是关闭状态。

1. #### 道具使用后UI更新

   先解决物品栏UI更新的问题，在Interactive中我们还没有实现注释的道具使用的逻辑代码。添加一个新事件ItemUsedEvent：

   ```C#
   public static event Action<ItemName> ItemUsedEvent;
   public static void CallItemUsedEvent(ItemName itemName)
   {
       ItemUsedEvent?.Invoke(itemName);
   }
   ```

   在Interactive.cs的注释处调用事件触发，然后在InventoryManager中添加订阅和退订以及事件处理方法：

   ```C#
   private void OnEnable()
   {
       EventHandler.ItemUsedEvent += OnItemUsedEvent;
   }
   private void OnDisable()
   {
       EventHandler.ItemUsedEvent -= OnItemUsedEvent;
   }
   
   public void OnItemUsedEvent(ItemName itemName)
   {
       itemList.Remove(itemName);
       // 尚未实现物品栏中道具的切换，暂时只实现使用单一物品效果
       if(itemList.Count == 0)
       {
           // 调用UI更新事件触发，消息会发送给InventoryUI和ObjectManager
           EventHandler.CallUpdateUIEvent(null, -1);
       }
   }
   ```

   现在不仅是Slot的显示更新，包括两边的切换按钮也会置为不可点击的状态。

   2. #### 手的UI在互动后更新

      然后实现互动后更新手的状态：

      在CursorManager中添加ItemUsedEvent事件的订阅和退订以及处理方法：

      ```C#
      private void OnEnable()
      {
          EventHandler.ItemSelectedEvent += OnItemSelectedEvent;
          EventHandler.ItemUsedEvent += OnItemUsedEvent;
      }
      private void OnDisable()
      {
          EventHandler.ItemSelectedEvent -= OnItemSelectedEvent;
          EventHandler.ItemUsedEvent -= OnItemUsedEvent;
      }
      
      private void OnItemUsedEvent(ItemName obj)
      {
          // 当前手持道具状态和道具名字也要同步更新
          currentItemName = ItemName.None;
          holdItem = false;
          hand.gameObject.SetActive(false);
      }
      ```

   3. #### 保存可互动物品的状态

      方法和保存场景中道具状态类似：

      ```C#
      using ……
      
      public class ObjectManager : MonoBehaviour
      {
          private Dictionary<ItemName, bool> itemAvailableDict = new Dictionary<ItemName, bool>();
          // 这里以可互动物品对象在场景中的名字作为Key
          private Dictionary<string, bool> interactiveStateDict = new Dictionary<string, bool>();
      
          private void OnEnable()
          {
              ……
              EventHandler.UpdateUIEvent += OnUpdateUIEvent;
          }
          private void OnDisable()
          {
              ……
              EventHandler.UpdateUIEvent -= OnUpdateUIEvent;
          }
          /// <summary>
          /// 当场景即将被卸载时触发
          /// </summary>
          private void OnBeforeSceneUnloadEvent()
          {
              ……
              // 跟道具状态字典是一个道理
              foreach(var interactive in FindObjectsOfType<Interactive>())
              {
                  if (interactiveStateDict.ContainsKey(interactive.name))
                      interactiveStateDict[interactive.name] = interactive.isDone;
                  else
                      interactiveStateDict.Add(interactive.name, interactive.isDone);
              }
          }
          /// <summary>
          /// 当场景被加载后触发
          /// </summary>
          private void OnAfterSceneLoadEvent()
          {
              ……
              foreach(var interactive in FindObjectsOfType<Interactive>())
              {
                  // 如果字典中包含该交互物体，根据存储的状态bool值设置物体的激活状态
                  if (interactiveStateDict.ContainsKey(interactive.name))
                      interactive.isDone = interactiveStateDict[interactive.name];
                  else
                      interactiveStateDict.Add(interactive.name, interactive.isDone);
              }
          }
          
      	……
      }
      ```

      现在再次运行游戏，上面三个问题就全部解决啦，你还可以测试一下打开邮箱但不拿走船票，当切换回来的时候船票也依然会在打开的邮箱中。



# 背包道具切换

首先在InventoryUI中添加切换的方法：

```C#
/// <summary>
/// 切换道具栏的物品
/// </summary>
/// <param name="offset">向左为-1，向右为1</param>
public void SwitchItem(int offset)
{
    var index = currentIndex + offset;
    // 控制左右按钮的点击状态
    leftButton.interactable = (index > 0);
    rightButton.interactable = (index < (total - 1));

    // 调用道具切换触发事件以驱动InventoryManager中当前道具的更新
    EventHandler.CallChangeItemEvent(index);
}
```

**这里和教程不一样的是，教程代码的逻辑只能维持上限为二的道具之间的切换，为此在InventoryUI中添加了一个新的变量total记录道具总数， 并添加一个事件，当InventoryManager添加或者删除道具时，调用该事件。InventoryUI订阅该事件以更新total即可。**

在EventSystem中添加道具切换事件：

```C#
// 物品栏中道具切换事件
public static event Action<int> ChangeItemEvent;
public static void CallChangeItemEvent(int offset)
{
    ChangeItemEvent?.Invoke(offset);
}
```

在InventoryManager中添加ChangeItemEvent事件的订阅和退订。添加处理方法：

```C#
private void OnEnable()
{
    ……
    EventHandler.ChangeItemEvent += OnChangeItemEvent;
}
private void OnDisable()
{
    ……
    EventHandler.ChangeItemEvent -= OnChangeItemEvent;
}
private void OnChangeItemEvent(int index){
    if(index >= 0 && index < itemList.Count)
    {
        ItemDetail item = itemDetailList.GetItemDetail(itemList[index]);
        // 再调用CallUpdateUIEvent事件反过来驱动UI更新
        EventHandler.CallUpdateUIEvent(item, index);
    }
}
```

按说这个时候切换的逻辑就已经完成了，但是运行游戏，发现拾取两个道具之后，两边的切换按钮还是不可点击状态。这是当然的，因为当拾起道具的时候根本没触发`SwitchItem`啊!

所以这个地方的主干逻辑就是：按钮触发`InventoryUI`的`SwitchItem`方法，`InventoryUI`通知`InventoryManager`以获取切换后的当前道具，然后`InventoryManager`用`UpdateUIEvent`事件在将当前道具的信息传递回来（虽然同时通知了`ObjectManager`，但是实际`ObjectManager`中只负责在临时性存储中更新道具的拾取状态，所以并无影响），然后`InventoryUI`根据信息控制左右按钮的状态以及`SlotUI`的显示（后者之前已经实现了）。

理完逻辑你就发现：InventoryManager中的CallUpdateUIEvent不仅是这里需要调用，当场景切换（加载）的时候也会调用，这不就完了嘛！直接添加`InventoryUI`的`OnUpdateUIEvent`的逻辑，让它也能更新左右按钮的状态，完事儿！

所以对`InventoryUI`的`OnUpdateUIEvent`添加左右按钮的状态更新代码：

```C#
private void OnUpdateUIEvent(ItemDetail itemDetail, int index)
{
    if (itemDetail == null)
    {
        slotUI.SetEmpty();
        currentIndex = -1;
        leftButton.interactable = false;
        rightButton.interactable = false;
    }
    else
    {
        currentIndex = index;
        slotUI.SetItem(itemDetail);
		// 添加以下代码
        if(index > 0)
        {
            leftButton.interactable = true;
        }
        else if(index == -1)
        {
            leftButton.interactable = false;
            rightButton.interactable = false;
        }
    }
}
```

现在拾取两个道具就能切换自如了。还有一点点小瑕疵就是——刚进入游戏是左右按钮没有置为不可点击状态。这个也简单，只要在`InventoryManager`接受`AfterSceneLoadedEvent`事件，判断如果`itemList`为空即`CallUpdateUIEvent(null, -1)`使`InventoryUI`按上面这个逻辑跑一下即可。

```C#
private void OnEnable()
{
    ……
    EventHandler.AfterSceneLoadedEvent += OnAfterSceneLoadedEvent;
}
private void OnDisable()
{
    ……
    EventHandler.AfterSceneLoadedEvent -= OnAfterSceneLoadedEvent;
}

private void OnAfterSceneLoadedEvent()
{
    if(itemList.Count == 0)
    {
        EventHandler.CallUpdateUIEvent(null, -1);
    }
}
```

# UI角色对话

根据要求，需要实现与人物的点击对话，并且当有关键道具时触发额外的对话。

新建Dialog文件夹，对对话内容实现持久数据化：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName = "DialogData_SO", menuName = "Dialog/DialogData_SO")]
public class DialogData_SO : ScriptableObject
{
    public List<string> dialogueList;
}
```

保存代码并编译后，在Assets/Game Data下新建Dialog文件夹，新建Dialog>DialogData_SO资源，命名为CharacterEmpty（不明所以的文件名），将介绍.txt的内容一句句添加进资源，然后再添加一个新的DialogData_SO资源，命名为Finish，将特殊触发对话的句子添加进去，这样对话数据就好了。



接下来制作UI，找到素材中的对话框精灵，我们需要对其进行修改。进入Sprite Editor，右下角设置border值如图：

![image-20241009004123359](picture/image-20241009004123359.png)

Apply过后，在SampleScene的MainCanvas中添加panel，将对话框素材作为其Source Image，并且选择Image Type为**Sliced**，调整大小和位置，将其放在H2老奶奶旁边。

现在我们来解释为什么要这样操作：如果不选择Image Type为Slice，那么经过放缩后效果是这样的：

![image-20241009004501118](picture/image-20241009004501118.png)

而选择Image Type为Sliced，同样情况下显示是这样的：

![image-20241009004557613](picture/image-20241009004557613.png)

发现区别了吗？区别在于边框是否根据缩放而缩放，当选择Sliced的时候，缩放的填充都是border内部的部分，所以边框不会有明显的变粗或者锯齿，因而看起来也更美观。



现在我们要将对话框的尖角添加进来：在Panel下添加Image，将对话框尖角的素材添加进来，翻转图像将Scale.X设为-1，设置Anchor和位置在左下角，细调位置和大小使得看上去无缝衔接即可。



添加TMP子对象，调整字体和大小以适配对话框。（注意TMP放在Image的下面，以控制显示顺序）

## 设置自适应

设置自适应的原因是：我们不会为每一句对话都添加一个这样的对话框对象，我们需要复用对话框并且完美显示不同内容。

先为对话框尖角对象添加Layout Element组件，勾选Ignore Layout，这样尖角会保持自己的RectTransfrom规则，不受自适应调整的影响。

然后为Panel添加Content Size Fitter组件，选择Vertical Fit属性值为Preferred Size，这样文本框的Height会随文本框子对象的大小变化而变化；再添加Vertical Layout Group组件，修改Padding均为15，勾选**Control Child Size**(: 控制布局组是否控制其子布局元素的宽度和高度)的Width和Height，现在对话框就既美观又可复用了。

简而言之，在这里这三个组件的作用分别是：

* Layout Element：排除不应纳入自适应的子对象
* Content Size Fitter：文本框自适应父对象的大小
* Vertical Layout Group：父对象管控子对象的边缘宽度以及子对象自己的宽高（即当文本变化时，TMP自动更改）

## 控制对话UI显示

现在需要设置点击老奶奶触发普通对话，将船票与老奶奶交互触发特殊对话的功能。

先着手dialogue的Controller脚本的编写，脚本里需要拿到之前编写好的两个DialogData_SO资产，并且需要用栈或者下标维护当前显示到哪了，这个Controller与DialogueUI**并非死绑**，因此还是需要用事件通知来调控UI显示。

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DialogueController : MonoBehaviour
{
    public DialogData_SO dialogueEmpty;
    public DialogData_SO dialogueFinish;
    /// <summary>
    /// 控制是否正在进行对话
    /// </summary>
    private bool isTalking = false;
    private Stack<string> dialogueEmptyStack;
    private Stack<string> dialogueFinishStack;

    private void Awake()
    {
        FillDialogueStack();
    }
    /// <summary>
    /// 重填对话栈，开始时也要调用
    /// </summary>
    private void FillDialogueStack()
    {
        dialogueEmptyStack = new Stack<string>();
        dialogueFinishStack = new Stack<string>();
        for (int i = dialogueEmpty.dialogueList.Count - 1; i >= 0; i--)
        {
            dialogueEmptyStack.Push(dialogueEmpty.dialogueList[i]);
        }
        for (int i = dialogueFinish.dialogueList.Count - 1; i >= 0; i--)
        {
            dialogueFinishStack.Push(dialogueFinish.dialogueList[i]);
        }
    }
    /// <summary>
    /// 展示普通对话
    /// </summary>
    public void ShowDialogueEmpty()
    {
        if (!isTalking)
            StartCoroutine(DialogueRoutine(dialogueEmptyStack));
    }
    /// <summary>
    /// 展示特殊对话
    /// </summary>
    public void ShowDialogueFinish()
    {
        
        if(!isTalking)
            StartCoroutine(DialogueRoutine(dialogueFinishStack));
    }
    /// <summary>
    /// 用协程来播放对话并维护对话栈
    /// </summary>
    /// <param name="data">如果为栈空，则隐藏对话框</param>
    /// <returns></returns>
    private IEnumerator DialogueRoutine(Stack<string> data)
    {
        isTalking = true;
        if (data.TryPop(out string dialogue))
        {
            EventHandler.CallShowDialogueEvent(dialogue);
            yield return null;
            isTalking = false;
        }
        else
        {
            EventHandler.CallShowDialogueEvent(string.Empty);
            FillDialogueStack();
            isTalking = false;
        }
    }
}
```

对于这段代码的一些理解：

* 为什么要用协程，尝试将其改为常规写法，运行起来也没什么问题。我在思考isTalking的作用后有所理解。isTalking实际就是线程的一个锁，当显示对话时拿取这个锁，显示后释放这个锁，在这里看似没什么用（毕竟连当显示常规对话时不能播放特殊对话都做不到），原因在于没有播放的持续动作（比如逐字显示或者等待声音播放等等），本身也没有想要限制播放速度什么的。为了防止等待动作阻塞当前进程，必须使用协程。



编写好DialogueController脚本后，在之前建好的对话的Panel上新建一个空的父对象Dialogue，添加新脚本DialogueUI，监听ShowDialogueEvent事件并更改显示：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;
public class DialogueUI : MonoBehaviour
{
    public GameObject panel;
    public TextMeshProUGUI text;


    private void OnEnable()
    {
        EventHandler.ShowDialogueEvent += ShowDialogue;
    }
    private void OnDisable()
    {
        EventHandler.ShowDialogueEvent -= ShowDialogue;
    }
    private void ShowDialogue(string dialogue)
    {
        if(dialogue != string.Empty)
        {
            panel.SetActive(true);
        }
        else
        {
            panel.SetActive(false);
        }
        text.text = dialogue;
    }
}
```

在老奶奶对象上挂载刚刚写的DialogueController脚本，并修改之前添加的空脚本CharacterH2。之前我们已经将老奶奶对象打上了Interactive标签，当鼠标点击老奶奶时会触发Interactive类的`CheckItem`或者`EmptyClicked`方法：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(DialogueController))]
public class CharacterH2 : Interactive
{
    DialogueController dialogueController;
    private void Awake()
    {
        dialogueController = GetComponent<DialogueController>();
    }
    /// <summary>
    /// 编写空点时的逻辑——如果已经触发过特殊对话，则展示特殊对话，否则展示普通对话
    /// </summary>
    public override void EmptyClicked()
    {
        if(!isDone)
            dialogueController.ShowDialogueEmpty();
        else
            dialogueController.ShowDialogueFinish();
    }

    protected override void OnClickedAction()
    {
        dialogueController.ShowDialogueFinish();
    }
}
```

在Unity Editor中设置公开变量的值，这样对话的播放就算完成了，空点和用船票触发特殊事件都没有问题。



但是当我们切换场景时，发现对话框并未消失，这是因为我们没有控制好场景切换和对话播放之间的关系。这里要将逻辑改为：“如果正在播放对话，则不可切换场景。”

在Enums.cs中声明新的枚举类：

```C#
public enum GameState
{
    Pause, GamePlay
}
```

添加新事件`Action<GameState> GameStateChangedEvent`，在TransitionManager脚本中添加事件订阅，编写处理方法和新变量：

```C#
private bool canTransition = true; // 提前设置，否则进入游戏是如果是H2场景，则不可切换场景

public void Transition(Teleport.Scene sceneFrom, Teleport.Scene sceneToGo)
{
    if(!isFade && canTransition)//添加新的判断
        StartCoroutine(TransitionToScene(sceneFrom, sceneToGo));
}

/// <summary>
/// 监听游戏状态改变事件
/// 如果改变为Pause状态，则不允许切换场景
/// </summary>
void OnGameStateChangeEvent(GameState current)
{
    canTransition = current == GameState.GamePlay;
}
```

在DialogueController中添加事件广播：

```C#
if (data.TryPop(out string dialogue))
{
    ...
    EventHandler.CallGameStateChangedEvent(GameState.Pause);
}
else
{
    ...
    EventHandler.CallGameStateChangedEvent(GameState.GamePlay);
}
```

这样就不会在播放对话时能切换场景了。

# 小游戏设计与制作

![H2效果图介绍](picture/H2效果图介绍.png)

我们要实现H2场景中这个密码门上的小游戏，这个地方的处理让我感到非常精妙的是——将小游戏也作为一个另外的场景。这个思路确实非常的正确，因为小游戏和正常的场景一样：铺满屏幕，有可用鼠标交互的对象（我甚至觉得哪怕没有共同点都可以设为一个新的场景），在能实现效果的同时也简化了代码和场景设计。

先为小游戏添加新场景H2A，添加基本的背景图片，在Teleport脚本中的Scene枚举类型下添加H2A枚举值，*注意最好添加在最后，否则Editor中所有已设置的Scene的值可能会发生错位改变*。将新场景添加进Hierarchy中。

按照Teleport的做法，给门对象添加Collider和Teleport脚本（别忘了打上标签，由于使用CursorManager来控制的点击对象的判断和点击事件的触发，所以标签一定是必须的），将SceneFrom和SceneTo参数配置好后就可以转移到H2A场景了。

## 小游戏的初始化（静态生成）

![H2A效果图](picture/H2A效果图-17290130420523.png)

小游戏场景中的基本元素有哪些？有槽位、带符号的球，连接不同槽位的线也是，因为这是游戏逻辑的具象化，并且在槽位固定的情况下，我们应该实现存储球和线的情况并且在游戏开始或者进入场景时自动生成，这样策划设计游戏时也不用频繁地给场景添加对象。

先制作固定的槽位，记得添加监听鼠标事件用的collider并打上标签Interactive（鼠标按住对象并Ctrl+D可直接生成复制体并被鼠标拖动），规定一个顺序——从最下的开始顺时针旋转一周进行命名。添加脚本Holder：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Holder : Interactive
{
    public bool isEmpty;
}
```

然后我们来制作线，素材给的是很短的一截线，我们想要达到连接任意两个槽位之间的功能，这时就应该请出新的魔法——Line Effect了，这个Line Effect厉害在于可以使用LineRenderer将材质以有宽度的线条渲染出来，这也就意味着能通过设置顶点的形式来确定其位置。

在Hierarchy中新建Effect>Line对象，这时候是看不到线条的，因为没有设置顶点和宽度，先临时设置顶点和宽度的值，调整Sorting Layer使其能够在背景之上槽位之下显示出来，将任意两个槽位的Position赋给LineRenderer的Positions下的成员，这样一条白线就连通了两个槽位了。

然后来制作渲染用的材质，查看线条素材发现两侧是有半透明的效果的，为了保留这种效果，在Assets下新建一个文件夹Material并新建一个Material材质，在Shader一行设为Unlit>Transparent，将素材拖入即制作完成。将该材质赋给LineRenderer下的Materials列表，调整宽度水平线的y值至效果合适，线对象就制作好了，将其保存为Prefab备用。

在制作Ball对象之前，我们先得设计关卡数据的保存和读取。添加脚本GameH2A_SO以定义球、线的相关参数以及ScriptableObject子类:

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName = "GameH2A_SO", menuName = "MiniGame Data/GameH2A_SO", order = 1)]
public class GameH2A_SO : ScriptableObject
{
    // Header标签使得在Editor中编辑资产时会在该属性前添加描述
    [Header("球的名字和对应的图片")]
    public List<BallDetails> ballDetailList;
    [Header("连接线的起始点和终点")]
    public List<Connection> connections;
    [Header("球的初始顺序")]
    public List<BallName> startBallOrder;
    public BallDetails GetBallDetails(BallName ballName)
    {
        return ballDetailList.Find(b => b.ballName == ballName);
    }
}

[System.Serializable]
public class BallDetails
{
    public BallName ballName;
    public Sprite wrongSprite;
    public Sprite rightSprite;
}

[System.Serializable]
public class Connection
{
    public int from;
    public int to;
}
```

忘了BallName枚举类还没定义，在Enum.cs中添加：

```C#
public enum BallName
{
    None,
    B1,
    B2,
    B3,
    B4,
    B5,
    B6
}
```

在Assets>Game Data>MiniGame Data下添加新的GameH2A_SO资产，命名为MiniGame_Week1，在该资产中添加球和线的参数和球的初始顺序：

![image-20241016015928933](picture/image-20241016015928933.png)

有了BallDetail我们再来定义Ball类：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Ball : MonoBehaviour
{
    public SpriteRenderer spriteRenderer;
    public BallDetails ballDetails;
    public bool isMatch;
    private void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
    }

    public void SetBall(BallDetails ball)
    {
        this.ballDetails = ball;
        if (isMatch)
        {
            SetRight();
        }
        else
        {
            SerWrong();
        }
    }

    public void SetRight()
    {
        spriteRenderer.sprite = ballDetails.rightSprite;
    }
    public void SerWrong()
    {
        spriteRenderer.sprite = ballDetails.wrongSprite;
    }
}
```

新建Ball对象，挂载该脚本，确认能够正常显示后，将Sprite设置为None，然后制作成预制件。



现在万事俱备！我们需要一个Controller来（先实现）控制进入场景时读取MiniGame_Week1.asset中的数据并在对应的槽位生成球和线：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameController : MonoBehaviour
{
    public GameH2A_SO gameData;
    // 方便控制所有线条，将所有的线对象生成在已有的一个lineParent空对象下
    public GameObject lineParent;
    public LineRenderer linePrefab;
    public Ball ballPrefab;
    public Transform[] holderTransforms;

    public void Start()
    {
        DrawLine();
        CreateBall();
    }
    public void DrawLine()
    {
        foreach (var Item in gameData.connections)
        {
            LineRenderer line = Instantiate(linePrefab, lineParent.transform);
            line.SetPosition(0, holderTransforms[Item.from].position);
            line.SetPosition(1, holderTransforms[Item.to].position);
        }
    }

    public void CreateBall()
    {
        for (int i = 0; i < gameData.startBallOrder.Count; i++)
        {
            if (gameData.startBallOrder[i] == BallName.None)
            {
                holderTransforms[i].GetComponent<Holder>().isEmpty = true;
                continue;
            }
            Ball ball = Instantiate(ballPrefab, holderTransforms[i]);
            holderTransforms[i].GetComponent<Holder>().isEmpty = false;
            ball.SetBall(gameData.GetBallDetails(gameData.startBallOrder[i]));
        }
    }
}
```

在场景下创建新的空对象GameController，添加GameController脚本，将相应的东西拖入参数中（注意：1. 创建一个LineParent空对象 2. 拖入Holders时一定要按照下标顺序一一对应 3. 预先要设置好Ball和Line预制件的Position，不然会错位）。测试进入小游戏场景是否正确生成球和线即可。

现在这个代码还是存在问题的，比如没有判定初始位置的Ball是否在正确的位置上。

## 小游戏的逻辑实现

这个小游戏的逻辑是：点击一个有球的槽位能将球传递给其连接的另一个空的槽位（只存在一个空的槽位）。当所有槽位的状态态都是isMatch的时候，游戏通过。

首先为Holder添加检查逻辑：

```C#
public class Holder : Interactive
{
    public BallName matchBall;
    public Ball currentBall;
    public HashSet<Holder> linkHolders = new HashSet<Holder>();
    public bool isEmpty;
    /// <summary>
    /// 初始化或者发生移动时，更改槽内球的信息，并且检查是否匹配
    /// </summary>
    /// <param name="ball"></param>
    public void SetAndCheckIfMatch(Ball ball)
    {
        currentBall = ball;
        if(ball.ballDetails.ballName == matchBall)
        {
            currentBall.isMatch = true;
            currentBall.SetRight();
        }
        else
        {
            currentBall.isMatch = false;
            currentBall.SerWrong();
        }
    }
}
```

不难发现当且仅当初始化和交换两个槽位的球时需要调用`SetAndCheckIfMatch`方法。

在GameController中添加初始化时添加槽位连接关系和检查球的位置的代码：

```C#
public void DrawLine()
{
    foreach (var connection in gameData.connections)
    {
        ......

        // 创建每一个Holder的连接关系
        holderTransforms[connection.from].GetComponent<Holder>().linkHolders.Add(holderTransforms[connection.to].GetComponent<Holder>());
        holderTransforms[connection.to].GetComponent<Holder>().linkHolders.Add(holderTransforms[connection.from].GetComponent<Holder>());
    }
}

public void CreateBall()
{
    for (int i = 0; i < gameData.startBallOrder.Count; i++)
    {
        if (gameData.startBallOrder[i] == BallName.None)
        {
            holderTransforms[i].GetComponent<Holder>().isEmpty = true;
            continue;
        }
        Ball ball = Instantiate(ballPrefab, holderTransforms[i]);
		// ball.SetBall(gameData.GetBallDetails(gameData.startBallOrder[i]));放在这更合理
        // 初始化时设置槽位中球的初始状态
        holderTransforms[i].GetComponent<Holder>().SetAndCheckIfMatch(ball);
        holderTransforms[i].GetComponent<Holder>().isEmpty = false;
        ball.SetBall(gameData.GetBallDetails(gameData.startBallOrder[i]));
    }
}
```

然后再回到Holder重载Interactive类的空点事件方法：

```C#
/// <summary>
/// 逻辑是当该槽位有球时，将球移动到与之连接的空槽位上（如果有），并且更新两个槽位以及球的信息
/// </summary>
public override void EmptyClicked()
{
    foreach (var holder in linkHolders)
    {
        if(holder.isEmpty){
            // 移动球
            currentBall.transform.position = holder.transform.position;
            currentBall.transform.SetParent(holder.transform);// 更改球的所属槽位

            // 交换球
            holder.SetAndCheckIfMatch(currentBall);
            this.currentBall = null;

            // 改变两个槽位信息
            this.isEmpty = true;
            holder.isEmpty = false;
        }
    }
}
```

运行小游戏，发现刚开始时所有的小球都处于位置正确的状态，debug发现是因为所有的Holder还未设置matchBall参数。设置好后暂时没问题了，但是自己测试的时候又发现如果初始本来就在正确的位置上的球又还是显示的是错误时的状态，想要解决这个问题，将GameController中`ball.SetBall(gameData.GetBallDetails(gameData.startBallOrder[i]));`置于` holderTransforms[i].GetComponent<Holder>().SetAndCheckIfMatch(ball);`前即可。

现在可以游玩小游戏了，但是还没有写判断通关以及通关后的行为的逻辑。

那我们所要做的，其实就是每一次球发生移动时使GameController检查所有球是否都是isMatch了。

可以选择让GameController变成单例然后直接由Holder调用检查方法，也可以新加事件由Holder发送、GameController订阅并处理。

```C#
// EventHandler
public static event Action CheckMiniGameStateEvent;
public static void CallCheckMiniGameStateEvent()
{
    CheckMiniGameStateEvent?.Invoke();
}

// Holder
public override void EmptyClicked()
{
    foreach (var holder in linkHolders)
    {
        if(holder.isEmpty){
            ......
            EventHandler.CallCheckMiniGameStateEvent();
        }
    }
}

// GameController
private void OnEnable()
{
    EventHandler.CheckMiniGameStateEvent += CheckMiniGameState;
}
private void OnDisable()
{
    EventHandler.CheckMiniGameStateEvent -= CheckMiniGameState;
}
/// <summary>
/// 处理检查小游戏状态事件方法
/// </summary>
void CheckMiniGameState()
{
    // 用此法获得了所有的Ball
    foreach (var ball in FindObjectsOfType<Ball>())
    {
        if (!ball.isMatch)
        {
            return ;
        }
    }
    Debug.Log("小游戏结束");
}
```

启动游戏，能触发小游戏结束通知即完成。



然后我们来实现小游戏完成之后的行动，这里就是简单的切换到指定场景：

```C#
// GameController.cs
using UnityEngine.Events;

public UnityEvent OnFinish;
[Header("游戏数据")]

void CheckMiniGameState()
{
    ...
    Debug.Log("小游戏结束");
    OnFinish?.Invoke();
}

```

`UnityEvent`是一种特殊的委托，它**可以在编辑器中配置**，并且可以在运行时触发。

`OnFinish?.Invoke();` 这行代码的含义是：如果 `OnFinish` 不是 null，则调用它指向的所有方法。这通常用于在某个操作完成时触发一个事件，而不需要担心 `OnFinish` 是否已经被赋值。

上面这串代码实际效果与自己编写的EventHandler基本一致，不同的是在Unity Editor中可以看到这样的公开变量：

![image-20241016212239530](picture/image-20241016212239530.png)

这下看懂了，不就跟Button的触发方法列表一样的嘛，所以在背景箭头处创建Teleport，设置转移到H2，将该Teleport拖到列表的对象参数框中，调用其切换场景方法即可。



最后再添加一个重置按钮，添加按钮框及按钮齿轮子对象，添加碰撞体，添加H2AResetButton脚本：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class H2AResetButton : Interactive
{
    private Transform gearSprite;

    private void Awake()
    {
        gearSprite = transform.GetChild(0);
    }

    public override void EmptyClicked()
    {
        // 重置游戏
        StartCoroutine(GearRotation.DoRotation(gearSprite, 60.0f, 1.0f));
        GameController.Instance.ResetGame();
    }
}
```

这里自己编写了一个齿轮旋转的异步方法，就是一个简单的匀速旋转的方法，实际可以尝试写一个加速再减速直至停止的代码。

再在GameController中将类更改为单例，添加重置小游戏的方法：

```C#
public void ResetGame()
{
    // 线是不用删除的，本身也不会改变

    foreach (var holder in holderTransforms)
    {
        if (holder.childCount > 0)// 用childCount判定槽位是否有球
        {
            Destroy(holder.GetChild(0).gameObject);
        }
    }

    CreateBall();
}
```

这样整个小游戏，包括初始化、主体逻辑、结束行为以及重置方法都写好了。

## 保存小游戏状态

根据H2场景中小游戏的作用，当从小游戏返回时，不只是转移会H2场景，还要根据小游戏是否通过的状态设置电子门是否显示。

先定义MiniGame用于存储小游戏的相关变量并控制场景对象的显示：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

public class MiniGame : MonoBehaviour
{
    public UnityEvent OnGameFinish;
    [SceneName] public string gameName;
    public bool isPass;
	
    public void UpdateMiniGameState()
    {
        if (isPass)
        {
            GetComponent<Collider2D>().enabled = false;
            GetComponent<SpriteRenderer>().color = new Color(1, 1, 1, 0);
            OnGameFinish?.Invoke();
        }
    }
}
```

这里定义了小游戏名称（限制为SceneName的枚举值）、Unity触发事件以及通过状态。保存代码，在Door上挂载MiniGame脚本，在门（楼梯）处新建转到H3的Teleport，并设Active为false。

修改脚本公开变量：

![image-20241016230524122](picture/image-20241016230524122.png)

在GameController的`CheckMiniGameState`方法中添加对小游戏状态的处理。由于一个场景中一个GameController负责控制一个小游戏，并且不是全局存在的（也就意味着数据不能持久存储），我们需要想办法将数据持久化存储（便于存储也便于修改），并且由一个全局存在的控制器对小游戏对象的状态进行判断并对场景中相关对象的状态进行更新。

考虑到GameController中已经包含了GameH2A_SO资产，可以直接对GameH2A_SO类编辑以存储资产对应的小游戏的名字：

```C#
public class GameH2A_SO : ScriptableObject
{
    [SceneName] public string gameName;
    ...
}

```

那么GameController就可以通过gameData成员变量拿到其控制的小游戏的名称了。当检查确认小游戏已经完成时，GameController通过事件系统发送消息让另一个在SampleScene中的控制器GameManager的脚本获取到该游戏的名称以及已经完成的消息。

```C#
 void CheckMiniGameState()
 {
     ...
     EventHandler.CallGamePassEvent(gameData.gameName);
     OnFinish?.Invoke();
 }

// EventHandler中添加
public static event Action<string> GamePassEvent;
public static void CallGamePassEvent(string gameName)
{
    GamePassEvent?.Invoke(gameName);
}
```

编写GameManager脚本用于存储所有小游戏的状态并编写处理方法，其主要控制方式为监听场景切换消息并更新场景中所有已被置为初始状态的MiniGame脚本对象的isPass成员状态并调用其状态更新方法：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameManager : MonoBehaviour
{
    // 用于存储所有小游戏的名称和状态
    private Dictionary<string, bool> miniGameStateDict = new Dictionary<string, bool>();
    private void OnEnable()
    {
        EventHandler.AfterSceneLoadedEvent += OnAfterSceneLoaded;
        EventHandler.GamePassEvent += OnGamePassEvent;
    }
    private void OnDisable()
    {
        EventHandler.AfterSceneLoadedEvent -= OnAfterSceneLoaded;
        EventHandler.GamePassEvent -= OnGamePassEvent;
    }
    void Start()
    {
        // 在游戏刚加载时将游戏状态设置为常规状态
        EventHandler.CallGameStateChangedEvent(GameState.GamePlay);
    }
    void OnAfterSceneLoaded()
    {
        // 当加载场景时，遍历所有被加载的MiniGame脚本对象，将字典中相应名字的小游戏的状态赋值给脚本对象，并调用其更新状态方法（如果isPass为false，不会触发任何改变）
        foreach(var miniGame in FindObjectsOfType<MiniGame>())
        {
            if(miniGameStateDict.TryGetValue(miniGame.gameName, out bool isPass))
            {
                miniGame.isPass = isPass;
                miniGame.UpdateMiniGameState();
            }
        }
    }
    void OnGamePassEvent(string gameName)
    {
        // **如果字典中不存在键值gameName，则添加键值对<gameName, true>**
        miniGameStateDict[gameName] = true;
    }
}

```

在编辑好所有的脚本后，保存并回到Unity Editor对MiniGame、MiniGame_Week1资产等进行赋值和确认。运行游戏，完成小游戏后自动转回H2场景，密码门已经消失并且能够点击楼梯进入场景H3，这一块功能就全部实现了。

*思考这里是否真正需要数据持久化存储？我目前的想法是，存在资产中和在GameController的一个公开变量中实际感觉并无太大差异。何况要存储小游戏的名称用SceneName枚举类来限制其值也是不合理的，建议还是自行在Enums.cs中另外定义一个枚举类。*

## *UI遮挡

在现在的H3场景中我们会发现这样的蜜汁操作——

![image-20241017004527395](picture/image-20241017004527395.png)

我们回到H2的Teleport处在屏幕的左下角，部分与InventoryUI重合，当我们点击SlotUI时，竟然也能触发场景转移，这显然是我们所不期望的。

在CursorManager中添加判断鼠标是否悬停在UI组件上的方法：

```C#
private bool InteractWithUI()
{
    if (EventSystem.current != null && EventSystem.current.IsPointerOverGameObject()) return true;
    else return false;
}
```

1. `EventSystem.current != null`：首先检查 `EventSystem` 是否存在。`EventSystem` 是Unity中处理事件系统的一个组件，负责处理如鼠标点击、拖拽等输入事件。如果当前场景中没有 `EventSystem` 组件，这个方法会返回 `false`。
2. `EventSystem.current.IsPointerOverGameObject()`：如果 `EventSystem` 存在，方法会调用 `IsPointerOverGameObject` 函数。这个函数检查当前是否有任何指针（如鼠标）悬停在任何UI元素上。如果玩家的指针正在与UI元素交互，这个函数会返回 `true`。

然后在Update方法中调用该方法进行判断，如果返回值为true，直接return以忽略后续的所有操作：

（这里建议将不同鼠标事件类型划分开来）

```C#
private void Update()
{
    canClick = ObjectAtMousePosition();
    // 鼠标移动相关
    // 如果手激活，就让手跟随鼠标移动
    if (hand.gameObject.activeInHierarchy)
        hand.position = Input.mousePosition;
    // 鼠标点击相关
    if (InteractWithUI()) return;
    if (canClick && Input.GetMouseButtonDown(0))
    {
        ClickAction(ObjectAtMousePosition().gameObject);
    }

}
```

这样就完成了。

# 制作游戏菜单

因为教程中的素材没有找到，在此就不多讲述教程内容了，只给出几个比较重要的点:

* 主菜单界面单独制作一个Scene，这样子就可以利用前面的场景转移停止游戏并快捷地显示主菜单。实际只要该保存的数据能够保存和复现，所有的游戏都可以应用这种方法。
* 在主菜单中可能会需要多级菜单栏，教程给出的处理方法是：将显示的每一组Button制成一个Tab父对象的子对象，然后在Button的触发事件中只需控制当前Tab的隐藏和目标Tab的显示即可。

# 多周目的实现

教程中多周目实际改变的就是小游戏的内容，看似只需要让GameController根据gameWeek去load相应的GameH2A_SO资产并重新绘制小游戏即可。但是实际我们还要对所有存储过游戏进度相关数据的所有脚本进行数据清空——包括GameManager、InventoryManager、MiniGame、ObjectManager等等。然后为了方便读取，我们将GameController中的`public GameH2A_SO gameData;`更改为List并将所有周目的资产拖入。

# 游戏数据存储和加载

Unity自己有json数据化工具`JsonUtility`类，但是这种工具只能序列和反序列化普通类，对于继承自`UnityEngine.Object`类比如`MonoBeheavier`或者`ScriptableObject`则无能为力，我们这里使用了另一种格式化工具`Newtonsoft.Json`，这是一种更全面的 JSON 库。在Package Manager中使用URL `com.unity.nuget.newtonsoft-json`导入包。

新建存储数据类`GameSaveData`，当我们有需要存储时变量时，将该变量的声明复制到这个类下即可。

很自然地能够想到，需要存储的数据是分散在各个脚本中的，为此我们可以创建一个全局的数据实例或者单例来一次性存储完所有的数据。这种方法就意味着每一个需要存储的脚本中都得拿到数据类型的实例以修改指定成员变量的值。脑测似乎也没有太大的问题和毛病，并且不会大量地浪费内存。

这里教程为了进行标准化管理，选择了让每一个模块都自行创建一个数据实例，但只对自己负责的数据进行赋值，并且传递给SaveDataManager以形成一个<脚本名，数据实例>格式的数据列表，将该列表存储为json格式。读取的时候也只需要按脚本名传递给不同的脚本调用其恢复数据的方法即可。这个方法也还不错，唯一的不足就是当存储的脚本过多时会大量浪费空间。

按照教程给的方法，首先需要一个接口以规定生成数据实例和根据数据实例恢复先前状态的方法：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public interface ISaveable 
{
    /*从 C# 8.0 开始，引入了默认接口方法（default interface methods）的概念，这允许在接口中提供方法的默认实现。这是一种特殊情况，允许接口中的方法包含方法体，这样实现接口的类可以选择使用接口提供的默认实现，或者提供自己的实现。*/
    void SaveableRegister()
    {
        SaveLoadManager.Instance.Register(this);
    }
    GameSaveData GenerateSaveData();
    void RestoreGameData(GameSaveData saveData);
}

```

编写数据管理类脚本并继承自Singleton:

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Newtonsoft.Json;
using System.IO;
public class SaveLoadManager : Singleton<SaveLoadManager>
{
    /// <summary>
    /// 存储数据文件的路径
    /// </summary>
    private string jsonFolder;
    /// <summary>
    /// 数据接口列表
    /// </summary>
    private List<ISaveable> saveableList = new List<ISaveable>();
    /// <summary>
    /// 数据实例字典，存储了类名和数据实例的对应关系
    /// </summary>
    private Dictionary<string, GameSaveData> saveDataDict = new Dictionary<string, GameSaveData>();
    /// <summary>
    /// 启动时创建存储路径并检查是否存在
    /// </summary>
    protected override void Awake()
    {
        // 因为继承了Singleton，所以需要在Awake中调用基类的Awake方法
        base.Awake();
        jsonFolder = Application.persistentDataPath + "/SAVE/";
        if (!System.IO.Directory.Exists(jsonFolder))
        {
            System.IO.Directory.CreateDirectory(jsonFolder);
        }
    }
    private void OnEnable()
    {
        EventHandler.StartNewGameEvent += OnStartNewGameEvent;
    }
    private void OnDisable()
    {
        EventHandler.StartNewGameEvent -= OnStartNewGameEvent;
    }
    /// <summary>
    /// 当启动新游戏时，删除已有的存储的数据文件
    /// </summary>
    /// <param name="obj"></param>
    private void OnStartNewGameEvent(int obj)
    {
        var resultPath = jsonFolder + "data.sav";
        if (File.Exists(resultPath))
        {
            File.Delete(resultPath);
        }
        Debug.Log(resultPath);
    }
    /// <summary>
    /// 注册数据接口方法
    /// </summary>
    /// <param name="saveable"></param>
    public void Register(ISaveable saveable)
    {
        saveableList.Add(saveable);
    }
    /// <summary>
    /// 保存数据方法
    /// </summary>
    public void Save()
    {
        saveDataDict.Clear();

        foreach (var saveable in saveableList)
        {
            // GetType()返回表示该对象类型的 System.Type 对象，当子类实例向上转型为父类或者接口实例时，GetType()方法返回的是子类的类型
            saveDataDict.Add(saveable.GetType().Name, saveable.GenerateSaveData());
        }
        var resultPath = jsonFolder + "data.sav";
        // 将数据序列化为json格式
        var jsonData = JsonConvert.SerializeObject(saveDataDict, Formatting.Indented);

        if (!File.Exists(resultPath))
        {
            Directory.CreateDirectory(jsonFolder);
        }

        File.WriteAllText(resultPath, jsonData);
    }
    /// <summary>
    /// 加载数据方法，同时会调用每个数据接口的RestoreGameData方法以恢复其状态
    /// </summary>
    public void Load()
    {
        var resultPath = jsonFolder + "data.sav";
        if (!File.Exists(resultPath))
        {
            return;
        }

        var stringData = File.ReadAllText(resultPath);
        var jsonData = JsonConvert.DeserializeObject<Dictionary<string, GameSaveData>>(stringData);

        foreach(var saveable in saveableList)
        {
            // 通过类名在字典中查找到其对应的ISaveable实例并调用该类的恢复方法
            saveable.RestoreGameData(jsonData[saveable.GetType().Name]);
        }
    }
}
```

*看不懂上面代码逻辑建议再学习一遍面向对象编程的多态性。*

编写好数据存储管理类的脚本后，在SampleScene中新建一个Empty Object并挂载该脚本。

接下来对所有需要存储的类引用ISaveable接口并实现`GenerateSaveData`和`RestoreGameData`方法。

首先是TransitionManager，我们需要存储当前所处的场景并且当继续游戏时，由Menu场景转换到先前的场景:

```C#
public class TransitionManager : SingletonMono<TransitionManager>, ISaveable
{
    ...
    void Start()
    {
        ...
        ISaveable saveable = this;
        saveable.SaveableRegister();
    }
	
    public GameSaveData GenerateSaveData()
    {
        GameSaveData gameSaveData = new GameSaveData();
        gameSaveData.currentScene = SceneManager.GetActiveScene().name;
        return gameSaveData;
    }

    public void RestoreGameData(GameSaveData gameSaveData)
    {
        // 将字符串转换为枚举类的值
        Enum.TryParse(gameSaveData.currentScene, out Teleport.Scene sceneTo);
        Transition(Teleport.Scene.Menu, sceneTo);
    }
}

```

然后是InventoryManager，存储背包中的道具信息：

```C#
public class InventoryManager : SingletonMono<InventoryManager>, ISaveable
{
    public ItemDetailList itemDetailList;
    [SerializeField] private List<ItemName> itemList = new List<ItemName>();
    ...
    void Start()
    {
        ...
        ISaveable saveable = this;
        saveable.SaveableRegister();
    }
    public GameSaveData GenerateSaveData()
    {
        GameSaveData saveData = new GameSaveData();
        saveData.itemDetailList = this.itemDetailList;
        return saveData;
    }
    public void RestoreGameData(GameSaveData gameSaveData)
    {
        this.itemDetailList = gameSaveData.itemDetailList;
    }
}

```

然后是ObjectManager，存储所有的可用道具和可交互对象的状态：

```C#
public class ObjectManager : MonoBehaviour, ISaveable
{
    ...
    void Start()
    {
        ...
        ISaveable saveable = this;
        saveable.SaveableRegister();
    }
    public GameSaveData GenerateSaveData()
    {
        GameSaveData gameSaveData = new GameSaveData();
        gameSaveData.itemAvailableDict = itemAvailableDict;
        gameSaveData.interactiveStateDict = interactiveStateDict;
        return gameSaveData;
    }

    public void RestoreGameData(GameSaveData saveData)
    {
        this.itemAvailableDict = saveData.itemAvailableDict;
        this.interactiveStateDict = saveData.interactiveStateDict;
    }
}
```

最后是GameController，存储小游戏的状态和周目数：

```C#
public class GameManager : MonoBehaviour, ISaveable
{
	...
    void Start()
    {
        ...
        ISaveable saveable = this;
        saveable.SaveableRegister();
    }
    public GameSaveData GenerateSaveData()
    {
        GameSaveData saveData = new GameSaveData();
        saveData.gameWeek = gameWeek;
        saveData.miniGameStateDict = miniGameStateDict;
        return saveData;
    }

    public void RestoreGameData(GameSaveData saveData)
    {
        this.gameWeek = saveData.gameWeek;
        this.miniGameStateDict = saveData.miniGameStateDict;
    }
}
```

这里这些脚本没有在`RestoreGameData`中刷新显示，原因是在场景切换时会自动刷新，因此基本只需要覆盖持久化变量的值即可。

**想要测试查看实现的效果，需要按照之前两P的教程完善代码后再运行。**

上面的都编写好后，在Menu.cs中的`ContinueGame()`方法中调用`SaveLoadManager.Instance.Load()`方法，当运行游戏并退出后，点击继续游戏按钮，之前的所有进度都正确的显示，这样就完成了。

还可以根据路径（输出一下resultPath）去指定文件夹中查看data.sav文件，打开后能够看到所存储的所有的对象及数据。



这样我们就实现了全部基本内容了！完结撒花:happy::happy::happy::happy:
