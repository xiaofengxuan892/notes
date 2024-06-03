[TOC]



##### Submesh：

每个Mesh中至少有一个submesh，每个submesh对应一个材质，各个submesh间独立渲染。

美术在制作模型时经常需要针对模型的不同部位使用不同的材质，实现丰富的效果，此时即可借助submesh来实现：每个submesh对应不同的部位，分别进行渲染，既保证了模型的统一，又可以实现模型丰富的效果。此即为submesh真正的作用。

##### Mesh与Submesh的关系：

Mesh中存储该物体所有的顶点Verticles、三角面Triangles、法线Normals、颜色Colors、纹理UV，Submesh并不拥有任何的顶点Verticles信息，仅拥有该submesh所包含的三角面triangles。结合Mesh中的Verticles、Submesh中的三角面，即可确定该Submesh在整个模型Mesh中实际包含的区域

##### 使用Submesh绘制一个正方形Mesh中“一半为红色，一半为蓝色”的效果：

```c#
//创建一个包含submesh的物体实例
void CreateSubmeshObj()
{
    Mesh mesh = new Mesh();
    mesh.subMeshCount = 2;
    mesh.vertices = new Vector3[]
    {
    new Vector3(0, 1),
    new Vector3(1, 1),
    new Vector3(0, 0),
    new Vector3(1, 0),
    };

    int[] triangles = new int[] { 0, 1, 2 };
    mesh.SetTriangles(triangles, 0);    //submesh1
    triangles = new int[] { 1, 3, 2 };
    mesh.SetTriangles(triangles, 1);    //submesh2

    GameObject go = new GameObject("SubmeshObj");
    go.AddComponent<MeshFilter>().mesh = mesh;
    MeshRenderer mr = go.AddComponent<MeshRenderer>();
    Material mat1 = new Material(Shader.Find("Unlit/Color"));
    mat1.SetColor("_Color", Color.red);
    Material mat2 = new Material(Shader.Find("Unlit/Color"));
    mat2.SetColor("_Color", Color.blue);
    mr.materials = new Material[] { mat1, mat2 };   //每个submesh对应一个material
}
```

效果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111213546946.png" alt="image-20231111213546946" style="zoom:80%;" />

**注意**：设置submesh的triangles三角面时使用**<font color=red>左手法则——大拇指方向为正方向</font>**，当三角面中**<font color=blue>“顶点的顺序”为“顺时针”</font>**时，则该mesh正朝向玩家



##### Mesh合并对Drawcall的影响：

###### Mesh合并的方式：

```c#
GameObject go = new GameObject("OnlyCombineMesh");
MeshFilter goMF = go.AddComponent<MeshFilter>();
MeshRenderer goMR = go.AddComponent<MeshRenderer>();

MeshFilter[] mfs = this.GetComponentsInChildren<MeshFilter>();
//当需要合并material时要先存储各个mesh对应的uv信息
List<Vector2[]> uvList = new List<Vector2[]>(); 
List<CombineInstance> allCI = new List<CombineInstance>();

for (int i = 0; i < mfs.Length; ++i)
{
    MeshFilter temp = mfs[i];
    //充分考虑各个mesh中所包含的submesh的情况
    for (int j = 0; j < temp.sharedMesh.subMeshCount; ++j)  
    {
        CombineInstance ci = new CombineInstance();
        ci.mesh = temp.sharedMesh;
        ci.subMeshIndex = j;
        ci.transform = temp.transform.localToWorldMatrix;  //合并的矩阵信息
        uvList.Add(temp.sharedMesh.uv);   //如果不合并各个mesh的material，则不需要存储uv信息
        allCI.Add(ci);
    }
}

Mesh finalMesh = new Mesh();
finalMesh.CombineMeshes(allCI.ToArray(), true/false, true);
goMF.sharedMesh = finalMesh;
```

**注意**：1).如果仅合并mesh，不对使用的材质做任何处理，则以上代码在任何情况下均可使用

