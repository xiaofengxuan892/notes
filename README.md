# notes





```c#
bool IsRectTransformInsideViewport(RectTransform viewport, RectTransform target)
{
    // 将目标物体的四个角点转换到视口的局部坐标系中
    Vector3[] targetCorners = new Vector3[4];
    target.GetWorldCorners(targetCorners);

    Vector3[] viewportCorners = new Vector3[4];
    viewport.GetWorldCorners(viewportCorners);

    Rect viewportRect = new Rect(
        viewportCorners[0].x,
        viewportCorners[0].y,
        viewportCorners[2].x - viewportCorners[0].x,
        viewportCorners[2].y - viewportCorners[0].y
    );

    foreach (Vector3 corner in targetCorners)
    {
        if (viewportRect.Contains(corner))
        {
            return true;
        }
    }

    return false;
}
```

