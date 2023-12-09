## [Android]接入虹软人脸检测、单目活体检测，sdk基于v2.+版本

---

[TOC]

### 1、准备工作

-   说明：
    -   1、请事先准备身份证照片等，用于实名认证，否则不能使用虹软最新的sdk，只能使用旧版v1.0，而旧版不支持活体检测
    -   2、虹软目前android最新版本号为v2.2，ios为v2.1，接入流程变化不大
    -   3、官网地址：[ai.arcsoft.com.cn](https://ai.arcsoft.com.cn/)

#### 1.1、注册帐号

> 略去不再详细说明，记得注册帐号之后进行实名认证，我认证的为个人实名，如果能够进行企业实名也可以.

#### 1.2、新建项目，下载sdk

进入开发者中心》我的应用。点击新建应用，输入项随意选择，没有什么影响。

新建完成后，点击右侧添加sdk，选择人脸识别（以为自2.0版本开始，人脸识别库已经内置了活体检测，所以不必独立集成）；  
  
之后的平台选择android，版本v2.2，语言java，应用选择我们刚刚新建的应用。确认即可。

点击sdk右侧的下载按钮，将sdk压缩包下载到本地，内含api文档、库本地和一个示例工程。

#### 1.3、集成sdk

将so以及jar复制到项目的对应位置，注意so仅支持v7a，然后在gradle文件中添加对so和jar的引用(我将so放在了libs\armeabi-v7a)

```java
android{
    sourceSets {
        main {
            //引入so
            jniLibs.srcDirs = ['libs']
            // 如果是单个文件夹 可以直接这样如下配置
            // jniLibs.srcDir 'libs'
        }
    }
}
dependencies{
    //人脸检测 https://www.arcsoft.com.cn/index.html
    implementation files('libs/arcsoft_face.jar')
}
```

执行sync，同步项目。

#### 1.4、实现相机预览并拿到摄像头的帧数据

> android想实现相机预览并拿到帧数据有多种方式，就不再一一列举，我这里选择了使用官方新出的jetpack组件-cameraX来实现。

值得注意的是，在以前的时候，onPreviewFrame拿到的图像帧格式为nv21，而我使用cameraX的图像分析拿到的为YUV_420_888格式，需要进行转换

```java
ImageAnalysisConfig analysisConfig =...
ImageAnalysis analysis = new ImageAnalysis(analysisConfig);
analysis.setAnalyzer(new ImageAnalysis.Analyzer() {
                @Override
                public void analyze(ImageProxy image, int rotationDegrees) {
                        int width = image.getWidth();
                        int height = image.getHeight();
                        byte[] bytes = YUVUtil.yuv_420_888to_nv21(image.getImage());
                        image.close();
                }
            });
```

转换工具类如下

```java
public class YUVUtil {
    public static byte[] yuv_420_888to_nv21(Image image) {

        int width = image.getWidth();
        int height = image.getHeight();
        int ySize = width * height;
        int uvSize = width * height / 4;

        byte[] nv21 = new byte[ySize + uvSize * 2];

        ByteBuffer yBuffer = image.getPlanes()[0].getBuffer(); // Y
        ByteBuffer uBuffer = image.getPlanes()[1].getBuffer(); // U
        ByteBuffer vBuffer = image.getPlanes()[2].getBuffer(); // V

        int rowStride = image.getPlanes()[0].getRowStride();
        if (image.getPlanes()[0].getPixelStride() != 1) {
            return null;
        }

        int pos = 0;

        if (rowStride == width) { // likely
            yBuffer.get(nv21, 0, ySize);
            pos += ySize;
        } else {
            int bufferPos = yBuffer.position();
            for (; pos < ySize; pos += width) {
                yBuffer.position(bufferPos);
                yBuffer.get(nv21, pos, width);
                bufferPos += rowStride;
            }
        }

        rowStride = image.getPlanes()[2].getRowStride();
        int pixelStride = image.getPlanes()[2].getPixelStride();

        assert (rowStride == image.getPlanes()[1].getRowStride());
        assert (pixelStride == image.getPlanes()[1].getPixelStride());

        if (pixelStride == 2 && rowStride == width && uBuffer.get(0) == vBuffer.get(1)) {
            // maybe V an U planes overlap as per NV21, which means vBuffer[1] is alias of uBuffer[0]
            byte savePixel = vBuffer.get(1);
            vBuffer.put(1, (byte) 0);
            if (uBuffer.get(0) == 0) {
                vBuffer.put(1, (byte) 255);
                if (uBuffer.get(0) == 255) {
                    vBuffer.put(1, savePixel);
                    vBuffer.get(nv21, ySize, uvSize);

                    return nv21; // shortcut
                }
            }

            // unfortunately, the check failed. We must save U and V pixel by pixel
            vBuffer.put(1, savePixel);
        }

        // other optimizations could check if (pixelStride == 1) or (pixelStride == 2),
        // but performance gain would be less significant

        for (int row = 0; row < height / 2; row++) {
            for (int col = 0; col < width / 2; col++) {
                int vuPos = col * pixelStride + row * rowStride;
                nv21[pos++] = vBuffer.get(vuPos);
                nv21[pos++] = uBuffer.get(vuPos);
            }
        }

        return nv21;
    }
}
```

### 2、实现人脸检测

#### 2.1、初始化虹软的人脸引擎

初始化分两部分，在线激活（官方说仅执行一次即可）和初始化能力。

这里我选择将初始化的两步工作都放在当前页面的onCreate中执行，其实可以将在线激活放在Application的onCreate中进行，而将初始化能力放在当前页面，我选择放在一起。

```java
faceEngine = new FaceEngine();
//两个参数从网站控制台获取
//这里的返回值
int ret = faceEngine.activeOnline(App.getApp(), appId, sdkKey);
if (ret != ErrorInfo.MOK && ret != ErrorInfo.MERR_ASF_ALREADY_ACTIVATED) {
    Log.d(TAG, "faceEngine.activeOnline fail" + ret);
    faceEngine = null;
    return;
}
//第二个参数为检测模式，有图片和视频两种模式，这里为视频模式
//第三个参数为识别角度，这里为全向检测，也可以指定固定角度检测，则人脸倾斜就检测不到
//第四个参数为图片宽边与人脸宽边的最小比值，eg：前置摄像头，分辨率720x1280，宽边为垂直边，这里设置为6，也就是说画面中人脸最小尺寸为1280/6=213.3分辨率，不足的时候认为检测不到人脸，该参数可以用来限制人脸距离屏幕太远就被检测到的问题，范围2-32
//第五个参数为最大检测人脸个数，特别要说明的是进行活体检测时最大支持1个人脸，即使传递了大于1的参数，在之后的活体检测中，除第一个之外的人脸的活体信息都为未知
ret = faceEngine.init(App.getApp(), FaceEngine.ASF_DETECT_MODE_VIDEO, FaceEngine.ASF_OP_0_HIGHER_EXT, 6, 1, FaceEngine.ASF_FACE_DETECT | FaceEngine.ASF_LIVENESS);//这里初始化了活体检测和人脸检测两个能力，还有性别、年龄等其他检测项，以api文档为准
if (ret != ErrorInfo.MOK) {
    Log.d(TAG, "faceEngine.init fail" + ret);
    faceEngine = null;
    return;
}
```

#### 2.2、从图像帧中得到人脸数量信息

在analyze()方法中增加如下逻辑，进行人脸检测

```java
List faceInfoList = ...;
List livenessInfoList = ...;

@Override
public void analyze(ImageProxy image, int rotationDegrees) {
        if (faceEngine == null) {
            Log.d(TAG, "faceEngine == null");
            image.close();
            return;
        }
        int width = image.getWidth();
        int height = image.getHeight();
        byte[] bytes = YUVUtil.yuv_420_888to_nv21(image.getImage());
        image.close();
        faceInfoList.clear();
        livenessInfoList.clear();
        int ret;
        ret = faceEngine.detectFaces(bytes, width, height, FaceEngine.CP_PAF_NV21, faceInfoList);
        if (ret != ErrorInfo.MOK) {
            Log.d(TAG, "faceEngine.detectFaces" + ret);
            return;
        }
        if (faceInfoList.size() != 1) {
            Log.d(TAG, "faceInfoList.size1() != 1:" + faceInfoList.size());
            return;
        }
        //TODO do something     
}
```

### 3、实现单目活体检测

在上一步后，就已经可以拿到人脸个数和尺寸等数据了，继续执行如下操作即可拿到活体检测数据

-   注意，上一步如果拿到的人脸个数超过1个，一定要将faceInfoList中的数据删除到只剩一个，否则不能进行活体检测

```java
public void analyze(ImageProxy image, int rotationDegrees) {
        if (faceEngine == null) {
            Log.d(TAG, "faceEngine == null");
            image.close();
            return;
        }
        int width = image.getWidth();
        int height = image.getHeight();
        byte[] bytes = YUVUtil.yuv_420_888to_nv21(image.getImage());
        image.close();
        faceInfoList.clear();
        livenessInfoList.clear();
        int ret;
        ret = faceEngine.detectFaces(bytes, width, height, FaceEngine.CP_PAF_NV21, faceInfoList);
        if (ret != ErrorInfo.MOK) {
            Log.d(TAG, "faceEngine.detectFaces" + ret);
            return;
        }
        if (faceInfoList.size() != 1) {
            Log.d(TAG, "faceInfoList.size1() != 1:" + faceInfoList.size());
            return;
        }
        ret = faceEngine.process(bytes, width, height, FaceEngine.CP_PAF_NV21, faceInfoList, FaceEngine.ASF_LIVENESS);
        if (ret != ErrorInfo.MOK) {
            Log.d(TAG, "faceEngine.process" + ret);
            return;
        }
        if (faceInfoList.size() != 1) {
            Log.d(TAG, "faceInfoList.size2() != 1:" + faceInfoList.size());
            return;
        }
        ret = faceEngine.getLiveness(livenessInfoList);
        if (ret != ErrorInfo.MOK) {
            Log.d(TAG, "faceEngine.getLiveness" + ret);
            return;
        }
        if (livenessInfoList.size() < 1) {
            Log.d(TAG, "livenessInfoList.size() < 1:" + livenessInfoList.size());
            return;
        }
        LivenessInfo livenessInfo = livenessInfoList.get(0);
        if (livenessInfo.getLiveness() != LivenessInfo.ALIVE) {
            Log.d(TAG, "livenessInfo.getLiveness() != LivenessInfo.ALIVE" + livenessInfo.getLiveness());
            return;
        }
        //TODO do something
}
```

### 4、测试&其他

到此为止已经能够检测到一个人脸并且保证是活体人脸数据了，之后就可以继续操作了。我们的用法是确认为活体人脸后从原数据中裁剪出人脸照片，再保存成图片到本地。这里需要用到nv21转bitmap，再将bitmap存为图片的工具类。

```java
    public static Bitmap nv21ToBitmap(byte[] data, int width, int height) {
        YuvImage image = new YuvImage(data, ImageFormat.NV21, width, height, null);
        ByteArrayOutputStream stream = new ByteArrayOutputStream();
        image.compressToJpeg(new Rect(0, 0, width, height), 80, stream);
        return BitmapFactory.decodeByteArray(stream.toByteArray(), 0, stream.size());
    }
```

​