2).`CombineMeshes`方法中第二个参数“true/false”代表“合并后的mesh中是否只包含一个submesh”：如果为false，则代表合并后的mesh中会包含多个submesh，这些submesh分别对应原有的各个物体所使用的mesh。由于各个submesh分别使用“合并后的材质列表”中其对应的材质进行渲染，因此在“显示效果”上不会有任何错乱的情况

3).`CombineMeshes`方法中第三个参数代表合并的过程中是否使用矩阵 —— 默认的矩阵为0，需要在”combineInstance“中设置合并矩阵才可使用，即`ci.transform = temp.transform.localToWorldMatrix`。但**<font color=red>如果是“蒙皮骨骼Mesh合并”则该参数需要设置为“false”</font>**，因此“蒙皮骨骼”中有其自带的“变换矩阵”，外部无需设置

4).由于以上方法中需要使用`GetComponentsInChildren<MeshFilter>()`获取所有的MeshFilter组件，为了避免“合并后的mesh”对父对象Root有任何影响，**<font color=red>父对象Root中最好不要添加“MeshFilter组件”</font>**，**<font color=red>即“空GameObject”</font>**

**PS**：各个submesh会按照顺序分别使用材质列表中各个材质进行渲染，如果材质数量不足，则后续的submesh无法显示出来(因为材质是负责显示图像的)；如果材质数量过多，则后续的材质不会被使用到

###### 问题：仅仅合并多个物体的mesh，但不对这些物体使用的材质做任何处理，是否可以降低drawcall？

**验证**：在场景中“空物体”Root下有三个物体，分别为Cube、Sphere、Capsule，其结构如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111214359972.png" alt="image-20231111214359972" style="zoom:80%;" />

此时场景的drawcall为“18”：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111214441203.png" alt="image-20231111214441203" style="zoom:80%;" />

1).对三个物体进行mesh合并：合并后的mesh中包含3个submesh，且每个submesh对应原来使用的材质

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111214610722.png" alt="image-20231111214610722" style="zoom: 50%;" />

**结果**：drawcall依然为“18”，没有任何降低

2).在“步骤1”的基础上，将三个submesh的材质替换成一样的

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111214734173.png" alt="image-20231111214734173" style="zoom:50%;" />

**结果**：drawcall依然为“18”，没有降低

**总结**：合并后的mesh中包含多个submesh，分别对应原有的多个物体的mesh。由于每个submesh都是独立渲染的，即使这些submesh都使用相同的材质，也依然不会降低drawcall。并且如果一个物体的mesh中submesh数量越多，则其渲染占用的drawcall数值越高

###### 问题：为什么不将合并后的mesh中只包含一个submesh，却保留三个submesh？

**解答**：如果合并后的mesh中只有一个submesh，则必须将各个物体的材质合并成一个，对应该submesh，并且重新设置该mesh的UV等信息。否则会出现“图像错乱”的情况。

而“合并材质”在很多情况下是无法实现的，尤其在每个物体使用的材质shader都不一致时。因此大多情况下“合并后的mesh”中包含有多个submesh，分别对应原有各个物体的mesh

###### 同时合并Mesh和材质的方式：

