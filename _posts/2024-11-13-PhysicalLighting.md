---
layout: post
title: UE真实灯光环境设置
date: 2024-11-13
author: domehall
tags: [Lighting, Theoretical, Practical]
comments: true
toc: true
pinned: false
---

# 一、理论

成像的三个要素 ~~（我自己编的）~~：光源、物体表面（材质）、人眼（相机）

想排除物体表面的影响，搭建物理正确的光照环境，只需管理**光源**，其次是**相机**

## 1. 光度学

| 物理量                       |   单位                                   |
|:----------------------------|:----------------------------------------|
| 发光强度（Luminous Intensity）|  坎德拉（Candelas，cd）                   |
| 发光通量（Luminous Flux）     |  流明（Lumens，lm）                       |
| 亮度（Luminance）            |  尼特（Nit）= cd/m^2^                     |
| 照度（Illuminance）          |  勒克斯（Lux）、英尺烛光（Foot Candles，fc） |

## 2. 摄影学

**光比**：被摄物体向光面和背光面的亮度比值；或者，主光和补光的强度比值

![952X429/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/jW2f/952X429/image.png)

**曝光**：

  - EV = Exposure Value：代表摄像机的快门速度和光圈（F 数）的组合值

  - EV 本质上是曝光的度量值，使得产生相同曝光水平的快门速度和 F 数的所有组合具有相同的 EV

- EV100：使用 ISO = 100 胶片时的 EV

**中灰**：sRGB 119（0.46），sRGB Linear 0.18（反射率 18%）

![646X428/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/UhCO/646X428/image.png)

校准曝光：使用中灰材质的板子（灰板），使灰板面向光源，相机测光点对准灰板，调整曝光，使得中灰材质在相机图像内为中灰亮度

**自动曝光**：使相机图像**平均亮度**为中灰亮度

  - 场景越亮，相机内平均亮度达到中灰亮度所需要的 EV100 就越高

## 3. 人眼视觉

**浦肯野效应**：从昼视觉向夜视觉转变时，人眼对光的最大敏感性向高频（蓝光）方向移动，此时场景中的蓝色部分在人眼视觉中更明亮（因为是视觉改变而不是环境改变，所以相对严谨的模拟方式是通过相机后期来模拟，如将色彩平衡调整为偏蓝）

![739X375/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/L7V7/739X375/image.png)

# 二、真实世界的数据

- 来源见【参考】节
- 可根据实际情况调整，真实测量数据作只为基准参考

##  1. 照度、EV100和光比

- 经验 EV100：仅代表对应场景在此 EV100 下看着舒服，不代表自动曝光值
- 光比：随氛围变化，可视为范围内最值

| 场景 | 测光板的照度（lux） | 经验EV1OO | 光比 |
|:--|:--|:--|:--|
| 正午时晴朗无云的室外（自然光） | 110000 ~ 120000 | 14 | 16：1 |
| 正午时多云晴天或者晴天阴影处的室外（自然光） | 20000 | 10 ~ 12 | 2.5 : 1 |
| 阴云密布的室外（自然光） | 1000 ~ 2000 | 10 | 1.65:1 |
| 日落黄昏时的室外（自然光） | 400？ | 7 ~ 12 | |
| 夜晚时的室外（自然光） | < 1 | -6 ~ -2 | |
| 夜晚时的室外（城市内人造光源） | 30 | 5 ~ 8 | |
| 办公室、家庭类型的室内 | 100 ~ 750？ | 5 ~ 8 | |
| 展厅、体育场、舞台类型的室内 | ？ | 8 ~ 11 | |

上表为个人总结，照度一列有部分不明确的地方（以问号标注），所以附上一个感觉也很合理的参考数据：

![618X942/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/F1mA/618X942/image.png)

## 2. 人造光源

| 人造光源类型 | 光源发光通量（lm） |
|:--|:--|
| 烛光 | 12 |
| 小型装饰灯 | < 100 |
| 装饰灯 | 200 ~ 300 |
| 普通房间的吸顶灯 | 400 ~ 800 |
| 大型房间的吸顶灯 | 800 ~ 1200 |
| 路灯 | 1000 ~ 40000 |

# 三、引擎中的实现

基本思路：

![886X83/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/GwcS/886X83/image.png)

1.  **根据相机 EV100 的经验取值固定 EV100**

2.  **按以上真实测量数据确定所有光源的强度（以达成目标照度、光比）**

3.  **恢复到自动曝光并调整自动曝光**，比如调整曝光补偿曲线，以达到符合人眼感知的结果而不是相机自动曝光结果；视情况**调整其他后期参数**，如白平衡等

    - 这一步算视觉调整，并不影响灯光环境本身的真实物理性，"摄影创作中没有正确只有合适"

## 1. 引擎设置

- Extend Default Luminance 开启

- Pre-Exposure 开启

- 精确光源（点光源、聚光源和矩形光源）的反比平方衰减开启（默认开启）

## 2. 固定 EV100

