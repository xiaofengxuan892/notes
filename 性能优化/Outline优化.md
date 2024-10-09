[TOC]



UGUI自带的“Outline组件”会极大增加顶点数，且会打断合批，因此==基于Shader实现描边功能==

实现过程主要分成两部分：

#### 1.描边组件XTextOutline:

添加到Text上的“XTextOutline.cs”组件，可自由设置描边的Color和Width。其中要点如下：

- 该脚本继承自“BaseMeshEffect.cs”，可重写组件生命周期中的方法，且由于“BaseMeshEffect.cs”前有“[ExecuteAlways]”标记，因此其在“Editor模式”下也会执行，如“Awake, OnEnable, OnDisable”等

- “==OnValidate==”方法专用于在“Editor模式”下==修改Inspector面板中该组件的任何参数后可以马上刷新显示最新效果==，且该方法==在“PlayMode模式”下不会自动执行==

- 针对==该“XTextOutline组件”被移除或者未启用==时，可分别==调用“OnDestroy”和“OnDisable”==执行对应逻辑。
  **从GameObject上移除某个组件，对该组件自身来讲也是将其Destroy，因此会触发组件的“OnDestroy”方法**

- 为方便合批优化，针对相同color和width的outline组件，默认使用相同的材质，因此使用Dictionary集合存储每个不同outline所对应的material。故封装新的类型“OutlineDataInfo”

- 由于需要向Shader中传递顶点相关的数据，因此==必须保证UI根部的Canvas的“Addtional Channel”中至少包含“TexCoord1, TexCoord2”两个通道==，这个非常重要，否则**shader内部无法获取到正确的顶点UV等信息**。最好的情况是==勾选“Everything”，开启所有通道==(根据RenderMode不同，“Normal”和“Tangent”有时不需要，无需开启)

- <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20241009194719456.png" alt="image-20241009194719456" style="zoom:80%;" />

  ```c#
  void SetShaderChannels()
  {
      if (base.graphic.canvas)
      {
          var v1 = base.graphic.canvas.additionalShaderChannels;
          var v2 = AdditionalCanvasShaderChannels.TexCoord1;
          if ((v1 & v2) != v2)         //位的"与运算"用于检测其中是否包含“v2”
          {
              base.graphic.canvas.additionalShaderChannels |= v2;
          }
          v2 = AdditionalCanvasShaderChannels.TexCoord2;
          if ((v1 & v2) != v2)
          {
              base.graphic.canvas.additionalShaderChannels |= v2;
          }
      }
  }
  ```

完成代码如下：

