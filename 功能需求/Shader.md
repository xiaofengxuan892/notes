[TOC]



##### Shader基础知识：

1.shader中unlit是不发光，vertex lit是顶点光照，specular是高光—金属/玻璃，normal map也被称为凹凸贴图，即法线贴图。其中unlit由于不受光照影响，只需要将纹理Texture显示出来，因此常用于UI

shaderlab语法中，多个subshader是为了满足不同的硬件条件，按从上到下顺序依次选择，但任何一种情况下只会有唯一的一个subshader被执行。最后的fallback是当所有的subshader都不能满足当前硬件的渲染时，由unity本身提供的可以被硬件广泛支持的最低限度的subshader

Shader中的透明通道比较耗费性能，通常不开启，只有在有特殊需求时才会开启

GPU相比于CPU，其并行执行任务的能力更强。CPU更多时候是按照顺序执行。

OpenGL是跨平台的，但使用的语言GLSL自成一家，DirectX主要用于底层为windows的平台——HLSL

**Unity自带的.cginc文件所在路径**：C:\Program Files\Unity\Editor\Data\CGIncludes

2.Shader中不要使用复杂的数学运算，少用“if/else”语句或“循环”语句。声明的数据类型尽量从低精度的类型开始选择：fixed -> half ->float

3.Shader中各个Pass通道的作用：

Shader中每一个Pass通道对应一次drawcall

案例研究：

Cube物体使用材质球“BlendTwoColorMat”—— 包含两个Pass通道的混合， 

Sphere使用材质球“BlueMat”—— 只有一个Pass通道

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231108203643418.png" alt="image-20231108203643418" style="zoom:80%;" />

操作：每增加一个Cube的复制体，则drawcall增加2；每增加一个Sphere的复制体，则drawcall增加1

由此可以验证Shader中Pass通道的数量直接决定了drawcall的数量

###### 获取RGB颜色的灰度值：

表示颜色的方式有多种，如RGB，HSV，CMYK等。其中最早期的颜色使用“黑白模式”来表示，即灰度值：数值越高则越亮，反之则最暗。

将RGB颜色转换成“黑白模式”的灰度值通常使用公式：Grey = 0.299 * R + 0.587 * G + 0.114 * B，如：

```
half gray = dot(col.rgb, fixed3(0.299, 0.587, 0.114));
return half4(gray, gray, gray, col.a);
```

###### Lambert光照模型：

当物体表面发生“漫反射”时可使用本模型用于计算颜色值，即`dot(lDir, nDir)`，但本模型在计算物体背面的颜色时，数值直接为0，在画面上无法体现“由亮到暗的逐渐变化”。

为了解决此问题，由Value公司提出新的“HalfLambert光照模型”，其计算方式为：`dot(lDir, nDir) * 0.5 + 0.5`。这样在物体背面可以得到“由0.5到0的逐渐变化趋势”，与真实场景的效果更为接近

###### Shader中“Vertex”和“fragment”程序的区别：

“Vertex”程序专用于处理物体顶点的空间变换，“Fragment”程序则用于计算物体的颜色。前者由于顶点数量有限，因此执行耗时较短，而后者计算光照较慢，但最终物体显示的颜色较为平滑。而且随着硬件性能提升，“Fragment”程序计算光照耗时也明显缩短，不会称为性能瓶颈

###### Mobile平台专用Shader：

为了在手机上有较好的性能，Untiy内部专门设计了一些常用Shader，均为“Mobile”一栏中：

“Unlit”：仅使用纹理颜色，不受光照影响

“VertexLit”：顶点光照      “Diffuse”：漫反射    “Specular”：在漫反射基础上增加高光计算

“Parallax Normal Mapped”：视差法线贴图

“Emission”是自发光

###### Surface Shader的作用：

Surface Shader内部包含很多自定义标记，无需手动计算光照，因此当场景中有多个光源时，使用该类型shader更为方便。在Unity中创建Shader时，默认都是Surface Shader

关键代码使用CG语言编写，将编号的CG代码使用 CGPROGRAM............. ENDCG 嵌在ShaderLab语言中。

Surface shader不使用pass段。代码如下:

```
Shader "Custom/MyShader" {
     Properties {
          _MainTex ("Base (RGB)", 2D) = "white" {}
          _ReflectionTex("Environment Reflection",2D)=""{}
          _BumpMap("Bumpmap",2D)=""{}
     }

     SubShader {
          Tags { "RenderType"="Opaque" }        //标签
          LOD 200
         
          CGPROGRAM
          #pragma surface surf Lambert          //固定结构，“surf”是表面函数，“Lambert”是光照模型
          sampler2D _MainTex;
          sampler2D _BumpMap;

          struct Input {                        //固定结构
               float2 uv_MainTex;               //固定书写
               float2 uv_BumpMap;
          };

          void surf (Input IN, inout SurfaceOutput o) {         //固定结构
               half4 c = tex2D (_MainTex, IN.uv_MainTex);       //固定结构
               o.Albedo = c.rgb;
               o.Alpha = c.a;
               o.Normal=UnpackNormal(tex2D(_BumpMap,IN.uv_BumpMap));
          }
          ENDCG
     }
     FallBack "Diffuse"
}
```

PS：sufaceShader是对vertex shader 和fragment shader 的封装，本质上的执行还是靠后者

###### 固定管线着色器的作用：

该类型shader通常用于某些性能较低的设备，因此效果较为单一，不支持“丰富的自定义效果”。常用格式如下：

```
Shader "Custom/NewShader" {                              //该shader的name
     Properties {   设置该材质会用到的参数
          _MainTex ("Base (RGB)", 2D) = "white" {}       //2D纹理，white可以写也可以不写
          _Color("Main Color", Color)=(1,2,3,1)         
          //第一个_Color 是在shader中会用到的该参数的name
          //第二个"Main Color"是当把该shader赋值给某个材质后，该材质在inspector面板中显示出来的名字
          //第三个参数Color指的是shader中的参数_Color的所属类型，后面的就是这个变量的赋值
          _Shininess("Shininess", Range(0, 1.5))= 0.7     //Range(0, 1.5)代表的是这个变量是某个范围内的float值
          _ReflectionTex("Environment Reflection", 2D)=""{}
          _BumpMap("Bumpmap(RGB)", 2D)=""{}
          _Cube("Cubemap", CUBE)=""{}     //CUBE代表的是该变量是立方体贴图属性
    }
    SubShader {    
          //一个shader中有多个子shader，游戏运行时依次往下查找，找到第一个能用的子shader即可。
          //每个子shader中有多个pass段，每个pass段都会被渲染一次，因此需要把子shader中pass段设置的少些
          Pass{                             //pass可以设置name，如此在别的shader中可以调用该pass段，代码的复用
                                            //例如代码：UsePass"Specular/BASE"就是使用Specular这个shader的Pass段BASE
             Material{                      //设置光照所需的材质参数
                Diffuse[_Color]
             }
            Lighting On                     //打开光源
            Fog{Mode off}                   //关闭雾效
            SeparateSpecular On             //开启高光颜色
            SetTexture[_MainTex]{           //设置纹理
                 constantColor[_Color]
            }
          }
    }
    SubShader{
         .....................
    }
     FallBack "Diffuse"                  
     //当所有的子shader都不适用时，再调用备用的着色器，因此该着色器对硬件等的要求很低，目的是保证可以正常运行。
     //“Diffuse”就是备用情况下会调用的shader
}    //当代码中为“Fallback Off”时代表不使用备用着色器
```

##### Shader在实际应用中出现的问题：

###### 1.将模型导入Unity后其表面出现“粉红色”

建议方案：导入模型到Unity后该物体表面出现“粉红色”，通常都是因为该模型使用的Material的Shader错误，只需要改下该Shader类型即可，通常选择“Legacy栏”下的Shader，如diffuse等



##### 各个常用美术效果的Shader实现：

###### 1.碎屏效果的Shader：