预先固定 EV100 ，防止"受被摄物影响的自动曝光"或"不合适的
EV100"影响打光时的感官判断

可以在视口里设置，也可以直接在后期盒子里固定

![289X682/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/IkGN/289X682/image.png)

![310X106/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/JNSM/310X106/image.png)

## 3. 确定光源强度

### 1）如何在引擎内测光

打开 HDR (Eye Adaptation) 视图

![1041X522/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/wPZ6/1041X522/image.png)

测**照度**：

![430X286/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/P0E2/430X286/image.png)

1.  在场景中放置一个面朝定向光的面片作为测光板，测光板的材质：Basecolor= 1（纯白色），Metal = 0，Specular = 0，Rough = 1（全漫反射）

2.  打开 HDR 视图，观察测光板上的 Illuminance（上方第一个探针的值），该值代表当前整个法线半球方向对此测光点的照度，单位 Lux

- 注：测照度时要注意周围场景的开阔程度影响

测**光比**：

![670X358/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/JWuZ/670X358/image.png)

1.  在场景中放置一个面片和一个圆柱体（一般用全漫反射中灰材质），如图所示，放置于定向光直射下

2.  在 HDR (Eye Adaption) 中观察这个装置的亮面和暗面的 Luminance，二者比值即为光比
    - 并不严格


### 2）确定光照方案

![1190X757/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/GPiQ/1190X757/image.png)

### 3）天光（Sky Light）

除间接光外，室外照度主要受定向光和天光共同影响，暗部受天光影响，亮部受定向光 + 天光影响，所以建议**先确定天光**

**天光 Intensity Scale** （即 BP_LookDev 中的 Sky Light Intensity）要通过测光确定，以达到符合要求的照度和光比

对于 HDRI 天光，建议 Intensity Scale 基于 power(2, EV100) 调整

- 因为在 UE 中，最终的 HDRI 天光强度 = Sky Light Intensity \* HDR 图像采样出的线性值，而 UE 对 HDR 图像的采样基于 EV100 = 0

对于捕捉的天光，建议 Intensity Scale 先取 1，调整定向光后，再回来调整天光，Intensity Scale 基于 1 微调

可打开 Visualize SkyLigfht Illuminance 查看当前的天光采样结果

![256X181/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/3B3f/256X181/image.png)

![830X483/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/dm7E/830X483/image.png)

### 4）定向光（Directional Light）

定向光的调节和是否作为大气太阳光相关：

- 定向光不作为大气太阳光时，主要调节：强度、色温、Source Angle（控制光质）

- 定向光作为大气太阳光时，主要调节：强度

  - 强度的调节也可由天空大气的 Mie Scattering & Mie Absorption
    参数替代，自身保持日光/月光的默认强度值即可

  - 照到物体表面的色温由天空大气的吸收、散射过程自动计算

在制作真实环境 LookDev 时，如果参考数据：

- 很充足，推荐定向光不作为大气太阳光，更可控（仅对于制作 LookDev 而言，实际引擎中有一部分功能和大气光有关，还是得用）
- 不充足，就只能让定向光作为大气太阳光，依靠天空大气组件的计算决定色温等

