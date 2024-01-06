---
title: 技术学习！Android对接华为AI--文本识别
date: 2024-01-06 20:00:00
categories:
  - Android
tags:
  - 技术学习
  - 华为AI
  - HMS Core SDK
  - AppGallery Connect
  - ML Kit云侧服务
description: Android对接华为AI--文本识别
cover: https://s2.loli.net/2024/01/06/KQBtcDz4jrbXhms.png
---
![image.png](https://s2.loli.net/2024/01/06/KQBtcDz4jrbXhms.png)
## 准备工作
在开发应用前：
- 1、需要在AppGallery Connect中配置相关信息，包括：注册成为开发者和创建应用。
- 2、使用ML Kit云侧服务（端侧服务可不开通）需要开发者在AppGallery Connect上打开ML Kit服务开关。

## 集成HMS Core SDK
工程根目录build.gradle文件

```undefined

buildscript {
    repositories {
        google()
        mavenCentral()
        maven { url 'https://jitpack.io' }
        // 配置HMS Core SDK的Maven仓地址。
        maven {url 'https://developer.huawei.com/repo/'}
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.0.4'
        classpath 'com.huawei.agconnect:agcp:1.6.2.300'
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
        maven { url 'https://jitpack.io' }
        // 配置HMS Core SDK的Maven仓地址。
        maven {url 'https://developer.huawei.com/repo/'}
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

app module下的build.gradle依赖华为基础SDK包与语言识别模型包：

```undefined
// 引入基础SDK
implementation 'com.huawei.hms:ml-computer-vision-ocr:3.11.0.301'
// 引入拉丁语文字识别模型包
implementation 'com.huawei.hms:ml-computer-vision-ocr-latin-model:3.11.0.301'
// 引入日韩语文字识别模型包
implementation 'com.huawei.hms:ml-computer-vision-ocr-jk-model:3.11.0.301'
// 引入中英文文字识别模型包
implementation 'com.huawei.hms:ml-computer-vision-ocr-cn-model:3.11.0.301'
```
## 配置混淆脚本

```undefined
-dontwarn com.huawei.**
-keep class com.huawei.** {*;}
-dontwarn org.slf4j.**
-keep class org.slf4j.** {*;}
-dontwarn org.springframework.**
-keep class org.springframework.** {*;}
-dontwarn com.fasterxml.jackson.**
-keep class com.fasterxml.jackson.** {*;}
-keep class com.huawei.noah.bolttranslator.**{*;}
-dontwarn com.huawei.hisi.**
-keep class com.huawei.hisi.** {*;}
```

## 添加权限
```undefined
    <!--相机权限-->
    <uses-permission android:name="android.permission.CAMERA" />
    <!--使用网络权限-->
    <uses-permission android:name="android.permission.INTERNET" />
    <!--写权限-->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <!--读权限-->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <!--录音权限-->
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <!--获取网络状态权限-->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <!--获取wifi状态权限-->
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
```
## 端侧识别

```undefined
    /**
     * 端侧文本识别
     */
    private void textAnalyzer() {
        long startTime =  (new Date()).getTime();
        Log.d("textAnalyzer", "start: " + startTime);
        MLLocalTextSetting setting = new MLLocalTextSetting.Factory()
                .setOCRMode(MLLocalTextSetting.OCR_DETECT_MODE)
                // 设置识别语种。
                .setLanguage("zh")
                .create();
        MLTextAnalyzer analyzer = MLAnalyzerFactory.getInstance().getLocalTextAnalyzer(setting);
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.test2);
        // 通过bitmap创建MLFrame，bitmap为输入的Bitmap格式图片数据。
        MLFrame frame = MLFrame.fromBitmap(bitmap);
        Task<MLText> task = analyzer.asyncAnalyseFrame(frame);
        task.addOnSuccessListener(new OnSuccessListener<MLText>() {
            @Override
            public void onSuccess(MLText text) {
                List<MLText.Block> blocks = text.getBlocks();
                StringBuilder sb = new StringBuilder();
                for (MLText.Block block : blocks) {
                    sb.append(block.getStringValue());
                }
                // 识别成功处理。
                tv.setText("识别成功: " + sb.toString());
                Log.d("textAnalyzer", "识别成功:\n " + sb.toString());
                long endTime =  (new Date()).getTime();
                Log.d("textAnalyzer", "end: " + endTime);
                Log.d("textAnalyzer", "耗时: " + (endTime-startTime));
            }
        }).addOnFailureListener(new OnFailureListener() {
            @Override
            public void onFailure(Exception e) {
                // 识别失败处理。
                tv.setText("识别失败");
            }
        });
    }
