[TOC]



项目开发中经常需要通过重构物体的Mesh来实现一些美术效果

##### 重构Mesh实现“物体倒影”效果

脚本代码如下：

```c#
void DrawNewMesh()
{
    Mesh mesh = this.GetComponent<MeshFilter>().mesh;
    int[] oldTris = mesh.triangles;
    Vector3[] oldVerts = mesh.vertices;
    Vector2[] oldUVs = mesh.uv;
    Vector3[] oldNors = mesh.normals;
    Color[] oldColors = mesh.colors; //对于不支持vertex color的shader，则此oldColors.Length = 0
    foreach (var vert in oldVerts)
    {
        Debug.Log(vert.ToString() + " Vert ");
    }
    int i = 0;
    foreach (var index in oldTris)
    {
        Debug.Log(index + "  Tris " + i);
        ++i;
    }
    int j = 0;
    foreach (var uv in oldUVs)
    {
        Debug.Log(uv.ToString() + "   UV  " + j);
        ++j;
    }
    int k = 0;
    foreach (var nor in oldNors)
    {
        Debug.Log(nor.ToString() + "  Nor   " + k);
        ++k;
    }

    /********************* 倒影的效果 **********************/
    /******** 正立和倒立的图像呈现在两个不同的物体上 *******/
    /*tris的顺序顺时针或逆时针改变的仅仅只是图像的正面，只有顺时针(正面)camera才会渲染，背面的图像camera是不会管的
     *要得到倒立或者“屏风”对称的效果，其关键在于改变verticles的顺序
     *在以上mesh.verticles中，其顶点顺序是按照从上到下(控制是否倒立),从左到右(控制是否对称)
     *因此若要得到图像倒立的效果，只需要从下到上依次改变顶点顺序，对称效果也是一样的效果
     *要注意：mesh.verticles各个顶点的顺序是按照上下左右严格顺序排列的，不存在顺逆时针排列顶点顺序的情况
     *顺逆时针改变的是triangles——图像的正反面
     *改变了顶点顺序后，需要相应的改变triangles顺序，使之为顺时针
     */
    //Vector3[] newVerts = new Vector3[4];
    //newVerts[0] = oldVerts[2];
    //newVerts[1] = oldVerts[3];
    //newVerts[2] = oldVerts[0];
    //newVerts[3] = oldVerts[1];
    //mesh.SetVertices(newVerts);
    //int[] newTris = { 0, 1, 3, 0, 3, 2 };
    //mesh.SetTriangles(newTris, 0);
    //由于分散在两个物体中，因此UV坐标并不需要进行改变

    /********** 正立和倒立图像集中在一个物体上 ***************/
    //由于集中在一个mesh中，因此pivot会发生改变，因此需要重新计算各个verticles相对于改变后的pivot的坐标
    float width_x = oldVerts[3].x, height_y = oldVerts[3].y * 2, z = oldVerts[3].z;
    //添加gap，方便体现效果
    float gap = 0.2f;
    Vector3[] newVerts2 = new Vector3[8];
    newVerts2[0] = new Vector3(-width_x, gap, z);
    newVerts2[1] = new Vector3(width_x, gap, z);
    newVerts2[2] = new Vector3(-width_x, gap + height_y, z);
    newVerts2[3] = new Vector3(width_x, gap + height_y, z);
    newVerts2[4] = new Vector3(-width_x, -gap, z);
    newVerts2[5] = new Vector3(width_x, -gap, z);
    newVerts2[6] = new Vector3(-width_x, (-gap) - height_y, z);
    newVerts2[7] = new Vector3(width_x, (-gap) - height_y, z);
    mesh.SetVertices(newVerts2);
    int[] newTris2 = { 0, 3, 1, 0, 2, 3, 4, 5, 7, 4, 7, 6 }; //tris仅仅控制正反方向，对图像的倒立或对称不起作用
    mesh.SetTriangles(newTris2, 0);

    /********* UV 和 Normal 的作用 ************/
    //UV代表的是需要截取原有texture中哪一区域的图像，坐标中的x, y分别代表截图texture中的区域
    //以texture左下角为中心点
    //由于集中在一个物体上，原有uv只有4个，若不增加则仍然只有正立图像，倒立图像由于没有UV表示的区域而没有图像显示
    //若不为新增加的顶点添加相应的UV，则该新增加的顶点所包含的区域会因为缺少贴图而没有图像显示
    Vector2[] newUVs = new Vector2[8];
    newUVs[0] = oldUVs[0];
    newUVs[1] = oldUVs[1];
    newUVs[2] = oldUVs[2];
    newUVs[3] = oldUVs[3];
    newUVs[4] = oldUVs[0]; //由于新增加的verticles已经改变了顺序，因此UV则不需要再改变，否则会抵消倒立效果
    newUVs[5] = oldUVs[1];
    newUVs[6] = oldUVs[2];
    newUVs[7] = oldUVs[3];
    mesh.SetUVs(0, newUVs);
    //Normal的作用是为新增加的顶点设置正方向。若顶点没有normal则多个顶点覆盖的区域会显示异常黑色
    Vector3[] newNors = new Vector3[8];
    for (int m = 0; m < 8; ++m)
    {
        newNors[m] = new Vector3(0, 0, -1); //normal的“1”，“-1”设置很重要，若方向错误会导致异常黑色
    }
    mesh.SetNormals(newNors);
    /*对于新增加的verticles，其normal不可缺少——若缺少则显示异常黑色
     *UV代表所使用的贴图中截取的区域，若缺少则没有图像显示
     *Tris表示图像的正反方向，顺时针为正方向
     */

    /*还可以设置各个verticles的color，在color中加入alpha的控制，可以制造透明度逐渐变化的效果
     *SetColor需要该物体使用的shader支持vertex color。若shader本身不支持vertex  color，
     *则使用mesh.colors获取其原本的color时为空，使用SetColor也不会有反应
     */
    Color[] newColors = new Color[8];
    for (int n = 0; n < 4; ++n)
    {
        newColors[n] = new Color(1, 1, 1, 1);
    }
    float alpha = 0.5f;
    newColors[4] = new Color(1, 1, 1, alpha);
    newColors[5] = newColors[4];
    newColors[6] = new Color(1, 1, 1, 0);
    newColors[7] = newColors[6];
    mesh.SetColors(newColors);
}
```