```c#
using System;
using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;
using System.Runtime.InteropServices;

[AddComponentMenu("UI/Effects/XTextOutline")]
public class XTextOutline : BaseMeshEffect
{
    [SerializeField]
    private Color OutlineColor = Color.white;
    [SerializeField][Range(0, 8)]
    private int OutlineWidth = 1;
    private static List<UIVertex> m_VetexList = new List<UIVertex>();
    //相同color和width的outline，使用相同的材质进行渲染，以方便合批优化
    //PS:考虑到游戏中不同outline的数量不会太多，且占用的内存空间不会太大，因此这里不提供“material的计数逻辑”。仅在“退出游戏”时统一清空本集合
    private static Dictionary<OutlineDataInfo, Material> allTextOutlineMat = new Dictionary<OutlineDataInfo, Material>();

    private const string OUTLINE_SHADER_NAME = "Unlit/XTextOutline";

    // ============================= START: “生命周期”相关方法 ======================
    protected override void Awake()
    {
        base.Awake();
        if (!SetMat()) return;
        SetShaderChannels();
        RefreshView();
    }

    protected override void OnEnable()
    {
        base.OnEnable();
        if(!SetMat()) return;
        RefreshView();
    }

    protected override void OnDisable()
    {
        base.OnDisable();
        if(null == base.graphic) return;
        base.graphic.material = null;
    }

    //退出游戏时清空集合
    protected void OnApplicationQuit()
    {
        allTextOutlineMat.Clear();
    }

    //1.“OnValidate”方法在“PlayMode模式”下不会自动执行
    //2.当组件在Inspector中有任何数值被改变时，均会自动触发该方法
    protected override void OnValidate()
    {
        base.OnValidate();
        if (!SetMat()) return;
        RefreshView();
    }

    //移除outline组件后会重置graphic为原本使用的材质
    protected override void OnDestroy()
    {
        base.OnDestroy();
        if(null == base.graphic) return;
        //在UGUI中设置为null时会自动使用默认材质“UI/Default”
        base.graphic.material = null;
    }
    // ============================= END: “生命周期”相关方法 ======================


    //相同color、width的outline使用同一个材质，以方便合批优化
    bool SetMat()
    {
        if (null == base.graphic)
        {
            Debug.LogError("No Graphic Component.");
            return false;
        }

        //为方便合批，相同color和width的outline使用相同的材质
        var mOutlineData = new OutlineDataInfo(OutlineColor, OutlineWidth);
        if (allTextOutlineMat.TryGetValue(mOutlineData, out var mat))
        {
            base.graphic.material = mat;
            return true;
        }

        //创建新的材质
        var mShader = Shader.Find(OUTLINE_SHADER_NAME);
        if (null == mShader)
        {
            Debug.LogError($"Can't find outline shader: {OUTLINE_SHADER_NAME}");
            return false;
        }
        var mMat = new Material(mShader);
        base.graphic.material = mMat;
        //放入集合
        allTextOutlineMat.Add(mOutlineData, mMat);
        return true;
    }

    //非常重要：设置根部Canvas的“Addtional Shader Channel”，至少包含“TexCoord1, TexCoord2”两个通道，
    //        用于向Shader中传递顶点相关的纹理TexCoord1-n数值
    void SetShaderChannels()
    {
        if (base.graphic.canvas)
        {
            var v1 = base.graphic.canvas.additionalShaderChannels;
            var v2 = AdditionalCanvasShaderChannels.TexCoord1;
            if ((v1 & v2) != v2)
            {
                base.graphic.canvas.additionalShaderChannels |= v2;
            }
            v2 = AdditionalCanvasShaderChannels.TexCoord2;
            if ((v1 & v2) != v2)
            {
                base.graphic.canvas.additionalShaderChannels |= v2;
            }
        }
    }

    //刷新视图
    private void RefreshView()
    {
        //设置参数
        if (base.graphic.material != null)
        {
            base.graphic.material.SetColor("_OutlineColor", OutlineColor);
            base.graphic.material.SetFloat("_OutlineWidth", OutlineWidth);
        }
        //刷新
        base.graphic.SetVerticesDirty();
    }

    // =================================== START：核心逻辑 =============================
    public override void ModifyMesh(VertexHelper vh)
    {
        vh.GetUIVertexStream(m_VetexList);

        this._ProcessVertices();

        vh.Clear();
        vh.AddUIVertexTriangleStream(m_VetexList);
    }

    private void _ProcessVertices()
    {
        for (int i = 0, count = m_VetexList.Count - 3; i <= count; i += 3)
        {
            var v1 = m_VetexList[i];
            var v2 = m_VetexList[i + 1];
            var v3 = m_VetexList[i + 2];
            // 计算原顶点坐标中心点
            var minX = _Min(v1.position.x, v2.position.x, v3.position.x);
            var minY = _Min(v1.position.y, v2.position.y, v3.position.y);
            var maxX = _Max(v1.position.x, v2.position.x, v3.position.x);
            var maxY = _Max(v1.position.y, v2.position.y, v3.position.y);
            var posCenter = new Vector2(minX + maxX, minY + maxY) * 0.5f;
            // 计算原始顶点坐标和UV的方向
            Vector2 triX, triY, uvX, uvY;
            Vector2 pos1 = v1.position;
            Vector2 pos2 = v2.position;
            Vector2 pos3 = v3.position;
            if (Mathf.Abs(Vector2.Dot((pos2 - pos1).normalized, Vector2.right))
                > Mathf.Abs(Vector2.Dot((pos3 - pos2).normalized, Vector2.right)))
            {
                triX = pos2 - pos1;
                triY = pos3 - pos2;
                uvX = v2.uv0 - v1.uv0;
                uvY = v3.uv0 - v2.uv0;
            }
            else
            {
                triX = pos3 - pos2;
                triY = pos2 - pos1;
                uvX = v3.uv0 - v2.uv0;
                uvY = v2.uv0 - v1.uv0;
            }
            // 计算原始UV框
            var uvMin = _Min(v1.uv0, v2.uv0, v3.uv0);
            var uvMax = _Max(v1.uv0, v2.uv0, v3.uv0);
            //OutlineColor 和 OutlineWidth 也传入，避免出现不同的材质球
            //描边颜色 用uv3 和 tangent的 zw传递
            //var col_rg = new Vector2(OutlineColor.r, OutlineColor.g);     
            //var col_ba = new Vector4(0, 0, OutlineColor.b, OutlineColor.a);
            //描边的宽度 用normal的z传递
            //var normal = new Vector3(0, 0, OutlineWidth);      

            // 为每个顶点设置新的Position和UV，并传入原始UV框
            v1 = _SetNewPosAndUV(v1, this.OutlineWidth, posCenter, triX, triY, uvX, uvY, uvMin, uvMax);
            v2 = _SetNewPosAndUV(v2, this.OutlineWidth, posCenter, triX, triY, uvX, uvY, uvMin, uvMax);
            v3 = _SetNewPosAndUV(v3, this.OutlineWidth, posCenter, triX, triY, uvX, uvY, uvMin, uvMax);

            // 应用设置后的UIVertex
            m_VetexList[i] = v1;
            m_VetexList[i + 1] = v2;
            m_VetexList[i + 2] = v3;
        }
    }

    private static UIVertex _SetNewPosAndUV(UIVertex pVertex, int pOutLineWidth,
        Vector2 pPosCenter,
        Vector2 pTriangleX, Vector2 pTriangleY,
        Vector2 pUVX, Vector2 pUVY,
        Vector2 pUVOriginMin, Vector2 pUVOriginMax)
    {
        // Position
        var pos = pVertex.position;
        var posXOffset = pos.x > pPosCenter.x ? pOutLineWidth : -pOutLineWidth;
        var posYOffset = pos.y > pPosCenter.y ? pOutLineWidth : -pOutLineWidth;
        pos.x += posXOffset;
        pos.y += posYOffset;
        pVertex.position = pos;
        // UV
        var uv = pVertex.uv0;
        uv += (Vector4)pUVX / pTriangleX.magnitude * posXOffset * (Vector2.Dot(pTriangleX, Vector2.right) > 0 ? 1 : -1);
        uv += (Vector4)pUVY / pTriangleY.magnitude * posYOffset * (Vector2.Dot(pTriangleY, Vector2.up) > 0 ? 1 : -1);
        pVertex.uv0 = uv;

        pVertex.uv1 = pUVOriginMin;     //uv1 uv2 可用  tangent  normal 在缩放情况 会有问题
        pVertex.uv2 = pUVOriginMax;

        return pVertex;
    }
    // =================================== END：核心逻辑 =============================


    // =================================== START：工具方法 =============================
    private static float _Min(float pA, float pB, float pC)
    {
        return Mathf.Min(Mathf.Min(pA, pB), pC);
    }

    private static float _Max(float pA, float pB, float pC)
    {
        return Mathf.Max(Mathf.Max(pA, pB), pC);
    }

    private static Vector2 _Min(Vector2 pA, Vector2 pB, Vector2 pC)
    {
        return new Vector2(_Min(pA.x, pB.x, pC.x), _Min(pA.y, pB.y, pC.y));
    }

    private static Vector2 _Max(Vector2 pA, Vector2 pB, Vector2 pC)
    {
        return new Vector2(_Max(pA.x, pB.x, pC.x), _Max(pA.y, pB.y, pC.y));
    }
    // =================================== END：工具方法 =============================
}

//存储描边相关信息，如“color、width”等
[StructLayout(LayoutKind.Auto)]
public struct OutlineDataInfo : IEquatable<OutlineDataInfo>
{
    private Color outlineColor;
    private int outlineWidth;

    public OutlineDataInfo(Color m_outlineColor, int m_outlineWidth)
    {
        outlineColor = m_outlineColor;
        outlineWidth = m_outlineWidth;
    }

    public bool Equals(OutlineDataInfo other)
    {
        return outlineColor == other.outlineColor && outlineWidth == other.outlineWidth;
    }

    public static bool operator ==(OutlineDataInfo a, OutlineDataInfo b)
    {
        return a.Equals(b);
    }

    public static bool operator !=(OutlineDataInfo a, OutlineDataInfo b)
    {
        return !(a == b);
    }

    public override bool Equals(object obj)
    {
        return obj is OutlineDataInfo info && Equals(info);
    }

    public override int GetHashCode()
    {
        return outlineColor.GetHashCode() ^ outlineWidth.GetHashCode();
    }
}
```