```c#
void CombineMesh(bool mergeMat)
{
    GameObject go = new GameObject("CombineMesh");
    MeshFilter goMF = go.AddComponent<MeshFilter>();
    MeshRenderer goMR = go.AddComponent<MeshRenderer>();

    MeshFilter[] mfs = this.GetComponentsInChildren<MeshFilter>();
    List<Vector2[]> uvList = new List<Vector2[]>(); //当需要合并material时要先存储各个mesh对应的uv信息
    List<CombineInstance> allCI = new List<CombineInstance>();
    //考虑submesh，对于单独的mesh合并更为全面
    for (int i = 0; i < mfs.Length; ++i)
    {
        MeshFilter temp = mfs[i];
        for (int j = 0; j < temp.sharedMesh.subMeshCount; ++j)
        {
            CombineInstance ci = new CombineInstance();
            ci.mesh = temp.sharedMesh;
            ci.subMeshIndex = j;
            ci.transform = temp.transform.localToWorldMatrix;
            uvList.Add(temp.sharedMesh.uv);
            allCI.Add(ci);
        }
    }

    MeshRenderer[] mrs = this.GetComponentsInChildren<MeshRenderer>();
    List<Material> allMats = new List<Material>();
    for (int i = 0; i < mrs.Length; ++i)
    {
        //这里使用的是“AddRange”，因此该物体使用的所有材质都会被添加到新的集合中
        allMats.AddRange(mrs[i].sharedMaterials);
    }
    if (!mergeMat)
        goMR.sharedMaterials = allMats.ToArray(); //当不需要合并材质时则直接使用该方式即可
    else
    {
        //对主纹理图片进行合并
        List<Texture2D> allTextures = new List<Texture2D>();
        for (int i = 0; i < allMats.Count; ++i)
        {
            allTextures.Add(allMats[i].GetTexture("_MainTex") as Texture2D);
        }
        Texture2D finalTexture = new Texture2D(1024, 1024, TextureFormat.RGBA32,  false);
        Rect[] uvs = finalTexture.PackTextures(allTextures.ToArray(), 0);
        Material finalMat = new Material(Shader.Find("Mobile/Diffuse"));
        finalMat.SetTexture("_MainTex", finalTexture);
        goMR.sharedMaterial = finalMat;
        //将合并后的总纹理中各个图片和原mesh的uv进行一一对应
        for (int i = 0; i < uvList.Count; ++i)
        {
            Vector2[] oldUV = uvList[i];
            Vector2[] newUV = new Vector2[oldUV.Length];
            for (int j = 0; j < oldUV.Length; ++j)
            {
                newUV[j].x = uvs[i].x + uvs[i].width * oldUV[j].x;
                newUV[j].y = uvs[i].y + uvs[i].height * oldUV[j].y;
            }
            allCI[i].mesh.uv = newUV;
        }
    }

    Mesh finalMesh = new Mesh();
    finalMesh.CombineMeshes(allCI.ToArray(), mergeMat, true); //只有在mergeMat为true时，才能起到降低drawcall的作用
    goMF.sharedMesh = finalMesh;
    //如果进行了材质合并，此时各个combineInstance的uv都已发生改变，为了不影响下次合并mesh，需要将mesh的uv重置才行
    if (mergeMat)
    {
        for (int i = 0; i < uvList.Count; ++i)
        {
            allCI[i].mesh.uv = uvList[i];
        }
    }

    this.gameObject.SetActive(false);

    //额外操作：当不需要合并材质时，则将包含有多个submesh的物体保存下来，用于下次检测包含有多个submesh合并时的情况时使用
    if (!mergeMat)
    {
        AssetDatabase.CreateAsset(finalMesh,  "Assets/FrankOwn/MeshAfterCombine.asset");
        PrefabUtility.CreatePrefab("Assets/FrankOwn/" + go.name + ".prefab", go);
    }
}
```

**注意**：1).由于“合并后的mesh”中仅有一个submesh，因此可以降低drawcall

2).以上“合并材质”时仅对材质使用的“_MainTex”主纹理贴图进行合并，因此需要各个材质使用的shader一致，否则合并后是无法达到正确的显示效果的

###### 蒙皮骨骼的Mesh合并：

蒙皮骨骼组件SkinnedMeshRender中包含该物体的Mesh、骨骼以及渲染用的材质，因此该物体并不需要另外添加MeshFilter和MeshRender组件。SkinnedMeshRender必须要以真实存在的骨骼实体GameObject - Skeleton为基础，否则该组件无法发挥正常的功能。在进行Mesh合并时，由于骨骼结构中有很复杂的变换矩阵，涉及骨骼空间，mesh局部空间，世界空间之间的转换，因此在使用“CombineMesh”方法时，默认不使用第三方矩阵，即“第三个参数为false”，由该骨骼结构内部的矩阵完成转换。具体的“Mesh合并”方法如下：