以上是经过验证过的，可以实现修改mesh来得到倒影的效果。但修改mesh，尤其是大量的修改，通常针对于mesh结构较为单一的情况，因为可以明确的得到顶点的数量和顶点信息

###### Mesh顶点数据代表的意义

```c#
using UnityEngine;
using System.Collections;
public class Test: MonoBehaviour
{
    void Start()
    {
        Vector3[] newVertices = { new Vector3(0, 0, 0), new Vector3(0, 1, 0), new Vector3(1, 1, 0), new Vector3(1, 0, 0) };
        Vector2[] newUV = { new Vector2(0, 0), new Vector2(0, 1), new Vector2(1, 1), new Vector2(1, 0) };
        int[] newTriangles = {0,2,1,0,3,2};  //应用左手法则，四指成拳头的方向即为正方向
        Mesh mesh = new Mesh();
        mesh.vertices = newVertices;
        mesh.uv = newUV;
        mesh.triangles = newTriangles;
        GetComponent<MeshFilter>().mesh = mesh;
    }
}
```

解析：

1.顶点数组 包括这个mesh中所有的顶点，如 `Vector3[] newVertices = { new Vector3(0, 0, 0), new Vector3(0, 1, 0), new Vector3(1, 1, 0), new Vector3(1, 0, 0) }`，即Mesh的四个顶点

三角形数组 指定如何构成每个三角形，同时也确定了三角形的个数，如 `int[] newTriangles = {0,2,1,0,3,2}`; 这行代码表示0，2，1（对应着newVertices[0],newVertices [2],newVertices [1]）,构成第一个三角形的三个角，数字的顺序表示顺逆时针，同时确定了三角形的正反面；后面的0，3，2同理，这就代表此mesh有两个三角形。

