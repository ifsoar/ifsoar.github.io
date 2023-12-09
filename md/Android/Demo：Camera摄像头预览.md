camera开发分辨率相关问题总结

[http://blog.csdn.net/jiayite/article/details/52039929](http://blog.csdn.net/jiayite/article/details/52039929)

```java
private int cameraCount;//摄像头个数

private int currentCameraIndex;//当前使用当摄像头index

private final float minScale = 1.7F;

private final float maxScale = 1.9F;

  

  

//设置屏幕高亮

private void keepScreenOnHighLight() {

//设置亮度最高

WindowManager.LayoutParams params = getWindow().getAttributes();

params.screenBrightness = 1f; //取值范围为0 - 1f

getWindow().setAttributes(params);

//保持常亮且亮度不变

getWindow().setFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON, WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);

}
```
  

  

初始化摄像头的方法
```
//初始化相机并开始预览  

private void initCamera() {

try {

//启用指定摄像头

camera = Camera.open(currentCameraIndex);

//获取该摄像头支持当预览尺寸

List sizes = camera.getParameters().getSupportedPreviewSizes();

Camera.Size previewSize = sizes.get(0);

//循环获取宽高比例在1.7到1.9之间的最大尺寸

for (Camera.Size size : sizes) {

Log.d("soar_", "size:" + size.width + "x" + size.height);

if (1F * size.width / size.height minScale 1F * size.width / size.height maxScale) {

previewSize = size;

break;

}

}

//设置画面旋转（该方法只有竖屏时使用，因为默认情况下摄像头参数都是横屏的）

// camera.setDisplayOrientation(90);

Camera.Parameters parameters = camera.getParameters();

//设置预览尺寸

parameters.setPreviewSize(previewSize.width, previewSize.height);

//设置对焦模式（否则不会自动对焦）

parameters.setFocusMode(Camera.Parameters.FOCUS_MODE_CONTINUOUS_PICTURE);

//应用上面的参数

camera.setParameters(parameters);

//设置预览surface

camera.setPreviewTexture(mSurfaceTexture);

//开始预览

camera.startPreview();

//取消自动对焦（与上面的自动对焦不冲突）

camera.cancelAutoFocus();

} catch (IOException e) {

e.printStackTrace();

}

}
```

  

停止的方法
```
//停止相机预览并释放资源

private void stopCameraAndRelease() {

if (camera != null) {

//停止预览并销毁相机

camera.stopPreview();

camera.release();

camera = null;

}

}

  
```

textureListener  
```
private class SurfaceTextureListener implements TextureView.SurfaceTextureListener {

  

//当textureView可用时

@Override

public void onSurfaceTextureAvailable(SurfaceTexture surfaceTexture, int i, int i1) {

mSurfaceTexture = surfaceTexture;

cameraCount = Camera.getNumberOfCameras();

if (cameraCount < 1) {

return;

}

currentCameraIndex = 0;

//初始化相机并开始预览

initCamera();

}

  

//尺寸变化（很少使用）

@Override

public void onSurfaceTextureSizeChanged(SurfaceTexture surfaceTexture, int i, int i1) {

}

  

//销毁时

@Override

public boolean onSurfaceTextureDestroyed(SurfaceTexture surfaceTexture) {

stopCameraAndRelease();

return true;

}

  

//update？？

@Override

public void onSurfaceTextureUpdated(SurfaceTexture surfaceTexture) {

  

}

}
```