```
Shader "Custom/GreyShader"
 {
		   Properties{
			   _NormalMap("normal map",2D) = "White"{}
			   _MainTex("mainTex",2D) = "White"{}
			   _ScaleX("scaleX", Range(1, 5)) = 1
			   _ScaleY("scaleY", Range(1, 6)) = 2
		   }

		   SubShader{
		   /*正常的MainTex的采样，normalDir ,lDir都是正常的*/
		   Pass{
			  TAGS{"LightMode" = "ForwardAdd"}
			  CGPROGRAM
			  #pragma vertex vert
			  #pragma fragment frag
			  #include "UnityCG.cginc"
			  #include "Lighting.cginc"

			  sampler2D _MainTex;
			  sampler2D _NormalMap;
			  float _ScaleX;   //增加可以外部调节采样UV的功能
			  float _ScaleY;
			  struct v2f {
						  float4 pos:SV_POSITION;
						  float2 uv_MainTex:TEXCOORD0;
						  float3 myNormal:TEXCOORD1;
						  float4 myVertex:TEXCOORD2;
			  };

			  v2f vert(appdata_base v) {
						  v2f o;
						  o.uv_MainTex = v.texcoord;
						  o.pos = UnityObjectToClipPos(v.vertex);
						  o.myNormal = v.normal;
						  o.myVertex = v.vertex;
						  return o;
			  }

			  fixed4 frag(v2f In) :SV_TARGET{
						  fixed4 col;
				   
						  //碎屏效果：使用normalMap提供的贴图UV进行采样
						  half2 bumpUV = In.uv_MainTex - 0.5;
						  bumpUV *= float2(_ScaleX, _ScaleY);
						  bumpUV += 0.5;
						  half2 bump = UnpackNormal(tex2D(_NormalMap, bumpUV)).rg;
						  //使用"rg","rb","gb"是可以得到不同的效果
						  float2 mainTexUV = bump * 0.5 + In.uv_MainTex;
						  //如果直接使用“bump”作为“_MainTex”的采样UV则只能得到normal map的图像——原因其实我现在还不清楚
						  //因此这里需要另外加上“In.uv_MainTex”
						  col = tex2D(_MainTex, mainTexUV);
						  /*暂时总结：
						  从以上的代码及最终结果来看：
						  碎屏效果的重点在于改变采样MainTex的UV，根据不同的normalmap会呈现出不同的碎屏效果
						  在normalmap本身就是一个碎屏的图像时效果是比较好的
						  */

						  //根据以上总结可对代码做如下简化：
						  //half2 simplifyBumpUV = UnpackNormal(tex2D(_NormalMap,  In.uv_MainTex)).rg;
						  ////为了使最终效果适于调节，引入"scaleX","scaleY"
						  //simplifyBumpUV *= float2(_ScaleX, _ScaleY);
						  ////不论normalMAP采样如何，必然要使用“_MainTex”相应的UV才行
						  //float2 simplifyMainTexUV = simplifyBumpUV * 0.5 +  In.uv_MainTex;
						  //col = tex2D(_MainTex, simplifyMainTexUV);

						  //普通采样：
						  //正常的使用该物体本身的顶点UV数据对贴图进行采样
						  //col = tex2D(_MainTex, In.uv_MainTex);
						  //正常的计算光照对颜色产生的影响
						  /*float3 nDir =  normalize(mul(unity_ObjectToWorld,In.myNormal));
						  float3 lDir =  normalize(UnityWorldSpaceLightDir(mul(unity_ObjectToWorld,In.myVertex)));
						  col *= (dot(nDir,lDir) * 0.5 + 0.5) * _LightColor0;*/
						  
						  //对最终得到的图像进行置灰处理
						  /*float grey = dot(col.rgb, fixed3(0.299, 0.587, 0.114));
						  return half4(grey, grey, grey, col.a);*/
						  
						  return col;
			  }
			  ENDCG
		   }
		 }
	   FallBack "Diffuse"
}
```

效果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112123506577.png" alt="image-20231112123506577" style="zoom:80%;" />

**注意**：碎屏效果会根据使用的NormalMap而变化，不同的NormalMap会有不同的“碎屏效果”。以上效果使用的NormalMap为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112123636705.png" alt="image-20231112123636705" style="zoom:80%;" />



###### 2.ZTest深度比较：

```
Shader "Custom/myShaderZwrite" {
       Properties{
            _UpperColor("UpperColor",color)=(1,1,1,1)
            _LowerColor("LowerColor",color)=(1,1,1,1)
       }
       
       SubShader{
           /*************************不透明物体的ztest********************/
           /*显示上半部分没有被遮挡的颜色*/
           Pass{
              CGPROGRAM
              #pragma vertex vert
              #pragma fragment frag
              
              fixed4 _UpperColor;
              
              struct a2v {
                   float4 myVertex:POSITION;
              };
              struct v2f {
                   float4 pos:SV_POSITION;
              };
              
              v2f vert(a2v v){
                   v2f o;
                   o.pos=mul(UNITY_MATRIX_MVP,v.myVertex);
                   return o;
              }
              fixed4 frag(v2f In):SV_TARGET{
                   fixed4 col;
                   col=_UpperColor;
                   return col;
              }
              ENDCG
           }
          
          /*用于显示下半部分被遮挡的颜色。先渲染上半部分的颜色，后渲染下面的颜色。
            两者的顺序不可颠倒，因为Pass通道会把物体的颜色重新渲染一遍，依据是Ztest的比较.除非添加blend语句，否则显示的为最后一个pass渲染的结果，即颜色覆盖
            当然如果这个新的颜色没有通过ztest比较，那么显示的还会是原来的颜色
            总之：1.Pass通道中自己定义了一个颜色——这是物体在没有任何遮挡时原本的颜色  
                 2.通过ztest的比较，只有在符合条件时，该像素点原本应该出现的颜色才会保存到color数组中输出到屏幕上
             通过不同的比较依据则可以出现物体各个部分不同颜色的效果
             该比较的最原始依据是：从camera到物体，如果没有遮挡，则为默认的最大。ztest默认是lEqual*/
            Pass{
              ztest Greater
              /*由于下半部分前面有不透明物体遮挡，而unity默认是zwrite on的，且对于不透明物体必须为zwrite on才能在camera中看到
                camera渲染顺序是按照离camera从近到远的顺序
                因此前面遮挡物渲染之后该屏幕像素点对应的depth被重写，值就是该遮挡物的Z轴depth，该遮挡物后面物体是否被看到则取决于跟此depth的比较
                且如果后面的颜色通过depth比较了，那么会用后面颜色代替之前的颜色，而不是两者的混合.
                颜色的比较总是两者之间相互比较，不会出现与第三者比较，然后某两者的颜色一起输出的情况，因为这两者之间还会再次重新比较。最终取符合条件的唯一一个颜色
                对于半透明物体而言，之所以会看到两者颜色的混合是因为透明所导致的，因为透明所以其他的颜色也可以露出一部分。但严格来讲永远只会有某一个物体的颜色输出。*/
              CGPROGRAM
              #pragma vertex vert
              #pragma fragment frag
              
             fixed4 _LowerColor;
              
              struct a2v {
                   float4 myVertex:POSITION;
              };
              struct v2f {
                   float4 pos:SV_POSITION;
              };
              
              v2f vert(a2v v){
                   v2f o;
                   o.pos=mul(UNITY_MATRIX_MVP,v.myVertex);
                   return o;
              }
              fixed4 frag(v2f In):SV_TARGET{
                   fixed4 col;
                   col=_LowerColor;
                   return col;
              }
              ENDCG
           }
    }
    FallBack "Diffuse"
}
```

###### 3.漫反射、高光反射、Alpha混合

