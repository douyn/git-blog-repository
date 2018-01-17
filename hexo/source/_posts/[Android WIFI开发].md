---
title: [Android WIFI开发]
date: 2018年1月17日15:22:42
---
# [Android WIFI开发]

## 前言
在Android应用层的开发中，使用到wifi相关知识点的地方并不多，所以之前对wifi开发并不熟悉，最近接到的两个硬件项目都有用到wifi并且知识点越来越深入，所以有必要记一下笔记，做一些注释。
## 基本使用
#### 1.权限
Android中要使用系统功能一般都要申请权限，在6.0上可能还要手动申请权限，这里wifi需要的权限有

    	<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE"/>
        <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
        <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
        <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
        <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/> // 需要系统权限 [定位权限]
其中在6.0以上设备，定位权限需要主动申请，并且如果要获取扫描wifi列表需要打开系统的定位开关。
#### 2. WiFi相关API
![WIFI相关API](http://img.blog.csdn.net/20160301131454334)

ScanResult类用于存放wifi扫描结果信息，包含ssid(wifi名称), bssid(网络接入点地址),capabilities(加密类型),frequency(传输频率),level(信号强度)等。这里的解释并不太标准，但是对应功能很形象，如果你要了解更多，可以去具体的查阅资料。

WifiConfiguration类用于存放wifi的配置信息。包括wifi的密码，加密类型，网络id(用于连接wifi)等。他的几个子类分别对应秘钥加密方式，安全协议等，这些在设置wifi配置的时候会被用到

wifiinfo类用来描述wifi属性和连接状态。暴露了一些方法给开发者调用。getBSSID(), getMacAddress(), getIpAddress()，getSSID()等

WifiManager类是framework层暴露的api，用来管理wifi。通过调用 Context.getSystemService(Context.WIFI_SERVICE)可以得到类的实例。通过他可以得到:1.已经配置的网络列表。2.当前连接的wifi。3.扫描到的wifi。4.以及一些常量表示广播的意图等

#### 3. wifi状态及开关
	/**
     * 判断wifi是否打开
     * <p>需添加权限 {@code <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>}</p>
     *
     * @return {@code true}: 是<br>{@code false}: 否
     */
    public static boolean getWifiEnabled() {
        @SuppressLint("WifiManagerLeak")
        WifiManager wifiManager = (WifiManager) Utils.getApp().getSystemService(Context.WIFI_SERVICE);
        return wifiManager.isWifiEnabled();
    }

    /**
     * 打开或关闭wifi
     * <p>需添加权限 {@code <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>}</p>
     *
     * @param enabled {@code true}: 打开<br>{@code false}: 关闭
     */
    public static void setWifiEnabled(final boolean enabled) {
        @SuppressLint("WifiManagerLeak")
        WifiManager wifiManager = (WifiManager) Utils.getApp().getSystemService(Context.WIFI_SERVICE);
        if (enabled) {
            if (!wifiManager.isWifiEnabled()) {
                wifiManager.setWifiEnabled(true);
            }
        } else {
            if (wifiManager.isWifiEnabled()) {
                wifiManager.setWifiEnabled(false);
            }
        }
    }
#### 4. 扫描wifi

	 /**
     *
     * 获取WIFI列表
     * <p>需要权限{@code <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/> <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>}</p>
     * <p>注意Android6.0上需要主动申请定位权限，并且打开定位开关</p>
     *
     * @param context 上下文
     * @return wifi列表
     */
    public static List<ScanResult> getWifiList(Context context) {
        WifiManager wm = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
        List<ScanResult> scanResults = wm.getScanResults();

        Collections.sort(scanResults, new Comparator<ScanResult>() {
            @Override
            public int compare(ScanResult scanResult1, ScanResult scanResult2) {
                return scanResult2.level - scanResult1.level;
            }
        });
        return scanResultsCopy;
    }
	
     /**
     * 获取当前链接的WiFi信息
     *
     * @param context 上下文
     * @return 当前wifi数据
     */
    public static WifiInfo getCurrentWifi (Context context) {
        WifiManager wm = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
        return wm.getConnectionInfo();
    }
    
    
    public static String getWifiEncryptTypeStr (String capabilitie) {
        if (TextUtils.isEmpty(capabilitie)) return null;

        String encryptType;

        if (capabilitie.contains("WPA") && capabilitie.contains("WPA2")) {
            encryptType = "WPA/WPA2 PSK";
        } else if (capabilitie.contains("WPA2")) {
            encryptType = "WPA2 PSK";
        } else if (capabilitie.contains("WPA")) {
            encryptType = "WPA PSK";
        } else if (capabilitie.contains("WEP")) {
            encryptType = "WEP";
        } else {
            encryptType = "NONE";
        }

        return encryptType;
    }

    /**
     * wifi加密方式有5种
     * 0 - WPA/WPA2 PSK
     * 1 - WPA2 PSK
     * 2 - WPA PSK
     * 3 - WEP
     * 4 - NONE
     * @param capabilitie
     * @return
     */
    public static int getWifiEncryptType (String capabilitie) {
        if (TextUtils.isEmpty(capabilitie)) return -1;

        int encryptType;

        if (capabilitie.contains("WPA") && capabilitie.contains("WPA2")) {
            encryptType = 0;
        } else if (capabilitie.contains("WPA2")) {
            encryptType = 1;
        } else if (capabilitie.contains("WPA")) {
            encryptType = 2;
        } else if (capabilitie.contains("WEP")) {
            encryptType = 3;
        } else {
            encryptType = 4;
        }

        return encryptType;
    }

    
#### 5. 连接wifi

	 /**
     * @des 连接已经保存过配置的wifi
     * @param context
     * @param ssid
     */
    public static void connectWifi (Context context, String ssid) {

        Log.d(TAG, "connectWifi: 去连接wifi: " + ssid);

        if (!getWifiEnabled()) return;

        WifiManager wm = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
        WifiConfiguration wc = new WifiConfiguration();

        wc.SSID = "\"" + ssid + "\"";

        WifiConfiguration configuration = getWifiConfig(context, ssid);
        if (configuration != null) {
            wm.enableNetwork(configuration.networkId, true);
        }

    }

    /**
     * @des 连接没有配置过的wifi
     * @param context
     * @param ssid
     * @param password
     * @param encryptType
     */
    public static void connectWifi (Context context, String ssid, String password, int encryptType) {

        Log.d(TAG, "connectWifi: 去连接wifi: " + ssid);

        if (!getWifiEnabled()) return;

        WifiManager wm = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
        WifiConfiguration wc = new WifiConfiguration();
        wc.allowedAuthAlgorithms.clear();
        wc.allowedGroupCiphers.clear();
        wc.allowedKeyManagement.clear();
        wc.allowedPairwiseCiphers.clear();
        wc.allowedProtocols.clear();

        wc.SSID = "\"" + ssid + "\"";

        WifiConfiguration configuration = getWifiConfig(context, ssid);
        if (configuration != null) {
            wm.removeNetwork(configuration.networkId);
        }

        switch (encryptType) {
            case 4://不加密
                wc.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.NONE);
                break;

            case 3://wep加密
                wc.hiddenSSID = true;
                wc.wepKeys[0] = "\"" + password +"\"";
                wc.allowedAuthAlgorithms.set(WifiConfiguration.AuthAlgorithm.SHARED);
                wc.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.CCMP);
                wc.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.TKIP);
                wc.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.WEP40);
                wc.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.WEP104);
                wc.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.NONE);

                break;
            case 0: //wpa/wap2加密
            case 1: //wpa2加密
            case 2: //wpa加密

                wc.preSharedKey = "\"" + password + "\"";
                wc.hiddenSSID = true;
                wc.allowedAuthAlgorithms.set(WifiConfiguration.AuthAlgorithm.OPEN);
                wc.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.TKIP);
                wc.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.CCMP);
                wc.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.WPA_PSK);
                wc.allowedPairwiseCiphers.set(WifiConfiguration.PairwiseCipher.CCMP);
                wc.allowedPairwiseCiphers.set(WifiConfiguration.PairwiseCipher.TKIP);
                wc.status = WifiConfiguration.Status.ENABLED;
                break;
        }

        int network = wm.addNetwork(wc);
        wm.enableNetwork(network, true);
    }
    
    public static void disConnectWifi (Context context, int networkId) {
        WifiManager wm = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
        wm.disableNetwork(networkId);
        wm.disconnect();
    }
