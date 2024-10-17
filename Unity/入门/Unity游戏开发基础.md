说明：

本人纯Unity萌新，有一定编程基础。使用的Unity版本为2022.3

对应视频教程：https://www.bilibili.com/video/BV1Mr4y1X76H?p=129&vd_source=28e3994bf6aa5fa346ca67b74f2c51f6，老师讲的很好，语速较慢，可以开倍速听讲。

老师自己也有讲义：https://gitee.com/chutianshu1981/AwesomeUnityTutorial

笔记从视频P47开始，即：零代码/低代码/0编程基础游戏制作不建议看本笔记。P141之后的内容可能与游戏编程本身无关，故先如此，未来做好全部笔记之后应该会再发一次。本笔记也修正了一些学习中遇到的问题，期待与大家一同学习Unity游戏开发。

[toc]

# 2D基本编程

## 游戏对象的坐标生成

```c#
//引入命名空间 UnityEngine
using UnityEngine;

// 生成器示例类
// 这个生成器会在不同位置，生成三个指定的游戏对象
public class SpawnerSample : MonoBehaviour
{
    // 声明 一个attribute（字段），是一个游戏对象，用来获取生成器生成的 gameobject
    public GameObject ObjectToSpawn;
    // 声明一个变量，用来存放距离
    public int radius;
    // start事件方法，是包含此脚本的游戏对象 Start is called just before any of the Update methods is called the first time
    // 在 update 前执行，且在游戏对象生命周期只执行一次
    // Start is only called once in the lifetime of the behaviour.
    void Start()
    {
        // 声明整型变量 angle 并赋值为 15
        int angle = 15;

        //声明一个 3D Vector (矢量) 对象 spawnPosition 并赋值为当前游戏对象的位置
        Vector3 spawnPosition = transform.position;

        // 声明一个 3D Vector (矢量) 对象 direction，并使用 Quaternion（四元数）类的静态方法  Euler（欧拉）返回值 * Vector3(1,0,0)
        // 四元数 Quaternion 是 unity 3d 系统中，用来处理旋转的类
        // 说白了就是创建一个 方向对象，指定一个方向，旋转多少角度，然后生成一个向量坐标
        Vector3 direction = Quaternion.Euler(0, angle, 0) * Vector3.right;
        // 重新计算 spawnPosition 的值，原来的位置坐标 + 一个方向 * 距离
        spawnPosition = transform.position + direction * radius;
        // 用 Object 的 Instantiate（实例化） 静态方法实例化游戏对象
        Instantiate(ObjectToSpawn, spawnPosition, Quaternion.identity);

        // 用一套新值，在新的位置重新生成一个游戏对象
        angle = 55;
        direction = Quaternion.Euler(0, angle, 0) * Vector3.right;
        spawnPosition = transform.position + direction * radius;
        Instantiate(ObjectToSpawn, spawnPosition, Quaternion.identity);

        // 用一套新值，在新的位置重新生成一个游戏对象
        angle = 95;
        direction = Quaternion.Euler(0, angle, 0) * Vector3.right;
        spawnPosition = transform.position + direction * radius;
        Instantiate(ObjectToSpawn, spawnPosition, Quaternion.identity);
    }
}


```

* 如何理解`Vector3`、`Quaternion.Euler`以及方向向量运算？

  `Vector3`变量是三维向量，可以是向量也可以是坐标，`Quaternion.Euler`是三维的旋转角，`Quaternion.Euler(0, angle, 0) * Vector3.right`就是取标准向量(1,0,0)并按照(0, angle, 0)的旋转角在三个维度上进行旋转所得到的**标准方向向量**。

  标准方向向量 \* 距离`radius`得到的就是新对象相对游戏对象`spawnPosition`的相对坐标，加上`spawnPosition`的绝对坐标即可得到其在Scene中的绝对坐标。

## Sprite的pivot