```
Shader "Custom/myShaderVertFrag"{
     Properties{
         _Diffuse("myDiffuse",Color)=(1,1,1,1)
         _valveLight("myValve",Range(0,1))= 0.5     /*用于处理物体背面没有被照射到的frag，不至于太黑了*/
         _Specular("mySpecular",Color)=(1,1,1,1)   /*全部为1代表白色*/
         _Gloss("myGloss",Range(0,255))=0.5
         _Alpha("myAlpha",Range(0,1))=0.5    /*颜色的值在0-1之间，这和gloss是不同的*/
      }
      /*在shader中添加高光反射的功能，实质就是镜面反射的效果，高光发射与view的位置有关，不同view下的效果不同
        这与漫反射是不同的，漫反射只与vertex的位置有关，物体背面接收不到light故是暗的，这与view无关*/
      SubShader{
          Pass{
              TAGS {"LightMode"="ForwardBase"}
              
              Blend SrcAlpha OneMinusSrcAlpha   
              /*开启Alpha混合，从实际效果来看就是接收自定义颜色的Alpha的影响；没有这句话效果中就不会有透明效果*/

              CGPROGRAM
              #pragma vertex vert
              #pragma fragment frag
              #include "UnityCG.cginc"
              #include "Lighting.cginc"
              
              fixed4 _Diffuse;
              float _valveLight;
              fixed4 _Specular;
              float _Gloss;
              float _Alpha;
              
              struct a2v {
                   float4 myVertex:POSITION;
                   float3 myNormal:NORMAL;
              };
              struct v2f {
                   float4 pos:SV_POSITION;     /*SV_POSITION代表裁剪空间的坐标，和世界空间的坐标是不同的*/
                   float3 worldNormal:TEXCOORD0;
                   float4 worldVertex:TEXCOORD1;
              };
              v2f vert(a2v v){
                   v2f o;
                   o.pos =mul(UNITY_MATRIX_MVP,v.myVertex);
                   /*在计算世界空间中的法向量时，第一种即使在物体非等比例拉伸时依然可以得到正确的法向量。
                     由于物体非等比例拉伸，因此物体原来的法向量已经改变了。若用第二种，则无法得到此时物体真实的法向量*/
                   o.worldNormal = normalize(mul(v.myNormal,(float3x3)_World2Object));
                   //o.worldNormal = normalize(mul((float3x3)_Object2World,v.myNormal));
                   o.worldVertex=mul(_Object2World,v.myVertex);     /*_Object2World可用于顶点和向量都行，目标是转成世界空间*/
                   return o;
              }
              fixed4 frag(v2f In) :SV_TARGET{
                   float3 vDir = normalize(UnityWorldSpaceViewDir(In.worldVertex));
                   /*通过unity提供的方法直接得到view方向：从顶点到camera的位置的方向*/
                   float3 lDir= normalize(UnityWorldSpaceLightDir(In.worldVertex));
                   /*unity提供的方法需要引用库："UnityCG.cginc" ，得到光源到该顶点的方向.该方法可以计算各种类型的光源的方向
                     _WorldSpaceLightPos0.xyz 只是方向光的方向。如果是点光源则要用这个方法得到*/
                   
                  // float3 vDir = normalize(_WorldSpaceCameraPos-In.worldVertex);  
                    /*获取camera的pos时_WorldSpaceCameraPos不需要加后缀‘0’，这里只考虑主摄像机了*/
                  // float3 lDir = normalize(_WorldSpaceLightPos0.xyz);
                  
                   /*方法一：计算夹角的另外一种方式，计算量小，因此常用这种方式，效果与方法二都是一样的*/
                   float3 sDir=normalize(lDir+vDir);
                   fixed3 mySpecular=pow(saturate(dot(sDir,In.worldNormal)),_Gloss)*_Specular*_LightColor0;
                   
                   /*方法二：计算specular，在获取反射光线方向时使用reflect方法，计算量较大*/
//                   float3 rDir=reflect(-lDir,In.worldNormal);
//                   fixed3 mySpecular=pow(saturate(dot(vDir,rDir)),_Gloss)*_Specular*_LightColor0;

                   fixed3 myDiffuse= saturate(dot(lDir,In.worldNormal)*_valveLight+(1-_valveLight))*_Diffuse*_LightColor0;
                   fixed3 myAmbient=UNITY_LIGHTMODEL_AMBIENT.xyz;
                   
                   /*当场景中除了directional light外还有其他的point light时，则使用此方法.缺点在于此方法只对点光源有效，对方向光无效
                     而且只有在point light不是重要光的情况下才使用。
                     如果空间中只有一个点光源或者也需要对点光源进行逐像素的计算，
                     那么需要再添加一个pass对点光源进行计算，同时tags="ForwardAdd"，只有在forwardAdd下才会计算点光源。ForwardBase下默认只计算平行光
                     两者blend，得到真实的效果*/
                   float3 pointLightColor= Shade4PointLights(unity_4LightPosX0,unity_4LightPosY0,unity_4LightPosZ0,
                                                             unity_LightColor[0],unity_LightColor[1],unity_LightColor[2],unity_LightColor[3],
                                                             unity_4LightAtten0,
                                                             In.worldVertex,In.worldNormal);


                   fixed3 myCol=mySpecular+myDiffuse+myAmbient+pointLightColor;
                   return fixed4(myCol,_Alpha);
              }
              ENDCG
          }
      }
      
      /*首选的shader方案，优势在把颜色的计算放到frag中，效果更加的真实平滑*/
      SubShader{
          Pass{
             TAGS{ "LightMode"="ForwardBase"}  /*作用是阳光正常照射的效果，如果没有则会正面是黑的，背面是亮的，跟正常情况相反*/
             
             CGPROGRAM
             #pragma vertex vert
             #pragma fragment frag
             #include "UnityCG.cginc"   
             #include "Lighting.cginc"
                       
             fixed4 _Diffuse ; //float -32bit,half -16bit,fixed -12bit  : color [0,1]
             float _valveLight;
             float _Alpha;
             
             struct a2v {
                  float4 myVertex :POSITION;  /*告诉unity：把模型的顶点坐标存放到myVertex变量中*/
                  float3 myNormal :NORMAL;    /*告诉unity：把模型的法线向量存放到myNormal变量中*/
             };
             struct v2f{          /*struct v2f  中只是存储模型相对应在不同空间下的值即可*/
                  fixed4 pos : SV_POSITION;
                  float3 nDir: TEXCOORD0;    /*寄存器TEXCOORD0-7 共7个寄存器，用于暂时存储数据的*/
             };       /*特别注意：struct后面一定要加分号，不然会报无法理解的错*/
             v2f vert(a2v v){       
                  v2f o;
                  o.pos=mul(UNITY_MATRIX_MVP,v.myVertex);
                  o.nDir=normalize(mul((float3x3)_Object2World,v.myNormal));
                  return o;
             }    /*方法则不用加分号*/
             fixed4 frag(v2f In) :SV_TARGET {     /*shader的核心是SV_TARGET，所以在表示最后的显示结果时一定要加上此语义，不然GPU会不知道哪里去取这个数据*/    
                  float3 lDir=normalize(_WorldSpaceLightPos0.xyz);
                 // fixed3 myDiffuse=  saturate(dot(In.nDir,lDir))*_Diffuse*_LightColor0;
                  fixed3 myDiffuse = saturate(dot(In.nDir,lDir)*_valveLight+(1-_valveLight))*_Diffuse*_LightColor0;
                  fixed3 myAmbient=UNITY_LIGHTMODEL_AMBIENT.xyz;
                  
                  fixed3 myCol=myDiffuse+myAmbient;
                  return fixed4(myCol,_Alpha);
             }
             ENDCG
             }
      }
      
      /*颜色计算放在vert中，verts之间系统自动lerp，有锯齿情况
        锯齿：由于verts之间是系统自动的lerp得到，因此就会出现有些地方lerp的不大如意，显示出来的效果就是锯齿了
             而直接计算frag的color则是很有规律的平滑的  */
      SubShader{
         Pass{
             TAGS{ "LightMode"="ForwardBase"}  /*作用是阳光正常照射的效果，如果没有则会正面是黑的，背面是亮的，跟正常情况相反*/
             CGPROGRAM
             #pragma vertex vert
             #pragma fragment frag
             #include "UnityCG.cginc"   
             #include "Lighting.cginc"
                       
             fixed4 _Diffuse ; //float -32bit,half -16bit,fixed -9bit,包含1位符号位  : color [0,1]
             float _valveLight;
             float _Alpha;
             
             struct a2v {
                  float4 myVertex :POSITION;  /*告诉unity：把模型的顶点坐标存放到myVertex变量中*/
                  float3 myNormal :NORMAL;    /*告诉unity：把模型的法线向量存放到myNormal变量中*/
             };
             struct v2f {
                 fixed3 col :COLOR0;         /*告诉unity：col变量中存放的是颜色信息——注意当有多个color变量时需要加后缀，相当于一个数据寄存器*/
                 float4 pos :SV_POSITION;    /*告诉unity：pos变量中包含有该模型的裁剪空间的坐标*/
             };
             v2f  vert(a2v v){
                  /*此处方法的名称是“vert”,这个名称是定死的，跟上面“ #pragma vertex vert”中定义的vert保持一致，表示此方法是上面的vert的入口解释*/
                  v2f o;
                  /*方法命中的“v2f”只是返回值而已，“vert”则是上面入口vert的解释。故需要重新声明个v2f用于最后返回值*/
                  
                  o.pos = mul(UNITY_MATRIX_MVP,v.myVertex);  /*从模型空间转换到裁剪空间——裁剪空间之后就是屏幕空间*/
                  float3 nDir= normalize( mul((float3x3)_Object2World,v.myNormal));
                  /*unity默认提供的矩阵都是4x4的，所以此处需要把_Object2World强制转换为3x3的*/
                  float3 lDir=normalize(_WorldSpaceLightPos0.xyz);  
                  /*_WorldSpaceLightPos0——注意后缀0，该变量只适用于Direction light,因为方向光照射所有物体方向都是一样的，但point light就不行——其照射到同一的物体由于角度不同，光源的方向也是不同的
                    注意：_WorldSpaceLightPos0本身实际上4x4的，此处只需要得到xyz即可。
                   注：1.transform的缩放scale，rotate都可以用3次矩阵表示，但平移需要4次矩阵，故为了用一个矩阵就能完全表示物体的几何变换——全部用4次矩阵表示
                      2.以上两个都是得到归一化的向量，都转换到世界空间中进行计算*/
                  
                 // fixed3 myDiffuse=  saturate(dot(nDir,lDir))*_Diffuse*_LightColor0;
                  /*saturate(m)——使m的值保持在[0,1],实际所代表的是该顶点与light照射的方向的夹角，如果小于0代表该vert在light的背面，light无法直接照射到
                    “*Diffuse”代表因为阳光照射的缘故，自定义的颜色能够显示到什么程度
                    以及当前场景的light的影响“_LightColor0”——注意后面加0后缀，该变量需要引用库"Lighting.cginc"*/
                  
                  fixed3 myDiffuse=saturate(dot(nDir,lDir)*_valveLight+(1-_valveLight))*_Diffuse*_LightColor0;


                  fixed3 myAmbient = UNITY_LIGHTMODEL_AMBIENT.xyz;  /*获取环境光的颜色影响*/
                  o.col =  myAmbient+myDiffuse;
                  return o;
             }
             fixed4 frag(v2f In):SV_TARGET{      
             /*SV_TARGET是最终显示在屏幕上的颜色 ，frag的核心其实也就是颜色，输入源是v2f中得到的数据。verts之间lerp得到
              但此种模式的缺点在于将颜色的计算全部放在顶点处，verts之间则使用lerp得到。因此这样显示的结果就显得有点锯齿。
              为了避免，应该把颜色的计算全放在frag中，仔细的计算出frag的颜色*/
                 return fixed4(In.col,_Alpha);
             }
             ENDCG
         }
      }
      
      /*场景中的对象能够产生shadow需要两个条件：1.light的shadowType设置为soft/hard shadow   2.物体所使用的shader支持shadow
        因此通常都需要在shader中定义shadow相关的功能。
        但有个简便的方法：
        1.选择FallBack中的备用shader，系统会自动的调用SubShader，对于SubShader中没有声明的内容则自动的调用FallBack
        所以备用的shader中选择能够产生shadow的。一般的都可以，如“Diffuse”,"Specular"等
        2.在SubShader中添加另一个Pass通道：Pass{TAGS{"LightMode"="ShadowCaster"}}  ,即可，其他的都不用加。这种形式就不需要FallBack中对阴影的相关定义了
        */
      FallBack "Diffuse"
}
```



