# IndoorNavigationAndroidWebSDK

3d-navigation-android-sdk 是一个室内导航SDK，支持院内3D地图展示、导航等功能。

## 獲取AppKey

**appKey mapId targetName targetId 請向richard.chin@locnavi.com 申請**

## AndroidX工程的Library引用
通过jitpack将github上的aar引入到工程中。
在setting.gradle中添加
```bash
    maven { url 'https://jitpack.io' }
```

在app的build.gradle中添加
```bash
    // use jitpack from github
    implementation 'com.github.locnavi:3d-navigation-android-sdk:0.0.11'
    implementation 'com.orhanobut:logger:2.2.0'
    implementation 'org.altbeacon:android-beacon-library:2.19.4'
```


## 加入权限
在AndroidMainfest.xxml中添加
```bash
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <!-- 调用定位权限 -->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <!-- 调用摄像头的权限 -->
    <uses-permission android:name="android.permission.CAMERA"/>
    <!-- 调用麦克风的权限 -->
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
```

## SDK的使用

### 初始化
在Application的onCreate方法中添加
```bash
        //初始化SDK
        LocNaviWebSDK.init(new LocNaviWebSDK.Configuration
                .Builder(this)
                .appKey(Constants.appKey)   //传入申请到的appKey
                .uploadApi("https://xxxx/putInfo")     //定时上传定位的api地址
                .uploadInterval(5000)   //定时上传时间间隔
                .uuids(new String[] {"FDA50693-A4E2-4FB1-AFCF-C6EB07647825"}) //设置蓝牙扫描的uuid
                .debug(true)    //控制是否使用测试地图
                .build());
```

### 显示地图列表
```js
    LocNaviWebSDK.openMapList(this);
```

### 显示室内地图
```js
    LocNaviWebSDK.openMap(this, mapId);
```

### 导航至具体地址
poi数据需要在导航系统中录入过
```js
    LocNaviWebSDK.openMap(this, mapId, poi);
```

### 开启定时上传定位功能
不在LocNaviWebSDK.init传值也可以单独设定定时上传参数（仅Webview加载前管用）
```js
    LocNaviWebSDK.setUploadLocationApi("https://xxxx.com/putinfo", 5000);
```
