# AGVMod插件

* ## 目录结构
  ![Alt text](image.png) 
  * ### Content：  

    **插件的资源文件夹，包括机器人模型，材质，纹理，蓝图等。**  
    ![Alt text](image-1.png) 

  * ### Source：
    **插件的代码文件夹。**  
    ![Alt text](image-2.png)  

* ## 插件使用方法  
  ### 1. 将BP_AGVManager放到场景中并配置相应参数  
  ![Alt text](image-3.png)  
   **_UMG_FirstPerson_,*StorageManager*和*SmapManager*可为空，此时将生成默认的对象实例。**

  ### 2. 在蓝图中获取BP_AGVManager并调用方法
  ![Alt text](image-4.png)  
  **调用此方法后，将会请求对应的Core接口以及scene接口。**  

  ### 3. 创建库位并更新库位状态  
  ![Alt text](image-5.png)  
  **或者(使用默认StorageManager)：**  
  ![Alt text](image-6.png)  

  ### 4. 库位自动校准  
  **当SmapManager请求到scene数据并且库位创建完成后，将自动进行库位校准操作**

  ### 5. 机器人状态更新  
  **当正常请求到Core数据后，会根据数据创建并更新机器人状态。效果如下图：**  
  ![Alt text](image-7.png)  

* ## 核心代码介绍  
  * ### 机器人相关  
    ![Alt text](image-8.png)  
    * #### AAGVManager  
      控制机器人的生命周期，库位自动校准并且提供机器人统一行为的对外接口(比如机器人显示隐藏，点击事件等)。

    * #### UAGVConfigInfo  
      Agv配置相关信息。只要包括Agv类型的对应表，机器人自动校准偏差以及机器人虚影效果的配置。  
      ![Alt text](image-41.png)

    * #### AAGVBaseController  
      机器人基类，定义并实现机器人统一的行为(比如路径的更新，充电效果的显示以及机器人运动等)。
      可以按照机器人类型(料箱，叉车，顶升)实现子类，目前按照机器人型号实现了4个子类：  
      1.AHRController  海柔机器人(对应料箱机器人)  
      2.ASW500Controller SW500(对应顶升机器人)  
      3.ACDD14Controller CDD14(对应叉车机器人)  
      4.ADNController 地牛(对应叉车机器人)  

      其中CDD14和DN可合并为一种。  
      不同类型机器人可以重写TickAction方法实现不同的动作。

    * #### AHRController  
      料箱机器人。料箱机器人的核心部件为：Lift(升降机构)和StretchHand(伸缩臂)  
      ![Alt text](image-9.png) 

      ![Alt text](image-10.png)  

    * #### ASW500Controller  
      顶升机器人。顶升机器人的核心部件为：Jack(千斤顶)
      ![Alt text](image-11.png)  

    * #### ACDD14Controller和ADNController  
      叉车机器人。叉车机器人的核心部件为：Fork(货叉)
      ![Alt text](image-12.png)  

      ![Alt text](image-13.png)  
    
    * #### UMoveLightBaseComponent  
      机器人状态灯基类。  
      状态灯由两部分组成：一种反应机器人的运动状态，另一种反应机器人的工作状态。  
      运动状态有5种：静止，前进，后退，左转，右转  
      工作状态有6种：空闲，报错，急停，阻挡，工作，充电  
      有些机器人没有运动状态指示灯  
      每种机器人灯的变化需要重写对应的方法  
      以CDD14为例：  
      空闲状态： 
      ![Alt text](image-14.png)  

      错误状态：  
      ![Alt text](image-15.png)  

      急停状态：
      ![Alt text](image-16.png)  

      阻挡状态：
      ![Alt text](image-17.png)  

      正常工作状态：
      ![Alt text](image-18.png)  

      充电状态：
      ![Alt text](image-19.png)  

      后退：
      ![Alt text](image-20.png)  

      左转：
      ![Alt text](image-21.png)  

      右转：
      ![Alt text](image-22.png)

  * ### Smap相关  
    ![Alt text](image-23.png)  
    * #### ASmapManager  
      Smap相关的接口和数据容器(主要包括站点和路线)。

    * #### APathBaseActor  
      机器人行驶路线基类。行驶路线由四部分组成：已完成路线(暂时未用到)、当前占用路径、未完成路径和目标点。效果使用Niagara粒子系统实现，通过蓝图传入路线的点集来改变路线形状。  
      ![Alt text](image-25.png)  

      ![Alt text](image-24.png)  

    * #### USmapStyle  
      Smap元素(主要包括站点和路线)的样式配置。  
      ![Alt text](image-26.png)  

    * #### AStationBaseActor  
      站点基类。站点主要有4种类型LM，AP，PP和CP。当在设置中打开绘制站点选项时，会在解析完成Smap数据后，在场景中创建站点Actor。  
      ![Alt text](image-27.png)  

      ![Alt text](image-28.png)

      注意:目前站点没有显示，要想显示站点打开站点BP，关闭HideinGame。 
      ![Alt text](image-29.png)  

    * #### AStationPathBaseActor  
      站点连线基类。当需要显示站点路线时，在设置中打开绘制路线选项。  
      ![Alt text](image-30.png)  

      要修改路线的样式可在样式配置中更换路线材质，也可使用自定义的BP。
      ![Alt text](image-31.png)
      

  * ### 库位相关  
    ![Alt text](image-32.png)  

    * #### AStorageManager  
      库位相关的对象容器和状态管理接口。确定库位类型与库位BP的对应关系。  
      ![Alt text](image-33.png)

    * #### AStorageBaseActor  
      库位基类。一个库位包含一下几个部分:  
      1.碰撞盒:用于辅助检测机器人取放货。碰撞盒的碰撞预设设置为Storage(机器人的取货部位的碰撞盒得设置为Device)，如果没有碰撞预设，可在项目设置里增加。  
      ![Alt text](image-34.png)  

      ![Alt text](image-35.png)  

      2.货物：一般是一个ChildActor组件，类型为该库位货物的BP类型。  

      3.虚拟货物:虚拟货物有两种效果，一种是机器人要来该库位取货时的效果，另一种是机器人要到该库位放货时的效果。  
      
      取货效果
      ![Alt text](image-36.png)  

      放货效果
      ![Alt text](image-37.png)  

    * #### AShelfBaseActor  
      货架基类，暂时没用。 

    * #### APileStorageBase  
      堆叠库位，目前仅针对杭纺项目。

  * ### UI相关  
    ![Alt text](image-38.png)  

    ![Alt text](image-39.png)

    * #### UUMG_FirstPersonBase  
      第一人称相关的UI操作。
      ![Alt text](image-40.png)  

    * #### UUMG_AGVTag  
      机器人标签UI。现在机器人标签UI有两种显示样式：当相机距离机器人较近时展示机器人基本信息；当距离较远时，显示机器人图标。UI随距离变化的配置可在AgvManager的细节面板配置。  
      ![Alt text](image-42.png)  

  * ## 新增机器人  

    **新增机器人有以下几个步骤：**  
    1. 确定增加的机器人类型(这里以叉车类型机器人为例)
    2. 创建对应机器人的蓝图类，添加状态灯组件(暂时得手动添加)
    ![Alt text](image-44.png)
    3. 机器人模型绑定，货物绑定，添加碰撞盒并调整至合适大小(将碰撞预设改为Device)  
    ![Alt text](image-46.png)  

        在构造脚本里给对应的变量赋值
    ![Alt text](image-47.png)

    4. 在蓝图中实现需要的方法(取放货部位的碰撞检测，充电效果的显示隐藏等)  
    ![Alt text](image-48.png) 

    5. 在代码中找到ERobotType枚举并增加对应的机器人型号(如果不存在冲突可复用已有的型号，比如项目中有CDD15型号而没有CDD14型号，那么可以不新增CDD15型号直接用CDD14，只要把机器人的BP绑定为CDD15就行)，在机器人配置信息中配置该型号的对应关系
    ![Alt text](image-43.png)   
      
       绑定型号
    ![Alt text](image-49.png)
    

    6. 正确绑定的情况下实际运行效果如下图：
    ![Alt text](image-50.png)  

    **完整的机器人绑定过程可参考CDD14的BP**