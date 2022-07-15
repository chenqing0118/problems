## crossengine中的材质fx文件

fx文件代表一个材质模板，包括含有的属性以及初始值

## 类的函数定义与内联函数

在头文件中，将类的成员函数定义写在类定义内，是不会导致重复定义的，相当于内联函数。
如果在头文件中使用 A::f(){}这种方式定义，此时函数不是内联，在多个源文件中包含，会导致重复定义。

## 左右手系的转换

只需要任意一个坐标取反即可，判断左右手系，大拇指永远代表x，食指永远代表y，中指指向即为z

## git相关操作

- git upstream

  表示本地branch和远程branch之间的关联关系，可以从.git/config处查看
  
- git push <远程主机名> <本地分支名>:<远程分支名>

- 如果只想取回特定分支的更新
  git fetch origin develop/Main

- 删除远程分支
  git push origin --delete 分支名

- git rebase <分支名> 将当前分支进行换基，新的基底为<分支名>上的最新提交

## shader的定义过程

- **.shader文件**

  以ffs项目中为例，litforward.shader表示一个lit材质的前向渲染的shader。
  与unity hdrp 中的lit.shader比较而言，这里只定义了一个forwardpass。而hdrp中含有多个pass。

```hlsl
#include "../../ShaderLibrary/GlobalModelVariables.hlsl"
#include "../../Material/Lit/LitUEVariables.hlsl"

#include "../../ShaderLibrary/Vertex.hlsl"

#include "../../Material/Material.hlsl"
#include "../../Lighting/Lighting.hlsl"

#include "../../Material/Lit/LitDataUE.hlsl"
#include "../../Lighting/LightLoop/LightLoop.hlsl"
#include "../../RenderPass/ShaderPassForward.hlsl"
```

- **ShaderPassForward**

  里面包含前向渲染vert和frag的流程，里面使用到的数据结构和方法则在具体的材质中有定义。
  不同的材质（本例中是lit）在数据结构和方法会有区别，而前向渲染的流程一般不变，也就是Pass一般不用修改。

- **具体到某个材质**

  材质由三个文件构成，如：

  lit.hlsl，litVariables.hlsl，litData.hlsl

  - 先看lit.hlsl，里面包含了SurfaceData，BSDFData，PreLightData等用于光照计算的数据结构，ConvertSurfaceDataToBSDFData方法
  - 再看litVariables.hlsl，包含了材质用到的数据，其中的space0，1，2代表不同的更新频率，model，material，glabal更新的频率不同
  - 最后是litData.hlsl，包含了一个 GetSurfaceData(float3 V, PSInput psInput, bool isFrontFace)方法，
    ==为什么不把这个函数放到lit.hlsl中呢？==

## raytracing中的材质

- 漫反射材质，Lambertian ，每次选择一个随机的方向进行散射，可使用单位球面随机向量。

- 金属材质，metal，菱形法则求反射方向。可以使用单位球随机向量做出模糊的效果，fuzz越大，随机偏移越多，越模糊(有生锈的感觉)

所有的散射，根据hit_record获得hitable的材质，当散射次数depth=0或未打到物体或能量耗尽时停止。

## lit.hlsl中surfacedata和bsdfdata属性解读

- surfacedata

  - uint materialType;

    例中只有两种材质类型standard和SubsurfaceScattering。次表面散射模拟的是一大类常见的半透明材质。

  - float3 baseColor;

  - float alpha;

    不透明度，当alpha小于一个阈值时可以discard。

  - float3 normalWS;

    ==好像与切线空间有关，再看==

  - float3 geomNormalWS;

    直接等于psinput里的normalws

  - float roughness;

  - float ambientOcclusion;

  - float metallic;

  - float3 emissive;

    自身发光

  - float thickness;

    ==好像是次表面反射中需要用到厚度==

  - float subsurfaceMask;

  - int diffusionProfileID;

  - float grassOcclusion;

    开启植被系统后的参数

  - float treeOcclusion;

    开启植被系统后的参数

- bsdfdata

  - uint materialType;
  
    直接继承
  
  - float3 diffuseColor;
  
    通过basecolor和metallic计算
  
  - float3 fresnel0;
  
    菲涅尔项，通过basecolor和metallic计算
  
  - float3 normalWS; 
  
    直接继承
  
  - float3 geomNormalWS;
  
    直接继承
  
  - float roughness;
  
    直接继承
  
  - float3 S;
  
  - float thickness;
  
    直接继承
  
  - float3 transmittance;
  
  - float subsurfaceMask;
  
    直接继承
  
  - int diffusionProfileID;
  
    直接继承

## 次表面散射（SSS）

指光线进入物体内部再射出的情况，金属（导体）不会有这个现象

## 绝缘体/电介质

- 折射

  snell定律，$\sin(\theta)\eta=\sin(\theta')\eta'$

  如何计算折射出的方向，分为与法线水平与垂直的方向。其中垂直方向的长度可以根据snell定律计算得出，水平方向长度勾股定理简单得出

  折射需要注意$\sin(\theta)\frac{\eta}{\eta'}>1$的情况，此时发生的是全反射，出现在由折射率大进入折射率小的介质的情况

- schlik 是菲涅尔的近似简单版