[游戏对象定位 - Unity 手册](https://docs.unity.cn/cn/2021.1/Manual/PositioningGameObjects.html)

* 在2D游戏中，因为Z轴垂直于纸面，而我们需要的是一个Z轴斜向的一个透视效果，该效果反映在不同精灵的图层变化上

* 每一个精灵都有一个中心点，由pivot属性决定其位置。pivot的坐标代表物体的坐标。如何在2D图像中表现出物体之间的前后顺序？

  可以认为：pivot更”靠下“的在前面。因此不同物体的pivot的位置很关键。

  具体还需更改Project Settings中的如下设置：

  <img src="E:/Typora/picture/image-20240728172026472.png" alt="image-20240728172026472" style="zoom:50%;" />

  在不同视角的2D游戏中，pivot的判断也不尽相同（比如斜45度视角的2D游戏）
  
  然后在物体/预制件中更改：
  ![image-20240730104615729](E:/Typora/picture/image-20240730104615729.png)

## 按键控制

~~~C#
public class RubyController : MonoBehaviour
{
   // 每帧调用一次 Update
    // 让游戏对象每帧右移 0.1
    void Update()
    {
        // 获取水平输入，按向左，会获得 -1.0 f ; 按向右，会获得 1.0 f
        float horizontal = Input.GetAxis("Horizontal");
        // 获取垂直输入，按向下，会获得 -1.0 f ; 按向上，会获得 1.0 f
        float vertical = Input.GetAxis("Vertical");

        // 获取对象当前位置
        Vector2 position = transform.position;
        // 更改位置
        position.x = position.x + 0.1f * horizontal;
        position.y = position.y + 0.1f * vertical;
        // 新位置给游戏对象
        transform.position = position;
    }
}
~~~

*在 Unity 项目设置中，可以通过 Input Manager 进行默认的游戏输入控制设置*

***Edit > Project Settings > Input***

**注意：`Input.GetButton` 仅用于事件等操作。不要将它用于移动操作。因为这样移动就绑定死了按键，不能在设置里更改。**

常用的一些关于按键控制的函数：

`Input.GetKeyDown()` 参数为KeyCode.\*，*指按键名如C、X，返回值为Bool类型

`Input.GetAxis()` 参数为String，指Unity设置的格式输入（即Input Manager中的绑定了按键的专用控制名），如"Horizontal"、"Fire1"等。其返回值为float，如果用该方法判断是否按下某个按键，应该设为if(~ != 0)。但是该方法对于按键的监听过于灵敏，会导致固定时间内触发次数过多，一般不建议用于判断按下按键。


## 移速不受帧率影响

```C#
// 将速度暴露出来，使其可调
   public float speed=0.1f;
// 每帧调用一次 Update
   void Update()
   {
       float horizontal = Input.GetAxis("Horizontal");
       float vertical = Input.GetAxis("Vertical");
       Vector2 position = transform.position;
       position.x = position.x + speed * horizontal * Time.deltaTime;
       position.y = position.y + speed * vertical * Time.deltaTime;
       transform.position = position;
   }
```

* 固定时间内稳定帧率与帧间时间间隔成反比，移动速度=单位时间帧数\*帧间移动距离

## 素材拆分

* 一些素材会以多个图像集合的形式存储，需要对其进行拆分后再使用

  常规的拆分方法就是拆成等比的小素材。

* 点击目标素材，Sprite Mode改为Multiple，点击Sprite Editor，选择Apply。
* 进入Sprite Editor，点击Slice，选择按数量拆分或者按大小拆分（实际效果相同），点击Slice，然后会自动拆分并检测每块中是否存在素材。拆分好后点击Apply，退出，在Project中原素材下就能看到拆分后的素材了。

## 地图瓦片（Tile）

https://gitee.com/chutianshu1981/xyz-s-free-course-for-full-stack-web-dev/blob/main/GameDesign/Unity/%E5%AE%98%E6%96%B9%E6%95%99%E7%A8%8B%E9%A1%B9%E7%9B%AE%E7%B3%BB%E5%88%97/%E5%AE%98%E6%96%B9%E6%95%99%E7%A8%8B05_2D%E6%B8%B8%E6%88%8F/RubyAdventure/02-%E4%BD%BF%E7%94%A8%E7%93%A6%E7%89%87%E5%9C%B0%E5%9B%BE%E5%88%9B%E5%BB%BA%E4%B8%96%E7%95%8C.md#3-%E7%93%A6%E7%89%87%E5%9C%B0%E5%9B%BE%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B

（之前看的时候忘了做笔记了，简单补一下）

先在Hierarchy中创建一个2D Object>TileMap>Rectangle。将地图瓦片素材切分好后，更改Pixels per Unit，在Tile文件夹创建一个Rule Tile，制作好RuleTile后，其他的瓦片就可以使用Advanced Rule Tile了。

如果需要考虑不同的瓦片之间的连接时相互不受影响，我们需要添加一个控制脚本：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Tilemaps;

[CreateAssetMenu]
public class RuleTile03 : RuleTile<RuleTile03.Neighbor> {
    public bool customField;

    public class Neighbor : RuleTile.TilingRule.Neighbor {
        public const int Null = 3;
        public const int NotNull = 4;
    }

    public override bool RuleMatch(int neighbor, TileBase tile) {
        switch (neighbor) {
            case Neighbor.Null: return tile == null;
            case Neighbor.NotNull: return tile != null;
        }
        return base.RuleMatch(neighbor, tile);
    }
}
```

该脚本的作用就是添加某格是否存在瓦片而不是存在同一个RuleTile下的瓦片。

建好该脚本后，Create时顶部会出现一个额外的与脚本对应的一个类名，以该类创建RuleTile即可应用该脚本。

我们还需要对Tilemap添加碰撞体：

在Tilemap对象中添加Tilemap专用的碰撞体组件Tilemap Collider 2D，然后添加Composite Collider 2D，此时会自动添加一个刚体组件，注意设置重力和限制旋转。

对于斜坡等一些形状不规则的瓦片，还需要自行设置碰撞体：

[Unity2D中自定义TileMap的碰撞体_unity 2d 让tilemap collider 2d平滑-CSDN博客](https://blog.csdn.net/qq_60512625/article/details/124654111#:~:text=解决方法是进入S,顺畅的游戏体验。)

## 触发器

触发器（触发碰撞器）是一种特殊类型的碰撞体。触发器不会阻止移动，但是物理系统仍会检查角色是否会与触发器碰撞。当你的角色进入触发器时，你将收到一条消息，以便你可以处理该事件。

一句话总结：触发器碰撞体只监测碰撞，不阻止移动；碰撞时，可接收到消息，根据需求添加相关处理代码

使用步骤：

1. 为碰撞体组件选中 “Is Trigger” 复选框，再测试游戏，会发现碰撞体不再阻止移动了；但实际上，Unity 还是会通过碰撞体记录碰撞；

   ![img](https://gitee.com/chutianshu1981/xyz-s-free-course-for-full-stack-web-dev/raw/main/imgs/unity_isTrigger.png)

2. 在使用触发器的游戏对象上添加脚本组件，在其中添加事件
   void OnTrigger***2D(Collider2D other)
   此事件会在每次\*\*\*时执行，将你需要的逻辑代码写入事件方法中即可。

   ==注意为预制件添加新脚本之后需保存一次，才能编辑并应用脚本，否则会消失。==

### 对于OnTriggerStay2D的一些说明

1. 控制碰撞后保持不动时是否触发——对other对象的刚体组件中的Sleeping Mode属性进行修改：Never Sleep表示一直触发，Start Awake表示不动时休眠（不触发）。

2. Never Sleep下事件会逐帧触发，需要自行设置一下触发间隔。

   

## 获取游戏对象

在触发器以及其他一些脚本中常常需要对游戏对象进行操作和属性修改。那实际在代码中实现如下：

1. 获取游戏对象的脚本对象(Controller为脚本对象的命名空间)

   ```C#
   Controller controller = other.GetComponent<Controller>();//other为刚体对象
   Controller controller = other.gameObject.GetComponent<Controller>();//other为碰撞体对象，注意如果other是碰撞相关的组件，一般获取到的是碰撞到的对象的组件
   ```

2. 判断对象是否为空。（建议用try catch语句）

3. 利用controller实例调用其修改属性的方法（需设定保护为public）

## 销毁游戏对象

在需要销毁的对象的脚本下指定的方法下的最后加入`Destroy(gameObject)`，`gameObject`自动获取到当前对象。

## C#的getter和setter

如果要设置不能读也不能写: `private string name;`

如果设置只能读不能写: `public string name{get;}`

如果设置既能读也能写: `public string name{set;get;}`

![image-20240808210335715](E:/Typora/picture/image-20240808210335715.png)

## 受伤后的无敌帧和间隔性伤害

设置角色受伤后的暂时无敌——

在脚本下新增变量：

```c#
public float invincibleTime = 2.0f;//无敌时间
bool isInvincible;
float invincibleTimer;//倒计时器
```

在Update()中添加：

```C#
if (isInvincible)//如果处于无敌状态，invicibleTimer进行倒计时
{
    invincibleTimer -= Time.deltaTime;//deltaTime获取帧间隔
    if(invincibleTimer < 0)
    {
        isInvincible = false;
    }
}
```

然后在角色脚本中的ChangeHealth方法中增加：

```c#
if(amount < 0)//如果生命值即将降低
{
    if (isInvicible)//如果处于无敌状态，直接返回且不结算伤害
    {
        return;
    }
    isInvicible = true;//否则设置角色为无敌
    invicibleTimer = invicibleTime;//计时器重置
}
```

## 平铺区域

Unity提供了将图像根据缩放平行密铺而非放缩的方法：

1. 选中对象，更改Draw Mode：

   ![image-20240812210718810](E:/Typora/picture/image-20240812210718810.png)

2. 弹出警告，提示更改精灵属性，找到原精灵（拖拽进Hierarchy层级界面的东西），更改mesh type属性：
   ![image-20240812211005338](E:/Typora/picture/image-20240812211005338.png)

3. 此时再缩放对象，发现可以平铺：（别忘了更改碰撞体）
   ![image-20240812211117307](E:/Typora/picture/image-20240812211117307.png)

4. 通过调整对象的Scale属性可以更改单元的大小。

## 制作敌人

对象加入——编辑参数和脚本（基本与操控对象类似）——制作事件

### 碰撞伤害

类似于OnTriggerEnter2D，可以使用OnCollisionEnter2D（刚体碰撞事件）函数

## 动画入门

1. 选择对象对应的预制件
2. 添加Animator组件
3. 在Animator组件中添加Controller

### Animation制作

* 新建Animation

* 双击打开Animation窗口，*注意先选中植入动画的预制件*，将指定的精灵批量拖入时间轴

* 更改采样率（一秒播放多少帧），如果找不到在右上角三点中勾选Show Sample Rate，播放以查看效果，可以适当调整

  ![image-20240819104428753](E:/Typora/picture/image-20240819104428753.png)

* 左上角轻松更换或新建Animation文件

Tips：

1. 在Add Property中可以对精灵进行快捷编辑，如水平、垂直翻转

    * 注意：在添加Property之前先选中需要添加属性的动画帧，并将当前指针移动至其开头时再添加，否则会出现一些帧动画没有更改的情况。
      常用的有：Flip X/Y——水平/垂直翻转，Enabled——是否显示该帧动画
      
      还有，避免浪费资源，请将更改后无用的帧给删除。

    ![image-20240819104822012](E:/Typora/picture/image-20240819104822012.png)

2. delete操作可以用delete键，拖拽使用鼠标中键。

3. 为了使动画看起来更合理流畅，自己逐帧拖入2D精灵时应当在最后一帧动画后再加一个周期并插入第一帧动画。

4. 想要将所有帧的帧间距离调整，可以全选中然后拖动末尾的竖条：
    ![image-20240819223333135](E:/Typora/picture/image-20240819223333135.png)

### 构建Controller

* 双击指定Animator Controller或在Window-Animation-Animator打开Animator窗口：

![image-20240819140832032](E:/Typora/picture/image-20240819140832032.png)

Animator Controller本质就是一个状态机，当为预制件装载Animation的时候会将Animation引入Animator，在状态上右键选择make transition即可引申新的转换方向。

默认对Animator Controller编辑会使不同状态周期循环转换，进行下面我们来为Controller与对象自身状态/属性进行挂接：

### 创建混合树

在Animator界面中右键选择创建新混合树：

![image-20240819142103122](E:/Typora/picture/image-20240819142103122.png)

双击Blend Tree打开：

![image-20240819142511718](E:/Typora/picture/image-20240819142511718.png)

点击左上角Parameter，删除参数Blend，新建自己的相应类型的参数：

![image-20240819142759599](E:/Typora/picture/image-20240819142759599.png)

* Bool类型和Trigger类型参数的区别：
  如果在transition中勾选了Has Exit Time，那么Trigger类型参数在触发后（发生`animator.SetTrigger("Name")`之后）相应的Exit Time之后自动返回未触发状态，而Bool类型参数需要手动Set之后才会更改值。

在Inspector界面中更改Blend Tree的Blend Type属性（暂时理解为有x个参数选xD），将Parameters改为需要的参数，然后在motion下添加条目：

![image-20240819144545879](E:/Typora/picture/image-20240819144545879.png)

打开最下面双杠的预览，调整红点位置以查看效果。

### 将参数发送到Animator Controller

在对象的控制脚本中添加：

```C#
//声明
Animator animator;
//获取
animator = GetComponent<Animator>();
//使用
animator.SetFloat("Move X", 0);//Float对应变量类型，Move X对应变量名称
```

\*注意：设置值不能模糊（比如介于两界限之间），否则动画效果可能出现差错。建议先在预览时找好合适的取值。

### 动画过渡

在Base Layer中的一个Blend Tree上右键Create一个指向其他Blend Tree的transition.

点击新生成的箭头，可以对过渡条件和属性进行修改，具体属性功能解释见：
[动画过渡 - Unity 手册](https://docs.unity.cn/cn/2022.3/Manual/class-Transition.html)

目前最重要的：

* **Transition Duration**：相对于当前状态持续时间的过渡持续时间，以标准化时间或秒为单位（具体取决于 **Fixed Duration** 模式）。此时间在过渡图中显示为两个蓝色标记之间的部分。
* **Conditions**：添加过渡条件。如果没有条件，则进入该状态之后立刻进行过渡。如果不需条件只需要持续一段时间后回到原状态则勾选Has Exit Time并设置Exit Time即可。

### 总结

[官方教程项目系列/官方教程05_2D游戏/RubyAdventure/03-2D精灵动画.md · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/blob/main/官方教程项目系列/官方教程05_2D游戏/RubyAdventure/03-2D精灵动画.md)

## \*优化角色控制

为了适配原教程提供的Ruby的Animator，我们需要对原来的控制脚本进行优化细化：

### 获取角色静止时的朝向

如果依然沿用之前XY方向各自设置一个浮点类型的矢量来判断，则角色静止时无法得到理应的朝向（即静止前最后一刻角色的朝向），为此需要手动设置：

```C#
/*声明*/
Animator animator;
Vector2 lookDirection = new Vector2(1, 0);
Vector2 move;
// 将速度暴露出来，使其可调
public float speed = 0.1f; 

/*Start*/
animator = GetComponent<Animator>();

/*Update*/
move = new Vector2(horizontal, vertical);//存储当前方向值
//如果正在移动
if(!Mathf.Approximately(move.x, 0.0f) || !Mathf.Approximately(move.y, 0.0f))
{
    //赋值给lookDirection
    lookDirection.Set(move.x, move.y);
    //因为blend tree中表示方向的参数值取值范围是[-1.0, 1.0]，所以一般需要对向量进行归一化处理
    lookDirection.Normalize();
}//如果静止状态，则lookDirection就会保持上一次Update时的方向
//将Ruby面朝方向传给blend tree
animator.SetFloat("Look X", lookDirection.x);
animator.SetFloat("Look Y", lookDirection.y);
//将Ruby当前的速度传给blend tree
animator.SetFloat("Speed", move.magnitude);//magnitude返回向量的长度

/*ChangeHealth*/
//如果判定受伤
animator.SetTrigger("Hit");
```

## 飞弹

* 先更改飞弹2D精灵的Pixels Per Unit属性使其缩小到合适的大小。

* 将精灵拖拽进Hierarchy中，再将素材添加到Prefabs中，对预制件进行编辑

* 飞弹要进行移动和碰撞，所以为其添加刚体和碰撞体。

* 添加控制脚本，为其添加飞行方法和碰撞方法：

  ```C#
  public void launch(Vector2 direction, float force)
  {
      //通过刚体对象调用物理系统的AddForce方法
      //对游戏对象施加一个力，使其移动
      rigidbody2D.AddForce(direction * force);
  }
  
  private void OnCollisionEnter2D(Collision2D collision)
  {
      Debug.Log($"齿轮子弹碰到了：{collision.gameObject}");
      //碰撞后销毁当前游戏对象
      Destroy(gameObject);
  }
  
  //如果还想控制子弹的最大飞行距离，在Update()下这样设置：
      if(transform.position.magnitude > 100.0f)
      {
          Destroy(gameObject);
      }
  ```

* 在角色脚本中添加发射子弹的脚本：

  ```C#
  //声明，将projectilePrefab暴露出来使得在Unity中可以自行设置projectilePrefab对应的素材
  public GameObject projectilePrefab;
  
  //发射函数Launch
  void Launch(){
      //将飞弹的预制件实例化，并设置其初始位置为角色的手的大致坐标，初始旋转角度为0（使用其本身的旋转角度）
      GameObject projectileObject = Instantiate(projectilePrefab, rigidbody2D.position + Vector2.up * 0.5f, Quaternion.identity);
      //调用飞弹的脚本对象
  	Bullet bullet = projectileObject.GetComponent<Bullet>();
      bullet.Launch(lookDirection, 300);
  }
  
  //Update()中将Launch()绑定到键盘输入
       if (Input.GetKeyDown(KeyCode.C))
       {
           Launch();
       }
  ```
  
  `public GameObject projectilePrefab`的作用：
  
  ![image-20240830230759917](E:/Typora/picture/image-20240830230759917.png)

* 运行后发现没有飞弹实例显示，并且报错，但是实际确实有飞弹与敌人发生碰撞，经debug发现角色调用`bullet.Launch(lookDirection, 300)`方法时，飞弹的Rigidbody对象为null：
  ![image-20240830234648468](E:/Typora/picture/image-20240830234648468.png)
  
  将Bullet的Start方法改为Awake方法，报错即消除。

* 然而再次运行时发现依然看不到飞弹（或者看到一瞬间即消失），这是因为飞弹的图层设置不对。

* 如果图层更改后依然看不到飞弹，那很有可能是角色给飞弹施加的力过大导致飞弹速度过快。

* 最后为Robot添加被命中后更改动画并移除刚体物理引擎效果：

  ```c#
  Animator animator;
  //Start中添加animator对象的自动获取
  animator = GetComponent<Animator>();
  public void Fix()
  {
      broken = false;//更改状态
  
      rigidbody2d.simulated = false;//关闭刚体物理引擎效果
  
      animator.SetTrigger("Fixed");//控制动画过渡
  }
  ```

  

## 图层

* 点击角色对象>Layer>Add Layer，为角色对象添加一个新的图层Character（高于当前所有图层），再添加一个高于Charater的图层设为Projectile。
* 为控制角色、飞弹的预制件设置为新的图层。

### 通过图层控制对象的碰撞

* 图层还有一个作用就是可以控制不同图层之间对象的是否碰撞。在**Edit>ProjectSettings>Physics 2D>Layer Collision Matrix**中勾除Projectile和Character之间的碰撞：
  ![image-20240831134241548](E:/Typora/picture/image-20240831134241548.png)

  这样角色发射飞弹时，就不会触发两者之间的碰撞了。

## VS2022挂载到Unity并进行debug

* 在**Unity>Edit>Preferences**中进行如下修改以进行挂载：

  ![image-20240830232936016](E:/Typora/picture/image-20240830232936016.png)

* 在VS2022中点击 **附加到Unity**：

  ![image-20240830233149476](E:/Typora/picture/image-20240830233149476.png)

* 设置断点，代码运行到断点时自动暂停，在下放窗口中可以搜索查看变量值和堆栈情况。

## 视角跟随

目前只介绍一种简单的方法：

[Ruby's Adventure：2D 初学者 | Unity 中文课堂 (u3d.cn)](https://learn.u3d.cn/tutorial/unity-ruby-adventure)

**Window>Package Manager**，选择**Packages:Unity Registry**，搜索**Cinemachine**，安装。
![image-20240831202303339](E:/Typora/picture/image-20240831202303339.png)

在Hierarchy中，右键>Cinemachaine>2D Camera

(本节请查看官方教程)

## 粒子入门

[粒子系统 - Unity 手册](https://docs.unity.cn/cn/2022.3/Manual/ParticleSystems.html)

* 在Hierarchy中右键，创建一个默认的特效：

  ![image-20240901100746641](E:/Typora/picture/image-20240901100746641.png)

  查看默认特效的效果：

  ![image-20240901104116746](E:/Typora/picture/image-20240901104116746.png)

* 特效对象下的Particle System即为其专有属性。我们想要一个烟雾效果，想要更换素材，首先需要更改Texture Sheet Animation栏（注意更改前需先勾选该属性）：

  ![image-20240901101943181](E:/Typora/picture/image-20240901101943181.png)

  更改Mode为Sprites，点击下面的+键即可为特效添加多种粒子。


### 为粒子添加随机性

* 此时查看效果发现虽然引入两种粒子但是特效只使用了一种。修改Start Frame为Random Between Two Constants，变成下面这种样子：
  ![image-20240901102642898](E:/Typora/picture/image-20240901102642898.png)

  两个变量的意义是，随机在[x,y)之间生成随机数，现在有两种粒子，将右边值修改为2，那么每个粒子在**其生命周期中**会在两种粒子中随机挂接。

* 但是又出现了新的问题：在粒子往上飘的过程中会随机改变其形态，而我们只想要其在刚生成时随机。
  点击Frame Over Time属性框内的白线，变成红线后打开Inspector最下面的预览界面（图中蓝圈标记）。

  ![image-20240901103523145](E:/Typora/picture/image-20240901103523145.png)

​		将红线调整为如下样子：

![image-20240901103913594](E:/Typora/picture/image-20240901103913594.png)

* 接下来调整粒子的锥形移动轨迹：

  在Shape栏中，更改Angle（扩散角度）和Radius（起始生成位置范围），然后再更改Start Speed和Start Size改为Random Between Two Constants。如下：

  ![image-20240901105237407](E:/Typora/picture/image-20240901105237407.png)
  
  Start Lifetime也需要更改。

* 我们还想粒子不要直接消失而是慢慢变透明：

  ![image-20240901110841752](E:/Typora/picture/image-20240901110841752.png)

  点击白框，弹出Gradient Editor窗口，点击右上角箭头，修改Alpha为0，即可做出渐变效果：

  ![image-20240901111017907](E:/Typora/picture/image-20240901111017907.png)

* 我们还希望粒子越往上飘越小，点击Size over Lifetime栏做如下更改：

  ![image-20240901111407897](E:/Typora/picture/image-20240901111407897.png)

### 特效绑定其他对象

* 为Robot绑定Smoke特效。先将特效制成预制件，然后点开Robot预制件，将SmokeEffect拖拽进去并成为Robot的子对象即可（**这一步的作用是使特效随Robot实例化时一同实例化，即Robot开始即显示特效**）。

* 运行查看效果，如果发现角色身上的特效不显示，检查特效对象的Z轴值是否为0，若不为更改即可。

* 绑定好后运行游戏，发现烟雾效果在移动的机器人上仍然是直着往上飞，这显然不符合物理规律：

  ![image-20240901222217255](E:/Typora/picture/image-20240901222217255.png)

  我们需要的效果是什么？烟应该相对于机器人向后飘，因此可以用将粒子位置的参考坐标系(**Simulation Space**)更改为World：

  ![image-20240901222521166](E:/Typora/picture/image-20240901222521166.png)

![image-20240901222632063](E:/Typora/picture/image-20240901222632063.png)

### 使用代码来控制粒子特效

* 获取烟雾特效对象：

    ```C#
    public ParticleSystem smokeEffect; //暴露获取对象的端口，ParticleSystem为粒子系统类
    ```

	![image-20240901225715711](E:/Typora/picture/image-20240901225715711.png)
	
	注意：拖拽进来的应该是Hierarchy中Robot对象下的Smoke Effect子组件而不是预制件中的。
	
	![image-20240901225842497](E:/Typora/picture/image-20240901225842497.png)

* 在脚本的`Fix()`下销毁粒子特效：

  ```c#
  Destroy(smokeEffect);
  ```

  如此便可使Robot被命中时即可去除烟雾效果。

* 考虑得更周到一点，烟雾特效不应该立刻消除。我们可以调用ParticleSystem对象的`Stop()`方法，即使特效停止产生新的粒子，但是原有的粒子仍会走完其生命周期：

  ```C#
  //替换原Destroy方法
  smokeEffect.Stop();
  ```

  

### 更多的例子

详见[Ruby's Adventure：2D 初学者 | Unity 中文课堂 (u3d.cn)](https://learn.u3d.cn/tutorial/unity-ruby-adventure?chapterId=63562b27edca72001f21d0af#61f10757a1f573001fa2396b)对应章节

==很多时候GPT确实能够辅助你实现一些粒子特效。==

还需要注意的是，Effect

#### 循环系统

你制作的**烟雾效果**是循环的效果，所以始终会生成新的粒子。但是，你可以创建只在特定时间内有效的粒子系统，完成任务后便自行销毁。为此，请进入 **Inspector** 中的 **Particle System** 主要部分：

- 取消勾选 **Looping**。
- 将 **Duration** 设置为你希望效果持续的时间。
- 将该部分底部的 **Stop Action** 设置为 **Destroy**。只有在所有粒子的生命周期都结束后，**粒子系统**才会被销毁，所以不会像手动销毁循环系统时那样出现“所有粒子立即消失”的问题。

#### 爆发发射

你的**烟雾效果**在之前是以稳定的速率发射粒子。你可以在 **Emission** 部分中设置该速率。**Rate over Time** 控制每秒发射多少粒子。

**Rate over Distance** 可以用于让系统在移动时发出粒子，例如，如果你希望只有在汽车移动时才发出烟雾粒子，那么这个功能很有用。

但对于某些效果，你可能想要一次发射很多粒子，仅此而已。比如，爆炸时会产生大量烟雾粒子，但爆炸结束后便不再产生烟雾粒子。

这就是 **Bursts** 子部分的用途。单击 **+ 按钮**可添加一个**爆发**，然后设置在粒子的生命周期内何时应该发生爆发，应生成多少粒子，以及其他一些设置。

例如，对于**命中效果粒子**，你可以：

- 禁用 **Looping** 并将 **Stop Action** 设置为 **Destroy**。
- 将 **Rate over Time** 设置为 **0**，这样就不会随时间生成任何粒子。
- 将 **20** 个粒子的爆发时间设置为 **0.0**。

然后，一旦创建了系统，便会生成许多粒子，然后在粒子都结束生命周期时自行销毁。

#### 实例化粒子系统

与**飞弹**一样，从**粒子系统**制作**预制件**后，便可以使用 **Instantiate 函数**创建粒子系统。

因此，对于仅发生一次的效果（例如被击中或拾取**生命值可收集对象**），你可以将对**预制件**的引用存储在公共变量中，并在应该产生效果时调用 **Instantiate**。如果 **Particle System** 未设置为 **Looping** 并且 **Stop Action** 设置为 **Destroy**，则会播放效果，然后销毁。

## UI设计

[官方教程项目系列/官方教程05_2D游戏/RubyAdventure/04-UI用户界面.md · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/blob/main/官方教程项目系列/官方教程05_2D游戏/RubyAdventure/04-UI用户界面.md)

**教程主UGUI，建议多接触一下最新的UI Toolkit，IMGUI也应该了解（程序员调试方便）**

所有的GUI都是在UI画布上设计的：

* 在Hierarchy中，**右键>UI>Canvas**

  ![image-20240902222638164](E:/Typora/picture/image-20240902222638164.png)

* 在Hierarchy中出现Canvas和EventSystem两个新对象，其中Canvas就是UI画布对象。

* 初始的画布会很大，点击Canvas对象后，需要缩小N多倍才能看到完整的画布的边框：

  ![image-20240902223142415](E:/Typora/picture/image-20240902223142415.png)

* 介绍Canvas的几个组件：
  * **Rect Transfrom**：拥有普通的Transfrom的功能，同时还有额外的UI的参数。
  * **Canvas Scaler**：缩放模式，UI Scale Mode属性有三种值：
    * **Constant Pixel Size**:固定像素大小
    * **Scale With Screen Size**:根据画面大小
    * **Constant Physical Size**:固定物理大小
  * **Canvas**:通过Render Mode控制UI的图层：
    * **Screen Space - Overlay**：在这种模式下，Canvas会覆盖整个屏幕，UI元素的位置坐标是屏幕空间的坐标。Overlay模式下，Canvas会填满整个屏幕空间，并将画布下面的所有UI元素置于屏幕的最上层。这意味着UI元素会始终显示在最前面，不受视角变化的影响。如果屏幕尺寸被改变，Canvas将自动改变尺寸来匹配屏幕。这种模式适用于2D UI元素，如菜单、得分、生命值等。
    * **Screen Space - Camera**：这种模式与Overlay模式类似，但Canvas会被放置在指定摄像机的前方。在这种渲染模式下，UI元素由摄像机渲染，这意味着摄像机的设置会影响UI的外观。如果摄像机设置为透视模式，UI元素将以透视图渲染，透视失真量可由摄像机视野控制。这种模式适用于需要与3D场景交互的UI元素，例如在3D游戏中显示的HUD（头上显示器）。
    * **World Space**：在这种模式下，Canvas被视为3D场景中的一个物体，其大小和位置可以在世界空间中自由设置。UI元素的位置坐标是世界空间的坐标，这意味着它们可以与其他3D对象交互，如遮挡和深度。这种模式适用于需要作为场景一部分的UI元素，例如在游戏世界中显示的指示牌或交互界面。
  * **Graphic Reycaster**：可以监测玩家是否点击了画布中的对象（比如按钮）*教程暂时不用*

* 接下来开始设计左上角头像及血条：

  * 在Canvas（重命名为UI）对象下右键创建一个UI>Image对象，将对应的Sprite拖入Image的Source Image。

  * 点击Image对象，按住Shift进行缩放（保持长宽高）。然后移动到UI的合适位置。

  * 虽然血槽目前的位置看上去很合适，但是更改UI界面长宽时会发现血条的位置也发生了改变（具体操作：点击Canvas的子对象，然后拖动Canvas边框以预览长宽更改后的UI）：

    ![image-20240902225726220](E:/Typora/picture/image-20240902225726220.png)

  * 仔细观察发现，更改中Canvas中间的锚点标记是没有改变相对位置的，我们想要血槽保持在UI的左上方，应该更改Image的锚点。

    点击Image，在Rect Transform中点击左边的方框：

    ![image-20240902230155569](E:/Typora/picture/image-20240902230155569.png)

    选择以下锚点：

    ![image-20240902230236545](E:/Typora/picture/image-20240902230236545.png)

    发现Canvas中间的锚点标记图形移到了左上角，说明设置成功。

  * 然后开始添加角色头像：

    在血槽Image下再新建头像Image，重复操作。

    此时更改血槽大小，发现角色头像不随其一起缩放。这是因为锚点不仅定义位置，还定义了大小。

    点击头像Image，将其锚点的四个角分别拖到与头像边框最为贴合的位置：

    ![image-20240902231110871](E:/Typora/picture/image-20240902231110871.png)

    ![image-20240902231139925](E:/Typora/picture/image-20240902231139925.png)

    再次拖拽发现可以跟随缩放了。
  
  * 接下来为添加可伸缩血条：
  
    先解释一个操作：**遮罩**
  
    *两个图像，一个是原图像，一个是遮罩图像，显示的是原图像，遮罩图像控制原图像的显示范围，只显示背遮罩区域的原图像。*
  
    *所以可以通过更改遮罩图像的大小，来不失真地显示原图像的部分区域*
  
    在血槽Image下创建一个新的遮罩Image，遮罩层无关自身的图像和颜色。调整图片的大小以刚好完全遮住整个血槽：
  
    ![image-20240903082320813](E:/Typora/picture/image-20240903082320813.png)
  
    最后我们需要移动遮罩层的轴心，原因是待会儿用代码调整遮罩层大小时会根据轴心位置来调整四周的收缩，因此为了收缩时右边收缩而左边不动，应当将轴心移动至最左侧。
  
    *注意这里如果不能拖动轴心的话需要将scene视图的工具栏左上角两个设置为pivot和globe。*
  
    添加好遮罩层后，为UI添加真正的血条：在遮罩Image下创建血条Image并设置SourceImage。调整位置以完全适合遮罩。
  
    *还有一种快速填充的方法——在血条Image中点击Anchor Assets中按住Alt并选中右下角的设置方法，注意确保血条Image是遮罩Image的子组件*
  
    ![image-20240903083815504](E:/Typora/picture/image-20240903083815504.png)
  
    最后为遮罩层设定遮罩——为遮罩Image添加Mask组件，并取消Show Mask Graphic。缩小遮罩层时，发现血条也随之而缩小。设置成功。
  
  * 接下来通过新建脚本代码绑定角色生命值并控制血条收缩。
  
    ```c#
    //创建公有静态成员属性，获取当前血条本身，即静态实例
    public static UIHaelthBar Instance { get; private set; }
    public Image mask;//需要导入UnityEngine.UI
    //记录遮罩层初始长度
    float originalSize;
    // Start is called before the first frame update
    void Awake()
    {
        //设置静态实例为当前对象
        Instance = this;
    }
    private void Start()
    {
        //获取遮罩层图像的初始宽度
        originalSize = mask.rectTransform.rect.width;
    }
    
    public void SetValue(float value)
    {
        //设置更改的是mask遮罩层的宽度，并且根据传参进行更改
    	mask.rectTransform.SetSizeWithCurrentAnchors
            (RectTransform.Axis.Horizontal, originalSize * value);
    }
    ```
  
    在角色的ChangeHealth方法中调用UIHaelthBar的SetValue方法：
  
    ```C#
    UIHaelthBar.Instance.SetValue(currentHealth / (float)maxHealth);
    ```
  
    最后在遮罩Image中添加控制脚本并将遮罩Image拖入脚本的Mask属性。
  
    现在可以运行游戏测试效果了。

## 世界交互——对话

* 创建NPC角色，绑定Animator，调整Pivot，添加碰撞体组件，制成预制件。（略）


### 射线投射

* 新建一个图层"NPC"并将NPC设置在此图层。（**这一步非常关键**）


* 想要角色靠近NPC时即可使用交互按键进行对话，可以使用之前讲过的Collider Trigger，然而存在的问题是：我们背对着NPC时不应该能与它交谈。这里就可以用到新的物理系统——射线投射。

  [Ruby's Adventure：2D 初学者 | Unity 中文课堂 (u3d.cn)](https://learn.u3d.cn/tutorial/unity-ruby-adventure?chapterId=63562b27edca72001f21d0af#61f10787eb46bf001f564044)

  在角色的控制脚本的Update()下添加以下代码：

  ```C#
  if (Input.GetKeyDown(KeyCode.X)){
      //创建一个射线投射碰撞对象，来接收射线投射碰撞信息
      //Physics2D.Raycast方法的参数为：
      //射线投射的位置，投射方向，投射距离，射线生效的层
      RaycastHit2D hit = Physics2D.Raycast(rigidbody2D.position + Vector2.up * 0.2f, lookDirection, 1.5f, LayerMask.GetMask("NPC")); //此处的图层名字应和NPC所在图层名字相同
      if (hit.collider != null)
      {
          Debug.Log($"射线投射碰到的对象是：{hit.collider.gameObject}");
      }
  }
  ```

### 对话UI

* 为对话设置一个对话框（注意，这里要实现的对话框是类似气泡的在NPC旁边的小对话框）：

  在NPC对象下创建一个UI>Canvas，将Render Mode设置成World Space以自定义大小，然后设置Canvas的position、wid/hei、scale（position应该大致在绑定NPC的上下左右的位置，其他的看心情，但最后的大小是同时由宽高和scale决定的。）

  ![image-20240903181841883](E:/Typora/picture/image-20240903181841883.png)

* 在Canvas里面设计对话内容。添加Image，填充整个Canvas。

* 在Image下添加UI>Test(TMP)对象，第一次使用时会弹出提示窗口：

  ![image-20240903182554685](E:/Typora/picture/image-20240903182554685.png)

  *点击第一个按钮，Unity会在你的项目下自动生成一个TextMesh Pro的文件夹，包含了一些TMP需要的资源、预设、配置。*

  *第二个按钮顾名思义，为你提供一些样例和额外的资源，如果想进一步学习、了解，可再次Import。*

  *如果未有过有过Import TMP Examples & Extras操作，之后可自行设置**Window>TextMeshPro>Import TMP Examples & Extras***

* 同样将文本框填充到整个Image，如果想预留出边框的宽度，请自行调整锚点和大小。

* 在Text对象的Inspector下面输入文本，在Main Settings>Font Asset中可以设置字体

  *注意：中文字体需要自己导入，否则会出现显示问题*

  *导入中文字体：*

  [【Unity TMP外部字体导入问题】TMP中文，将字体生成为TMP_FontAsset常见问题_unity font asset-CSDN博客](https://blog.csdn.net/PinaColadaONE/article/details/124887148)（尚未试过）

### 显示对话

* 上面测操作之后对话框会保持显示，我们需要设置成只有角色与NPC交互时才显示对话内容。

* 为对话交互新建一个脚本：

  ```C#
  public class NonPlayerCharacter : MonoBehaviour
  {
      //对话框显示时长
      public float displayTime = 4.0f;
      //获取对话框游戏对象
      public GameObject dialogBox;
      //计时器，倒计时文本显示时间
      float timerDisplay;
      // Start is called before the first frame update
      void Start()
      {
          //设置对话框初始时不显示
          dialogBox.SetActive(false);
          //计时器设置成不可用的状态
          timerDisplay = -1.0f;
      }
  
      // Update is called once per frame
      void Update()
      {
          //如果倒计时尚未结束，进行倒计时
          if(timerDisplay >= 0.0f)
          {
              timerDisplay -= Time.deltaTime;
              if(timerDisplay < 0.0f)
              {//否则，倒计时结束，隐藏对话框
                  dialogBox.SetActive(false);
              }
          }
      }
  
      public void DisplayDialog()
      {
          //重置计时器
          timerDisplay = displayTime;
          //显示对话框
          dialogBox.SetActive(true);
      }
  }
  
  ```

  然后在控制角色中调用交互脚本暴露的方法：

  ```C#
  if (Input.GetKeyDown(KeyCode.X)){
      //创建一个射线投射碰撞对象，来接收射线投射碰撞信息
      //Physics2D.Raycast方法的参数为：
      //射线投射的位置，投射方向，投射距离，射线生效的层
      RaycastHit2D hit = Physics2D.Raycast(rigidbody2D.position + Vector2.up * 0.2f, lookDirection, 1.5f, LayerMask.GetMask("NPC")); 
      if (hit.collider != null)
      {
          Debug.Log($"射线投射碰到的对象是：{hit.collider.gameObject}");
          //new here
          //获取hit碰到的对象的NonPlayerCharacter组件
          NonPlayerCharacter npc = hit.collider.GetComponent<NonPlayerCharacter>();
          if (npc != null)
          {
              npc.DisplayDialog();
          }
      }
  }
  ```

  接下来注意：将交互脚本NonPlayerCharacter.cs挂载到==NPC对象==，然后将Dialog Box属性设置为其下的==DialogCanvas对象==

  *这里的原因在于：*

  1. *hit.collider一定是NPC身上的碰撞体（因为NPC对象的子对象并没有任何碰撞体），所以交互脚本不能挂载到其子对象上。*
  2. *至于能否将Dialog Box属性设为Image，其实也可以，但是能看出如此设置后DialogCanvas不会隐藏，确保万无一失还是应该设为DialogCanvas对象。*

* 最后可以运行查看效果了。

### TextMeshPro介绍

在TMP对象下有一个固定生成的组件——TextMeshPro，下面我们来详细介绍一下该组件（虽然仍然只是冰山一角）

![image-20240904015621742](E:/Typora/picture/image-20240904015621742.png)

#### 使用中文

使用TMP自带的字体，尝试输入中文发现无法正常显示。

[Unity常用包/TextMeshPro/TextMeshPro使用中文方法.md · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/blob/main/Unity常用包/TextMeshPro/TextMeshPro使用中文方法.md)

【原视频教程】https://www.bilibili.com/video/BV1Mr4y1X76H?p=110&vd_source=28e3994bf6aa5fa346ca67b74f2c51f6

* 动态加载字体相对比较简单方便，但是如果是文字内容过多的游戏这种方法的性能较差。
* 静态字体效率会比动态的高，但是设置比较麻烦（还是要基于导入项目的动态字体）。

#### 大段文字翻页显示

* 更改TMP对象的如下属性：

  ![image-20240904022219383](E:/Typora/picture/image-20240904022219383.png)

  这样大段文字就不会溢出文本框了。

  *Page值自设，你设置几页Scene就会显示第几页的预览，并不影响Unity将文本内容拆分成多少页。*

* 然后我们需要通过脚本激活翻页显示文本：

  更改NonPlayerCharater脚本如下：
  
  ```C#
  public class NonPlayerCharacter : MonoBehaviour
  {
      //对话框显示时长
      public float displayTime = 4.0f;
      //获取对话框游戏对象
      public GameObject dialogBox;
      //计时器，倒计时文本显示时间
      float timerDisplay;
      //获取TMP游戏对象
      public GameObject TMPGameObject;
      //获取游戏组件类对象
      TextMeshProUGUI textMeshPro;
      //设置初始页标
      int currentPage = 1;
      //获取当前页数
      int totalPages;
      // Start is called before the first frame update
      void Awake()
      {
          
      }
  
      void Start()
      {
          //设置对话框初始时不显示
          dialogBox.SetActive(false);
          //计时器设置成不可用的状态
          timerDisplay = -1.0f;
          //获取对话框组件
          textMeshPro = TMPGameObject.GetComponent<TextMeshProUGUI>();
          
      }
  
      // Update is called once per frame
      void Update()
      {
          if (timerDisplay >= 0.0f){
              totalPages = textMeshPro.textInfo.pageCount;
              if (Input.GetKeyUp(KeyCode.Space)){
                  timerDisplay = displayTime;
                  if (currentPage < totalPages)//如果没到最后一页，向后翻页
                  {
                      Debug.Log($"{currentPage} page count: {totalPages}");
                      currentPage++;
                  }
                  //显示当前一页
                  textMeshPro.pageToDisplay = currentPage;
              }
              timerDisplay -= Time.deltaTime;
          }
          else
          {
              currentPage = 1;
              dialogBox.SetActive(false);
          }
  
      }
  
      public void DisplayDialog()
      {
          //重置计时器
          timerDisplay = displayTime;
          //显示对话框
          dialogBox.SetActive(true);        
      }
  }
  
  ```
  
  * 这个代码中存在的一些雷点：
    1. 如果TMP对象实例化不完全，`totalPages = textMeshPro.textInfo.pageCount`获取不到正确的Page Count，将这句话放到Update的if内，一是需要时能持续获取正确的值，二是优化性能（避免持续忙）。尝试过在`DisplayDialog()`中用While语句以等待完全实例化后再一次赋值，但是调用`DisplayDialog()`后Unity会卡死，故放弃。
    2. 计时器的持续判断不建议嵌套二次判断其对立条件，很难把握二次判断的时机，最好的解决方法就是单独拎出来成一个else。

## 音频

[官方教程项目系列/官方教程05_2D游戏/RubyAdventure/05-音频入门.md · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/blob/main/官方教程项目系列/官方教程05_2D游戏/RubyAdventure/05-音频入门.md)

### 添加背景音乐

* 在**Project>Assets>Audio**下找到相应的背景音乐的Audio Clip。

* 在Hierarchy Create一个新的空对象，并为其挂载一个Audio Source，将背景音乐的Audio Clip拖入AudioClip属性。
* 确认Audio Source组件的Special Blend属性的滑块在2D一边。否则角色移动时BGM会有方向感。

### 代码控制音乐播放

* 在本次教程中选用一对多的声音播放方法，需要注意的是：这个方法同样能实现听声辨位，但是依赖大量的脚本代码来控制。这种方法的好处是减少音频源的数量，从而提升性能。

尝试为吃加血道具设置音效：

* 在控制角色预制件中添加Audio Source。确认勾选了Play On Awake。

* 进入角色的控制脚本。第一步需要声明音频源对象：

  ```C#
  //声明音频源对象，用来后期进行播放控制
  AudioSource audioSource;
  ```

  在Start时，获取声音组件源对象：

  ```C#
  audioSource = GetComponent<AudioSource>();
  ```

  新增一个公有方法，其他脚本一经调用会播放指定的音频剪辑：

  ```C#
  public void PlaySound(AudioClip clip, float volumn)
  {
      //调用音频源的PlayOneShot方法，播放指定的音频
      audioSource.PlayOneShot(clip, volumn);
  }
  ```

  在回血道具的脚本中，添加以下代码：

  ```C#
  //声明公开AudioCLip对象和volumn对象
  public AudioClip collectClip;
  public float soundVolumn;
  //在合适的位置添加播放音频剪辑的方法
  rubyController.PlaySound(collectClip, soundVolumn);
  ```

  同样的，可以在其他地方尝试挂载音频播放。

* 个人认为，如果真要以上述方式实现一对多的音频播放的话，应该将AudioSource对象和PlaySound方法均设为static，这样其他脚本中不用获取当前角色的脚本对象就能进行播放（实测可以，但是不知道会不会有隐藏的bug，感觉3D的话不一定可行）。

### 空间化

为敌人添加3D脚步音频：

* 同样添加Audio Source组件，将Special Blend属性的滑块拖到3D一边，勾选Loop属性。

* 点开组件最下面的3D Sound Settings，通过这个设置可以编辑最大传播范围和音量衰减：

  ![image-20240904213832441](E:/Typora/picture/image-20240904213832441.png)

  点击敌人对象，蓝圈之内就是可听到脚步声的范围：
  ![image-20240904213951698](E:/Typora/picture/image-20240904213951698.png)

  

* 运行发现听不到脚步声，我们需要切换到3D界面：

  ![image-20240904214119356](E:/Typora/picture/image-20240904214119356.png)

  不难看出：因为我们设置的摄像头的图层过高，导致摄像头中的listener不在Robot的脚步声传播范围内。

  * 教程给的解决方法是：创建一个Visual Camera（控制MainCamera行为的虚拟摄像头）的子对象，设置其position.z=-maincamera.transfrom.position.z来抵消在Z轴上的坐标偏移。然后在这个子对象上添加一个Audio Listener并删除原来的。

  * 也有同学提出：可以直接给控制角色挂载Listener，这还比上面的方法简单，同时还避免了角色走到地图边缘时摄像机并不完全跟随角色导致视觉和听觉上出现方向差的问题。

  * 教程给的这个游戏项目是可以将Listener挂载到角色身上的。但有同学指出——这种方法会导致切换摄像头还是会接收到角色身上Listener收到的声音（可以参考一些FPS游戏的观战视角）。只能说各有各的缺陷，根据情况自选即可。

    

* 还有一个问题是：当Robot被修复后应该关闭声音。

  解决方法也很简单，通过代码获取机器人的Audio Source组件，调用Stop方法停止播放。

  ```C#
  GetComponent<AudioSource>().Stop();
  ```

  利用之前的知识，我们可以通过脚本实现机器人被修复后切换音频。在此不多赘述。

## 构建、运行、分发

这里老师一P带过，本身该项目也是学习用，故不再赘述。

[Ruby's Adventure：2D 初学者 | Unity 中文课堂 (u3d.cn)](https://learn.u3d.cn/tutorial/unity-ruby-adventure?chapterId=63562b27edca72001f21d0af#61f107cdec04af00209b9fa8)

# 2D经验

## 关于素材

一定是要先确定素材是否好用，裁切和绘制网格是否容易，否则很容易出现项目做到一半发现因为素材质量差导致进度阻塞。

## 放大素材

在像素游戏制作中，第一个遇到的问题就是角色在画面中太小。尝试更改分辨率也没太大用处。实际上想要更改素材在编辑器中的显示大小，除了更改Scale（我不觉得这是个好方法）以外，还有一个东西可以影响——图片素材的Pixels per Unit：

*PPU是一个重要的设置，它定义了每单位长度包含多少像素。默认情况下，Unity中的PPU值是100，这意味着每单位长度显示100像素。*

对于像素素材，设置为64/32/16这些值比较合适。

## Layer和Sorting Layer

被这个问题硬控一小时。我以为渲染时会根据Layer由高到低的顺序来显示，然而添加角色动画后发现会随机有几帧动画会被背景覆盖。搜了一下才发现：对象的Order in Layer属性中的"Layer"指的是Sorting Layer。

总的来说，Sorting Layer控制显示顺序（谁覆盖谁），Layer负责控制Camera的渲染和不同物体的碰撞。

将角色始终显示在背景前面的方式有两种：更改Sorting Layer为更高的值，或者在同一Sorting Layer中设置更高的Order In Layer。

## 角色移动应该用什么控制

使用RigidBody控制Position会发现匀直移动时帧率不高且略有卡顿，而使用Transform则没有这个问题，但是使用Transfrom的时候碰撞时会抖动。

有大佬使用velocity来解决，我试了一下，确实是非常丝滑，下面是GPT给出的使用velocity的：

是的，`velocity`属性在Unity的2D游戏开发中同样可以使用。在2D游戏中，`Rigidbody`组件被替换为`Rigidbody2D`组件，它具有类似的功能，但是专门针对2D环境进行了优化。

在2D游戏中，`Rigidbody2D`的`velocity`属性是一个`Vector2`类型，它只包含水平（x轴）和垂直（y轴）两个方向的速度分量。以下是如何在2D游戏中使用`Rigidbody2D`的`velocity`来控制移动的步骤：

1. **获取Rigidbody2D组件**：首先，获取你想要控制移动的游戏对象上的`Rigidbody2D`组件。

```csharp
Rigidbody2D rb2D;
void Start()
{
    rb2D = GetComponent<Rigidbody2D>();
}
```

2. **在FixedUpdate中更新速度**：在2D游戏中，物理更新同样发生在`FixedUpdate`方法中。

```csharp
void FixedUpdate()
{
    // 你的移动逻辑
}
```

3. **计算移动方向**：使用`Input.GetAxis`来获取水平和垂直输入，并创建一个`Vector2`来表示移动方向。

```csharp
Vector2 movement;
float moveHorizontal = Input.GetAxis("Horizontal");
float moveVertical = Input.GetAxis("Vertical");
movement = new Vector2(moveHorizontal, moveVertical);
```

4. **设置速度**：将计算出的移动方向乘以一个速度因子，然后赋值给`Rigidbody2D`的`velocity`属性。

```csharp
void FixedUpdate()
{
    Vector2 movement = new Vector2(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical"));
    rb2D.velocity = movement * speed;
}
```

在这里，`speed`是你定义的一个变量，用于控制移动的速度。

5. **考虑重力**：如果你的`Rigidbody2D`设置了受重力影响，你可能需要在计算速度时考虑垂直方向的重力影响。

```csharp
void FixedUpdate()
{
    Vector2 movement = new Vector2(Input.GetAxis("Horizontal"), 0.0f);
    rb2D.velocity = movement * speed;
}
```

在这个例子中，我们只更新了水平方向的速度，垂直方向的速度保持不变，这样可以让角色在跳跃时保持当前的垂直速度，直到重力或其他力（如碰撞）改变它。

6. **平滑停止**：如果你希望角色在停止输入后能够平滑地停下来，可以在没有输入时逐渐减小速度。

```csharp
void FixedUpdate()
{
    if (Input.GetAxis("Horizontal") == 0 && Input.GetAxis("Vertical") == 0)
    {
        rb2D.velocity = Vector2.Lerp(rb2D.velocity, Vector2.zero, deceleration * Time.fixedDeltaTime);
    }
    else
    {
        Vector2 movement = new Vector2(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical"));
        rb2D.velocity = movement * speed;
    }
}
```

在这里，`deceleration`是一个用于控制减速快慢的变量。

使用`Rigidbody2D`的`velocity`属性来控制2D游戏中的移动是一种有效的方法，它可以提供流畅和自然的物理行为。

查看了一下RubyAdventure的代码，感觉确实是没把坐标控制放进FixedUpdate的问题。

## 移动优化

[游戏中角色运动的一些问题（附代码） - 技术专栏 - Unity官方开发者社区](https://developer.unity.cn/projects/6650b1a0edbc2a001ddaafae)

## 施加瞬时力

直接使用AddForce(Vector2.Up * force)没有起作用，看博客才知道原来Unity中力也是分种类的，使用AddForce(Vector2.Up * force, ForceMode.Impulse)就解决问题了：

在Unity中，`ForceMode` 是一个枚举类型，用于指定如何应用力到 `Rigidbody` 上。不同的 `ForceMode` 值会影响力的计算方式，从而产生不同的物理效果。以下是 `ForceMode` 的几种模式及其用途：

1. **ForceMode.Force**：
   - 应用一个持续的力。这个力会考虑物体的质量，并且会持续影响物体，直到力不再被应用。
   - 适合模拟如风力或连续推力的效果。

2. **ForceMode.Impulse**：
   - 应用一个瞬间的脉冲力。这个力不考虑物体的质量，并且会在下一帧立即改变物体的速度。
   - 适合模拟如跳跃或被击中的效果。

3. **ForceMode.Acceleration**：
   - 应用一个持续的加速度。这个加速度会考虑物体的质量，并且会持续影响物体，直到加速度不再被应用。
   - 适合模拟如引擎加速的效果。

4. **ForceMode.VelocityChange**：
   - 应用一个瞬间的速度变化。这个速度变化不考虑物体的质量，并且会在下一帧立即改变物体的速度。
   - 适合模拟如爆炸或快速停止的效果。

5. **ForceMode.Force** 和 **ForceMode.Impulse** 的区别：
   - `Force` 模式下，力会持续作用，并且会根据物体的质量来计算加速度。
   - `Impulse` 模式下，力是瞬间的，不考虑物体的质量，直接改变速度。

使用示例：

```csharp
using UnityEngine;

public class ApplyForceExample : MonoBehaviour
{
    private Rigidbody rb;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            // 应用一个瞬间的脉冲力，用于跳跃
            rb.AddForce(Vector3.up * 10, ForceMode.Impulse);
        }
    }
}
```

在这个例子中，当按下空格键时，会通过 `ForceMode.Impulse` 模式给 `Rigidbody` 施加一个向上的脉冲力，这通常用于模拟跳跃。

选择正确的 `ForceMode` 对于实现预期的物理效果非常重要。你需要根据你的游戏逻辑和物理行为的需求来选择合适的模式。

## 攻击判定

[Unity2D游戏制作入门 | 14( 之人物实装攻击判定 )_unity 2d动作攻击检测-CSDN博客](https://blog.csdn.net/widhejd393/article/details/139770048#:~:text=Unity)

## UGUI

### 组件详解

[Unity UGUI 完整系列教程-Chinar中文图解_unity ui-CSDN博客](https://blog.csdn.net/ChinarCSDN/article/details/90313617#:~:text=文章浏览阅读)

### 菜单切换

【unity-UI界面切换方法】https://www.bilibili.com/video/BV13q4y1D72Y?vd_source=28e3994bf6aa5fa346ca67b74f2c51f6

### 事件系统与用户交互

事件系统在创建Canvas时会伴随生成。在Unity中，`EventSystem`组件负责处理输入事件（如鼠标点击、触摸屏幕等）并将其发送给UI元素，如按钮、滑块等。`EventSystem`通常不需要额外的脚本来执行其基本功能，因为它自动处理大部分的交互逻辑。

具体添加用户交互见这篇博客：[Unity UGUI的EventSystem（事件系统）组件的介绍及使用_event selected unity-CSDN博客](https://blog.csdn.net/alianhome/article/details/131906274)

### 简易暂停菜单

在Canvas下新建一个Panel再在Panel下新建三个Button，分别对应继续、重新开始、退出。再在Scene下建一个监听ESC键的空对象。

```C#
// 监听器
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class OnEsc : MonoBehaviour
{
    public GameObject pauseMenuUI; // 指向你的Panel对象
    [SerializeField] bool isPaused = false; // 游戏是否暂停的标志

    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Escape))
        {
            if (isPaused)
            {
                ResumeGame();
            }
            else
            {
                PauseGame();
            }
        }
    }

    void PauseGame()
    {
        // 暂停游戏
        Time.timeScale = 0f;
        isPaused = true;

        // 显示Panel并设置Alpha值为1
        pauseMenuUI.SetActive(true);
        pauseMenuUI.GetComponent<CanvasGroup>().alpha = 1;
        //Image panelImage = pauseMenuUI.GetComponent<Image>();
        //if (panelImage != null)
        //{
        //    panelImage.color = new Color(panelImage.color.r, panelImage.color.g, panelImage.color.b, 1f);
        //}
    }

   public void ResumeGame()
    {
        // 恢复游戏
        Time.timeScale = 1f;
        isPaused = false;

        // 隐藏Panel并设置Alpha值为0
        //Image panelImage = pauseMenuUI.GetComponent<Image>();
        //if (panelImage != null)
        //{
        //    panelImage.color = new Color(panelImage.color.r, panelImage.color.g, panelImage.color.b, 0f);
        //}
        pauseMenuUI.SetActive(false);
        pauseMenuUI.GetComponent<CanvasGroup>().alpha = 0;
    }
}
```

```C#
// Continue
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class ContinueButton : MonoBehaviour
{
    public Button continueButton;

    public GameObject menuListener;
    void Start()
    {
        continueButton.onClick.AddListener(OnClick);
    }
    public void OnClick()
    {
        menuListener.GetComponent<OnEsc>().ResumeGame();
        Debug.Log("ContinueButton Clicked");
    }
}