```

## 端侧识别测试

原图：
![image](https://github.com/KXHH2021/xiao.xiaopengw.com/assets/88917933/2c9e6748-dcba-4b06-9231-221a6d98dad2)


识别结果：
![image.png](https://s2.loli.net/2024/01/06/aqHxsYr7GD9uSA2.png)

## 云侧文本识别
- 注意：此功能收费,但是精确度更高。

# 配置应用的鉴权信息

先申请apikey:

https://developer.huawei.com/consumer/cn/console/api/credentials/dev388421841221889538

然后配置apikey

![image.png](https://s2.loli.net/2024/01/06/oQ81ljbuRcLq56F.png)

# 云侧文本识别实现
```undefined
    MLApplicationInit.init();

    /**
     * 云侧文本识别
     */
    private void textAnalyzerNet() {
        long startTime =  (new Date()).getTime();
        Log.d("textAnalyzerNet", "start: " + startTime);
        // 方式一：使用自定义参数配置。
        // 创建语言集合。
        List<String> languageList = new ArrayList();
        languageList.add("zh");
        languageList.add("en");
        // 设置参数。
        MLRemoteTextSetting setting = new MLRemoteTextSetting.Factory()
                // 设置云侧文本字体模式：
                // 若选择手写体格式，文本检测模式仅支持稀疏文本，语言列表仅支持中文（zh），边界框格式仅支持NGON四顶点坐标。
                // setTextDensityScene、setLnaguageList、setBorderType等方法设置均不生效。
                // 若选择印刷体格式，则需要手动设置语言列表，检测模式和边框样式，或采取默认配置。
                // MLRemoteTextSetting.OCR_HANDWRITTENFONT_SCENE：手写体。
                // MLRemoteTextSetting.OCR_PRINTFONT_SCENE：印刷体。
                .setTextFontScene(MLRemoteTextSetting.OCR_HANDWRITTENFONT_SCENE)
                // 设置云侧文本检测模式：
                // MLRemoteTextSetting.OCR_COMPACT_SCENE：文本密集场景的文本识别。
                // MLRemoteTextSetting.OCR_LOOSE_SCENE：文本稀疏场景的文本识别。
                .setTextDensityScene(MLRemoteTextSetting.OCR_LOOSE_SCENE)
                // 设置识别语言列表，使用ISO 639-1标准。
                .setLanguageList(languageList)
                // 设置文本边界框返回格式。
                // MLRemoteTextSetting.NGON：返回四边形的四个顶点坐标。
                // MLRemoteTextSetting.ARC：返回文本排列为弧形的多边形边界的顶点，最多可返回多达72个顶点的坐标。
                .setBorderType(MLRemoteTextSetting.ARC)
                .create();
        MLTextAnalyzer analyzer = MLAnalyzerFactory.getInstance().getRemoteTextAnalyzer(setting);
        // 方式二：使用默认参数配置，自动检测语种进行识别，适用于文本稀疏场景，文本框返回格式为：MLRemoteTextSetting.NGON。
        //MLTextAnalyzer analyzer = MLAnalyzerFactory.getInstance().getRemoteTextAnalyzer();
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.test2);
        // 通过bitmap创建MLFrame，bitmap为输入的Bitmap格式图片数据。
        MLFrame frame = MLFrame.fromBitmap(bitmap);
        Task<MLText> task = analyzer.asyncAnalyseFrame(frame);
        task.addOnSuccessListener(new OnSuccessListener<MLText>() {
            @Override
            public void onSuccess(MLText text) {
                // 识别成功。
                List<MLText.Block> blocks = text.getBlocks();
                StringBuilder sb = new StringBuilder();
                for (MLText.Block block : blocks) {
                    sb.append(block.getStringValue());
                    sb.append("\n");
                }
                // 识别成功处理。
                tv.setText("识别成功: " + sb.toString());
                Log.d("textAnalyzerNet", "识别成功:\n " + sb.toString());
                long endTime =  (new Date()).getTime();
                Log.d("textAnalyzerNet", "end: " +endTime);
                Log.d("textAnalyzerNet", "耗时: " + (endTime-startTime));
            }
        }).addOnFailureListener(new OnFailureListener() {
            @Override
            public void onFailure(Exception e) {
                // 识别失败,获取相关异常信息。
                try {
                    MLException mlException = (MLException) e;
                    // 获取错误码，开发者可以对错误码进行处理，根据错误码进行差异化的页面提示。
                    int errorCode = mlException.getErrCode();
                    // 获取报错信息，开发者可以结合错误码，快速定位问题。
                    String errorMessage = mlException.getMessage();
                } catch (Exception error) {
                    // 转换错误处理。
                }
            }
        });
    }
```
# 云侧文本识别测试

原图：

![image.png](https://s2.loli.net/2024/01/06/4rKFew1oRODkTlf.png)

识别结果：

![image.png](https://s2.loli.net/2024/01/06/eYtZTNmQ6kLqSfD.png)

官网参考

更多内容，请参考官网：

https://developer.huawei.com/consumer/cn/doc/hiai-Guides/text-recognition-0000001050040053
