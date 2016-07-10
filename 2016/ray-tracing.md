# Ray Tracing
[Introduction to Ray Tracing: a Simple Method for Creating 3D Images](http://www.scratchapixel.com/lessons/3d-basic-rendering/introduction-to-ray-tracing/how-does-it-work)

**Forward Tracing**：光线从光源处开始。
**Backward Tracing**：光线从人眼开始，比起Forward Tracing，效率更高。
主要谈下**Backward Tracing**，几个关键词：光源，人眼，光线，反射，折射，阴影。
我们假设人眼是一个点，在人眼前面有一张由像素点构成的平面，我们把光照射到物体，再到达像素平面的时候记录下像素值，这样就能模拟3D图片。

[算法代码](http://www.scratchapixel.com/code.php?id=3&origin=/lessons/3d-basic-rendering/introduction-to-ray-tracing)
算法中用到了很多几何和物理的知识。在这份代码中，
main函数中初始化了5个球和一个光源。

```
void render(const std::vector<Sphere> &spheres);
```
render函数的作用是在原点出模拟了一个人眼，同时定了一像素平面，最终是作为输出。在像素平面中的每个像素值的确定调用了`trace`函数。
```
Vec3f trace(
    const Vec3f &rayorig,
    const Vec3f &raydir,
    const std::vector<Sphere> &spheres,
    const int &depth);
```
trace`函数,它接受一个反向的光线(即从人眼触发的光线)原点和方向，`spheres`代表所有球，depth是`trace`递归的深度，返回一个代表颜色的3维向量。

trace函数主要是找出最近的球，之后进入分支判断球是否可以反射和折射和递归深度，可以就结合两者递归调用，否则就会进入到判断阴影的计算环节。

它里面主要是计算反射，折射，还有阴影。计算很有数学味道，看到几何知识能够在这里发挥这么大的作用，这很喜欢。