```

```C#
// Restart
using UnityEngine;
using UnityEngine.SceneManagement; 
using UnityEngine.UI;
public class RestartButton : MonoBehaviour
{
    public Button restartButton;

    void Start()
    {
        // 获取按钮引用
        restartButton = GetComponent<Button>();
        // 添加点击事件监听器
        restartButton.onClick.AddListener(RestartGame);
    }

    // 重置游戏的方法
    void RestartGame()
    {
        SceneManager.LoadScene(0);
        Time.timeScale = 1f;// 这里是因为呼出菜单的时候暂停画面了，需要继续播放
    }
}
```

```C#
// Exit
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.UI;

public class ExitButton : MonoBehaviour
{
    public Button exitButton; 

    void Start()
    {
        // 添加点击事件监听器
        exitButton.onClick.AddListener(ExitGame);
    }

    // 退出游戏的方法
    void ExitGame()
    {
        // 退出游戏
        UnityEditor.EditorApplication.isPlaying = false;
    }
}
```

## 滚动背景

[Unity中的图片循环滚动实现_unity图片循环滚动-CSDN博客](https://blog.csdn.net/Wu_0526/article/details/128578967)

对于教程中非UI的图片滚动，在Hierarchy中创建的图片对象是直接将图片素材拖入Hierarchy创建的。

## 背包UI设计

[超级详细的游戏常见系统之背包系统制作以及优化_用户 背包 道具 游戏 底层表设计-CSDN博客](https://blog.csdn.net/qq_42385019/article/details/107922452)

## 对象禁用启用与方法是否可运行的关系

当Unity中的对象被禁用时，其上挂载的脚本的行为会有所不同，具体取决于你指的是哪种“运行”。

### 1. **更新循环中的方法不会运行**

- **不会运行**：如果一个游戏对象被禁用，那么Unity不会调用该对象上任何组件的更新循环中的方法，如`Update()`, `FixedUpdate()`, `LateUpdate()`等。这意味着，如果你依赖这些方法来执行游戏逻辑或计算，当对象被禁用时，这些逻辑将不会执行。

### 2. **生命周期方法**

- **Awake**：即使对象被禁用，`Awake` 方法仍然会运行，因为`Awake` 在对象首次激活时执行，并且仅执行一次，与对象的激活状态无关。
- **Start**：如果对象在游戏开始时是禁用的，`Start` 方法不会执行。但如果对象在游戏运行中被启用，且之前未执行过`Start`，则在它第一次被启用时`Start` 会执行。
- **OnEnable** 和 **OnDisable**：这两个方法与对象的启用和禁用直接相关。`OnEnable` 在对象被启用时调用，而`OnDisable` 在对象被禁用时调用。

### 3. **可以手动触发脚本中的方法**

- **手动调用**：即使对象被禁用，你仍然可以通过代码从其他激活的对象或组件中调用该对象上脚本的某些方法。例如，你可以从另一个游戏对象的`Update`方法中调用一个方法，即使该对象当前是禁用的。

### 4. **事件和委托**

- **事件**：对象上脚本的事件监听和触发仍然可以工作，即使对象本身被禁用。但是，如果事件的触发依赖于更新循环中的方法（如`Update`），那么在对象被禁用时这些事件不会被触发。

# 3D教程

## 角色设计

### 导入模型并设置

* 将Assets/Models/Charaters文件夹下的角色模型JohnLemon拖入Hierarchy。

  ![image-20240905085814330](E:/Typora/picture/image-20240905085814330.png)

* 点击JohnLemon对象查看其子对象，其中除了一个和对象自身同名的子对象（主要是皮肤网格渲染）外还有几个Root对象，按Alt键点击Root旁的三角就能展开其全部子对象，实际就是绑定的模型身体的各个部分的骨骼。

* 点击任一个骨骼对象，可以通过transform来改变其位置以影响模型外观。

* 将JohnLemon对象拖入Prefabs制成预制件。下面我们对角色的预制件进行操作。

* 在Assets/UnityTechnologies/Animation/Animators下新建一个Animation Controller用于制作角色动画。将其拖进（父）JohnLemon对象下的Animator组件的Controller属性中。

* 点开Animation Controller对象，将Assets/UnityTechnologies/Animation/Animation下所有JohnLemon的动画全部添加进来并添加transition，然后添加一个Bool类型的变量IsWorking：

  ![image-20240905091421122](E:/Typora/picture/image-20240905091421122.png)

  取消两个transition的Has Exit Time属性，添加IsWalking的Condition。

* 运行一下，就能看到角色静止时的动画了。

### 为角色添加物理系统

* 在预制件中添加RIgidBody组件，直接运行查看效果，发现角色在缓慢上升，为解释这种现象，这里附上官方教程的内容：

  ### **什么是根运动 (Root Motion)？**

  动画用于在特定层级视图中移动和旋转所有游戏对象。这些移动和旋转大多数都是相对于其父项完成的，但是层级视图的父游戏对象没有父项，因此它们的移动不是相对的。此父游戏对象也可以称为根 (Root)，因此其移动称为根运动 (Root Motion)。

  **重要注意事项！**在 JohnLemon 预制件的层级视图中称为根的游戏对象指的是其骨架的根，而不是实际的根游戏对象。根游戏对象是 Animator 组件所在的任何游戏对象，在本例中，该游戏对象称为 JohnLemon。

  在 Animator 组件上启用了 Apply Root Motion，因此根在动画中的任何移动都将应用于每一帧。由于 Animator 正在播放 Idle 动画，没有移动，因此 Animator 不会施加任何动作。那么，为什么 JohnLemon 游戏对象会移动呢？这是因为 Animator 的**更新模式 (Update Mode)**。

  ### **什么是更新循环？**

  游戏的工作方式与电影和电视类似：一幅图像显示在屏幕上，该图像每秒变化多次，给人以运动的感觉。我们称这些图像为**帧**；将这些帧绘制到屏幕上的过程称为**渲染**。对于电影和电视，通常会预先定义要在屏幕上显示的下一幅图像，但是在游戏中，下一幅图像可能会发生巨大变化，因为用户会对接下来发生的事情产生影响。每幅图像都需要根据用户输入进行计算 — 由于这种变化可能在转瞬间发生，因此计算显示内容的程序要以同样快的速度进行运算。这称为**更新循环**。

  每次显示帧时，都会依序发生许多事情。您现在只需要知道，自定义组件的 Update 方法被调用后，会在屏幕上渲染新图像。这些更新的长度会有所不同，具体取决于计算和渲染的复杂程度。但是，还有另一个单独的循环可以运行所有物理操作。此循环不会改变更新的频率，因此称为 **FixedUpdate**。

  Animator 组件可以更改其执行更新的时间。默认情况下根据渲染执行此更新。这意味着 Animator 在 Update 中移动角色，而 Rigidbody 同时在 Fixed Update 中移动角色。这就是造成您的问题的原因，很容易解决！

* 上面这么大段看不懂很正常，简单理解一下就是：在Animator中启用Apply Root Motion 之后，动画的运动会体现在实际游戏场景中，这里动画的运动和刚体的运动冲突了。想要解决这个问题，我们需要如下操作：

  将Animator的Update Mode属性选为Animate Physics，Normal值表示动画更新在Update()里面执行，而Animate Physics的刷新模式则为FixedUpdate()，这样就完成了动画和物理的同步更新。

  *其实取消Apply Root Motion也能解决这个问题，但是由于后续的控制中没有对角色坐标的控制，所以以上面的解决方法为准。*

* 然后我们还需要对刚体进行约束：首先，由于这是一个平面移动游戏，所以应该锁定Y轴上的位置，其次，我们允许在Y轴上的旋转而不允许另外两个的旋转（即只能转身而不能"磕头"），所以应该锁定XZ轴上的rotation。

### 为角色添加碰撞体

* 为角色添加一个Capsule Collider，调整碰撞体大小以贴合实际模型。

  Tips: 使用鼠标操作碰撞体大小可能并不精确，可以在Capsule Collider组件下直接编辑其参数：

  ![image-20240905101432047](E:/Typora/picture/image-20240905101432047.png)



### 添加控制角色的脚本

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    // 创建一个3D矢量，来表示玩家角色移动
    Vector3 _moveMent;
    // 创建获取用户输入的方向
    float horizontal;
    float vertical;
    Rigidbody _rigidbody;
    Animator _animator;
    // 用四元数对象来表示3D游戏中的旋转
    Quaternion _rotation = Quaternion.identity;// 初始化四元数对象，表示不旋转
    public float rotateSpeed;
    // Start is called before the first frame update
    void Start()
    {
        _rigidbody = GetComponent<Rigidbody>();
        _animator = GetComponent<Animator>();
    }

    // Update is called once per frame
    void Update()
    {
        horizontal = Input.GetAxis("Horizontal");
        vertical = Input.GetAxis("Vertical");
    }

    private void FixedUpdate()
    {
        // 将用户输入组装成3D运动需要的三维矢量
        _moveMent.Set(horizontal, 0.0f, vertical);
        // 将方向矢量归一化
        _moveMent.Normalize();
        // 判断是否有横向移动
        bool hasHorizontal = !Mathf.Approximately(horizontal, 0.0f);
        bool hasVertical = !Mathf.Approximately(vertical, 0.0f);
        // 只要有一个方向的移动，就认为角色处于移动状态
        bool isWalking = hasHorizontal || hasVertical;
        // 将变量传递给动画管理器
        _animator.SetBool("IsWalking", isWalking);
        // 用三维矢量来表示转向后的玩家角色的朝向
        Vector3 desiredForward = Vector3.RotateTowards(transform.forward, _moveMent, rotateSpeed * Time.fixedDeltaTime, 0.0f);
        // 设置四元数对象的值
        _rotation = Quaternion.LookRotation(desiredForward);
    }

    private void OnAnimatorMove()
    {
        // 使用动画中每次0.02秒移动的距离作为帧距离来移动
        _rigidbody.MovePosition(_rigidbody.position + _moveMent * _animator.deltaPosition.magnitude);
        // 旋转游戏对象
        _rigidbody.MoveRotation(_rotation);
    }
}

```