###### 4.UV动画、水波振荡纹理、高斯模糊：

```
Shader "Custom/myShaderUV" {
    Properties{
          _MainTex("MainTex",2D)="White"{}
          _SecondTex("SecondTex",2D)="White"{}
          _Speed("mySpeed",Range(1,5))=1
          _Extent("myExtent",Range(0,1))=0.1 /*UV水波的振幅*/
          _Radius("MyRadius",Range(0,0.5))=0.1 /*中心UV水波中只有固定的radius内才产生水波*/
    }
    SubShader{
        Pass{
             CGPROGRAM
             #pragma vertex vert
             #pragma fragment frag
             /*偏导数ddx/ddy需要在shader model 3.0 才能正确运行，因此需要添加*/
             #pragma target 3.0     
             #include "UnityCG.cginc"

             sampler2D  _MainTex;
             sampler2D _SecondTex;
             float4 _MainTex_ST;
             float4 _SecondTex_ST;
             float _Speed;
             float _Extent;
             float _Radius;
             uniform float4 _hitWorldPos; /*射线点击后得到的世界空间下的点*/
             /*这两个变量在UnityShaderVariable.cginc中自带了，所以不需要再次声明了，直接使用即可
               注意：unity_LightmapST 中“ST”没有下划线。跟之前UV的写法不同*/
             // sampler2D unity_Lightmap;
             // float4 unity_LightmapST;
             
             /*当有多套纹理采样时，简单的图形如plane等则无论纹理之间如何定顺序都没事。随便最终frag中选择哪个纹理输出都没事。总之这种情况下随便
             但是如果是sphere或其他特殊形状的物体，特征是有多个面，每个面都会取一套uv纹理以不同的uv分布时，TEXCOORD0_mainTex,而且采样到的纹理最好作为最终frag中颜色输出
                                                TECCOORD1_Lightmap，如果该shader需要接收lightmap，则最好是TEXCOORD1
                                                TEXCOORD2-7,作为其他纹理的寄存器，一样可以采样到，但是在特殊的形状下显示的样子会与TEXCOORD0中不同
                         总结：1.特殊形状下正常显示，TEXCOORD0为最终frag输出的颜色，如果启用lightmap，则TEXCOORD1存储lightmap
                              2.多套纹理采样都是可以的，只是对于特殊形状下的显示有些拼接歪曲。
             */
             struct a2v {
                  float4 myVertex:POSITION;
                  float4 myMainTex:TEXCOORD0;
                  float4 myUnityLightmap:TEXCOORD1;
                  float4 mySecondTex:TEXCOORD2;
             
             };
             struct v2f {
                  float4 pos:SV_POSITION;
                  float2 uv_MainTex:TEXCOORD3;
                  float2 uv_unity_Lightmap:TEXCOORD4;
                  float2 uv_SecondTex:TEXCOORD5;  
                  float4 objPos:TEXCOORD6;    /*用于在frag中计算物体世界空间下Z轴的变化，用于UV模糊效果*/
             };
             
             v2f vert (a2v v){
                   v2f o;
                   o.pos=mul(UNITY_MATRIX_MVP,v.myVertex);
                   o.objPos=v.myVertex;
                   /*针对多个纹理的采样。_MainTex_ST指本shader支持对纹理动态的修改。如果直接的o.uv_MainTex=v.myMainTex，则不支持inpsector中修改Tiling得到不同的采样*/
//                   o.uv_MainTex=_MainTex_ST.xy *v.myMainTex +_MainTex_ST.zw;
//                   o.uv_unity_Lightmap=unity_LightmapST.xy *v.myUnityLightmap +unity_LightmapST.zw;
//                   o.uv_SecondTex=_SecondTex_ST.xy * v.mySecondTex +_SecondTex_ST.zw;
                   

                   /*UV动画：核心在于改变UV偏移，随着时间逐渐变换
                     1."_Speed"指代变换的速度，可以用CSharp脚本传值，也可自由设定
                     2._MainTex_ST.xy是所取到的UV的界限大小，因此在inpsector中Tiling需要是小数类型，你懂的*/
//                   int rowNum= int((_Time.y/_Speed) /(1/_MainTex_ST.x));
//                   int columnNum=int((_Time.y/_Speed) %(1/_MainTex_ST.y));
//                   o.uv_MainTex= _MainTex_ST.xy*v.myMainTex+ float2(columnNum *_MainTex_ST.y,rowNum * _MainTex_ST.x);
                   
                   
                   /************小河流水缓缓流动******************/
                   /*小河流水是使纹理中的像素进行拉伸或缩小所达到的效果，跟动画中改变offset是完全不同的两回事。这里始终所采样到的都是同一个offset下的纹理。
                     最后呈现的结果之所以会出现不同offset下的，是因为某些像素拉伸后超出了原有的界限，根据repeat特性所以最后得到offset后的图像*/
                    o.uv_MainTex=_MainTex_ST.xy *v.myMainTex +_MainTex_ST.zw ;
                    
                    /*对采样到的纹理进行拉伸处理，从而得到水流的效果。因为像素被拉伸了所以仿佛看到内部纹理在流动的效果
                      因为像素拉伸后，如果拉伸的幅度过大，那么像素与像素之间差距过大就会出现花屏的情况。如此也证明流水只是拉伸了像素而已，并不是改变了offset
                      像素拉伸后，多多少少会超过原来的0，1边界，因此超出的部分由于纹理的“repeat”特性，所以可以自动补足*/


                   return o;
             }
             fixed4 frag (v2f In) :SV_TARGET{
                   fixed4 col;
                   
                    /************中心UV水波***************/
                   /*对于UV的计算完全可以放在vert中，但是如果想要根据不同的UV来实现不同的颜色则需要把uv计算放在frag中，这样更细致。
                     放在vert中计算后传值到frag也可以，只是顶点数量有限，得到的效果不会很细滑。
                     在边界上的某些uv点由于拉伸，超出了边界，因此最后的图像会有明显的拼接特点。
                     要解决拼接的点：1.美工支持，把图片边界改成无缝衔接的   
                                   2.和中心水波类似。中心水波的特点是采样始终不变，改变的是像素的分布来达到拉伸的效果。因此也就不会有拼接的问题了*/
                   /*sin(kx+b)：k值越大，则周期内拉伸的次数越频繁，看到的图像扭曲越明显
                     k:表现为相邻之间像素的间距变大了，但不可能控制最终振幅程度——由_Extent决定。
                       最终的结果是拉伸的幅度，这个完全由_Extent决定。像素拉伸幅度不能太大，否则图像不正常
                       但是因变量——顶点uv到中心UV的距离：
                       假设A到中心UV为1，添加K后，到中心UV为k，那么最终A点的拉伸程度为sin(k)，所以最终A点UV为原uv+原uv *sin(k).由于sin函数，所以最终拉伸程度不一定>sin(1)
                       原来距离中心UV为1/k的B点，添加K后到中心uv为1，最终拉伸程度为sin(1).
                       因此距离更近的点也呈现出了原来只有距离较远的点才能有的变化。呈现在图像上则是周期变短，同样条件下有更多的变化
                     k的作用：1.加深水波内部的剧烈程度  2.如下情况中添加白色圆环，但是圆环的区域太大了，希望圆环能够小一些变化多一些——此时则需要K值发生作用了*/
                   float uv_Scale= _Extent * sin(-length(In.uv_MainTex-float2(0.5,0.5))*20+_Time.y);
                   /*不同的拉伸幅度，因此是“*”*/
                   In.uv_MainTex+=uv_Scale;
                   /*为了添加白色圆环引人注目点：saturate(uv_Scale)*fixed4(1,1,1,1)*1000
                     因为saturate范围在0-1之间，因此得到的圆环也会有从0-1的变化，“*1000”是为了把原来值只有0.001的也变成1
                     这样呈现在图像上则大部分区域都是1，自然也就亮了*/
                   col=tex2D(_MainTex,In.uv_MainTex)+saturate(uv_Scale)*fixed4(1,1,1,1)*1000;
                   
                   /****************横向水波********************/
                   /*由于UV水波的核心是像素的拉伸，因此如果只是向一个方向做拉伸，那么拉伸幅度不变，那么图像在那一瞬间会静止；如果拉伸幅度一直增大，那么最后图像会变形
                     因此拉伸的幅度一定是有增大也有减小的。由于反复的变化，呈现在图像上就是往一个方向上的流动了
                     其实和平移中的区别：平移中一个点上下波动，最后呈现向一个方向的波动
                     总之：中心和横向两者都是采用振幅上下波动来达到流动的效果，只是中心的依据是中心点，横向的则是直线X=0或Y=0
                     不管是哪一种，通过因变量前加“-”来改变流动方向。但是不管哪种都是定向的往一个方向流动*/
//                   float uv_Scale_x= _Extent* sin(length(In.uv_MainTex.x)*20 +_Time.y);
//                   In.uv_MainTex.x+=In.uv_MainTex.x *uv_Scale_x;
//                   col=tex2D(_MainTex,In.uv_MainTex)+saturate(uv_Scale_x)*fixed4(1,1,1,1)*1000;

                   
                   /*************中心UV水波进化——只有固定Radius内才产生********************/
                   /*d代表从中心到周围，拉伸的比重越来越轻，中间对于波动影响最敏感，外围慢慢比较淡定
                     这样得到的结果是从中心到周围拉伸幅度越来越小
                     如果要实现点击后水波由中心扩散，扩散后中心的拉伸越来越小：
                                首先肯定是根据d权重确定中心到周围幅度越来越小，然后逐渐的减小_Extent的值，这样中心的拉伸也会减小，
                                如果_Extent始终不变，那么中心始终会不断的产生水波，然后扩散到周围，实际上第一个产生水波时能量最强，之后会逐渐减小，所以需要降低_Extent的幅度*/
//                   float d = saturate(1-length(In.uv_MainTex - float2(0.5,0.5))/_Radius) ;
//                   float uv_Scale= _Extent*sin(-length(In.uv_MainTex-float2(0.5,0.5))*50+_Time.y)*d;
//                   In.uv_MainTex+=In.uv_MainTex *uv_Scale;
//                   col=tex2D(_MainTex,In.uv_MainTex) +saturate(uv_Scale) *fixed4(1,1,1,1)*1000;
                   
                   
                   /******************射线点击后产生水波*******************/
                   /*核心在于获取射线点击得到的点的UV坐标。射线点击的点通过Csharp传值得到.这种情况下需要知道模型空间下物体每一个面的各个顶点坐标值，用于计算UV
                     UV始终计算的都是二维平面下的，当物体有多个面时每一个面都会得到一个纹理。所以只能取顶点坐标的其中两个组成所在的平面*/
               //    float4 hitObjPos=mul(_World2Object,_hitWorldPos);
                   /*转换空间之后得到该点在模型空间上的坐标值。
                     由于UV通常都是以左下角为(0,0).所以需要知道图像UV坐标为(0,0)所对应的vertex数值。这可以通过Csharp获取meshFilter得到模型顶点
                     本例中由于是plane，transorm都是初始值，所以取XZ平面。
                     本例中UV坐标为(0,0)所对应的vertex为(5,5),因此是如下的计算方式。plane大小是10x10的*/
                 //  float2 center = float2(abs(hitObjPos.x-5)/10,abs(hitObjPos.z-5)/10);
                   /*水波中心点和权重等都是以center为依据的*/
//                   float d = saturate(1-length(In.uv_MainTex - center)/_Radius) ;
//                   float uv_Scale= _Extent*sin(-length(In.uv_MainTex-center)*50+_Time.y)*d;
//                   In.uv_MainTex+=In.uv_MainTex *uv_Scale;
//                   col=tex2D(_MainTex,In.uv_MainTex) +saturate(uv_Scale) *fixed4(1,1,1,1)*1000;
                  
                  
                  /****************多次采样后得到UV模糊的效果*****************/
                  /*首先对于UV模糊是一定可以通过多次采样，每一次采样的offset有些许偏差，最终取多次采样后得到的col的平均值，即为模糊效果*/
                  /*使用偏导数来达到模糊效果,dds(a)表示基于a的变化率，“*10”是因为变化率过小的话是难以看到模糊的效果，这样增大变化率。
                    dds函数会自动的根据tex2D,tex1D得到float2,float,挺方便的。
                    对于tex2D,如果使用的是float x_ddx=ddx(In.uv_MainTex.x),那么实际上转换下就是float2(x_ddx,x_ddx) ——————你懂我为什么要写这句话的。*/
                  //float2 dsdx=ddx(In.uv_MainTex.x)*10;
                 // float2 dsdy=ddy(In.uv_MainTex.y)*10;
                  /*tex2D的重载函数，专门用于模糊效果的制作。后面两个参数分别指代在U,V上的一个变化情况——其本质还是在U,V方向上以不同offset做采样
                    所以当V对应的为0时，表示只在U方向上多次采样得到模糊效果*/
                  //col= tex2D(_MainTex,In.uv_MainTex,dsdx,0);
                  
                  /*本plane当绕Z或X轴旋转模型时，模型上各个顶点在世界空间上Y轴的坐标会变化，可以基于这个变化实现模糊效果*/
                  /*旋转模型时，模型本身各个顶点的坐标在模型空间中是不变的，因此如果要实现因为旋转而导致的模糊效果就需要转到世界空间上*/
                 // float4 objWorldPos=mul(_Object2World,In.objPos);
                 // float2 dsdx=ddx(objWorldPos.y)*10;
                  /*只对U基于Y坐标的变化率来采样。总的来讲就是基于某一数值的变化率来对U,V方向上多次采样。如果没有其他数值变化则基于自身的U,V变化率来做采样
                    参数值为0时代表，相对于的U,V方向上多次采样的offset为0，这样自然得到的都是一样的结果，最后取平均col也和原来的一样也就没有模糊效果了*/
                //  col=tex2D(_MainTex,In.uv_MainTex,dsdx,0);
                  
                  
                  /********************为shader添加lightmap支持**********************/
                  /*lightmap的计算，不要问我为什么最后是“*”。因为我也不知道，反正就是乘*/
                  /*启用光照贴图的原因：
                    制作好的lightmap，要使用需要shader支持，如果shader不支持，则关掉light场景会变黑。
                    关掉light场景仍然亮的需要条件：1.有场景bake好的lightmap   2.物体本身的shader支持lightmap
                    这个lightmap不需要手动指定，系统会自动的查找该物体对象所需要的lightmap自动填充*/
                 //  float3 lightmap= DecodeLightmap(UNITY_SAMPLE_TEX2D(unity_Lightmap,In.uv_unity_Lightmap));
                     /*直接将采样得到的col与光照系数相乘即可。“a”不管，仍然为col原本的Alpha*/
                  // col.xyz *=lightmap*2;
                   /*支持光照贴图后显示如果暗淡，可以“*2”，这样就亮了，不要问我为什么*/

                   return col;
             }
             ENDCG
             
        }
    }
    FallBack "Diffuse"
}
```

