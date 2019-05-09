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

// 
m.setTo( cv::Scalar( 1.0f, 0.0f, 1.0f ));
```

## cv::SparseMat