# IndoorNavigationAndroidWebSDK

IndoorNavigationAndroidWebSDK 是一个室内导航SDK，支持院内3D地图展示、导航等功能。
新开1.0*版本支持Android Support版本
0.0*及2.0*支持Android X版本

## 獲取AppKey

**appKey mapId targetName targetId 請向richard.chin@locnavi.com 申請**

## AndroidX工程的Library引用
通过jitpack将github上的aar引入到工程中。
在setting.gradle中添加
```bash
    maven { url 'https://jitpack.io' }
```

在app的build.gradle中添加
Android support版本 android-beacon-library需要使用2.16.2版本
```bash
    // use jitpack from github
    implementation 'com.github.locnavi:IndoorNavigationAndroidWebSDK:2.0.24'
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
```java
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
```java
    LocNaviWebSDK.openMapList(this);
```

### 显示室内地图
```java
    LocNaviWebSDK.openMap(this, mapId);
```

### 导航至具体地址
poi数据需要在导航系统中录入过
```java
    LocNaviWebSDK.openMap(this, mapId, poi);
```

### 显示室内地图并执行一些特殊操作
中文参数最好urlencode后再传入
```java
    LocNaviWebSDK.openMapWithParmas(this, mapId, "search=%E5%8E%95%E6%89%80&k=1,2,3");
```

### 关闭地图Activity
可以通过SDK获取地图的Activity，从而关闭地图界面
```java
    LocNaviWebMapActivity activity = LocNaviWebSDK.getMapActivity();
    if (activity != null) {
        //内部先尝试打开空白页面，再finish，据说可以避免内存泄漏
        activity.goBack();
    }
```

### 开启定时上传定位功能
不在LocNaviWebSDK.init传值也可以单独设定定时上传参数（仅Webview加载前管用）
```java
    LocNaviWebSDK.setUploadLocationApi("https://xxxx.com/putinfo", 5000);
```

### 导航事件-完成导航回调
LocNaviWebSDK添加监听器可以获取到H5传递过来的事件
```java
    LocNaviWebSDK.setNaviNavigationListener(new LocNaviNavigationListener() {
        @Override
        public void didFinishNavigation(LocNaviLocation location) {
                
        }
    });
```

### 不显示WebView时就能Beacon定位 (需要定位授权)
```java
    //提前设置相关参数
    LocNaviWebSDK.setMapId(Constants.mapId);
    //定位相关的url，一般情况可不用设置
    LocNaviLocationService service = LocNaviLocationService.getInstanceForApplication(this);
    service.setServerURL("https://l.locnavi.com");
    
    //开始定位
    LocNaviLocationService service = LocNaviLocationService.getInstanceForApplication(this);
    // service.start();
    //支持获取更详细的定位信息
    service.start(LocNaviConstants.LOCATION_MODE_AUTO, true);
    //可指定只开启蓝牙定位，暂时未使用GPS定位，默认使用LocNaviConstants.LOCATION_MODE_AUTO
    //service.start(LocNaviConstants.LOCATION_MODE_ONLY_BEACON);

    //设置扫描时长及扫描间隔(毫秒), 显示webview前建议改回较正常的扫描间隔以免影响体验。
    service.updateScanPeriods(1100, 0); //默认

    //添加广播监听
    IntentFilter filter = new IntentFilter();
    filter.addAction("location")
    LocalBroadcastManager.getInstance(this).registerReceiver(locationReceiver, filter);

    //停止定位
    LocNaviLocationService service = LocNaviLocationService.getInstanceForApplication(this);
    service.stop();
```


## 本地广播
目前添加了以下几个通知 取消(退出)导航：exit-navigation、退出路径规划：exit-route、 完成导航：navigation-done、 已在目的地：already-there、定位信息：location。
```java
    private BroadcastReceiver localReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {

            String str = intent.getStringExtra("data");
            try {
                JSONObject obj = new JSONObject(str);
                //数据解析
            } catch (JSONException e) {
                throw new RuntimeException(e);
            }
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //添加监听
        IntentFilter filter = new IntentFilter();
        filter.addAction("exit-navigation");
        filter.addAction("exit-route");
        filter.addAction("navigation-done");
        filter.addAction("already-there");
        LocalBroadcastManager.getInstance(this).registerReceiver(localReceiver, filter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        //移除监听
        LocalBroadcastManager.getInstance(this).unregisterReceiver(localReceiver);
    }
```
