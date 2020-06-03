## OPENCV
### 1. opencv里的opencv文件夹里的文件与opencv2里的文件有何不同呢？

    前者是过去式（opencv 1.0），后者是现在时和未来时（opencv 2.x 和 opencv 3.x）。建议使用后者。
**note:** opencv4之后不再兼容opencv1了，所以如果需要编译含有1版本的老代码可以使用opencv 3.4.X

### 2. opencv源码编译注意事项
- 如果编译python module需要安装对应版本的python编译器（x86或x64），如果暂不需要python module，直接由不想修改cmakelists，可直接将python文件夹删除
- 若要编译非默认的x86，并且需要生成一个总的opencv_world.lib的话请遵循以下步骤:
```
mkdir build
cd build
cmake -DCMAKE_GENERATOR_PLATFORM=win32 -DBUILD_opencv_world=ON ..
cmake --build . --target install
```
**note:** 一定要安装，不然include里面需要的众多的头文件散落在各个module.并且即使我们没有设置`-DCMAKE_INSTALL_PATH`,也会把默认Install to ***/opencv-3.4.4/build/install。反而设置安装路径会报错，还没找到原因

### 3. cv::VideoWriter保存视频失败（0kb或打不开）
提示信息如
```
OpenCV: FFMPEG: tag 0x47504a4d / 'MJPG' is not supported with codec id 8 and format 'mp4 / MP4 (MPEG-4 Part 14)' 
OpenCV : FFMPEG : fallback to use tag 0x7634706d / 'mp4v'
```
一般是`.open(,fourcc,...)`第二个参数，编码格式设置不对，按照提示设置`CV_FOURCC('m','p', '4', 'v')`即可
<br>实例如：
```
cv::VideoWriter writer;
string savePath = "..../*.mp4";
...
writer.open(savePath, CV_FOURCC('m','p', '4', 'v'),fps, frameSize,true);
...
while(...)
{
       writer<<img;
}
writer.release();
...
```

### 4.将homographyMat转化为mapX,mapY供remap使用时注意事项
- remap中使用的mapX和mapY表示dst图像上对应在src上的坐标（目标点到源点的映射）
- warpPerspective使用的homography是源到目标的映射，与remap正好相反

### 5.opencv二维码检测器（`cv::QRCodeDetector`)对于宽高比相差较大时，检测不出来