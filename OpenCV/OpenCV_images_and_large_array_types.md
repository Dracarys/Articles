# 【笔记】OpenCv 之 Images 及大数组类型

## cv::Mat
用于描述任意维度的密集数组，例如图片。

- flags 用来标识数组的内容
- dims 表示维度
- rows、cols 分别表示行数和列数（2 维以上无效）
- data 数组指针
- refcount 引用计数，这个稍稍复杂一些，主要是为了避免频繁的拷贝数据而开创的。

```cpp
cv::Mat m;

// 创建数据区，大小为 3 x 10，每个元素类型是 3 通道的 32 位 float 类型。
m.create(3, 10, CV_32FC3);

// 设置每个像素的三个通道，分别为 1.0、0.0、1.0
m.setTo( cv::Scalar( 1.0f, 0.0f, 1.0f ));

// 声明的同时初始化
cv::Mat m(3, 10, CV_32FC3, cv::Scalar( 1.0f, 0.0f, 1.0f));

```
构造方法（无拷贝）：

```cpp
cv::Mat;
cv::Mat(int rows, int cols, int type);
cv::Mat(int rows, int cols, int type, const Scalar& s);
cv::Mat(int rows, int cols, int type, void* data, size_t step=AUTO_STEP);
cv::Mat(int rows, int cols, int type, void* data, size_t step=AUTO_STEP);
cv::Mat(cv::Size sz, int type);
cv::Mat(cv::Size sz, int type, const Scalar& s);
cv::Mat(cv::Size sz, int type, void* data, size_t step=AUTO_STEP);
cv::Mat(int ndims, const int* sizes, int type);
cv::Mat(int ndims, const int* sizes, int type, const Scalar& s);
cv::Mat(int ndims, const int* sizes, int type, void* data, size_t step=AUTO_STEP);
```

构造方法（拷贝）：

```cpp
cv::Mat(const Mat& mat);
cv::Mat(const Mat& mat, const cv::Range& rows, const cv::Range& cols);
cv::Mat(const Mat& mat, const cv::Rect& roi);
cv::Mat(const Mat& mat, const cv::Range* ranges);
cv::Mat(cosnt cv::MatExpr& expr);
```

构造方法（兼容2.1版）：

```cpp
cv::Mat(const CvMat* old, bool copyDate=false);
cv::Mat(cosnt IplImage* old, bool copyData=false);
```

模板构造器：

```cpp
cv::Mat(const cv::Vec<T, n>& vec, bool copyData=true);
cv::Mat(const cv::Matx<T, m, n>& vec, bool copyData=true);
cv::Mat(const std::vector<T>& vec, bool copyData=true);
```

静态函数构造：

```cpp
cv::Mat::zeros( rows, cols, type );
cv::Mat::ones( rows, cols, type );
cv::Mat::eye( rows, cols, type );
```

访问数组元素：

```cpp
cv::Mat m = cv::Mat::eye(10, 10, 32FC1);
printf("Element (3, 3) is %f\n", m.at<float>(3, 3));

cv::Mat m = cv::Mat::eye(10, 10, 32FC2);
printf("Element (3, 3) is (%f, %f)\n", 
	m.at<cv::Vec2f>(3, 3)[0],
	m.at<cv::Vec2f>(3, 3)[1]
);

cv::Mat m = cv::Mat::eye(10, 10, cv::DataType<cv::Complexf>::type);
printf(
	"Element (3, 3) is %f + i%f\n",
	m.at<cv::Complexf>(3, 3).re,
	m.at<cv::Complexf>(3, 3).im
);

// 元素访问方法
M.at<int>(i);
M.at<flaot>(i, j); // Element (i, j) from float array M
M.at<int>(pt); // Element at location (pt.x, pt.y) in integer matrix M
M.at<float>(i, j, k);// Element at location (i, j, k) in three-demensional float array M
M.at<uchar>(idx); // Element at n-dimensional location indicated by idx[] in array M of unsigned characters
```

### cv::NAryMatIterator

```cpp
// 构建一个 3 维数组
const int n_mat_size = 5;
const int n_mat_sz[] = {n_mat_size, n_mat_size, n_mat_size };
cv::Mat n_mat(3, n_mat_sz, CV32FC1);

// 向其中填充数据
cv::RNG rng;
rng.fill( n_mat, cv::RNG::UNIFORM, 0.f, 1.f );


// 首先，创建一个 C 数组，包含所有需要遍历的 mat
// ⚠️注意，这个数组必须是以 0 或 `NULL` 结尾
const cv::Mat* arrays[] = { &n_mat, 0 };
// 另一个 C 数组，用于指向各个平面
// ⚠️注意，这里仅有一个，所以是 1
cv::Mat my_planes[1];
// 所需参数准备就绪，创建迭代器
cv::NAryMatIterator it( arrays, my_planes );

// N-ary iterator 创建完毕，开始对 m0 和 m1 求和
// 
float s = 0.f; // 所有平面之和，为什么是 float ，注意回顾开始的声明
int n = 0;// 平面数
for (int p = 0; p < it.nplanes; p++, ++it) {
	s += cv::sum(it.planes[0])[0];
	n++;
}
```
通过 N 元迭代器对两个数组进行求和

