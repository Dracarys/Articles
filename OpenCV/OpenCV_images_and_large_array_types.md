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

```


## cv::SparseMat