**PS**：获取鼠标点击后的坐标在模型空间中的UV值(仅对结构简单的物体可用，如Plane等)

```c#
GameObject go = GameObject.CreatePrimitive(PrimitiveType.Plane);
go.name = "PlaneGO";
MeshFilter mf = go.GetComponent<MeshFilter>();
//获取顶点数据
Debug.Log(mf.sharedMesh.vertexCount + "  count ");
Vector3[] arr = mf.sharedMesh.vertices;
foreach(var temp in arr)
{
    //Debug.Log(temp);
}
Debug.Log(arr.Max(v => v.x) + "  v max  ");
//获取mesh的uv数据
Vector2[] uvArr = mf.sharedMesh.uv;
Debug.Log(uvArr.Length + "   length  ");
foreach(var temp in uvArr)
{
    //Debug.Log(temp);
}
//通过以上数据的遍历可知，UV坐标为(0, 0)对应的顶点坐标为arr[0],
//UV坐标为(1, 1)对应的UV是"uvArr[uvArr.Length - 1]",顶点坐标对应的是"arr[arr.Length - 1]"
//如此即可知道模型空间中X,Y的范围区间，UV为(0, 0)对应的顶点的模型空间坐标值
Debug.Log(arr[0] + "   @@@   " + uvArr[0]);
Debug.Log(arr[arr.Length - 1] + "     !!!!!   " + uvArr[uvArr.Length - 1]);

//通过MeshFilter获取到的顶点坐标实际上是模型空间中该顶点的坐标a
//通过将鼠标点击得到的坐标使用Unity自带的"float4  hitObjPos=mul(_World2Object,_hitWorldPos)"转换
//得到该点击的点在模型空间中的坐标b
//把UV坐标为(0, 0)对应的坐标a与坐标b进行比对，借助X,Y的范围区间，即可知道鼠标点击的点对应的UV值
```

