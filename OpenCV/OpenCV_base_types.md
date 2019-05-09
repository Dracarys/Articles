# 【笔记】OpenCV 之基础数据类型

## CV::Point

声明：

```cpp
cv::Point2i p;
cv::Point3f p;
```

通过已有点构造一个点：

```cpp
cv::Point3f p2( p1 );
```

通过值构造：

```cpp
cv::Point2i p( x, y);
cv::Point3d p( x, y, z );
```

类型投射：

```cpp
（cv::Vec3f) p;
```

成员访问：

```cpp
p.x; 
p.y;
p.z;// 仅限于 3 维点
```

点积 & 双精度点积：

```cpp
float x = p1.dot( p2 );
double x = p1.ddot( p2 );
```

叉乘（Cross product）：

```cpp
p1.cross( p2 );// 仅限于 3 维点。
```
判断是点是否位于矩形内：

```cpp
p.inside( r );// 仅限于 2 维点
```