```cpp
// 创建两个矩阵
const int n_mat_size = 5;
const int n_mat_sz[] = { n_mat_size, n_mat_size, n_mat_size };
cv::Mat n_mat0( 3, n_mat_sz, CV_32FC1 );
cv::Mat n_mat1( 3, n_mat_sz, CV_32FC1 );

// 填充数据
cv::RNG rng;
rng.fill( n_mat0, cv::RNG::UNIFORM, 0.f, 1.f );
rng.fill( n_mat1, cv::RNG::UNIFORM, 0.f, 1.f );

// 创建迭代器
const cv::Mat* arrays[] = { &n_mat0, &n_mat1, 0 };
cv::Mat my_planes[2];
cv::NAryMatIterator it( arrays, my_planes );

// 准备求和
float s = 0.f;// Total sum over all planes in both arrays
int   n = 0;// Total number of planes

// 迭代求和
for(int p = 0; p < it.nplanes; p++, ++it) {
  s += cv::sum(it.planes[0])[0];
  s += cv::sum(it.planes[1])[0];
  n++;
}
```

### 访问某一块元素

```cpp
m.row(i);
m.col(j);
m.rowRange(i0, i1);
m.rowRange(cv::Range(i0, i1));
m.colRange(j0, j1);
m.colRange(cv::Range(j0, j1));
m.diag(d);
m(cv::Range(i0, i1), cv::Range(j0, j1));
m(cv::Rect(i0, i1, w, h));
m(ranges);
```
### 矩阵表达式
主要涉及的是矩阵运算

```cpp
m0 + m1, m0 - m1;
m0 + s; m0 -s; s + m0, s - m1;// s is singleton
-m0;
s * m0; m0 * s;
m0.mul( m1 ); m0/m1;
m0 * m1;
m0.inv( method )// default value of method is DECOMP_LU
m0.t()// Matrix transpose of m0 (no copy is done)
m0 > m1; m0 >= m1; m0 == m1; m0 <= m1; m0 < m1;

m0&m1; m0|m1; m0^m1; ~m0;
m0&s; s&m0; m0|s; s|m0; m0^s; s^m0;

min(m0,m1); max(m0,m1); min(m0,s); 
min(s,m0); max(m0,s); max(s,m0);

cv::abs(m0);

m0.cross(m1); m0.dot(m1);

cv::Mat::eye( Nr, Nc, type ); 
cv::Mat::zeros( Nr, Nc, type );
cv::Mat::ones( Nr, Nc, type );
```

### 其它成员函数

```cpp
m1 = m0.clone();// 会产生完整拷贝
m0.copyTo(m1); // 与上面等价，如有必要会 reallocating m1                    m0.copyTo(m1, mask); // 只有 mask 数组中标记的才会拷贝
m0.convertTo(m1, type, scale, offset);
m0.assignTo(m1, type);// 尽显内部调用
// Set all entris in m0 to singleton value s;
// if mask is present, set only those values corresponding to nonzero elements in mask
m0.setTo(s, mask);
// Changes effective of a two-dimensional matrix; chan or rows may be zero, which implies "no change"; ;data is not copied
m0.reshape(chan, rows);
m0.push_back(s);
m0.push_back(m1);
m0.pop_back(n);

// Write whole size of m0 to cv::Size size;
// if m0 is a 'view' of a large matrix,
// write location of starting corner to Point& offset
m0.locateROI(size, offset);
// Increase the size of a view by t pixels above, b pixels below, l pixels to the left, and r pixels to the right
m0.adjustROI(t, b, l, r);
m0.total();// Compute the total number of array elements(does not include channels)
m0.isContinuous();// Return true only if the rows in m0 are packed without space between them in memory
m0.elemSize();// In bytes, e.g., a three-channel float matrix world return 12 bytes
m0.elemSize1();// Subelements of m0 in bytes. e.g., a three-channel float matrix would return 4 bytes
m0.type()
m0.depth();
m0.channels();
m0.size();// Return the size of the m0 as a cv::Size object
m0.empty();
```

## cv::SparseMat

线性代数计算？暂时跳过。