###### 5.广告牌“彩带流光”、水波(基于顶点坐标实现)、光晕

```
Shader "Custom/myShaderScreenPos" {
    /*功能：1.基于屏幕坐标制作的流光效果  2.水波效果  3.边缘泛光及光晕的效果*/
    
    Properties{
        _Edge("MyEdge",Range(-5,5))=0  /*颜色缓冲带中分界线*/
        _Area("Area",Range(-1,1))=0
        _EdgeColor("EdgeColor",Color)=(1,1,1,1) /*边缘泛光中边缘的颜色*/
        _Alpha("MyAlpha",Range(0,1))=1   /*自主设定Alpha*/
        _Gloss("MyGloss",Range(-10,10))=1  /*结合pow，使某个颜色呈现衰减*/
        _Radius("MyRadius",Range(0,0.5))=0.2  /*缓冲带的半径范围*/
    }
    SubShader{
       Pass{
           TAGS{"LightMode"="ForwardBase"}
           /*表示会接收设定的Alpha的影响,只有alpha。如果是“Blend one one”则代表整体颜色都会受到影响*/
          // Blend SrcAlpha OneMinusSrcAlpha   
           
           CGPROGRAM
           #pragma vertex vert
           #pragma fragment frag
           #include "UnityCG.cginc"
           
           float _Edge;
           float _Area;
           fixed4 _EdgeColor;
           float _Alpha;
           float _Gloss;
           float _Radius;
           uniform float loopEdge;

           struct a2v{
               float4 myVertex:POSITION;
               float3 myNormal:NORMAL;
           };
           struct v2f{
               float4 pos:SV_POSITION;
               float4 objPos:TEXCOORD0;
               float4 worldPos:TEXCOORD1;
               float2 screenPos:TEXCOORD2;
               float3 worldNormal:TEXCOORD3;
           };
          
           v2f vert(a2v v){
                v2f o;
                /*制作中心水波效果，如果模型顶点较少，则不适合做周期太小的改变."-"负号的作用在于是向里还是向外扩散的*/
                v.myVertex.y+=0.5*cos(-length(v.myVertex.xz)*2+_Time.y);

                /*制作波浪定向扩展效果——在X轴上扩展*/
               // v.myVertex.y+=0.5*cos(-v.myVertex.x *2+_Time.y);
//                v.myVertex.y+=0.1*cos(-v.myVertex.x *5+_Time.y);
               
               /*制作斜向水波扩展——直线Z=x,(x,z)到直线Z=x的距离是length(float2(1,1))*(z-x)。但这里length(float2(1,1))只是表示一个幅度，所以直接省略了
                 波涛汹涌的效果核心在于：Z=X和Z=-X重合.可以自主设定周期、振幅和初相*/
//                v.myVertex.y+=0.5*cos(2*(v.myVertex.z-v.myVertex.x)+_Time.y);
//                v.myVertex.y+=0.5*cos((v.myVertex.z+v.myVertex.x)+_Time.w);                

                o.pos=mul(UNITY_MATRIX_MVP,v.myVertex);
                o.worldNormal=mul(v.myNormal,(float3x3)_World2Object);
                o.worldPos=mul(_Object2World,v.myVertex);
                o.objPos=v.myVertex;
               
                /*根据投影矩阵，由裁剪坐标得到屏幕坐标的值。由于屏幕坐标的特性，其值始终在(-1,1)之间*/
                o.screenPos=float2(o.pos.x/o.pos.w,o.pos.y/o.pos.w);
                return o;
           }
           
           fixed4 frag(v2f In):SV_TARGET{
                fixed3 col;
                
                /*制作水波由中心向外扩散的颜色,要使颜色是变化，模仿那种波光粼粼的效果，则颜色的设定也需要是变化的*/
                col=fixed3(1,0,sin(length(In.objPos.y))*0.5+0.5);
                
                 /*制作边缘发光的效果*/
//                float3 vDir= normalize(UnityWorldSpaceViewDir(In.worldPos));
//                float3 nDir= normalize(In.worldNormal);
                /*saturate(dot(vDir,nDir))得到的值从中心为1，向两边为0，背面为负值。若以此为alpha，则需要颠倒，故为1-saturate(dot(vDir,nDir))
                  saturate(dot(vDir,nDir))的特点是：向着light的是亮的，到两边开始变黑，背面完全是黑的。常用的Lambert则是考虑到背面所以做了改进。*/
//                float result=1-saturate(dot(vDir,nDir));
//                col =result *_EdgeColor;
//                return fixed4(col,result);
                
                /*if...else...用起来更方便，常用这种*/
//                if(In.objPos.x>_Edge)
//                    col=fixed4(1,0,0,1);
//                else
//                    col=fixed4(0,1,0,1);
                /*故弄玄虚的类型，核心是取到一个始终只有-1，1两种值的数据，然后saturate得到0和1两种值，如此作为分界线
                  GPU对于if...else..语句，当有多个或者嵌套时支持不是很好，所以这是if..else..一种替代方式。
                  虽然不常用，但仍然需要记住*/
//                float rate=saturate((In.objPos.x-_Edge)/abs(In.objPos.x-_Edge));
//                col=lerp(fixed4(0, 1, 0, 1), fixed4(1,0,0,1), rate);
                

                /*在不用if/else的情况下，添加分界线的缓冲带*/
                /*(In.objPos.x-_Edge)/_Radius得到的值从-1.x -> -0.x -> 0.x -> 1.x 的范围变化。使用lambert模式是为了把-1.x处理成功
                  但((In.objPos.x-_Edge)/_Radius)*0.5+0.5 处理得到的值并不是真实的数据，整体数值都增大了一点。
                  但是对于之前“-1.x -> -0.x”的范围值则好多了，不是直接处理成0.因此中心点左边的半径内部分也有了变化，而不是直接为0。对于超出-1的，则直接处理成0.*/
//               float rate2= saturate(((In.objPos.x-_Edge)/_Radius)*0.5+0.5);
//               col=lerp(fixed4(1,0,0,1),fixed4(0,1,0,1),rate2);


//                if(In.worldPos.x<-3)
//                    col=fixed4(0,0,1,1);
                /* 使用屏幕空间坐标制作流光效果 */
                /*CG语言中直接的“a<x<b”是不起作用的，必须要拆开写。由于cos的特性，效果是ping pong的*/
//                if(-_CosTime.w<In.screenPos.x && In.screenPos.x <-_CosTime.w+0.1)
//                    col=fixed4(1,0,1,1);
                   
                /*制作流光效果，由于CG中的unity_DeltaTime不会用，因此采用从外部传值的方法，限定loopEdge，达到loop而不是ping pong的效果*/     
//                if(loopEdge<In.screenPos.x && In.screenPos.x<loopEdge+0.1)
//                    col=fixed4(1,0,1,1);
                /*由于开启了Alpha，所以最后结果会受到设定的alpha的影响*/
                return fixed4(col,1);    
           }
           ENDCG
       }
       
       /*对于光晕效果，使用两个pass来blend.这个pass的核心在于在物体表面制造一层晕。上个pass的核心在于物体边缘发光，中心镂空。如果只是做个晕，那本pass足够了*/
    //   Pass{
           /*第一个参数表示当前pass所采用的，第二个参数代表之前渲染所得到的颜色，如果之前没有pass则表示环境的影响*/
//           Blend one one
//           CGPROGRAM
//           #pragma vertex vert
//           #pragma fragment frag
//           #include "UnityCG.cginc"
//           
//          
//           float _Gloss;
//           fixed4 _EdgeColor;
//           struct a2v{
//               float4 myVertex:POSITION;
//               float3 myNormal:NORMAL;
//           };
//           struct v2f{
//               float4 pos:SV_POSITION;
//               float4 worldPos:TEXCOORD1;
//               float3 worldNormal:TEXCOORD3;
//           };
//            v2f vert(a2v v){
//                v2f o;
//                o.worldPos=mul(_Object2World,v.myVertex);
//                o.worldNormal=mul(v.myNormal,(float3x3)_World2Object);
                /*照顾到边缘泛光，达到光晕的效果。故将顶点在normal方向上做平移*/
//                v.myVertex.xyz+=normalize(v.myNormal)*0.2;
//                o.pos=mul(UNITY_MATRIX_MVP,v.myVertex);
//                return o;
//                }
//           fixed4 frag(v2f In):SV_TARGET{
//                fixed3 col;
                /*注意：光晕的核心是以view的方向为准的，不受光线的影响，受view不同方向的影响。在任何角度看到的都是有一圈晕*/
//                float3 vDir=normalize(UnityWorldSpaceViewDir(In.worldPos));
//                float3 nDir=normalize(In.worldNormal);
                /*由于vertex沿normal方向平移后，采用从内到外正常衰减的方式，边缘会变模糊，中间清晰。之后两者叠加*/
//                float result2=saturate(dot(vDir,nDir));
//           
//                col=pow(result2,_Gloss)*_EdgeColor;
//           
//                return fixed4(col,result2);
//           }
//           ENDCG
//       }
       
    }
    FallBack "Diffuse"
}
```