UE 官方文档 [SkyAtmosphere](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/sky-atmosphere-component-in-unreal-engine?application_version=5.4#additionalinformation) 的、"其他注意事项、"部分给出了日光、月光和大气设定的建议，可参考

### 5）人造光源（Local Light）

| 真实世界 | 引擎内 |
|:--|:--|
| 室内照度受房间大小和灯泡个数影响，宽敞高大的室内会打很多灯来照明 | 如果项目内有很高很宽敞的房间，就没法用一两个真实灯泡亮度的光源去照亮；在引擎里，从性能考虑，不会真的暴力地去打很多个光源来还原 |
| 光源衰减距离是无限的 | 直接光衰减距离有限 |

解决办法：把单个光源视作很多个灯泡的集合，打灯后可以补光

——因此没有必要去给所有光源开反比平方衰减来追求物理性，只给实际作为一个灯具照明的光源开反比平方衰减就可以了（一般是装饰灯）

考虑到投影问题，只给主光源开投影

最终室内照度在合理范围内即可（测照度的点在人高度附近）

## 4. 调整相机后期

恢复到自动曝光：

![489X110/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/gGd6/489X110/image.png)

// 占位，待添加图片（给一个暗场景开关自动曝光）

相机自动曝光算法都是把所有场景都自动调节成中灰亮度，在场景过暗或过亮时会出现不符合人眼视觉经验的结果（该暗的时候没有暗下去，该亮的时候没有感觉很亮），需要调整，建议用曝光补偿曲线调整

![436X171/image.png](https://tc.z.wiki/autoupload/f/s94kodVkqlR-sch2mSQ86dxfv_1EF54v7RrPqsKp-Giyl5f0KlZfm6UsKj-HyTuv/20250719/SJVy/436X171/image.png)

- 模拟人眼对晴天等过亮环境的感知：曝光补偿 + （即 EV100 数值 -）
  - UE 默认设置的曝光补偿 +1：讨好明亮场景下的人眼视觉效果

// 占位，待添加图片

- 模拟人眼对夜晚等过暗环境的感知：白平衡调低

// 占位，待添加图片

**光源强度确定后，基本上所有视觉上过亮或过暗的问题都是曝光问题，不要再去调整光源强度**

e.g. 开启全局光照的情况下，室外正午晴天，室内无光源，视点位于室内，用未调整的自动曝光的相机观察室内场景，会感觉太亮

——并不是光照出问题，是曝光出问题

所以前期打光时固定曝光很重要，不要被自动曝光欺骗 （・∀・）

对于 LookDev，出于资产检查的需求，还是需要固定曝光

# Tips

## 1. 绝对亮度其实没那么重要

Q：为什么大部分项目都可以不采用这套真实物理的光照数值？

A：光的强度数值可以随 EV100 一起缩放而保持视觉效果不变（不考虑雾效等额外的亮度）

- 缩放规律：EV100 + 1，光强 * 2

- 举例：一个 EV100=14，HDRI 天光强度 = 1 * pow(2, 14) = 16384，定向光强度=120000 lux 的环境，等同于 EV100 = 0，HDRI 天光强度= 1，定向光强度= 7.3 lux 的环境

对于一般打光流程，只要能保证先固定 EV100（随便固定成哪个值，一般取 EV100=0）然后再打光（根据视觉感受调整光照数值），再调整自动曝光，也是能达到视觉上合理的光照的

只是使用真实物理光照数值有一些好处：

- 对相机和画面的调整可以直接沿用摄影圈经验

- （个人感觉）宽曝光范围下的高光比场景给人眼的视觉感受更强烈（亮部有过曝感）

## 2. 不想用网络参考数据来回调整，想用实景拍摄结果去 match？

一个更深入的话题……

在这里放一篇笔记：[Enabling a Look Development Workflow for UE4 \|
Unreal Fest Europe 2019 \| Unreal Engine
(youtube.com)](https://discreet-sandal-328.notion.site/Enabling-a-Look-Development-Workflow-for-UE4-Unreal-Fest-Europe-2019-Unreal-Engine-youtube-com-11038356a5fe802c977fd5a626c562dc)

内容包括：前期工具准备、拍摄布景、图像处理、UE 内虚拟环境匹配拍摄结果

# 问题记录

- 自发光特效材质：
  - 如果项目在中途改用此设置，此时应该会发现绝大部分特效材质都没有使用 eye adaptation——这导致使用真实物理光照时，在非常亮的日光导致的高 EV100 曝光下，会看不见发光特效，或者白色的烟雾变黑，解决办法就是给特效材质用上 eye adaptation

- 后期全屏特效材质：
  - 在 blendable location 和材质节点中都要谨慎使用 scene color
  - 因为使用物理光照设置后，scene color 是线性的反射光强，不再是所见即所得的颜色（事实上也本来不是，只是光强度小的时候问题不明显），日光下直接数值爆炸，对 scene color 还要 blend 一层自发光颜色的话曝光结果就坏掉了
  - 如果只是想对所见即所得的颜色进行处理，应该使用 tone mapping
    之后的颜色，blend location 也要选 after tone mapper 或者之后的

# 参考

五星级资料，看完武功大成

- [关于建立资产统一审查预览环境的一些说明 - Unreal Engine](https://www.unrealengine.com/zh-CN/tech-blog/a-few-tips-for-building-unified-assets-reviewing-enviroment)
- [The Challenges of Rendering an Open World in Far Cry 5](https://advances.realtimerendering.com/s2018/The%20Challenges%20of%20Rendering%20an%20Open%20World%20in%20Far%20Cry%205%20(With%20Notes).pdf)
- [Moving Frostbite to Physically Based Rendering 3.0](https://seblagarde.files.wordpress.com/2015/07/course_notes_moving_frostbite_to_pbr_v32.pdf)
- [What are LV and EV (kenrockwell.com)](https://www.kenrockwell.com/tech/ev.htm)
- [使用虚幻引擎中的物理光照单位 | 虚幻引擎 5.4 文档 | Epic Developer Community (epicgames.com)](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/using-physical-lighting-units-in-unreal-engine)

数据参考

- [Physically Based - The PBR values database](https://physicallybased.info/)
- [Daylight - Wikipedia](https://en.wikipedia.org/wiki/Daylight)
- [Exposure EV Table](http://www.photographers-resource.co.uk/downloads/Photography/EV_Guide.pdf)
- [Moving Frostbite to Physically Based Rendering 3.0](https://seblagarde.files.wordpress.com/2015/07/)
- [What are LV and EV (kenrockwell.com)](https://www.kenrockwell.com/tech/ev.htm)
- [Bulbs.com](https://www.bulbs.com/)
- [Understand physical light units | High Definition RP | 17.0.3 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.high-definition@17.0/manual/Physical-Light-Units.html)