uv数组 这是一个vector2数组，用来确定顶点数组中每个顶点的uv坐标

2.顶点坐标的基准点是这个对象的坐标，根据不同的基准点设置顶点的位置，可以将物体显示在不同的位置



##### 修改Mesh顶点将物体Z轴方向上的最高点拉高一倍

```c#
using UnityEngine;
using System.Collections;

[RequireComponent(typeof(MeshFilter))]
public class example : MonoBehaviour {
    void Update() {
        Mesh mesh = GetComponent<MeshFilter>().mesh;
        Vector3 [] vertices = mesh.vertices;

        int p = 0;
        int flag = -1;
        float maxheight = 0.0F;
        while (p < vertices.Length) {
            if(vertices[p].z > maxheight) {
                 maxheight = vertices[p].z;
                 flag = p;
            }
            p++;
         }
         vertices[flag] += new Vector3(0, 0, maxheight);

         mesh.vertices = vertices;
         mesh.RecalculateNormals();
    }
}
```



##### 重构Mesh实现“文言文的从右到左、从上到下排列”的效果

1).支持“文言文显示风格”的脚本如下：

```c#
using UnityEngine;
using UnityEngine.UI;
using System.Collections;
using System.Collections.Generic;
using UnityEditor;

public class Test03 : Text
{
    private bool _isRevert;
    public enum LetterType
    {
        chinese,
        english,
    }
    public bool isRevert {
        get
        {
            return _isRevert;
        }
        set{
            _isRevert = value;
            UpdateGeometry();
            EditorUtility.SetDirty(this);   //此句至关重要，及时更新scene中的图像
        }
    }

    protected override void OnPopulateMesh(VertexHelper toFill)
    {
        if (null == toFill)
            return;
        base.OnPopulateMesh(toFill);
        //获取所有的UIVertex,绘制一个字符对应6个UIVertex，绘制顺序为012 230
        List<UIVertex> listUIVertex = new List<UIVertex>();
        toFill.GetUIVertexStream(listUIVertex);
        var vertArray = listUIVertex.ToArray();
        for (int i = 0; i < vertArray.Length; i += 6)
        {
            float halfOfheight = Mathf.Abs(vertArray[i + 1].position.x - vertArray[i].position.x) / 2.0f;
            float halfOfwidth  = Mathf.Abs(vertArray[i + 1].position.y - vertArray[i +  2].position.y) / 2.0f;
            Vector3 centerPos = (vertArray[i].position + vertArray[i + 2].position) / 2.0f; //中心点的坐标并非为(0, 0)
            //先绕Z轴顺时针旋转90度
            float angle = Mathf.Deg2Rad * (90);
            Matrix4x4 rMatrix = new Matrix4x4();
            rMatrix.SetRow(0, new Vector4(Mathf.Cos(angle), Mathf.Sin(angle), 0, 0));
            rMatrix.SetRow(1, new Vector4(-Mathf.Sin(angle), Mathf.Cos(angle), 0, 0));
            rMatrix.SetRow(2, new Vector4(0, 0, 1, 0));
            rMatrix.SetRow(3, new Vector4(0, 0, 0, 1));
            centerPos = rMatrix.MultiplyPoint(centerPos);
            if (isRevert)
            {
                //再绕Y轴逆时针旋转180度
                float angle2 = Mathf.Deg2Rad * (-180);
                Matrix4x4 rMatrix2 = new Matrix4x4();
                rMatrix2.SetRow(0, new Vector4(Mathf.Cos(angle2), 0, -Mathf.Sin(angle2),  0));
                rMatrix2.SetRow(1, new Vector4(0, 1, 0, 0));
                rMatrix2.SetRow(2, new Vector4(Mathf.Sin(angle2), 0, Mathf.Cos(angle2),  0));
                rMatrix2.SetRow(3, new Vector4(0, 0, 0, 1));
                centerPos = rMatrix2.MultiplyPoint(centerPos);
            }
            Vector3[] temp = new Vector3[4];
            temp[0] = centerPos + new Vector3(-halfOfwidth, -halfOfheight, 0.0f);
            temp[1] = centerPos + new Vector3(halfOfwidth, -halfOfheight, 0.0f);
            temp[2] = centerPos + new Vector3(halfOfwidth, halfOfheight, 0.0f);
            temp[3] = centerPos + new Vector3(-halfOfwidth, halfOfheight, 0.0f);
            vertArray[i].position = temp[3];      //UIVertex的方向是从上往下，从左往右，而计算的temp则是从下往上，从左往右
            vertArray[i + 1].position = temp[2];
            vertArray[i + 2].position = temp[1];
            vertArray[i + 3].position = temp[1];
            vertArray[i + 4].position = temp[0];
            vertArray[i + 5].position = temp[3];
            listUIVertex[i] = vertArray[i];
            listUIVertex[i + 1] = vertArray[i + 1];
            listUIVertex[i + 2] = vertArray[i + 2];
            listUIVertex[i + 3] = vertArray[i + 3];
            listUIVertex[i + 4] = vertArray[i + 4];
            listUIVertex[i + 5] = vertArray[i + 5];
        }
        toFill.Clear();
        toFill.AddUIVertexTriangleStream(listUIVertex);
    }
}
```