```c#
#region 合并SkinnedMeshRender
public AnimationClip[] anims;

void CombineSkinnedMeshRender(bool mergeMat)
{
    List<CombineInstance> allCI = new List<CombineInstance>();
    List<Material> allMats = new List<Material>();
    List<Transform> allBones = new List<Transform>();

    //首先收集完整的骨骼结构
    List<Transform> fullBones = new List<Transform>();
    fullBones.AddRange(skeleton.GetComponentsInChildren<Transform>(true));

    SkinnedMeshRenderer[] smrs = new SkinnedMeshRenderer[allParts.Length];
    for(int i = 0; i < allParts.Length; ++i)
    {
        smrs[i] = allParts[i].GetComponentInChildren<SkinnedMeshRenderer>();
    }
    for(int i = 0; i < smrs.Length; ++i)
    {
        SkinnedMeshRenderer temp = smrs[i];
        allMats.AddRange(smrs[i].sharedMaterials);
        for(int j = 0; j < temp.sharedMesh.subMeshCount; ++j)
        {
            CombineInstance ci = new CombineInstance();
            ci.mesh = temp.sharedMesh;
            ci.subMeshIndex = j;
            allCI.Add(ci);
        }
        //收集骨骼
        //方式一：
        //allBones.AddRange(temp.bones);

        //方式二：
        for(int j = 0; j < temp.bones.Length; ++j)
        {
            for(int k = 0; k < fullBones.Count; ++k)
            {
                if(fullBones[k].name == temp.bones[j].name)
                {
                    allBones.Add(fullBones[k]);
                    break;
                }
            }
        }
    }
    Mesh finalMesh = new Mesh();
    finalMesh.CombineMeshes(allCI.ToArray(), false, false); //由于骨骼的关系不能使用矩阵
    SkinnedMeshRenderer newSmr = skeleton.AddComponent<SkinnedMeshRenderer>();
    newSmr.sharedMesh = finalMesh;
    newSmr.sharedMaterials = allMats.ToArray();

    newSmr.bones = allBones.ToArray();
    //保证合并后的物体是以骨骼结构设立的姿势站立，如果没有设置SkinnedMeshRender的bones，则物体会以finalMesh的姿势平躺着
    //同时也是因为此原因，对SkinnedMeshRender中的mesh进行CombineMesh操作时不能使用矩阵

    //播放动画:因为skeleton物体下拥有完整的骨骼实体结构，因此可以正常播放以骨骼实体为基础的动画
    Animation anim = skeleton.AddComponent<Animation>();
    anim.wrapMode = WrapMode.Loop;
    anim.AddClip(anims[0], anims[0].name);
    anim.Play(anims[0].name);

    //注意：动画正常播放是依附于完整的骨骼结构的，所以如果以上收集骨骼的过程有误则无法则无法正常播放动画
    //并且动画的正常播放是以真实存在的一些列“Transform”节点为基础的。
    //所以如果只是合并SkinnedMeshRender，而不要求合并后支持动画效果，则可以直接create一个新GameObject，
    //添加SkinnedMeshRender组件即可。该物体可以正常显示，但是不支持动画播放
    #region  验证以上设想：没有bones对应的GameObject实体，只拥有合并后的SkinnedMeshRender，是无法播放动画的
    GameObject go = new GameObject("ROLE");  //缺少骨骼实体
    SkinnedMeshRenderer goSmr = go.AddComponent<SkinnedMeshRenderer>();
    goSmr.sharedMesh = finalMesh;
    goSmr.sharedMaterials = allMats.ToArray();
    goSmr.bones = allBones.ToArray(); //保证物体以bones设置的姿势显示
    //以上代码执行结果：只拥有skinnedMeshRender的角色依靠“goSmr.bones”可以正常站立，显示都正常，但由于缺少骨骼实体，无法播放动画
    Animation goAnim = go.AddComponent<Animation>();  //该动画无法播放
    goAnim.wrapMode = WrapMode.Loop;
    goAnim.AddClip(anims[0], anims[0].name);
    goAnim.Play(anims[0].name);
    //不要设想新建完整的骨骼实体结构，因为根据各个SkinnedMeshRenderer收集到的“allBones”并不是完整的skeleton
    for(int i = 0; i < allBones.Count; ++i)
    {
        Debug.Log(allBones[i].name + "   " + allBones[i].position);
    }
    #endregion
}
#endregion
```