#### 6. 获取和清除WIFI配置

	/**
     * @des 清除wifi配置信息
     * @param context
     * @param ssid
     */
    public static void clearWifiInfo(Context context, String ssid) {

        Log.d(TAG, "clearWifiInfo: 清除WIFI配置信息: " + ssid);

        WifiManager wm = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);

        String newSSID = "\"" + ssid + "\"";

        if (!(ssid.startsWith("\"") && ssid.endsWith("\""))) {
            newSSID = "\"" + ssid + "\"";
        } else {
            newSSID = ssid;
        }

        WifiConfiguration configuration = getWifiConfig(context, newSSID);
        configuration.allowedAuthAlgorithms.clear();
        configuration.allowedGroupCiphers.clear();
        configuration.allowedKeyManagement.clear();
        configuration.allowedPairwiseCiphers.clear();
        configuration.allowedProtocols.clear();

        if (configuration != null) {

            wm.removeNetwork(configuration.networkId);
            wm.saveConfiguration();
        }
    }
    
    public static WifiConfiguration getWifiConfig (Context context, String ssid) {

        Log.d(TAG, "getWifiConfig: 获取wifi配置信息: " + ssid);

        if (TextUtils.isEmpty(ssid)) return null;

        String newSSID;

        if (!(ssid.startsWith("\"") && ssid.endsWith("\""))) {
            newSSID = "\"" + ssid + "\"";
        } else {
            newSSID = ssid;
        }

        WifiManager wm = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
        List<WifiConfiguration> configuredNetworks = wm.getConfiguredNetworks();

        for (WifiConfiguration configuration : configuredNetworks) {
            if (newSSID.equalsIgnoreCase(configuration.SSID)) {
                return configuration;
            }
        }

        return null;
    }
