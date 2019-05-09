# 【笔记】OpenCV 之基础数据类型

## CV::Point
```cpp
cv::Point2i p;
cv::Point3f p;
cv::Point3f p2( p1 );
cv::Point2i p( x, y);
cv::Point3d p( x, y, z );
(cv::Vec3f) p;// 类型投射
p.x; 
p.y;
p.z;// 仅限于 3 维点
// 点乘
float x = p1.dot( p2 );
double x = p1.ddot( p2 );
// 叉乘
p1.cross( p2 );// 仅限于 3 维点。
p.inside( r );// 判断是否位于矩形内，仅限于 2 维点
```

## cv::Scalar
```cpp
cv::Scalar s;
cv::Scalar s2( s1 ); // 通过一个已有的 scalar 来构造
cv::Scalar s( x0 );
cv::Scalar s( x0, x1, x2, x3 );
s1.mul( s2 ); // 逐元乘法
//(Quaternion)conjugation:
s1.conj( s2 );// (returns cv::Scalar(s0,-s1,-s2,-s2))
// (Quaternion)real test:
s1.isReal();// (returns true iff s1==s2==s3==0)
```
## cv::Size
```cpp
cv::Size sz;
cv::Size2i sz;
cv::Size2f sz;
cv::Size sz2( sz1 );
cv::Size2f sz( w, h );
sz.width; sz.height;

sz.area();
```

## cv::Rect
用来描述那些与坐标轴对齐的矩形

```cpp
cv::Rect r; 
cv::Rect r2( r1);
cv::Rect( x, y, w, h);
cv::Rect(p, sz);
cv::Rect(p1, p2); // 通过两个点来声明，对角线？
r.x; r.y; r.width; r.height;
r.area(); // 求面积
r.tl(); // top-left
r.br(); // bottom-right
r.contains( p ); // 是否包含点 p
```
重载的一些操作符：

```cpp
// 取交叠的矩形
cv::Rect r3 = r1 & r2; 
r1 &= r2;
// 
cv::Rect r3 = r1 | r2; 
r1 |= r2;
// 变换
cv::Rect rx = r + x;
r += x;
// 缩放
cv::Rect rs = r + sz; 
r += sz;
// 相等判断
bool eq = (r1 == r2);
// 非等判断
bool ne = (r1 != r2);
```

## cv::RotatedRect
用来描述那些坐标轴没对齐的矩形

```cpp
cv::RotatedRect rr();
cv::RotatedRect rr2( rr1 );
cv::RotatedRect( p1, p2 ); // 通过两个点构建
cv::RotatedRect rr( p, sz, theta ); // 逐元素构建
rr.center; rr.size; rr.angle; // 成员访问
rr.points( pts[4] ); // 返回所有的顶点
```

## cv::Matx*
矩阵

```cpp
cv::Matx33f m33f; // 3 x 3 float
cv::Mat43f m43d; // 4 x 3 double

cv::Matx22d m22d( n22d ); // 通过一个已有的矩阵构建一个新矩阵，注意维度必须相同
cv::Matx21f m( x0, x1); // 通过值直接构建一个 2 x 1 的矩阵
// 通过值构建一个 4 x 4 矩阵
cv::Matx44d m(x0,x1,x2,x3,x4,x5,x6,x7,x8,x9,x10,x11,x12,x13,x14,x15);

m33f = cv::Matx33f::all( x ); // Matrix of identical elements
m23d = cv::Matx23d::zeros(); // 全 0 元素
m16f = cv::Matx16f::ones(); // 全 1 元素
m33f = cv::Matx33f::eye(); // create a unit matrix

// 构建一个能容纳对角线的矩阵
m31f = cv::Matx33f::diag();

// 通过指定元素范围，构建一个随机矩阵
cv::Matx33f::randu( min, max );

// Create a matrix with normally distributed entries
cv::Matx33f::nrandn( mean, variance );

m( i, j ), m( i ); // 成员访问

// 矩阵运算
m1 = m0; m0 * m1; m0 + m1; m0 - m1;
m * a; a * m; m / a;
// 比较
m1 == m2; m1 != m2;

// 点乘
m1.dot( m2 );
m1.ddot( m2 );

// Reshape a matrix
m91f = m33f.reshap<9, 1>();

m44f = (Matx44f)m44d;// 投射转换
m44f.get_minor<2, 2>( i, j ); // Extract 2 x 2 submatrix at (i, j)
// Extract row and column
m14f = m44f.row( i );
m14f = m44f.col( j );
// Extract matrix diagnoal
m41f = m44f.diag();
// Compute transpose
n44f = m44f.t();
// Invert matrix
n44f = m44f.inv( method );// default method is cv::DECOMP_LU

// Solve linear system
m31f = m33f.solve( rhs31f, method )
m32f = m33f.solve<2>( rhs32f, method ); default method is DECOMP_LU

m1.mul( m2);// per-element multiplication
```

## cv::Vec*

```cpp
// 集中构造方法
Vec2s v2s; Vec6f v6f;
Vec3f u3f( v3f );
Vec2f v2f( x0, x1 ); Vec4d v6d(x0, x1, x2, x3, 4, 5);
// 成员访问
v4f[ i ]; v3w( j ); 
// 叉乘
v3f.cross( u3f );
```

## cv::Complex*

```cpp
// 构造方法
cv::Complexf z1; cv::Complexd z2;
cv::Complexf z2( z1 );
cv::Complexd z1( re0 ); cv::Complexd( reo, im1 );
cv::Complexf u2f( v2f );

// 成员访问
z1.re; z1.im;

// Complex conjugate
z2 = z1.conj();
```

## 辅助类

- cv::TermCriteria
- cv::Range
- cv::Ptr
- cv::Exception
- cv::DataType<>
- cv::InputArray & cv::OutputArray 表示所有支持的序列类型，

## 实用函数

- cv::alignPtr() 按指定字节数对齐指针
- cv::alignSize() 按指定字节数对齐 buffer size
- cv::allocate() 创建 C 风格的数组
- cvCeil() 对 float x 向上取整，不会小于 x
- cv::cubeRoot() 
- cv::CV_Asser()
- CV_Error()
- CV_Error_()
- cv::deallocate()
- cv::error()
- cv::fastAtan2()
- cv::fastFree()
- cv::fastMalloc()
- cvFloor()
- cv::format()
- cv::getCPUTickCount()
- cv::getNumThreads()
- cv::getOptimalDFTSize()
- cv::getThreadNum()
- cv::getTickCount()
- cv::getTickFrequency()
- cvIsInf()
- cvIsNaN()
- cvRound()
- cv::setNumThreads()
- cv::setUserOptimized()
- cv::useOptimized()