在Unity中修改Rotate Speed属性值并运行即可看到效果。

与2D的移动不一样的是，考虑到根运动，游戏工程师并不需要自己设定帧运动距离，可以直接使用动画的（动画相关并不是动画工程师的工作）。

还要再补充一下3D旋转的知识：

[Unity - Scripting API: Vector3.RotateTowards (unity3d.com)](https://docs.unity3d.com/ScriptReference/Vector3.RotateTowards.html)

关于Vector3.RotateTowards，有一点讲的不是很清楚的是，第二个参数**target**指的是目标方向而非转向，这一点在我自己理解时误解了很久。

[Unity - Scripting API: Quaternion (unity3d.com)](https://docs.unity3d.com/ScriptReference/Quaternion.html)

*总而言之，不会的时候就去查Unity Script API*

## 场景

将Prefabs中的Level预制件拖入Hierarchy中，然后修改控制角色的position为(-9.8, 0, -3.2)。

### 光照

基本概念：[官方教程项目系列/官方教程06_3D/John Lemon's Haunted Jaunt/02.光照简介.md · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/blob/main/官方教程项目系列/官方教程06_3D/John Lemon's Haunted Jaunt/02.光照简介.md)

#### 更改光照

点击创建项目时Unity自行创建的Light对象Directional Light：

![image-20240905142245289](E:/Typora/picture/image-20240905142245289.png)

点击**Emission>Color**，在新窗口中按如下设置光照颜色：

![image-20240905142425780](E:/Typora/picture/image-20240905142425780.png)

*Swatches用于取色，确认颜色设置后点击带边框的方块即可创建一个预设*

这样得到一个淡蓝的光照效果，以适配鬼屋的气氛。

然后更改**General>Intensity**属性为2，以增加亮度。

更改阴影效果：在**Shadow>Shadow Type**中更改其属性，由No->Hard->Soft，阴影效果越来越好。下面的Resolution同理。

最后修改**Transform>Rotation**中x=30,y=20,z=0。这样光照效果就符合我们的预期了。

更多的API详见：https://docs.unity.cn/cn/2020.3/ScriptReference/Light.html



#### 配置全局光照

打开**Window>Randering>Lighting**

初始时没有设置任何全局光照，点击New，将新建的Lighting Settings移动到Assets根目录下。取消**Mixed Lighting>Baked Global Illumination**属性。

切换到Environment设置，更改**Environment Lighting>Source**属性为Gradient。这个模式可以理解为一个半球笼罩整个场景，半球上不同高度发出的光也可以不同。

根据教程给出的设置来配置下面的三个属性：

Sky Color:(170, 180, 200)（提升水平面上的整体亮度）

Equator ~:(90, 110, 130)（提升垂直面上的亮度）

Ground ~:(0, 0, 0)（增加向上的光线，产生漂亮的全局光照效果，但是会使得整个场景过于明亮）

### 导航网格

[官方教程项目系列/官方教程06_3D/John Lemon's Haunted Jaunt/03.NavMesh导航网络实现简单AI.md · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/blob/main/官方教程项目系列/官方教程06_3D/John Lemon's Haunted Jaunt/03.NavMesh导航网络实现简单AI.md)

[导航和寻路 - Unity 手册](https://docs.unity.cn/cn/2020.3/Manual/Navigation.html)

* 首先将Level对象及其所有子对象设置为static，但是要把Level>Corridors>Dressing下的CeilingPlane(天花板组件)的static属性取消。

* 点击**Window>AI>Navigation(Obsolete)**（注意：该窗口已弃用，但是新的Navigation暂时不会使用（效果和该教程不符），所以仍沿用该教程）

* 切换到Bake分区，设置Agent Radius（半径是指AI物体横轴方向最大通过的路径）为0.25，然后点击Bake，新生成的蓝色区域（需打开Gizmos）即为新生成的导航网格。


导航网格的应用详见[设置导航网格代理](#设置导航网格代理)

## 摄像机

我们为什么不直接使用物理摄像机？因为想要使用物理摄像机进行跟随需要编写代码，而虚拟摄像机能很好的解决这个问题。（以下摘自官方教程）

### **Cinemachine 的工作原理**

Cinemachine 是 Unity 针对游戏中与摄像机有关的所有问题的解决方案。该系统简单总结如下：

- 在场景中创建一个或多个“虚拟”摄像机。
- 这些虚拟摄像机由一个名为 Cinemachine Brain 的组件进行管理。
- Cinemachine Brain 与 Camera 组件连接到相同的游戏对象，默认情况下，这个游戏对象将是 Main Camera 游戏对象。
- Cinemachine Brain 管理所有虚拟摄像机，并确定实际摄像机应跟随哪个虚拟摄像机（或虚拟摄像机的组合）。

在您的游戏中，摄像机只会跟随 JohnLemon，因此您只需要一个虚拟摄像机。您将确保此虚拟摄像机跟随 JohnLemon，然后 Main Camera 游戏对象会连接到这个虚拟摄像机。

### 添加并修改虚拟摄像机

* 点击控制角色，按一次F键以聚焦到该游戏对象，保持视角同时在Hierarchy中右键>Cinemachine>Virtual Camera创建新的虚拟摄像机。可以看到Game窗口中的视角变成了Scene的视角。

* 通过组件调整虚拟摄像机的属性：

  ![image-20240905164942177](E:/Typora/picture/image-20240905164942177.png)

  （再次引用官方教程的内容（很重要））

  ## **更改 Cinemachine Virtual Camera 组件设置**

  要设置虚拟摄像机以使其追踪 JohnLemon，请执行以下操作：

  **1.**在 Hierarchy 中，选择 **CM vcam1**。  

  **2.**在 Inspector 中，找到 Cinemachine Virtual Camera 组件。该组件有很多设置，但是现在您只需要关注三个部分：目标引用、Body 和 Aim。

  ![img](https://learn-public.cdn.u3d.cn/20220209/learn/images/0e53524c-46d0-4b30-8127-9f03a30630a1_image.jpeg)

  目标引用部分有两项设置：**Follow** 和 **Look At**。这些可选设置是对游戏对象的 Transform 组件的引用。

  如果要移动虚拟摄像机，虚拟摄像机需要知道*如何*移动。具体来说，虚拟摄像机需要引用要跟随的 Transform 组件。同样，如果想旋转虚拟摄像机，虚拟摄像机需要知道自己必须经过旋转才能观看的目标。

  下面两个部分是 **Body** 和 **Aim**。这些设置分别用于控制摄像机如何移动和旋转。您将要限制虚拟摄像机的移动，使摄像机的视野始终涵盖 JohnLemon 但实际上并没有观看该角色。   

  **3.**在 **Aim** 部分中，将右上角的下拉菜单从 **Composer** 更改为 **Do Nothing**。  

  **4.**将 JohnLemon 游戏对象从 Hierarchy 窗口拖放到 Cinemachine Virtual Camera 组件的 Follow 属性上。这样会将 **Follow** 设置更改为引用 JohnLemon 的 Transform 组件。

  **5.**在 **Body** 部分中，将该部分右上角的下拉菜单从 **Transposer** 更改为 **Framing Transposer。**将 Body 更改为 Framing Transposer 可以为虚拟摄像机提供关于其 **Follow** 目标必须在屏幕上所处位置的规则，从而控制虚拟摄像机的位置。  

  要了解有关不同 Body 设置值的更多信息，可查看 Cinemachine 文档（在顶部菜单中，选择 **Cinemachine > About**）。  

  现在，Game 窗口上应该有若干红色和蓝色框。这些是目标可以在屏幕上出现的位置的参考线框：

  ![img](https://learn-public.cdn.u3d.cn/20220209/learn/images/45bec1f5-afa7-4041-af09-ae95eae29064_image.jpeg)

  **6.**现在，让我们将虚拟摄像机设置为正确的角度。在 Hierarchy 中，选择 **CM vam1** 游戏对象。在 CM vam1 游戏对象的 Transform 组件上，将围绕 x 轴的 **Rotation** 设置为 **45**（45°是一个非常适合追尾视角的角度）。  

  现在，虚拟摄像机向下倾斜并正在从上往下观看角色。这大致就是您需要的视角。当您仅更改旋转时，为什么虚拟摄像机会移动？这就是 Cinemachine 的功能！  

  您知道自己需要更大的相对于摄像机的自顶向下角度，然后虚拟摄像机推断出：如果您希望将摄像机向下倾斜，则需要将摄像机升高才能将其目标显示在屏幕上。

  ![img](https://learn-public.cdn.u3d.cn/20220209/learn/images/446565b0-d7cf-4bf8-bd28-ccada826ac77_image.png)

  **7.**Framing Transposer 的大多数默认设置都适合您的游戏。唯一需要更改的是 **Camera Distance** 设置；角色在屏幕上有点太小，因此您需要将虚拟摄像机移近一点。  

  将 **Camera Distance** 设置从 **10** 更改为 **8**。

  ![img](https://learn-public.cdn.u3d.cn/20220209/learn/images/de273123-2eae-4a3b-a500-ff5e1ed9fd35_image.jpeg)

  **8.**在 Hierarchy 中，选择 **CM vcam1**，然后将其重命名为 **VirtualCamera**。  

  **9.**取消选择 VirtualCamera 游戏对象以从 Game 窗口中删除参考线。  

  **10.**从顶部菜单中选择 **File > Save Scene** 或按 Ctrl + S (Windows) 或 CMD + S (macOS)，保存该场景。  

  **11.**单击工具栏中的 Play 按钮并测试到目前为止完成的游戏。完成测试后，再次单击 Play。  

  就是这样 — 您已经设置了虚拟摄像机及其移动！现在，您可以开始为摄像机添加一些视觉效果，使游戏看起来更加出色。

### 添加后期处理效果

[官方教程项目系列/官方教程06_3D/John Lemon's Haunted Jaunt/05.后期处理.md · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/blob/main/官方教程项目系列/官方教程06_3D/John Lemon's Haunted Jaunt/05.后期处理.md)

==注意：现在这个项目的原Assets已使用URP，并非内置渲染管线，即使在构建项目时选择内置渲染管线，Import 项目Assets之后Project Settings>Grapics内的相关设置仍然会更改为URP。而且当前最新版本URP已不再支持Post Process包，因此教程中的后期处理效果没有生效。因此选择直接使用URP内置Post Processing来进行后期处理。==

==首先请不要将Main Camera和Volume设置到专门的一个图层，这样渲染同样会失效。==

想知道更详细的设置，建议根据效果名自行查阅：[Universal Render Pipeline overview | Universal RP | 14.0.11 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/index.html)

#### 添加抗锯齿

点击Main Camera，勾选Post Processing，更改其下的Anti-aliasing为其他任意一个值。

![image-20240906135608089](E:/Typora/picture/image-20240906135608089.png)

可以在Game界面查看应用抗锯齿前后的效果：

![image-20240906135929774](E:/Typora/picture/image-20240906135929774.png)

![image-20240906135945219](E:/Typora/picture/image-20240906135945219.png)

#### 全局后期处理

如果你的项目构建时使用的是URP，那么在Unity Editor中默认会创建一个Global Volume对象：

![image-20240906134837404](E:/Typora/picture/image-20240906134837404.png)

如果没有该对象也不要紧，Hierarchy中右键添加即可：

![image-20240906134918177](E:/Typora/picture/image-20240906134918177.png)

##### 颜色分级Color Grading

更改色调：

* 在Global Volume中点击Add Override>Post Processing>Tonemapping(色调映射)

  在新建的Tonemapping模块中勾选Mode并点击下拉菜单选择

  1. **None**：不应用任何色调映射。
  2. **Neutral**：应用中性色调映射，旨在保持图像的亮度和颜色平衡。
  3. **ACES**：应用学院色彩编码系统（Academy Color Encoding System）的色调映射，这是一种广泛使用的行业标准。
  4. **Custom**：允许开发者使用自定义的色调映射曲线。（此处没有该选项）

* 原教程中是更改曝光度Post-Exposure(EV)，然而在URP自带的Volume下我没有找到关于曝光度的相关设置，只能使用Color Curves模块进行调整。

* 想要达成视频中展示的光亮效果，可以按如下设置：

  ![image-20240906143344783](E:/Typora/picture/image-20240906143344783.png)

  ![image-20240906151437165](E:/Typora/picture/image-20240906151437165.png)

  Color Curves的作用就是更改图像的色调、亮度和对比度，曲线的意义是：横轴x为通道输入，纵轴y为通道输出，创建一个映射函数y=f(x)。

  效果大致如下：

  ![image-20240906143930443](E:/Typora/picture/image-20240906143930443.png)

然后是增加阴影深度：

* 添加Lift Gamma Gain模块，设置大致如下：

  ![image-20240906151103809](E:/Typora/picture/image-20240906151103809.png)

##### 泛光Bloom

* 添加Bloom模块（默认已添加），剩余操作跟视频是一样的。
* 点击ALL使得所有属性均生效。更改Intensity（强度）属性值为1，可以明显看出画面中的所有灯光变得更大更亮且模糊（即泛出光晕）。
* 更改Threshold（阈值）为0.75，意义是强于多少的光亮能够显示光晕。
* 发现光晕有些太过强烈了，所以又更改Intensity为0.25

##### 环境光遮罩

在**Edit>Project Setting>Graphics**中你能找到当前URP使用的渲染器资源：

![image-20240907083102257](E:/Typora/picture/image-20240907083102257.png)

点开资源文件，找到使用的渲染器：

![image-20240907083205367](E:/Typora/picture/image-20240907083205367.png)

点开渲染器文件，点击**Add Renderer Feature>Screen Space Ambient Occlusion**

![image-20240907083316959](E:/Typora/picture/image-20240907083316959.png)

略微更改一下配置：

![image-20240907083446549](E:/Typora/picture/image-20240907083446549.png)

可以看到Game中阴影明显更多且加深。

##### 渐晕Vignette

渐晕的效果就是是镜边缘变暗，让玩家更觉得幽闭恐怖。

* 在Global Volume中添加Vignette模块，点击All使全部生效。将Intensity更改为0.5，Smoothness更改为0.3。即可查看效果。

  ![image-20240907084453328](E:/Typora/picture/image-20240907084453328.png)

##### Lens Distortion镜头失真

* 在Global Volume中添加Lens Distortion模块，勾选ALL，更改Intensity（控制凸变/凹变程度）为35，Scaler（控制画面缩放，搭配Intensity更改）设为1.1：

  ![image-20240907084856379](E:/Typora/picture/image-20240907084856379.png)

## 结束游戏

本节的目标是：编写脚本以控制游戏结束触发以及游戏结束UI界面的设计。

### 制作UI

* 在Hierarchy界面右键>UI>Canvas，新建Canvas对象的同时还会生成一个EventSystem对象，该对象用于控制UI在不同条件下的显示。

* Canvas中的Rect Transfrom组件用于高效的平面定位（当Canvas填充整个屏幕的时候不需要），Canvas组件用于设置模式，Canvas Scaler用于缩放（当Canvas填充整个屏幕的时候不需要），最后Graphic Reycaster和用户交互相关。为提升游戏性能，我们将不需要的组件全部删掉，只保留一个Canvas组件和一个不能删除的Rect Transfrom组件。

* 然后是添加Image。首先制作一个全黑背景，在Canvas对象下新建Image命名为WinBackground，设置Anchor四角分布在Canvas的四个顶点（鼠标拖拽亦可）：

  ![image-20240907091848759](E:/Typora/picture/image-20240907091848759.png)

  然后使Image填充整个屏幕（除了之前使用Anchor Presets以外也可更改Scale属性为（1,1,1））

  在Background下再Create一个Image并选择Source Image为Won的图片。然后需要控制宽高比不变放大到全屏，只需勾选Preserve Aspect属性然后重复上面的操作即可：

  ![image-20240907092550946](E:/Typora/picture/image-20240907092550946.png)

  

### 控制胜利界面显示

* 首先需要调整Image为常时透明的状态。在WinBackground中添加一个Canvas Group组件。Canvas Group 组件是一个用于控制一组 UI 元素的组件，它可以同时改变多个 UI 元素的某些属性。这个组件主要用于优化性能和创建统一的 UI 交互效果。

  在Canvas Group中设置Alpha为0，即可隐藏Background及其子对象。

* 然后我们需要实现当角色走到终点的时候显示胜利画面。在Hierarchy中创建一个空的GameEnding对象，添加Box Collider组件，勾选Is Trigger并调整其位置和大小：

  ![image-20240907102448775](E:/Typora/picture/image-20240907102448775.png)

  * 新建一个脚本GameEnding：

    ```C#
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    
    public class GameEnding : MonoBehaviour
    {
        bool _isPlayerAtExit;
        // 在Unity编辑器中将玩家对象拖拽到这个字段中
        public GameObject player;
        // 更改透明度的时间
        public float fadeDuration = 1.0f;
        // 计时器
        float _Timer;
        // 正常（完全不透明状态）显示结束UI的时间
        public float displayImageDuration = 1.0f;
        // 声明一个Canvas Group组件用来更改UI的透明度
        public CanvasGroup exitBackgroundImageCanvasGroup;
        // Start is called before the first frame update
        void Start()
        {
            
        }
    
        // Update is called once per frame
        void Update()
        {
            if(_isPlayerAtExit)
            {
                EndLevel();
            }
        }
    
        private void OnTriggerEnter(Collider other)
        {
            if(other.gameObject == player)
            {
                _isPlayerAtExit = true;
            }
        }
    
        void EndLevel()
        {
            _Timer += Time.deltaTime;
            // 逐渐更改UI的透明度，alpha>=1时，UI完全显示
            exitBackgroundImageCanvasGroup.alpha = _Timer / fadeDuration;
            // 当时长大于设定的渐变时间加上显示时间时，退出游戏
            if (_Timer > fadeDuration + displayImageDuration)
            {
                // 当前只有一个关卡，所以直接退出游戏
                // 这个方法只有打包发布时才能生效
                // Application.Quit();
                // 想要在Unity编辑器中退出，可以使用下面这个方法
                UnityEditor.EditorApplication.isPlaying = false;
            }
        }
    }
    
    ```

    将该脚本挂载到GameEnding对象上并拖入角色对象作为player的引用值，拖入WinBackground作为exitBackgroundImageCanvasGroup的引用值。

    *注意打包发布的时候想退出游戏得改回*`application.Quit()`;

### 添加失败界面

点击WinBackground，Ctrl+D复制一个新对象，并重命名为CaughtBackground和CaughtImage，将Source Image更改为Caught图像。

在GameEnding脚本中修改：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameEnding : MonoBehaviour
{
    bool _isPlayerAtExit;
    // 在Unity编辑器中将玩家对象拖拽到这个字段中
    public GameObject player;
    // 更改透明度的时间
    public float fadeDuration = 1.0f;
    // 计时器
    float _Timer;
    // 正常（完全不透明状态）显示结束UI的时间
    public float displayImageDuration = 1.0f;

    public float displayWaitTime = 1.0f;
    // 声明一个Canvas Group组件用来更改UI的透明度
    public CanvasGroup exitBackgroundImageCanvasGroup;

    // 新增一个表示游戏失败的结束界面UI
    public CanvasGroup caughtBackgroundImageCanvasGroup;
    // 用一个bool类型的变量，用于判断玩家是否被抓住
    bool _isPlayerCaught;
    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        if(_isPlayerAtExit)
        {
            EndLevel(exitBackgroundImageCanvasGroup, false);
        }

        else if(_isPlayerCaught)
        {
            EndLevel(caughtBackgroundImageCanvasGroup, true);
        }
    }

    private void OnTriggerEnter(Collider other)
    {
        if(other.gameObject == player)
        {
            _isPlayerAtExit = true;
        }
    }

    void EndLevel(CanvasGroup imageCanvasGroup, bool doRestart)
    {
        _Timer += Time.deltaTime;
        // 逐渐更改UI的透明度，alpha>=1时，UI完全显示
        imageCanvasGroup.alpha = (_Timer - displayWaitTime) / fadeDuration;
        // 当时长大于设定的渐变时间加上显示时间时，退出游戏
        if (_Timer > fadeDuration + displayImageDuration + displayWaitTime)
        {
            if (doRestart)
            {
                //如果失败，重新加载场景（需要添加新的using引用）
                SceneManager.LoadScene(0);
            }
            // 必须要有else，否则会在重新加载场景后继续执行关闭游戏
            else
            {
                // 当前只有一个关卡，所以直接退出游戏
                // 这个方法只有打包发布时才能生效
                // Application.Quit();
                // 想要在Unity编辑器中退出，可以使用下面这个方法
                UnityEditor.EditorApplication.isPlaying = false;
            }
        }
    }

    public void CaughtPlayer()
    {
        _isPlayerCaught = true;
    }
}

```

[至于编辑敌人以触发失败界面详见](#敌人)

### 编码控制Image加载

添加多个Image并使用bool变量+判断分支以实现不同条件触发不同的UI显示，这个方法固然不错，但是如果考虑到会有N多个条件分支对应N多个Image，无疑对于资源过于浪费且操作麻烦。我们希望能够通过代码来控制同一个（组）Image对象能够挂载不同的Source Image。

首先删除Canvas对象下已经创建的一组Background+Image，然后在Assets下确认有一个Resources文件夹（只有Resources下面的文件才能使用` Resources.Load<>()`）。

修改GameEnding脚本代码如下：

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.UI;

public class GameEnding : MonoBehaviour
{
    bool _isPlayerAtExit;
    // 在Unity编辑器中将玩家对象拖拽到这个字段中
    public GameObject player;
    // 更改透明度的时间
    public float fadeDuration = 1.0f;
    // 计时器
    float _Timer;
    // 正常（完全不透明状态）显示结束UI的时间
    public float displayImageDuration = 1.0f;

    public float displayWaitTime = 1.0f;
    // 声明一个Canvas Group组件用来更改UI的透明度
    public CanvasGroup imageCanvasGroup;
    // 用一个bool类型的变量，用于判断玩家是否被抓住
    bool _isPlayerCaught;
    // 从Unity Editor挂载Image对象
    public Image image;
    // 创建两个对象，分别表示从目录中获取的两个图片素材（精灵）
    Sprite spriteCaught;
    Sprite spriteWon;
    // Start is called before the first frame update
    void Start()
    {
        // 预先从Resources文件夹中加载两个图片素材
        spriteCaught = Resources.Load<Sprite>("Caught");
        spriteWon = Resources.Load<Sprite>("Won");
    }

    // Update is called once per frame
    void Update()
    {
        if(_isPlayerAtExit)
        {
            EndLevel(spriteWon, false);
        }

        else if(_isPlayerCaught)
        {
            EndLevel(spriteCaught, true);
        }
    }

    private void OnTriggerEnter(Collider other)
    {
        if(other.gameObject == player)
        {
            _isPlayerAtExit = true;
        }
    }

    void EndLevel(Sprite sprite, bool doRestart)
    {
        _Timer += Time.deltaTime;
        // 指定结束UI Image的精灵
        image.sprite = sprite;
        // 逐渐更改UI的透明度，alpha>=1时，UI完全显示
        imageCanvasGroup.alpha = (_Timer - displayWaitTime) / fadeDuration;
        // 当时长大于设定的渐变时间加上显示时间时，退出游戏
        if (_Timer > fadeDuration + displayImageDuration + displayWaitTime)
        {
            if (doRestart)
            {
                //如果失败，重新加载场景（需要添加新的using引用）
                SceneManager.LoadScene(0);
            }
            // 必须要有else，否则会在重新加载场景后继续执行关闭游戏
            else
            {
                // 当前只有一个关卡，所以直接退出游戏
                // 这个方法只有打包发布时才能生效
                // Application.Quit();
                // 想要在Unity编辑器中退出，可以使用下面这个方法
                UnityEditor.EditorApplication.isPlaying = false;
            }
        }
    }

    public void CaughtPlayer()
    {
        _isPlayerCaught = true;
    }
}

```

保存后检查所有该挂载的属性是否挂载正确：

![image-20240907201722460](E:/Typora/picture/image-20240907201722460.png)

## 敌人

我们制作的是一款潜行游戏，只要被敌人发现就游戏失败。

### 添加静态敌人

* 将**Assets/UnityTechnologies/Models/Characters/Gargoyle.fbx**拖入Hierarchy，在**Assets/UnityTechnologies/Animation/Animators**中新建一个Gargoyle的Animation Controller。因为保持位置静止，所以只需要添加一个动画，对应在Assets/UnityTechnologies/Animation/Animation。

* 添加Capsule Collider：

  ![image-20240907113352358](E:/Typora/picture/image-20240907113352358.png)

* 制成预制件，在预制件中在Gargoyle下创建一个空的对象PointOfView用来模拟视线，按如下规则更改其Transfrom：

  ![image-20240907114544699](E:/Typora/picture/image-20240907114544699.png)

  

* 使用<img src="E:/Typora/picture/image-20240907114857651.png" alt="image-20240907114857651" style="zoom:50%;" />查看对象的可视化Transfrom（注意把Scene中的<img src="E:/Typora/picture/image-20240907114943748.png" alt="image-20240907114943748" style="zoom:60%;" />设为Local）：

  * **Global**：当选择Global时，旋转手柄会以世界坐标系的三个轴（X、Y、Z轴）作为参考轴。这意味着无论你的游戏对象如何旋转，旋转手柄始终保持与世界坐标轴对齐，这有助于进行精确的全局旋转控制。

  - **Local**：当选择Local时，旋转手柄会以游戏对象本身的三个坐标轴（局部轴）作为参考。这意味着手柄的旋转会随着游戏对象的旋转状态而变化，这有助于进行相对于对象自身的旋转操作。

  ![image-20240907115126997](E:/Typora/picture/image-20240907115126997.png)

* 为PointOfView添加Capsule Collider，用来监测是否发现玩家。

  ![image-20240907115920662](E:/Typora/picture/image-20240907115920662.png)

  ![image-20240907115938247](E:/Typora/picture/image-20240907115938247.png)

* 为PointOfView添加Observer脚本以监测是否发现玩家：

  ```C#
  public class Observer: MonoBehaviour
  {
      // 获取玩家对象的Tansform组件对象，用于挂接玩家
      public Transform player;
      // 声明一个bool类型的变量，用于判断玩家是否在视野内
      bool _isPlayerInSight;
      // 声明游戏结束脚本组件类对象，用于调用游戏结束方法
      public GameEnding gameEnding;
      private void OnTriggerEnter(Collider other)
      {
          // 判断触发器内的对象是否为玩家
          if (other.transform == player)
          {
              _isPlayerInSight = true;
          }
      }
      private void OnTriggerExit(Collider other)
      {
          // 判断触发器内的对象是否为玩家
          if (other.transform == player)
          {
              _isPlayerInSight = false;
          }
      }
  
      void Update()
      {
          // 判断玩家是否在视野内
          if (_isPlayerInSight)
          {
              // 获取玩家与观察者之间的方向
              Vector3 direction = player.position - transform.position + Vector3.up;
              // 创建一个射线，用于检测玩家与观察者之间是否有障碍物
              Ray ray = new Ray(transform.position, direction);
              RaycastHit hit;
              // 判断射线是否与障碍物相交
              if (Physics.Raycast(ray, out hit))// hit作为out参数，用于存储射线与障碍物相交的信息
              {
                  // 判断相交的对象是否为玩家
                  if (hit.transform == player)
                  {
                      // 调用游戏结束方法
                      gameEnding.CaughtPlayer();
                  }
              }
          }
      }
  }
  ```

  这个代码的逻辑是：Trigger做触发，Update里做检测，触发后只有当投射视线首先碰撞到玩家时才能判定为发现玩家（考虑玩家和Trigger碰撞体之间存在其他碰撞体的情况）。

  至于被敌人发现后触发游戏失败详见[结束游戏](#添加失败界面)

### 添加动态敌人

* 添加对象>添加Animation Controller>引入walk Animation>Animator组件中挂载Controller>制作预制件>添加碰撞体和触发器（略）

* 我们希望触发碰撞后幽灵不会被弹开，所以需要添加刚体，勾选Is Kinematic。

* 为幽灵添加视线，可以按照制作石像鬼时一步步添加和调整，也可以将Gargoyle**预制件**下的PointOfView对象直接制成预制件再修改复用。

* 现在幽灵尚不能移动，需要设置Nav Mesh Agent组件：

  在Ghost预制件下，添加Nav Mesh Agent组件。打开Gizmos看到Nav Mesh中定义的的体积远大于Ghost自己的大小：

  ![image-20240907211724908](E:/Typora/picture/image-20240907211724908.png)

  将radius改为0.2，height改为1.2。

### 设置导航网格代理

* 接下来添加一个新的脚本组件WaypointPatrol（路径点巡逻）：

  ```C#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  using UnityEngine.AI;
  
  public class WaypointPatrol : MonoBehaviour
  {
      // 设置NavMeshAgent组件对象，用于控制导航网格代理AI的移动
      NavMeshAgent _navMeshAgent;
      // 设置路径点数组，主要是利用其position
      public Transform[] wayPoints;
      // 设置当前路径点索引数，从0到wayPoints.Length-1
      int _currentWaypointIndex;
      // Start is called before the first frame update
      void Start()
      {
          // 获取组件对象
          _navMeshAgent = GetComponent<NavMeshAgent>();
          // 设置导航组件 导航路径的起始位置
          _navMeshAgent.SetDestination(wayPoints[0].position);
      }
  
      // Update is called once per frame
      // 每次刷新都尝试获取下一个路径点
      // 如果满足要求，指定下一个路径点
      // 并通过（除余）算法，让路径点位循环往复
      void Update()
      {
          // 当前游戏对象到指定的路径点的距离 如果小于等于导航组件的停止距离(Stopping Distance)视为已经到达目标路径点
          if (_navMeshAgent.remainingDistance < _navMeshAgent.stoppingDistance)
          {
              // 获取下一个路径点
              _currentWaypointIndex = (_currentWaypointIndex + 1) % wayPoints.Length;
              // 设置新的导航位置
              _navMeshAgent.SetDestination(wayPoints[_currentWaypointIndex].position);
          }
  
      }
  }
  
  ```

  注意：Nav Mesh Agent组件中的Stopping Distance属性默认为0，这是不合适的，建议更改为一个较小的浮点数：

  ![image-20240907215231313](E:/Typora/picture/image-20240907215231313.png)

* 然后，我们需要再Hierarchy中添加路径点对象（Empty Object）并将其设置在路径上合适的位置：

  ![image-20240907220510338](E:/Typora/picture/image-20240907220510338.png)

  点击Ghost，将新脚本添加进来，将路径点对象拖入Way Points下，运行游戏即可查看幽灵是否在指定路径上往返移动。

## 整理Hierarchy结构

* 在多添加几个敌人之后，发现Hierarchy内容爆满，有没有一种方法能够像文件系统一样整理Hierarchy结构吗？

  实际是有的，我们创建一个空对象，将其视为"文件夹"，然后将所有应当归在这个"文件夹"下的对象拉进来。

  注意：我们在拉对象之前，应该先将Empty Object的position设为(0,0,0)*（或者直接reset）*，因为对象拖进来作为Empty Object的子对象后，虽然其世界位置不变，但是其Transform.position变成了相对Empty Object的坐标，这样以后再更改其位置时就不方便输入调整了。

## 音频/声效

### 添加非剧情声音(环境音)

* 添加空对象，再添加Audio Source组件（或者直接将Audio Clip拖入Inspector），调整Loop和Volume。（略）

* 对于背景音乐，不需要控制播放。而对于游戏胜利和失败的音乐，首先先勾掉Play On Awake属性，然后再脚本中添加控制播放的代码。
  ```C#
  public class GameEnding : MonoBehaviour
  {
      bool _isPlayerAtExit;
      // 在Unity编辑器中将玩家对象拖拽到这个字段中
      public GameObject player;
      // 更改透明度的时间
      public float fadeDuration = 1.0f;
      // 计时器
      float _Timer;
      // 正常（完全不透明状态）显示结束UI的时间
      public float displayImageDuration = 1.0f;
  
      public float displayWaitTime = 1.0f;
      // 声明一个Canvas Group组件用来更改UI的透明度
      public CanvasGroup imageCanvasGroup;
      // 用一个bool类型的变量，用于判断玩家是否被抓住
      bool _isPlayerCaught;
      // 从Unity Editor挂载Image对象
      public Image image;
      // 创建两个对象，分别表示从目录中获取的两个图片素材（精灵）
      Sprite spriteCaught;
      Sprite spriteWon;
  	// 添加并挂载音频源
      public AudioSource winAudio;
      public AudioSource caughtAudio;
      //因为这两种声音只会播放一次，所以需要用一个bool变量来判断是否已经播放过
      bool _hasAudioPlayed;
      // Start is called before the first frame update
      void Start()
      {
          // 预先从Resources文件夹中加载两个图片素材
          spriteCaught = Resources.Load<Sprite>("Caught");
          spriteWon = Resources.Load<Sprite>("Won");
      }
  
      // Update is called once per frame
      void Update()
      {
          if(_isPlayerAtExit)
          {
              EndLevel(spriteWon, false, winAudio);
          }
  
          else if(_isPlayerCaught)
          {
              EndLevel(spriteCaught, true, caughtAudio);
          }
      }
  
      private void OnTriggerEnter(Collider other)
      {
          if(other.gameObject == player)
          {
              _isPlayerAtExit = true;
          }
      }
  
      void EndLevel(Sprite sprite, bool doRestart, AudioSource audioSource)
      {
          // 如果声音没有播放过，播放音频源
          if (!_hasAudioPlayed)
          {
              audioSource.Play();
              _hasAudioPlayed = true;
          }
          _Timer += Time.deltaTime;
          // 指定结束UI Image的精灵
          image.sprite = sprite;
          // 逐渐更改UI的透明度，alpha>=1时，UI完全显示
          imageCanvasGroup.alpha = (_Timer - displayWaitTime) / fadeDuration;
          // 当时长大于设定的渐变时间加上显示时间时，退出游戏
          if (_Timer > fadeDuration + displayImageDuration + displayWaitTime)
          {
              if (doRestart)
              {
                  //如果失败，重新加载场景（需要添加新的using引用）
                  SceneManager.LoadScene(0);
              }
              // 必须要有else，否则会在重新加载场景后继续执行关闭游戏
              else
              {
                  // 当前只有一个关卡，所以直接退出游戏
                  // 这个方法只有打包发布时才能生效
                  // Application.Quit();
                  // 想要在Unity编辑器中退出，可以使用下面这个方法
                  UnityEditor.EditorApplication.isPlaying = false;
              }
          }
      }
  
      public void CaughtPlayer()
      {
          _isPlayerCaught = true;
      }
  }
  ```

### 添加脚步声

因为脚步声不属于环境音，所以直接在角色下添加Audio Source组件，将对应的Audio Clip拖入，勾掉Play On Awake，并勾选Loop。然后修改角色控制脚本：

```C#
AudioSource _audioSource;

// Start
 _audioSource = GetComponent<AudioSource>();

//FixedUpdate末尾
if (isWalking)
{
    // 保证每次都不是从头播放
    if(!_audioSource.isPlaying)
    {
        _audioSource.Play();
    }
}
else // 如果停止运动，则停止播放音频
{
    _audioSource.Stop();
}
```

### 在角色身上添加音频监听器

默认的Listener是在Main Camera上的，为了使效果更好，应该设置在角色身上。

先删除Main Camera上的Listener组件，然后在角色上挂载一个。

然后为幽灵预制件挂载音频。勾选Loop，调整Special Blend为3D,调整距离曲线：

![image-20240908103639437](E:/Typora/picture/image-20240908103639437.png)

* 运行游戏后虽然幽灵的声音的确有方向感了，但是在我们控制角色移动时会发现有时声音的方向和我们所看到的幽灵的位置其实并不吻合。这是因为当我们把音频监听器放置在角色身上时，音频监听器的方向会随着角色的转动而转动。形象一点，我们听到的声音方向是角色的耳朵听到的方向，而我们眼睛方向是随着摄像机而锁死的（耳朵和眼睛明显 不在同一个头上）。

* 按照教程，将spread直线设在y（spread）=0.5上。

  ![image-20240908104920966](E:/Typora/picture/image-20240908104920966.png)

  然而我听出来的效果是：完全模糊了方向，只能分辨远近了。

* GPT是这么解释这个Spread曲线的：

  在Unity中，`AudioSource`组件的`Spread`属性用于设置3D立体声或多声道声音在扬声器空间中的扩散角度。这个属性以度为单位，可以调整声音在立体声场中的分布范围。

  - 当`Spread`设置为`0`度时，所有声道都位于同一个扬声器位置，声音表现为“单声道”。
  - 当`Spread`设置为`360`度时，所有子通道都位于与根据3D位置确定的扬声器位置相对的扬声器位置，这意味着声音会根据其在3D空间中的位置而分布在所有扬声器上，从而实现全范围的立体声效果。

* 又尝试了改为360，发现好像辨位和实际位置又完全相反了。我认为教程给的这种解决方法实乃下下策。
* 就上面这种情况，个人认为是游戏本身设计的缺陷。如果是玩TPP（第三人称视角），那么Listener就应该是在摄像头上，随摄像头的转动而转动而非控制角色的转动。

# 3D美术

源项目：https://github.com/amel-unity/ProBuilderPolyBrushDemo

下载zip解压后在Unity Hub中打开文件夹，使用现有版本打开，警告点Continue即可。

[3D游戏/3DLevelDesign · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/tree/main/3D游戏/3DLevelDesign)

## 3D游戏环境设计流程

### 准备工作

根据教程文档确认所有的包都安装好了之后，点击Tools>ProGrids>ProGrids Window

![image-20240908151500607](E:/Typora/picture/image-20240908151500607.png)

然后Scene中会出现这样一个侧边栏：

![image-20240908151643726](E:/Typora/picture/image-20240908151643726.png)

该侧边栏出现后，原本的网格系统[(Position GameObjects - Unity 手册)](https://docs.unity.cn/cn/2022.3/Manual/PositioningGameObjects.html)就不使用了。

点击Tools>Probuilder>ProBuilder Window以及Tools>PolyBrush>PolyBrush Window

### 导入蓝图

新建一个Scene，在Hierarchy中创建一个空的对象Level并重置其Transfrom。

在PodBuilder窗口点击New Shape，在Scene的右下角出现一个Create Shape悬浮窗口。

![image-20240908161448077](E:/Typora/picture/image-20240908161448077.png)

点击Plane（平面）图标，设置Size X=70，Z=40，Height Cut=0，Width Cut=0，切换至Y平面网格模式，按住shift+左键创建对象

![image-20240908163856778](E:/Typora/picture/image-20240908163856778.png)

*如果发现图形是透明的，说明你的视角在图形的反面*

在Materials文件夹下新建一个Material对象命名为HousePlan。然后在Assets/ProBuilderPolyBrushDemo/Textures文件夹下找到LevelPlan材质

![image-20240908164350853](E:/Typora/picture/image-20240908164350853.png)

将LevelPlan拖入HousePlan的Base Map属性前的方格中：

![image-20240908164656799](E:/Typora/picture/image-20240908164656799.png)

再将HousePlan拖入Plane对象的Mesh Renderer组件下的Material属性下，但是Plane并不显示图案。

![image-20240908164804059](E:/Typora/picture/image-20240908164804059.png)

点击Probuilder下的UV Editor，选中Plane对象，点击Scene上方的第四个按钮（对面编辑）<img src="E:/Typora/picture/image-20240908165200694.png" alt="image-20240908165200694" style="zoom:33%;" />，整个Plane被标黄（如果只被标记了部分，说明Plane的Height Cuts或Width Cuts不为0）。

在UV Editor窗口中更改Fill Mode为Stretch（拉伸）：

![image-20240908165433852](E:/Typora/picture/image-20240908165433852.png)

再将Anchor设为Upper Left（实际这里只要不设为None都可以），使平面和材质重合。关闭UV Editor，看到蓝图已经设计完成。

为了防止误操作使得视角改变，我们在可视化轴上锁定当前视角，即可对蓝图平面进行自由编辑。然后将蓝图plane放入Level对象下，设置其为不可选中。

### 添加墙体

墙体的建模方式有很多种，但各有各的优缺点，这里就不再一一列举。

下面这种方法添加墙体的基本原理是：在蓝图上用Plane描绘轮廓构成平面，在将平面拉伸成墙体。

#### 挤压边

在ProGrid中将格宽设为0.05，snap打开。放大蓝图，用ProBuilder绘制一条边的Plane，在利用Scene顶上的功能缩放和微调。

![image-20240908171320940](E:/Typora/picture/image-20240908171320940.png)

虽然可以照此方法一步步描绘蓝图上的所有边，但是显然麻烦，且还有一个更大的问题是——可能这样的墙体并不严丝合缝。为了达成这个目标，我们需要利用ProBuilder的一个功能——挤出边。

这个概念可能略显抽象，但是实际操作一下就能清楚地理解这个功能了：

使用Scene顶部栏的第三个功能选中已经建好的这堵墙Plane连接其他墙的一条边。

![image-20240908213757192](E:/Typora/picture/image-20240908213757192.png)

在ProBuilder中找到Extrude Edges功能，

![image-20240908214834003](E:/Typora/picture/image-20240908214834003.png)

点击右边的"+"，弹出一个配置窗口：

![image-20240908213946613](E:/Typora/picture/image-20240908213946613.png)

*As Group：将挤出的新的Plane与当前的Plane合并到一起；挤压为组决定在挤压时相邻的面是否彼此保持连接。*

*Distance：需要挤出的长度*

这里我们不需要合并，且难以确定需要的长度，所以将AS Group勾掉并将Distance设为0后，直接右上角关闭窗口。

现在我们再选中这条边，点击Extrude Edges，点住红色的X轴并向左拉，可以看到Plane延伸了出来并且中间出现了一条分割线：

![image-20240908214530504](E:/Typora/picture/image-20240908214530504.png)

现在理解什么是"挤出边"了吧。

注意：

1. 拉出来后中间没有分割线或者不符合预期效果，可能是你没有点击Extrude Edges。

			2. 点击Extrude Edges即自行挤压，说明没有更改Settings。
			2. Snap设置感觉还是很有必要的，因为描绘一些长边的时候不得不缩小画面，到时不好微调。
			2. 边的连接点一定要单独挤压一次，这样拐角的时候才能选到等于宽度的一条边。
			2. 一次拉不完一条边可以继续拖，只要没有点击Extrude Edges，视为仍然在挤出一条边。

现在，你可以自行描绘完蓝图中一个房间全部的边了。

![image-20240908221342512](E:/Typora/picture/image-20240908221342512.png)

可以解除视角轴锁定以查看所有的边是否在同一个面。

#### 挤压面

然后，我们需要由面到体，在远点透视模式下调整好角度后切换到平行透视模式并锁定，点击顶部栏的第四个功能，鼠标左键划出能概括整个房间面的区域以全部选中。

![image-20240908221832305](E:/Typora/picture/image-20240908221832305.png)

在ProBuilder中找到Extrude Faces，点击"+"，在弹窗中设置Distance为1.2并点击下面的按钮：

![image-20240908222021294](E:/Typora/picture/image-20240908222021294.png)

设置墙体高度为1.2的原因是我们还需要构造出窗体等额外的嵌入结构。最好的处理方法是先将墙体升到窗户的最低高度后再Extrude1.5，之后只对上半墙体进行编辑。

#### 环切

我们需要在后面的墙上开洞，得先划出窗体。

环切的效果是：选中一条边后，垂直于该边的面（默认交于中点）切割整个物体并留下一个环形的分割线。

使用顶部栏选择边，点击墙体内侧顶部的边，在ProBuilder中点击Insert Edge Loop，即可形成环切线。移动整个环切线（如果是重新选择，需要点击环的一条边后在ProBuilder中点击Select Edge Loop即可选择整个环）至合适的位置，再重新环切一次：

![image-20240908223648858](E:/Typora/picture/image-20240908223648858.png)

再横向切割一次：

![image-20240908224230403](E:/Typora/picture/image-20240908224230403.png)

#### 开出空洞

切割好之后，选择中间的一块墙体，按下Backspace键将其删除，此时正面看墙体已经删除。

然而从背面看发现墙体似乎依然存在，这是因为我们删除的只是一个面，在Unity中，面是单向显示的，定义一个面只有从其正面才能看见。将背面的面删除后，窗体空洞就形成了。

#### 补面

近距离查看空洞，发现挖掉部分周围并没有面：

![image-20240908224724664](E:/Typora/picture/image-20240908224724664.png)

选择一个空面的两条边（shift多选），点击ProBuilder下的Bridge Edges，即可形成新的面。重复操作以补全墙面。

然后还需要将地面给补全，先补上门槛处的面，然后重复补面操作即可。

*这一步可能会出现nonmanifoid的报错，在preferences->probuilder中设置一下允许非流形（即没有厚度的墙）就可以啦~*

这样一个完整的物理墙体就建好啦。

**弹幕评论：**

**建模顺序不太对，所以要修修补补，3D高手就不会这么弄了，不过这种修补以后总会用到的。**

**当个扩展知识把，建模还是用专业建模软件吧。**

个人体验下来确实，这种建模一般只用于白盒设计和低模设计，如果真要做细致的场景还是用专业的建模工具好了，而且Unity插件加多了过后会引起卡顿。

### 添加贴图

选中一个面，在ProBuilder中打开Material Editor：

![image-20240908231938008](E:/Typora/picture/image-20240908231938008.png)

在Alt+2一行添加Floor材质，然后按下Alt+2（或点击Alt+2按钮），即可将材质应用到选中的平面上。

![image-20240908232054050](E:/Typora/picture/image-20240908232054050.png)

添上材质后发现贴图又密又小，再打开UV Editor，更改Tiling和Offset属性：

![image-20240908232534715](E:/Typora/picture/image-20240908232534715.png)

即可正常显示：

![image-20240908232622438](E:/Typora/picture/image-20240908232622438.png)

### 拼接场景

在**Assets/Prefabs/Environment/Corridors**下找到相关模型，拖入场景中，贴合我们建好的白盒模型并合理拼接即可。（略）

![image-20240908233219119](E:/Typora/picture/image-20240908233219119.png)

拼接好后即可删除白盒。

### 户外场景设计

户外的场景不需要使用ProBuilder做白盒设计，我们将引入Unity自带的Terrain对象。

在蓝图上绘制新的Plane，设置其Transfrom，然后在脚本中更改其Size和Cuts（Cuts越小，整个模型就越精细）。

![image-20240909103958425](E:/Typora/picture/image-20240909103958425.png)

然后使用PolyBrush绘制网格。

![image-20240909104732159](E:/Typora/picture/image-20240909104732159.png)

顶部第一个功能：改变网格形状，通过Max/Min更改笔刷半径范围，Outer Radius对应画笔外圈，Inner对应内圈，当在Mesh上点按或拖拉时，内圈的网格全部以最大Strength强度上凸（按Ctrl则下凹），其余外圈以内部分根据Falloff Curve曲线来更改凹凸程度。Bursh Mirroring控制镜像对称绘制。Ignore Open Edges控制边缘绘制是否允许边形状改变。Direction控制每次拉伸时的方向。Sculpt Power决定一次拉伸的力度。

第二个功能是圆滑。

第三个功能是画刷，按住Ctrl即可擦除。Color Paint Settings可以选择扩散/按格/全部绘制。

第四个功能是素材预制件添加，可以自行设定多种素材绘制的概率。

最后一个功能是纹理绘制和混合。如果最下面出现这个警告：

![image-20240909111419691](E:/Typora/picture/image-20240909111419691.png)

说明当前选中的材质不可用。纹理和材质不一样，简而言之，纹理是图像数据，而材质是使用这些纹理以及其他渲染信息来定义物体外观的资源。在Unity中，你可以将纹理应用到材质上，然后材质再应用到模型上，以此来控制模型的视觉表现。

现在来创建一个材质：

在Assets/Materials下新建一个Material，将Shader设为PolyBrush>Lightweight Render Pipeline>Lightweight Blend LWRP，这是一个Polybrush专用的制作混合纹理的Shader。

![image-20240909113459178](E:/Typora/picture/image-20240909113459178.png)

在Assets/ProBuilderPolyBrushDemo/Textures中的材质拖入Albedo Texture，再搜索Ground找到Ground_Albedo拖进来。

点击刚刚绘制的Plane对象，在Mesh Renderer中将默认的Material更改为刚刚制作好的材质：

![image-20240909114035963](E:/Typora/picture/image-20240909114035963.png)

再进入PloyBrush界面，最底下的界面就改变了：

![image-20240909114117221](E:/Typora/picture/image-20240909114117221.png)

使用不同材质的笔刷进行绘制，这样一个户外场景就算绘制好了：

![image-20240909114421747](E:/Typora/picture/image-20240909114421747.png)

## Blender基础

[Blender中文手册介绍 — Blender Manual](https://docs.blender.org/manual/zh-hans/2.80/about/introduction.html)

[建模 · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/tree/main/建模)

Blender 是一款功能强大的开源3D创作套件，它能够用于各种3D建模、动画、渲染、模拟以及渲染视频和静态图像的制作。以下是Blender的一些主要功能：

1. **3D建模**：创建和编辑3D模型，包括硬表面建模、雕刻、纹理绘制和UV展开。

2. **动画**：制作角色和物体的动画，包括骨骼绑定、关键帧动画、路径动画等。

3. **渲染**：使用Cycles渲染器或Eevee渲染器进行高质量的图像渲染。

4. **视觉效果**：创建视觉效果，如粒子系统、流体模拟、烟雾和火焰效果。

5. **模拟**：进行物理模拟，如刚体、软体、布料、毛发和肌肉模拟。

6. **雕刻**：使用雕刻工具进行高细节的模型创作。

7. **纹理绘制**：直接在3D模型上绘制纹理。

8. **UV展开**：将3D模型的表面展开成2D图像，以便在纹理上进行绘制。

9. **合成**：在渲染完成后，对图像进行颜色校正、特效添加和合成。

10. **视频编辑**：剪辑和编辑视频，添加过渡效果和标题。

11. **运动跟踪**：将3D模型与实际拍摄的视频或图像进行匹配和跟踪。

12. **游戏制作**：使用Blender的内置游戏引擎进行游戏开发。

13. **VR/AR开发**：创建虚拟现实(VR)和增强现实(AR)内容。

14. **脚本和插件**：使用Python脚本自动化工作流程或开发自定义工具。

15. **实时渲染**：使用Eevee渲染器进行实时渲染，适合游戏开发和实时预览。

Blender因其强大的功能和免费开源的特性，被广泛应用于电影、游戏、动画、视觉效果、建筑可视化和科学可视化等领域。

### 基本介绍

#### 进入界面

![image-20240909155815346](E:/Typora/picture/image-20240909155815346.png)

最重要的功能是左下角的Recover Last Session，如果你的项目未保存而意外关闭，那么再次启动时可点击此按钮以恢复上次编辑。

#### 系统界面

先看右上角**坐标轴**部分：

![image-20240909154524789](E:/Typora/picture/image-20240909154524789.png)

首先，坐标轴基本与Unity一致，但有一点不同的是，Unity是左手系而Blender是右手系。除了点击坐标轴调整视角，鼠标中键也可以。

放大按钮按住并拖拽即可缩放（鼠标滚轮亦可），移动按钮按住并拖拽即可平移视角（鼠标中键+shift亦可），点击摄像机按钮即可查看最终动画的效果，点击网格按钮即可切换透视模式。

然后看**最下面状态栏**：

![image-20240909160038056](E:/Typora/picture/image-20240909160038056.png)

除了动画时间轴以外，最下面一行是操作提示，即你现在可以对当前物体进行什么操作。

如果不想要时间轴，可以直接右键>Close Area。

除Menu以外所有的窗口都可以缩放，除此之外，在两窗口的交界处右键会出现这样一个菜单栏：

![image-20240909160425185](E:/Typora/picture/image-20240909160425185.png)

可以对一个窗口进行切分，合并，交换。

每一个窗口的左上角的图标都可以将当前窗口切换为其他窗口。

还有一些窗口操作的快捷键：

Shift+Space 将目标窗口最大化 

Shift+Fx键 将当前窗口切换为某个窗口（可在左上角切换界面中查询）

然后是**顶部菜单栏**：

![image-20240909161308817](E:/Typora/picture/image-20240909161308817.png)

这个是主菜单栏，无需多言。

![image-20240909161342361](E:/Typora/picture/image-20240909161342361.png)

这个是3D Viewpoint窗口的不同工作模式切换栏，包括布局、建模、UV编辑（贴图编辑）、渲染、动画等等。可使用ctrl+Page Up/Down进行切换。

Layout是默认与布局界面，**快捷工具栏**可由T键或者Shift+Space键呼出，

右边两个窗口，一个是**大纲**，还有一个是**属性窗口**

在属性窗口中还有一个侧边栏：

![image-20240909163731019](E:/Typora/picture/image-20240909163731019.png)

第一个模块是工具（和下面的Tool一样），2-5均与场景相关。第二个是图像效果属性，第三个是输出相关设置，第四个是视图层相关属性，第五个是视图相关属性，第六个是世界环境相关属性。剩下的基本都是与3D对象相关。

大纲窗口中上方的菜单栏可以切换当前视图或视图层：

![image-20240909165210404](E:/Typora/picture/image-20240909165210404.png)

在大纲窗口可以更改以场景为根或以视图为根。

![image-20240909164549196](E:/Typora/picture/image-20240909164549196.png)

点击右上角的按钮即可新建一个Collection：

![image-20240909165446036](E:/Typora/picture/image-20240909165446036.png)

当切换时，属性窗口的侧边栏也会随之改变。

视图与视图层的关系：

![../../_images/scene-layout_view-layers_introduction_collections.png](E:/Typora/picture/scene-layout_view-layers_introduction_collections.png)

在坐标轴右边还有一个小箭头，点开（或者按快捷键N）是这样一个界面：

![image-20240909162406888](E:/Typora/picture/image-20240909162406888.png)

其中Item编辑物体属性，Tool编辑工具属性，View编辑视图属性。

在View窗格中，有一个3D Cursor模块，用来控制视图中3D光标的位置：

![image-20240909162751686](E:/Typora/picture/image-20240909162751686.png)

图中红白圈即为3D Cursor，其强大功能我们以后再说，想要移动它，可以使用Shift+鼠标右键移动。

还有一个隐藏的圆盘菜单：shift+s。

### 基本操作

#### 选择

按住<img src="E:/Typora/picture/image-20240909170009294.png" alt="image-20240909170009294" style="zoom:33%;" />可以查看几种选择方式：单选（不常用）、框选、刷取、套索，除了长按切换还可用W快捷切换。在<img src="E:/Typora/picture/image-20240909170009294.png" alt="image-20240909170009294" style="zoom:33%;" />上面还有一排<img src="E:/Typora/picture/image-20240909170153832.png" alt="image-20240909170153832" style="zoom:50%;" />选择规则：普通选择、扩选（再次选取不取消已经选取的对象）、减选（从已选取对象中剔除）、反选（选择范围内未被选择的并取消已被选择的，在普通选择模式下按一次B然后按住Shift选择亦可）、交集（选择范围中上次被选中过的）。

还有<img src="E:/Typora/picture/image-20240909173459478.png" alt="image-20240909173459478" style="zoom:50%;" />的Select中规定选择，这里面的选择更加全面，日后使用到时再详细说明。

#### 变换Transform

移动可以按照Unity中的那种变换方法，也有一大堆的快捷键辅助（还可以用右键取消）。具体的你可以在使用Unity中的变换方法时关注最下面的操作提示栏。

如果做变换时想以3D Cursor为轴心点，可以在顶部栏：

![image-20240909175213503](E:/Typora/picture/image-20240909175213503.png)

选择3D Cursor即可。

左边还可以设定Global/Local坐标。

至于精确变换和吸附（Snap）变换，只需按住鼠标左键后按住Shift或者Ctrl即可或者更改属性即可。

#### 飞行模式/步行模式

实际Blender也有和Unity一样的操作模式。使用Shift+~快捷键即可进入飞行模式，点左键确定点右键取消。或者在顶部栏View>Navigation中切换亦可。

#### 删除操作

Shift+A或3D ViewPoint顶部栏Add菜单来添加对象。使用X或Object>Delete以删除。

如果需要对物体的点线面进行操作，切换到Modeling视图。Modeling视图下选中默认以点为单位（可以在<img src="E:/Typora/picture/image-20240910091202282.png" alt="image-20240910091202282" style="zoom:50%;" />中切换或多选），当选中相邻的点或者围成一个面的点时，则会自动选中边/面。如果没有选中边/面，则对边/面的操作不生效。在删除时，如果直接删除，则物体不会改变其形状：

![image-20240910082907638](E:/Typora/picture/image-20240910082907638.png)



但是如果使用Dissolve会实实在在地消除（但是：

![image-20240910083149947](E:/Typora/picture/image-20240910083149947.png)

Limit Dissolve（有限融并）模式一般用于去除平坦区域中（多余的）的顶点和边线，简化网络。（对简单图形不起作用，在此不做展示）。

Collapse Edge & Face（塌陷边线和面）模式将多个顶点合并为一个顶点：

![image-20240910083851682](E:/Typora/picture/image-20240910083851682.png)

边缘循环（Egde Loop）模式用于将两组相接的循环的面用一组循环的面代替：

![image-20240910084058830](E:/Typora/picture/image-20240910084058830.png)

#### 集合（Collection）

集合用于逻辑地组织场景，或方便一步在文件间或跨场景添加或链接。

在大纲（Outline）窗口中，拖拽（或选中后按快捷键M）一对象可以拖拽进一个集合中，如果想保留该对象在原集合中并归入另一个集合（成为两个集合的公共元素），可以按住Ctrl再拖拽。

集合可以套娃（包含）。

为了方便查看，可以右键选择颜色图标。

### 建模基础

#### Add

先切换视图为Modeling，切换模式为Object Mode<img src="E:/Typora/picture/image-20240910090311479.png" alt="image-20240910090311479" style="zoom:50%;" />，Add一个新的模型，再选中新的模型后切换到Edit Mode。

在Edit Mode下，OutLine窗口中被编辑的物体左边会出现一个Edit Mode图标：

![image-20240910090619737](E:/Typora/picture/image-20240910090619737.png)

在Edit Mode下，其他物体无法被选中和编辑，所有的编辑都始终视为当前编辑的是同一个物体。如果在Edit Mode下Add新的模型，则这个模型会与当前选中的模型合并为一个（添加了一个子对象）。

#### Inset Faces

内插面跟上面的生成并扩缩复制面是一个道理，但是在非挤压模式下S键是扩缩并连带其他面进行更改。内插面的快捷键为I。

在内插操作过程中，按住Ctrl键可以是内插面上凸或者下凹。

#### Extrude

对应<img src="E:/Typora/picture/image-20240910091326525.png" alt="image-20240910091326525" style="zoom:50%;" />图标。

选中点/线/面后，会出现一个白圈，在白圈内点击并移动鼠标即可进行挤出操作，点击"+"号球则会固定向法线轴挤压，如果点击在白圈外则会取消。挤出操作会将原来的点/线/面复制并移动复制体并与原体连接，原体则会保留。

在挤压时可以按下快捷键S对复制体进行缩放（直接按S是对原体进行操作且不产生复制体），缩放后鼠标左键确认即可对更改后的复制体挤压。

*如果出现了面显示有问题的情况：*

*![image-20240910093016508](E:/Typora/picture/image-20240910093016508.png)*

*说明新的面是反向的，此时一些操作可能会出bug。*

可以仅凭挤压功能构造出以下模型：

![image-20240910092618645](E:/Typora/picture/image-20240910092618645.png)

长按<img src="E:/Typora/picture/image-20240910091326525.png" alt="image-20240910091326525" style="zoom:50%;" />或者快捷键Alt+E可以呼出挤出菜单。

![image-20240910102739057](E:/Typora/picture/image-20240910102739057.png)

第二个是不生成复制体直接挤出，实际效果类似于更改长宽高（但是很神奇的是，我居然找不到有长宽高的属性，并且在挤压过程中Scale属性页的确没有改变）

第三个是沿法线挤出。即普通挤出的"+"模式，会产生复制体。

第四个是独立挤出。单独挤出一个面的时候与普通挤出没有区别，但是同时选择三个面并进行操作的时候就会出现下面这种更改：

![image-20240910103634655](E:/Typora/picture/image-20240910103634655.png)

可以看出当对多个对象挤出时，独立挤出会使它们按照各自的法线方向扩展。

而对于沿法线挤出，挤出后的样子是这样的：

![image-20240910104014264](E:/Typora/picture/image-20240910104014264.png)

可以看出三个面似乎都在朝着一个方向扩展。

第五个是朝光标方向挤出。

Blender中可以使用Alt+E快捷键快速选择并挤出。但是左边Extrude格中的挤出模式并不改变。想要实现快捷键切换模式，请自行在侧边栏中查看对应快捷键。

#### Loop Cut

点击侧边栏环切<img src="E:/Typora/picture/image-20240910104624644.png" alt="image-20240910104624644" style="zoom:50%;" />图标。

鼠标悬停时自动在当前边的中点进行环切预览。按住鼠标左键并拖拽即可挑整环切位置，松开左键即确定。

现在，使用挤出搭配环切你就能建模出一把椅子了：

![image-20240910105134734](E:/Typora/picture/image-20240910105134734.png)

环切里还有另外一种模式：偏移环切边，先选中一个环切圈，然后切换模式后长按拖拽即可在该环切圈两边对称形成两个新的环切圈。

环切的快捷键为Ctrl+R，当按下该快捷键后滑动鼠标滚轮时，会产生多条间隔均匀的环切圈。

#### Bevel

在木工中有一种操作叫倒角，实际就是切边/顶点将模型的棱角抹去。

点击侧边栏<img src="E:/Typora/picture/image-20240910135615992.png" alt="image-20240910135615992" style="zoom:50%;" />，选中边/面，拖动黄球即可显示控制切去的多少：

![image-20240910135736129](E:/Typora/picture/image-20240910135736129.png)

Bevel的快捷键是Ctrl+B，还可以在顶部栏进行编辑，找到Vertex>Bevel Vertex或者Edge>Bevel Edge。

Bevel模式下，操作提示栏上方会出现一个（可能被折叠的）Bevel菜单栏，点开即是Bevel的操作参数设置：

![image-20240910141451122](E:/Typora/picture/image-20240910141451122.png)

具体的参数解释详见Blender官方文档。

#### Knife

点击knife<img src="E:/Typora/picture/image-20240910142300430.png" alt="image-20240910142300430" style="zoom:50%;" />光标会变成一把小刀，点击一次确认起点，再点一次时切割一次并以该点为下一刀的起点。按空格/回车确认。点击鼠标右键取消本次切割并重新选择起点。点击Esc退出切割。

切割的另外一种方式是：切换为Bisset切分，将目标物体全选后，点击鼠标并拖拽在整个物体上切一刀，同时物体上方出现一个圆环：

![image-20240910143248371](E:/Typora/picture/image-20240910143248371.png)

拖动圆环即更改切割方向，拖动黄色箭头即平移切割线，在左下的Bisset带单栏中，勾选Clear Inner即去掉内侧部分，Fill即将切面补充上。

![image-20240910143453489](E:/Typora/picture/image-20240910143453489.png)

#### Poly Build

点击<img src="E:/Typora/picture/image-20240910143555144.png" alt="image-20240910143555144" style="zoom:50%;" />对多边形进行操作。该模式下只能对点进行操作。选中点后拖拽即可更改多边体的形状。按下Ctrl创建一个新点并且与最近的点连线，如果能与其他面围成则生成面。按下Shift用红框标记面，鼠标左键即删除。

或者，你可以新建一个物体，在编辑模式下A全选并X删除，然后利用Poly Build绘制几个点，按F自动连成面。

#### Modifier

在属性的侧边栏中找到modify<img src="E:/Typora/picture/image-20240910135935166.png" alt="image-20240910135935166" style="zoom:67%;" />模块，Add Modifier搜索Bevel添加：

![image-20240910140201890](E:/Typora/picture/image-20240910140201890.png)

[简介 - Blender 4.3 Manual：](https://docs.blender.org/manual/zh-hans/dev/modeling/modifiers/introduction.html)

**修改器**是一种自动操作，以非破坏性的方式影响对象的几何形状。使用修改器，可以自动执行许多效果而且不会影响对象的基本几何形状，否则手动执行这些效果（例如细分曲面）会非常麻烦。

它们的工作原理是**改变对象的显示和渲染方式**，而不是更改可以直接编辑的几何图形。您可以向一个对象添加多个修改器，以形成 [修改器堆队列](https://docs.blender.org/manual/zh-hans/dev/modeling/modifiers/introduction.html#the-modifier-stack)，如果您希望该更改永久存在，则 *应用* 一个修改器。

#### Spin

在侧边栏点击Spin<img src="E:/Typora/picture/image-20240910150144000.png" alt="image-20240910150144000" style="zoom:50%;" />，在3D Cursor上会出现一个两个+球的圆弧。选中物体的点/线/面/整个，然后点击一个+球并旋转，则会出现这样的现象：

![image-20240910150917241](E:/Typora/picture/image-20240910150917241.png)

可以拖动轴和中心点调控旋转轴和旋转中心。

在顶部栏中可以调控沿XYZ轴旋转：

![image-20240910151454667](E:/Typora/picture/image-20240910151454667.png)

也可以在Spin的菜单栏中可更改参数:

![image-20240910150945711](E:/Typora/picture/image-20240910150945711.png)

现在你可以利用Spin来构建一个碗的模型了：

在New Scene下，新建任意的一个图形，然后删除点直至剩下最后一个点：

![image-20240910152141825](E:/Typora/picture/image-20240910152141825.png)

选中该点，Shift+S呼出圆盘快捷操作，选择Selection to Cursor:

![image-20240910152332506](E:/Typora/picture/image-20240910152332506.png)

这样点就被移动到3D Cursor上了。

将视角切换到平行于Y轴上，便于2D绘制。

点击点，按住E拖动鼠标以延伸成边，直至将形成一个"有厚度的曲线"：

![image-20240910153347116](E:/Typora/picture/image-20240910153347116.png)

然后使用Spin功能，将旋转轴设定为Z，旋转角度设为360，更换视角查看效果：

![image-20240910153615248](E:/Typora/picture/image-20240910153615248.png)

修改Steps使其达到一个合理的精度，发大以看到内侧最里面的一堆点，使用Select Circle模式(刷取模式)选择中间的所有点，使用快捷键M或右键菜单栏中的Merge>At Center以汇聚成一点。

这样一个碗就建好了。

### 渲染基础

[建模/Blender/01-Blender中渲染过程简介.md · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/blob/main/建模/Blender/01-Blender中渲染过程简介.md)

这里只浅讲基本操作，更多的请查看官方文档。

#### Shading

[建模/Blender/02-Shader Editor简介.md · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/blob/main/建模/Blender/02-Shader Editor简介.md)

着色的核心工具是Shader Editor，在这里可以用可视化工具编辑Shader程序。

点击3D ViewPoint界面的顶部栏中的Shading视图，教程则建议调整为以下视图：

![image-20240910170609015](E:/Typora/picture/image-20240910170609015.png)

左上角也是个3D ViewPoint，切换![image-20240910170634469](E:/Typora/picture/image-20240910170634469.png)中的第四个，关闭Gizmos，关闭Header即可变成类似预览界面。想要一直保留这个视图配置，还需要在File>Defaults>Save Startup Files.

点击Shader Editor界面的顶部栏的New，新建一个Material:

![image-20240910171255880](E:/Typora/picture/image-20240910171255880.png)

这个界面同样有很多的快捷键。

更改一下Base Color，发现一个物体被染色，勾掉Use Nodes，则当前节点不会应用到物体上。

点击Slot下拉栏，可以切换或新建不同的Slot，一个Slot中可以挂载多个Material，存储的是Material和对象的Link关系，对象通过Slot把多个不同的Material挂接到对象表面的不同区域。

新建一个Slot并切换，再新建一个Material，将Base Color设为绿色，勾选Use nodes，然而刚刚应用过Material的对象并没有更改颜色。这是因为3D ViewPoint处于Object Mode状态，不能对物体的材质进行编辑。

切换到Edit Mode，拉出右侧被隐藏的Properties属性窗口，点击Material<img src="E:/Typora/picture/image-20240911081308567.png" alt="image-20240911081308567" style="zoom:50%;" />选中刚刚新建的Material，确认已选中整个物体，在下面点击Assign，物体即被重新挂接新的材质。

删除Slot并不会删除材质，但是删除对象会删除建立联系的Slot，但是不会删除应用的材质。那么如何删除Material呢？一种方法是在OutLine中找到物体下的Material对象删除，还有一种方法是在OutLine中切换到Blender File模式：

![image-20240911082210176](E:/Typora/picture/image-20240911082210176.png)

找到Material，删除指定的材质即可。

##### 原理化BSDF

[原理化 BSDF - Blender 4.3 Manual](https://docs.blender.org/manual/zh-hans/dev/render/shader_nodes/shader/principled.html)

在BSDF结点的左侧会有一列点，根据颜色分辨其属性的值类型——黄色为RGBA，灰色为Float，紫色为向量。颜色分辨的作用在于——一般这些节点和其他节点连接时，往往都是颜色相同的连接。

##### 导入素材

推荐免费材质素材网站：[ambientCG - CC0 Textures, HDRIs and Models](https://ambientcg.com/#:~:text=Limitless Freedom. Assets from ambientCG are)

进入网站后点击Explore即可查看可用素材。

下载好素材后添加进项目文件夹，然后拖入Shading界面，即会产生一个新的文件Node。

我们把文件Node的Color节点连接到Material的Base Color节点：

![image-20240912223824465](E:/Typora/picture/image-20240912223824465.png)

即可在当前材质上应用图片（类似贴图的效果）

##### Mix Shader

选择一个新的物体，添加新的材质，删除原理化BSDF，新建一个Diffuse BSDF（漫反射BSDF)，连接到输出。再新建一个Glossy BSDF（光泽BSDF），重新连接到输出。

现在我们想同时将两个BSDF应用到输出上，这时就需要使用到Mix Shader。

![image-20240912225039126](E:/Typora/picture/image-20240912225039126.png)

##### Texture

打开UV Editor，选中目标物体并选中所有的面。

![image-20240912225802478](E:/Typora/picture/image-20240912225802478.png)

想要更改不同面上的贴图位置，可以先选中面，在UV Editor中移动该面的框以选择。

除了快捷键S移动R旋转以外，UV Editor界面还有很多必要的功能：

![image-20240912230143247](E:/Typora/picture/image-20240912230143247.png)切换由面选择对应贴图还是由贴图选择面，

![image-20240912230241322](E:/Typora/picture/image-20240912230241322.png)选择点/线/面，选择点和线时，同一个点/线会关联高亮，便于查看不同面的贴图的连接情况。第四个——岛模式，可以选中所有连接的面。

##### More...

看教程文档。里面还有一篇关于计算机图形学的开源文档。

[建模/Blender/Texture纹理.md · chutianshu/AwesomeUnityTutorial - 码云 - 开源中国 (gitee.com)](https://gitee.com/chutianshu1981/AwesomeUnityTutorial/blob/main/建模/Blender/Texture纹理.md#32-)

#### Light