###### 6.针对NormalMap纹理的采样：

```
Shader "Custom/myShaderNormalMap" {
    Properties {
        _NormalMap("normal map",2D)="White"{}
        _MainTex("mainTex",2D)="White"{}
    }
    SubShader {
        /*正常的MainTex的采样，normalDir ,lDir都是正常的*/
        Pass{
           TAGS{"LightMode"="ForwardAdd"}
           CGPROGRAM
           #pragma vertex vert
           #pragma fragment frag
           #include "UnityCG.cginc"
           #include "Lighting.cginc"
           
           sampler2D _MainTex;
           struct v2f {
                float2 uv_MainTex:TEXCOORD0;
                float4 pos:SV_POSITION;
                float3 myNormal:TEXCOORD1;
                float4 myVertex:TEXCOORD2;
           };
           v2f vert(appdata_base v){
                v2f o;
                o.uv_MainTex=v.texcoord;
                o.pos=mul(UNITY_MATRIX_MVP,v.vertex);
                o.myNormal=v.normal;
                o.myVertex=v.vertex;
                return o;
           }
           fixed4 frag (v2f In):SV_TARGET{
                fixed4 col;
                col=tex2D(_MainTex,In.uv_MainTex);
                float3 nDir=normalize(mul(_Object2World,In.myNormal));
                float3 lDir=normalize(UnityWorldSpaceLightDir(mul(_Object2World,In.myVertex)));
                col*=(dot(nDir,lDir)*0.5+0.5)*_LightColor0;
                return col;
           }
           ENDCG
        }
        
        /*对于normalMap的采样需要单独的建立一个pass通道，因为对于normalMap其所使用的lDir,nDir都和平常所使用的不一样，所以需要单独处理*/
        Pass{
        TAGS{"LightMode"="ForwardAdd"}
        Blend one one
        CGPROGRAM
        #pragma vertex vert
        #pragma fragment frag
        #include "UnityCG.cginc"
        #include "Lighting.cginc"
        
        sampler2D _NormalMap;

        struct v2f {
            float4 pos:SV_POSITION;
            float2 uv_NormalMap:TEXCOORD1;
            float4 objPos:TEXCOORD3;
            float3 lDir:TEXCOORD4;
        };
        
        v2f vert(appdata_full v){
            v2f o;
            o.pos=mul(UNITY_MATRIX_MVP,v.vertex);
            o.uv_NormalMap=v.texcoord;
            o.objPos=v.vertex;
            /*将光照方向转到纹理空间，这个lDir是针对normalMap的方向，正常的计算是不用这个lDir的*/
            TANGENT_SPACE_ROTATION;
            o.lDir=normalize(mul(rotation,UnityWorldSpaceLightDir(mul(_Object2World,v.vertex))));
            return o;
        }
        fixed4 frag(v2f In):SV_TARGET{
            fixed4 col;
            /*使用normalMap后真正的发现是发现贴图UnpackNormal得到的normal方向，而不是模型空间中物体原本的normal了*/
            float3  nDir = normalize(UnpackNormal(tex2D(_NormalMap,In.uv_NormalMap)));
            /*这里不需要用lambert那个方式处理，没有光的地方直接置为0，区分度大点*/
            col=  saturate(dot(In.lDir,nDir))*_LightColor0;
            return col;
        }
        ENDCG
        }
    }
    FallBack "Diffuse"
}
```

###### 7.自定义变换矩阵Matrix在Shader中的使用：

```
Shader "Custom/myShaderMatrix" {
    /*功能：主要是基于object和world空间下的中心点所做的变换，包括平移和旋转以及矩阵常用的一些注意事项*/
    
    Properties{
        _Vector("MyVector",vector)=(0,0,0,1)
        _Range("MyRange",Range(0,9))=3
        _XPivot("XPivot",Range(-10,10))=0  /*用于自主设定pivot，但只在以世界空间坐标为基准的情况*/
    }
    
    SubShader{
    
        Pass{
           
           CGPROGRAM
           #pragma vertex vert
           #pragma fragment frag
           #include "UnityCG.cginc"
           
           uniform float4 pivot; /*uniform 类型的定义时是区分大小写，总之关键字没高亮的就表示写的有问题*/
           float4 _Vector;
           float _Range;
           float _XPivot;
           
           struct a2v {
                float4 myVertex:POSITION;
           };
           struct v2f {
                float4 pos:SV_POSITION;
                fixed4 col:COLOR;
           };
           
            /*在CG语言中Matrix的赋值必须遵循：
                  1.每一个元素都要赋值，不能跟CSharp中只给非零元素赋值。所以通常使用向量形式直接给每一行一次性赋值
                  2.在赋值时一定要加上float4()的形式，不能只是直接写数值，前面要加float4才可以
              cos参数里只有_Time.w,表示所有顶点只是随着time而旋转，与顶点自身到pivot的距离无关，因此物体本身的形状不会改变
              注意：_Time需要运行之后才能看到随着时间的旋转
            */
           float4x4 RotateMatrix2(){
                float4x4 myMatrix;
                myMatrix[0]=float4(1,0,0,0);
                
                /*由于_Time包含有多倍数time的效果，因此可以看到快速旋转的效果，最高为3倍*/
                myMatrix[1]=float4(0,cos(_Time.w),-sin(_Time.w),0);
                myMatrix[2]=float4(0,sin(_Time.w),cos(_Time.w),0);
                
                /*是上面的sin,cos的简写，已经综合到_SinTime、_CosTime中，但缺点在于无法看到多倍数时间的效果，因为w最大就是1倍time*/
//                myMatrix[1]=float4(0,_CosTime.w,_SinTime.w,0);
//                myMatrix[2]=float4(0,-_SinTime.w,_CosTime.w,0);
                
                myMatrix[3]=float4(0,0,0,1);
                return myMatrix;
           }
           
           float4x4 RotateMatrix(a2v v){
                float4x4 RM;
                float4 worldPos=mul(_Object2World,v.myVertex);
                float angle=length(worldPos.xz);
                RM[0]=float4(cos(angle),0,-sin(angle),0);
                RM[1]=float4(0,1,0,0);
                RM[2]=float4(sin(angle),0,cos(angle),0);
                RM[3]=float4(0,0,0,1);
                return RM;
           }
           v2f vert(a2v v){
                v2f o;
                /*总结：当以模型自身的顶点到中心点距离来改变顶点位置——分为两种情况：
                       1.如果以模型自身的中心点为基准，那么只有一个这种类型的物体时是正常显示的。当有>=2时则系统自动校准为以世界坐标中的零点为基准。
                         表现形式则为：多个物体只是扩展了显示面积，中心点为世界空间中的零点，而非单个自身的零点
                       2.如果一开始就以世界空间中的零点为基准做变化，则不论一个还是多个相同物体其效果都是一样的
                         且最后使用的转化Matrix依情况改变，不一定是MVP
                       3.由于出现自动转换pivot的情况，因此根据需求适当选择。
                         并且以世界空间为基准的情况可以自主设定pivot，不一定是零点，但以模型空间则无法自由设定pivot，系统会自动选取模型的中心点为基准*/
                                    
                   
                /*以世界空间的零点为标准来旋转物体，依据和本体都是世界坐标，所以最后为UNITY_MATRIX_VP，而不是MVP.
                  这样的结果是正常的，始终以世界坐标中的零点为基准，不论一个还是多个相同物体*/
//                float4 worldPos=mul(_Object2World,v.myVertex);
//                float4 RotatePos=mul(RotateMatrix(v),worldPos);
//                o.pos=mul(UNITY_MATRIX_VP,RotatePos);  
//                o.col=fixed4(1,0,sin(length(v.myVertex.xz))*0.5+0.5,1);


                /*以模型的零点作为pivot平移，改变vert位置
                  不论哪种方式，使用v.myVertex与v.myVertex.xz，前者因为包含了Y和w坐标，所以整体显示出来的画面会缓和一些，后者则更直接。通常选用后者*/
//                float d=(length(v.myVertex.xz)<3)?(3-length(v.myVertex.xz)+v.myVertex.y):v.myVertex.y;
//                float4 transformPos=float4(v.myVertex.x,d,v.myVertex.z,v.myVertex.w);
//                o.pos=mul(UNITY_MATRIX_MVP,transformPos);
//                o.col=fixed4(1,0,sin(length(v.myVertex.xz))*0.5+0.5,1);


               /*以世界坐标中的零点作为pivot平移，根据到此的距离来更改vert位置.
                 优势是不论是有多少个同样的物体，始终依据的pivot都是世界坐标，因此多个物体也不过是扩展了显示面积而已*/
                float4 worldPos=mul(_Object2World,v.myVertex);
                float4 transformPos;
                /*1.“3”是代表需要改变的顶点的区域，半径为3以内的所有顶点.要注意：plane默认是10x10的大小.
                  2.只取vert的xz分量是为了避免因为在Y轴上拖动物体而造成区域减少的情况。即不论如何拖动物体，半径3以内的顶点都会改变 */
//                if(length(worldPos.xz)<3)  
//                      transformPos=float4(v.myVertex.x,3-length(worldPos.xz)+v.myVertex.y,v.myVertex.z,v.myVertex.w);
//                else
//                      transformPos=v.myVertex;
                
                /*平移矩阵，要注意：矩阵是写在左边的，因此要注意向量的写法
                  改变Pivot可以得到不同的效果*/
                float d=(length(worldPos.xz-float2(_XPivot,0))<3)?(3-length(worldPos.xz-float2(_XPivot,0))):0;
                float4x4 posMatrix={
                         float4(1,0,0,0), float4(0,1,0,d),float4(0,0,1,0),float4(0,0,0,1)
                };
                transformPos=mul(posMatrix,v.myVertex);
                
                /*SV_POSITION语义代表最终显示出来的顶点的信息，其与vert本身的信息是一一对应的，此处用的是transformPos*/
                o.pos=mul(UNITY_MATRIX_MVP,transformPos);
                o.col=fixed4(1,0,sin(length(worldPos.xz))*0.5+0.5,1);
                
                
                /*绕Y轴旋转,注意参数写法*/
//                float4x4 YRotateMatrix={
//                       float4(_CosTime.w,0,_SinTime.w,0),float4(0,1,0,0),float4(-_SinTime.w,0,_CosTime.w,0),float4(0,0,0,1)
//                };
//                float4 RotatePos=mul(YRotateMatrix,v.myVertex);
//                o.pos=mul(UNITY_MATRIX_MVP,RotatePos);
//                o.col=fixed4(1,0,1,0);
                return o;
           }
           fixed4 frag(v2f In) :SV_TARGET{
                return In.col;
           }
           ENDCG
        }
    }
    FallBack "Diffuse"
}
```