**注意**：

**1.合并后的SkinnedMeshRender一定要以完整的骨骼实体Skeleton的真实物体为基础而存在**，如果只有SkinnedMeshRender组件是无法支持动画的。

2.对于普通的MeshRender，MeshFilter合并之后，只要在新GameObject中添加组件MeshRender，MeshFilter即可支持一切操作，在合并MeshFilter时设置的矩阵“ci.transform = temp.transform.localToWorldMatrix”，是以世界坐标系下的位置为统一基准对所有的mesh进行合并，因此合并后的finalMesh也是根据Hierarchy中各个物体的相对位置来显示的，而SkinnedMeshRender的mesh合并则使用骨骼结构中自带的矩阵。

3.在合并完成后需要设置该组件的骨骼结构，否则不能正常显示，如设置SkinnedMeshRender组件的“newSmr.bones = allBones.ToArray()”后，角色以bones设置的姿势显示：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112111521057.png" alt="image-20231112111521057" style="zoom:80%;" />



##### 动态批处理和静态批处理：

###### “动态批处理”的限制条件：

1).Mesh的顶点数量限制：一般情况下单个mesh的顶点数量不超过900个，当mesh中包含有法线时则mesh的顶点数量不能超过300个；当mesh中的uv数量>= 2时，则该mesh的顶点数量被限制在150以下

2).多个动态批处理的物体需要同一缩放比例

3).动态批处理的材质球需要相同，且使用的Shader不能有多Pass通道

由于动态批处理的限制条件极为严格，因此虽然可以降低drawcall，但资源本身需要简洁单一，无法体现丰富的效果。所以部分情况下需要以“极为简单的资源”才能降低drawcall，这种方式是不可取的。因此“drawcall并非越低越好”

**PS**：动态批处理的过程是Unity自动执行，其会在运行时检测当前场景中所有物体，如果某些物体满足“动态批处理”的限制条件，则会自动针对这些物体执行“批处理”，整个过程外部无需介入，只要物体本身满足以上3个限制条件即可

###### “静态批处理”的限制条件：

静态批处理的条件较为宽松，但要求批处理的物体是静态的，即该物体不能移动、缩放、旋转等，一般用于**完全不动的固定物体。如果该物体包含“动画”，则其是不能“静态批处理”的

对于将多个旗帜合并完后依然会**随风摆动**则不能使用静态批处理

因此在很多条件下静态批处理都是无法满足条件的

**并且进行静态批处理会对所有需要批处理的物体的mesh进行复制以进行合并，因此内存占用会是原来的2倍**

因此当内存占用过高时，通常不用静态批处理，而使用自己写的合并mesh的代码来实现，至少可以在自己写的代码中destroy掉原有的mesh物体

```
1.对于合并后的网格，unity会判断其中使用同一个材质的子网格，然后对它们进行批处理。
2.在内部实现上，unity首先把这些静态物体变换到世界空间下，然后为他们构建一个更大的顶点和索引缓存。对于使用了同一材质的物体，unity只需要调用一个draw call就可以绘制全部物体。而对于使用了不同材质的物体，静态批处理同样可以提升渲染性能。尽管这些物体仍然需要调用多个draw call，但静态批处理可以减少这些draw call之间的状态切换，而这些切换往往是费时的操作。从合并后的网格结构中发现，尽管3个水壶模型使用了同一个网格，但合并后却变成了3个独立网格。而且我们可以从unity的profiler观察到在应用静态批处理前后VBO total(Vertex Buffer Object，顶点缓冲对象)的变化,数目会变大，这正是因为静态批处理会占用更多内存的缘故。
如果场景中包含了除平行光以外的其他光源，并且在shader中定义了额外的Pass来处理它们，这些额外的Pass部分是不会被批处理的。但是处理平行光的Base Pass部分仍然会被静态批处理，因此仍然可以节省两个draw call
```

**总结**：基于动态批处理和静态批处理的限制条件，很多情况下无法使用Unity自带的批处理来降低drawcall，此时只能手动写代码来合并mesh和材质球以达到优化drawcall的效果



















































