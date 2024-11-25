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
Android support版本 请使用com.github.locnavi:android-beacon-library:2.19.4版本
```bash
    // use jitpack from github
    implementation 'com.github.locnavi:IndoorNavigationAndroidWebSDK:2.0.24'
    implementation 'com.orhanobut:logger:2.2.0'
    implementation 'org.altbeacon:android-beacon-library:2.19.4'
    //非地图页面定位时需要用到
    implementation 'com.squareup.okhttp3:okhttp:4.9.2'
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
    <uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30"/>
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30"/>
    <!-- Android 12 ibeacon扫描需要用到，动态授权 -->
    <uses-permission android:name="android.permission.BLUETOOTH_SCAN"/>
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
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

    //背景定位功能 需要
    service.setupForegroundService(R.mipmap.ic_launcher, "背景定位中", "详情信息");
    //自行调整背景定位时的扫描时长及扫描间隔
    service.updateScanPeriods(1100, 0, 1100, 0);

    //添加广播监听
    IntentFilter filter = new IntentFilter();
    filter.addAction("location")
    LocalBroadcastManager.getInstance(this).registerReceiver(locationReceiver, filter);

    //停止定位
    LocNaviLocationService service = LocNaviLocationService.getInstanceForApplication(this);
    service.stop();
    //停止背景定位
    service.stopForegroundService();
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

## 动态申请权限代码参考
LocNaviWebSDK内的webActivity新增了权限申请的代码，无界面定位模块的权限申请还是需要自行实现。以下提供参考代码。
```java
    //判断手机定位功能是否开启
    private void verifyLocation() {
        if (!this.isLocationEnabled()) {
            AlertDialog.Builder builder = new AlertDialog.Builder(this)
                    .setTitle("定位功能未开启")
                    .setMessage("室内定位需要定位功能开启后使用")
                    .setNegativeButton(android.R.string.cancel, null)
                    .setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            //openGPS(MainActivity.this);
                            //跳转GPS设置界面
                            Intent intent = new Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS);
                            startActivity(intent);
                        }
                    });
            builder.setCancelable(true);
            builder.show();
        }
    }

    //判断定位功能是否开启
    private boolean isLocationEnabled() {
        int locationMode = 0;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            try {
                locationMode = Settings.Secure.getInt(getContentResolver(), Settings.Secure.LOCATION_MODE);
            } catch (Settings.SettingNotFoundException e) {
                e.printStackTrace();
                return false;
            }
            return locationMode != Settings.Secure.LOCATION_MODE_OFF;
        } else {
            String locationProviders = Settings.Secure.getString(getContentResolver(), Settings.Secure.LOCATION_PROVIDERS_ALLOWED);
            return !TextUtils.isEmpty(locationProviders);
        }
    }

    private void requestPermissions() {
        // 检查和请求蓝牙和位置权限
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.BLUETOOTH_SCAN) != PackageManager.PERMISSION_GRANTED ||
                    ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
                // 请求蓝牙扫描和定位权限
                ActivityCompat.requestPermissions(this,
                        new String[]{
                                Manifest.permission.ACCESS_FINE_LOCATION,
                                Manifest.permission.BLUETOOTH_SCAN,
                                Manifest.permission.BLUETOOTH_CONNECT
                        },
                        REQUEST_CODE_BLUETOOTH_SCAN
                );
            }
        }
    }

        @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case REQUEST_FINE_LOCATION:
                if (grantResults.length >= 1 && grantResults[0] == PackageManager.PERMISSION_GRANTED && geolocationCallback != null) {
                    geolocationCallback.invoke(geolocationOrigin, true, false);
                }
                break;
            case REQUEST_CODE_BLUETOOTH_SCAN: {
                if (grantResults.length >= 1) {
                    if (grantResults[0] == PackageManager.PERMISSION_GRANTED && grantResults[1] == PackageManager.PERMISSION_GRANTED) {
                        //同意授权
                        this.startRangeBeacons();
                    } else {
                        String msg;
                        if (grantResults[0] != PackageManager.PERMISSION_GRANTED) {
                            msg = "请在设置页面开启【位置信息】权限。";
                        } else if (grantResults[1] != PackageManager.PERMISSION_GRANTED) {
                            msg = "请在设置页面开启【附近的设备】权限。";
                        } else {
                            msg = "请在设置页面开启【附近的设备】和【位置信息】权限。";
                        }
                        //提示开启授权
                        AlertDialog.Builder builder = new AlertDialog.Builder(this)
                                .setTitle("室内定位功能受限")
                                .setMessage(msg)
                                .setNegativeButton(android.R.string.cancel, null)
                                .setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
                                    @Override
                                    public void onClick(DialogInterface dialog, int which) {
                                        //openGPS(MainActivity.this);
                                        Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
                                        intent.setData(Uri.parse("package:" + getPackageName()));
                                        startActivity(intent);
                                        //跳转GPS设置界面
//                                        Intent intent = new Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS);
//                                        startActivity(intent);
                                    }
                                });
                        builder.setCancelable(true);
                        builder.show();
                    }
                }
            }
            break;

        }
    }
```