#### 2.配合描边组件的XTextOutline.shader

该shader对==Mask、RectMask2D等也能很好的支持==，配合“XTextOutline组件”可以很好的实现描边效果。

完整代码如下：

```
Shader "Unlit/XTextOutline"
{
    Properties
    {
        [PerRendererData] _MainTex ("Main Texture", 2D) = "white" {}
        _Color ("Tint", Color) = (1, 1, 1, 1)
        _OutlineColor ("Outline Color", Color) = (1, 1, 1, 1)
        _OutlineWidth ("Outline Width", Int) = 1

        _StencilComp ("Stencil Comparison", Float) = 8
        _Stencil ("Stencil ID", Float) = 0
        _StencilOp ("Stencil Operation", Float) = 0
        _StencilWriteMask ("Stencil Write Mask", Float) = 255
        _StencilReadMask ("Stencil Read Mask", Float) = 255

        _ColorMask ("Color Mask", Float) = 15

        [Toggle(UNITY_UI_ALPHACLIP)] _UseUIAlphaClip ("Use Alpha Clip", Float) = 0
    }

    SubShader
    {
        Tags
        {
            "Queue"="Transparent"
            "IgnoreProjector"="True"
            "RenderType"="Transparent"
            "PreviewType"="Plane"
            "CanUseSpriteAtlas"="True"
        }

        Stencil
        {
            Ref [_Stencil]
            Comp [_StencilComp]
            Pass [_StencilOp]
            ReadMask [_StencilReadMask]
            WriteMask [_StencilWriteMask]
        }

        Cull Off
        Lighting Off
        ZWrite Off
        ZTest [unity_GUIZTestMode]
        Blend SrcAlpha OneMinusSrcAlpha
        // Blend Off
        ColorMask [_ColorMask]

        Pass
        {
            Name "OUTLINE"

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma target 2.0

            //Add for RectMask2D
            #include "UnityUI.cginc"
            //End for RectMask2D

            sampler2D _MainTex;
            fixed4 _Color;
            fixed4 _TextureSampleAdd;
            float4 _MainTex_TexelSize;

            float4 _OutlineColor;
            int _OutlineWidth;

            //Add for RectMask2D
            float4 _ClipRect;
            //End for RectMask2D

            struct appdata
            {
                float4 vertex : POSITION;
                float4 tangent : TANGENT;
                float4 normal : NORMAL;
                float2 texcoord : TEXCOORD0;
                float2 uv1 : TEXCOORD1;
                float2 uv2 : TEXCOORD2;
                float2 uv3 : TEXCOORD3;
                fixed4 color : COLOR;
            };

            struct v2f
            {
                float4 vertex : SV_POSITION;
                float4 tangent : TANGENT;
                float4 normal : NORMAL;
                float2 texcoord : TEXCOORD0;
                float2 uv1 : TEXCOORD1;
                float2 uv2 : TEXCOORD2;
                float2 uv3 : TEXCOORD3;
                //Add for RectMask2D
                float4 worldPosition : TEXCOORD4;
                //End for RectMask2D
                fixed4 color : COLOR;
            };

            v2f vert(appdata IN)
            {
                v2f o;

                //Add for RectMask2D
                o.worldPosition = IN.vertex;
                //End for RectMask2D

                o.vertex = UnityObjectToClipPos(IN.vertex);
                o.tangent = IN.tangent;
                o.texcoord = IN.texcoord;
                o.color = IN.color;
                o.uv1 = IN.uv1;
                o.uv2 = IN.uv2;
                o.uv3 = IN.uv3;
                o.normal = IN.normal;

                return o;
            }

            /*
            fixed IsInRect(float2 pPos, float4 pClipRect)
            {
                pPos = step(pClipRect.xy, pPos) * step(pPos, pClipRect.zw);
                return pPos.x * pPos.y;
            }
            */

            //该方法非常重要，用于判定“pPos”是否在矩形范围内，返回值只有0或1两种情况：
            //0 —— 代表pPos不在范围内，1 —— 代表pPos在范围内
            fixed IsInRect(float2 pPos, float2 pClipRectMin, float2 pClipRectMax)
            {
                pPos = step(pClipRectMin, pPos) * step(pPos, pClipRectMax);
                return pPos.x * pPos.y;
            }

            fixed SampleAlpha(int pIndex, v2f IN)
            {
                const fixed sinArray[12] = { 0, 0.5, 0.866, 1, 0.866, 0.5, 0, -0.5, -0.866, -1, -0.866, -0.5 };
                const fixed cosArray[12] = { 1, 0.866, 0.5, 0, -0.5, -0.866, -1, -0.866, -0.5, 0, 0.5, 0.866 };
                float2 pos = IN.texcoord + _MainTex_TexelSize.xy * float2(cosArray[pIndex], sinArray[pIndex]) * _OutlineWidth;	//normal.z 存放 _OutlineWidth
                return IsInRect(pos, IN.uv1, IN.uv2) * (tex2D(_MainTex, pos) + _TextureSampleAdd).a * _OutlineColor.a;		//tangent.w 存放 _OutlineColor.w
            }

            fixed4 frag(v2f IN) : SV_Target
            {
                fixed4 color = (tex2D(_MainTex, IN.texcoord) + _TextureSampleAdd) * IN.color;//默认的文字颜色
                if (_OutlineWidth > 0)	//normal.z 存放 _OutlineWidth
                {
                    color.w *= IsInRect(IN.texcoord, IN.uv1, IN.uv2);	//uv1 uv2 存着原始字的uv长方形区域大小

                    half4 val = half4(_OutlineColor.rgb, 0);		//uv3.xy tangent.z 分别存放着 _OutlineColor的rgb
                    //val 是 _OutlineColor的rgb，a是后面计算的
                    val.w += SampleAlpha(0, IN);
                    val.w += SampleAlpha(1, IN);
                    val.w += SampleAlpha(2, IN);
                    val.w += SampleAlpha(3, IN);
                    val.w += SampleAlpha(4, IN);
                    val.w += SampleAlpha(5, IN);
                    val.w += SampleAlpha(6, IN);
                    val.w += SampleAlpha(7, IN);
                    val.w += SampleAlpha(8, IN);
                    val.w += SampleAlpha(9, IN);
                    val.w += SampleAlpha(10, IN);
                    val.w += SampleAlpha(11, IN);

                    color = (val * (1.0 - color.a)) + (color * color.a);
                    color.a = saturate(color.a);
                    color.a *= IN.color.a;	//字逐渐隐藏时，描边也要隐藏
                }

                //Add for RectMask2D
                color.a *= UnityGet2DClipping(IN.worldPosition.xy, _ClipRect);
                #ifdef UNITY_UI_ALPHACLIP
                    clip(color.a - 0.001);
                #endif
                //End for RectMask2D

                return color;
            }

            ENDCG
        }
    }
}
```



以上两个脚本经过实际测试，可以很好的实现“描边效果”，顶点数量也极少，可对DC友好
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20241009200105198.png" alt="image-20241009200105198" style="zoom:80%;" />

![image-20241009200051050](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20241009200051050.png)

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20241009200239161.png" alt="image-20241009200239161" style="zoom:80%;" />



附上一个大神的链接：https://blog.csdn.net/HelloCLanguage/article/details/105836309