###### 8.固定管线着色器案例：

```
Shader "Custom/myShaderFixed" {
    Properties{
          _Diffuse("myDiffuse",Color)=(1,1,1,1)
          _Specular("mySpecular",Color)=(1,1,1,1)
          _Shiness("myShiness",Range(0,20))=10
          _Ambient("myAmbient",Color)=(1,1,1,0.5)
          _Emission("myEmission",Color)=(1,1,1,1)
          
          _ConstantColor("myConstantColor",Color)=(1,1,1,1)
          _MainTex("MainTex",2D)="White"{}
          _SecondTex("SecondTex",2D)="White"{}
    }
    
    SubShader{
         //  TAGS {"QUEUE"="Transparent"}
           /*指代该shader的绘制顺序，等所有的非透明物体都绘制完了之后再绘制本物体。这个顺序是硬件底层规定的。这句话的作用仅仅是指定本shader的绘制顺序*/
           
        Pass{
           Blend SrcAlpha OneMinusSrcAlpha   /*开启Alpha混合*/
            
           /*FixedShader的固定结构，Material{}表示对光照等的处理，特点是使用关键字直接显示light，specular等效果*/
           Material{
                Diffuse[_Diffuse]  
                Specular[_Specular]  
                Shininess[_Shiness]  /*表示高光specular的范围*/
                Ambient[_Ambient]  /*加入环境光的影响*/
                Emission[_Emission]  /*自发光————通常用于一些柴火、灯光等自主发光的物体*/
           }
           lighting on  /*fixedShader中需要手动开启light渲染，如果没有则无法显示自定义的颜色；总之这句话在FixedShader中很重要*/
           separateSpecular on  /*当shader中需要有specular效果时，表示开启高光效果；跟lighting on 一样，没有这句话则不会显示高光效果*/
           
           SetTexture[_MainTex]{   /*固定结构*/
                 constantColor[_ConstantColor]   
                 /*核心作用只是自定义texture的Alpha，调节此color的RGB没有作用，只有Alpha可以被应用到后面的“texture*Constant”中*/
                 
                 combine texture *primary double,texture*Constant
                 /*texture:指代参数“_MainTex”
                 primary:环境及自定义的颜色的影响，统称为非texture因素的影响
                 double：light照射到此texture的强度，quad则是4倍
                 texture*Constant：当强调要用指定的Alpha时则添加此参数texture——表示使用该Texture的Alpha通道,
                                   此时在texture的inspector界面中选中“Alpha from grayScale”——开启该texture的Alpha通道。如果没有勾选则Alpha为1，只是添加以上语句是看不到效果的
                                   倘若在此程度上还需要自定义Alpha，那么引入关键字"Constant"，但是该“Constant”需要与“constantColor[_ConstantColor]”配合
                                   如果“constantColor[_ConstantColor]”没有写，则“Constant”=0*/
           }
           SetTexture[_SecondTex]{
               combine texture *previous   
               /*previous指代之前所有处理的结果，包括上一个Texture的处理结果，primary仅仅指环境光照等非texture因素的影响*/
           }
        }
    }
    
    FallBack "Diffuse"
    
    /*总结：FixedShader大多使用固定结构语法的语句，所以自定义程度较低，但基本的效果都可以实现。需要记住基本语法结构*/
}
```

###### 9.通常C#脚本自定义Shader中的部分参数(uniform修饰)：

```c#
using UnityEngine;
using System.Collections;
using System.Linq;
using System.Collections.Generic;
using System;

public class Test01 : MonoBehaviour {
    //float loopEdge=-1;
    public Cubemap cube;
    // Use this for initialization
    void Start () {
        //this.GetComponent<Renderer> ().material.SetTextureScale ("_SecondTex",new Vector2(0.2f,1f));
        
//        Mesh mesh = this.GetComponent<MeshFilter> ().mesh;
//        Vector3[] vert= mesh.vertices;
//        Debug.Log(vert.Max (v=>v.x));
//        foreach (var v in vert)
//            Debug.Log (v);
     }
    
    void Update () {
        Camera.main.RenderToCubemap (cube);
//        loopEdge += Time.deltaTime;
//        loopEdge = (loopEdge > 1) ? -1 : loopEdge;
        //this.GetComponent<Renderer> ().material.SetMatrix ("testMatrix",testMatrix());
        //this.GetComponent<Renderer> ().material.SetFloat ("time",Time.realtimeSinceStartup);
        //this.GetComponent<Renderer> ().material.SetVector ("worldPivot",transform.position);
        //this.GetComponent<Renderer> ().material.SetInt ("randomAngle",Random.Range(1,100));
    //    this.GetComponent<Renderer> ().material.SetFloat ("loopEdge",loopEdge);
        //this.GetComponent<Renderer> ().material.SetTextureOffset ("_MainTex",new Vector2(0.5f,0.5f));

//        if (Input.GetMouseButtonDown(0)) {
//            Ray ray= Camera.main.ScreenPointToRay(Input.mousePosition);
//            RaycastHit hit;
//            if(Physics.Raycast(ray,out hit ,1000f)){
//                this.GetComponent<Renderer>().material.SetVector("_hitWorldPos",new Vector4(hit.point.x,hit.point.y,hit.point.z,1));
//            }
//        }
    }

    Matrix4x4 testMatrix(){
        Matrix4x4 myMatrix = new Matrix4x4 ();
        myMatrix[0,0]=1;
        myMatrix[1,1]=Mathf.Cos(Time.realtimeSinceStartup);
        myMatrix[1,2]=Mathf.Sin(Time.realtimeSinceStartup);
        myMatrix[2,1]=-Mathf.Sin(Time.realtimeSinceStartup);
        myMatrix[2,2]=Mathf.Cos(Time.realtimeSinceStartup);
        myMatrix[3,3]=1;
        return myMatrix;
    }
}
```









