2).针对该组件的“Editor脚本”如下：

```c#
using UnityEngine;
using System.Collections;
using UnityEditor;
using UnityEngine.UI;

[CanEditMultipleObjects()]
[CustomEditor(typeof(VerticalText), true)]
public class VerticalTextEditor : Editor
{
    VerticalText model;
    public override void OnInspectorGUI()
    {
        base.DrawDefaultInspector();
        model = target as VerticalText;
        bool isRevert = EditorGUILayout.Toggle("isRevert", model.isRevert);
        if (model.isRevert != isRevert)
            model.isRevert = isRevert;
    }
}
```

注意：isRevert的作用是 增加一种选择用于从左往右，从上往下的排列，“从右往左”和“从左往右”的效果分别为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111122429832.png" alt="image-20231111122429832" style="zoom:80%;" />   <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111122448367.png" alt="image-20231111122448367" style="zoom:80%;" />



###### 问题解析：以上代码中为什么绕Y轴逆时针旋转180度后仍然得到的是正面图像？

解答：第一次旋转时，绕Z轴顺时针旋转90度，从旋转结果可以看出，**是将整体的Text的绘制核心绕着Z轴顺时针旋转**，因此可以得到从上到下，从右到左的图像

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111122631895.png" alt="image-20231111122631895" style="zoom:80%;" />

**而真实决定每个字符的正面或背面图像的则是由UIVertex的顺序来决定的**——顺时针为正面，逆时针为背面

但是单纯的在transform组件中的Rotation中设置Z轴旋转90度，则无法得到绕Z轴旋转的效果，如图

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111122654050.png" alt="image-20231111122654050" style="zoom:80%;" />

设置Z轴逆时针旋转90度

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111122726279.png" alt="image-20231111122726279" style="zoom:80%;" />

因为直接更改transform是将Text组件所在的GameObject对象进行旋转，如当设置transform中的Rotation为180时，则得到以下图像：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111122744104.png" alt="image-20231111122744104" style="zoom:80%;" />

虽然两种情况都是旋转，**<font color=red>但前者是以绘制图像的中心点进行旋转，后者则是物体的中心点进行旋转</font>**

**<font color=red>设置UIVertex的顺序可以得到正面或背面的图像</font>**，但是图像本身的旋转颠倒则无法决定

因此当将绘制图像的中心点绕Z轴旋转90度时可以得到从上往下，从右到左的图像；此时再绕Y轴逆时针旋转180度得到从上倒下，从左到右的图像，虽然此时是背面的图像，但是设置UIVertex顺序后则可以始终得到正面的图像























