## 高级用法
#### 1. 监听wifi状态的变化并刷新wifi列表和连接状态

#### 2. 5Gwfif和2.4Gwifi
## 踩坑记录
#### 1. wifi的ssid是带有""，所以你如果要判空是不行的，并且在连接wifi的时候要手动给ssid加上""。

#### 2. 前文已经说了Android6.0以上要想扫描到wifi需要开启权限和手机的定位功能，不然得到的列表为空。

#### 3. wifi的秘钥安全模式网上并没有人给出标准的答案，各个手机厂商的wifi系统上也都不一样，对于我们开发者而言，主要有5种方式也可以总结为3中：NONE(无密码模式), WEP(安全性较低), WAP PSK, WAP2 PSK, WPA/WPA2 PSK(这三种安全性较高)。需要注意的是通过capabilitie属性得到的字符串是类似 [WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][ESS] 这样的，因为wifi的加密方式是安全模式+密码加密类型组合的方式。你要通过string.contains()方法解析出其中的安全模式，上文的扫描wifi中有解析的代码。

#### 4. 扫描列表中ssid可能为空或""，一般要剔除掉，而且wifi列表一般是根据信号强度排序。也有可能列表中出现同名的wifi,我之前遇到是因为现在的路由器同时支持5G wifi和2.4G wifi，两个wifi属于不同的频段。

#### 5. wifi是可以设置为隐藏wifi的，设置为隐藏wifi后不能被扫描到，但是可以通过直接输入ssid和秘钥的方式连接上。
## 后记
