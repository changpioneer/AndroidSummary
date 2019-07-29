基于android6.0.7.01.20

1. 默认使用Launcher2，修改Launcher2  
packages/apps/Launcher3/src/com/android/launcher3/Launcher.java

在onResume()的函数最后调用startFleetyMainActivity()
 

```java
   private void startFleetyMainActivity()
    {
        try
        {
            String action = "com.fleety.android.MAIN";
            System.out.println("Try to start fleety main, action =" + action);
            Intent intent = new Intent(action);
            Launcher.this.startActivity(intent);
        } catch (Exception e)
        {
            System.err.println("Fleety apk not installed");
        }
    }
```


2. sqlite3编译进USER版本
external/sqlite/dist/Android.mk中
修改成下面代码，
optional:指该模块在所有版本下都编译

```xml
LOCAL_MODULE_TAGS := optional

LOCAL_MODULE := sqlite3
```

3. 禁止休眠
frameworks/base/packages/SettingsProvider/res/values/defaults.xml

```xml
<integer name="def_screen_off_timeout">-1</integer>
<bool name="def_lockscreen_disabled">false</bool>
<bool name="def_device_provisioned">false</bool>
```

4. 默认禁止屏幕旋转
frameworks/base/packages/SettingsProvider/res/values/defaults.xml

```xml
<bool name="def_accelerometer_rotation">false</bool>设置为false

    <!-- Default for Settings.System.USER_ROTATION -->
    <integer name="def_user_rotation">0</integer>
取值：0,1,2,3（分别对应0°，90°，180°，270°）
```

5、允许未知源安装
frameworks/base/packages/SettingsProvider/res/values/defaults.xml

```xml
<!-- fleety chang modify false to true -->
<bool name="def_install_non_market_apps">false</bool>改为true
```

6、无需验证
frameworks/base/packages/SettingsProvider/res/values/defaults.xml

```xml
 <bool name="def_package_verifier_enable">true</bool>改为false
```

7、打开GPS
frameworks/base/packages/SettingsProvider/res/values/defaults.xml

```xml
<string name="def_location_providers_allowed" translatable="false">gps</string>
    <bool name="assisted_gps_enabled">true</bool>
```

8、关闭锁屏
frameworks/base/packages/SettingsProvider/res/values/defaults.xml

```xml
<bool name="def_lockscreen_disabled">false</bool>改为true
```

9. 禁止开机SIM卡相关信息Dialog
vendor/mediatek/proprietary/packages/apps/Stk/src/com/android/stk/StkAppService.java
的handleCmd(CatCmdMessage cmdMsg, int slotId)方法中注释掉type为DISPLAY_TEXT中launchTextDialog(slotId, false);


10. ANR自动确认
frameworks/base/services/core/java/com/android/server/am/AppNotRespondingDialog.java
中重载onCreate()方法，构建时发布一个关闭事件，然后在mHandler的关闭事件中关闭对话框，即在FORCE_CLOSE中添加
AppNotRespondingDialog.this.dismiss();

代码：


```java
import android.os.Bundle;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    mHandler.obtainMessage(FORCE_CLOSE, this).sendToTarget();
    Slog.d(TAG, "ANR send Message to close Dialog:"+this);
    }

private final Handler mHandler = new Handler() {
        public void handleMessage(Message msg) {
            Intent appErrorIntent = null;
            switch (msg.what) {
                case FORCE_CLOSE:
                    // Kill the application.
                    mService.killAppAtUsersRequest(mProc, AppNotRespondingDialog.this);
            // fleety min.chu add start
            
            AppNotRespondingDialog.this.dismiss();
                // fleety min.chu add end
            break;
```

11. 修改设备权限，添加开机启动项（未改）

device/mediatek/mt6735/init.mt6735.rc
中chmod 0660 /dev/ttyGS2这行下面添加
 # fleety min.chu add start
    chmod 0777 /dev/ttyMT0
    chmod 0777 /dev/ttyMT1
    chmod 0777 /dev/ttyMT2
    chmod 0777 /dev/ttyMT3
    chmod 0777 /dev/mtgpio
    chmod 0777 /dev/gps
    chmod 0755 /system/etc/nat.sh
    # fleety min.chu add end

和service MtkCodecService /system/bin/MtkCodecService下面添加
fleety min.chu add start
service SuAgentServer /system/bin/SuAgentServer 60001
    class main
    oneshot
   disabled
 fleety min.chu add end



12、 uboot logo、  kernel、Android is starting,startApp
1024*600：
/vendor/mediatek/proprietary/bootable/bootloader/lk/dev/logo/wsvganl/wsvganl_uboot.bmp

800×480：
vendor/mediatek/proprietary/bootable/bootloader/lk/dev/logo/wvgalnl/wvgalnl_uboot.bmp

修改启动时Android is starting...提示框为Launcher is starting...
frameworks/base/services/core/java/com/android/server/pm/PackageManagerService.java
/frameworks/base/core/res/res/values/strings.xml
frameworks/base/services/core/java/com/android/server/policy/PhoneWindowManager.java

PackageManagerService.java中弹出提示框：
        

```java
if (doTrim) {
                    if (!isFirstBoot()) {
                        try {
                            ActivityManagerNative.getDefault().showBootMessage(
                                    mContext.getResources().getString(
                                            R.string.android_upgrading_fstrim), true);
                        } catch (RemoteException e) {
                        }
                    }
                    ms.runMaintenance();
                }
```

strings.xml中设置提示框的内容

```xml
<!-- fleety chang modify <Android to Launcher> -->
    <!-- [CHAR LIMIT=40] Title of dialog that is shown when system is starting. -->
    <string name="android_start_title">Launcher is starting\u2026</string>
```

13. 默认联网

vendor/mediatek/proprietary/frameworks/base/packages/FwkPlugin/src/com/mediatek/op/telephony/TelephonyExt.java

```java
public boolean isDefaultDataOn() {
        // fleety min.chu modify (false to true)
    return true;
    }

    public boolean isAutoSwitchDataToEnabledSim() {
        // fleety min.chu modify (false to true)
        return true;
    }
    public boolean isDefaultEnable3GSIMDataWhenNewSIMInserted() {
        // fleety min.chu modify (false to true)
        return true;
    }
```

14. 自动安装APK

packages/apps/PackageInstaller/src/com/android/packageinstaller/InstallAppProgress.java
packages/apps/PackageInstaller/src/com/android/packageinstaller/PackageInstallerActivity.java

InstallAppProgress.java mHandler中给

```java
mLaunchButton.setOnClickListener(InstallAppProgress.this);添加if-else如下：
                // fleety min.chu modify (add if-false)
                            if (false)
                            {
                                mLaunchButton.setOnClickListener(InstallAppProgress.this);
                            } else {
                                startActivity(mLaunchIntent);
                                InstallAppProgress.this.finish();
                            }
                            // fleety min.chu modify (add if-false) end
```

PackageInstallerActivity.java中startInstallConfirm()方法最后加上install()方法

 

```java
private void install() {
        // Start subactivity to actually install the application
        mInstallFlowAnalytics.setInstallButtonClicked();
        Intent newIntent = new Intent();
        newIntent.putExtra(PackageUtil.INTENT_ATTR_APPLICATION_INFO,
                mPkgInfo.applicationInfo);
        newIntent.setData(mPackageURI);
        newIntent.setClass(this, InstallAppProgress.class);
        newIntent.putExtra(InstallAppProgress.EXTRA_MANIFEST_DIGEST, mPkgDigest);
        newIntent.putExtra(
                InstallAppProgress.EXTRA_INSTALL_FLOW_ANALYTICS, mInstallFlowAnalytics);
        String installerPackageName = getIntent().getStringExtra(
                Intent.EXTRA_INSTALLER_PACKAGE_NAME);
        if (mOriginatingURI != null) {
            newIntent.putExtra(Intent.EXTRA_ORIGINATING_URI, mOriginatingURI);
        }
        if (mReferrerURI != null) {
            newIntent.putExtra(Intent.EXTRA_REFERRER, mReferrerURI);
        }
        if (mOriginatingUid != VerificationParams.NO_UID) {
            newIntent.putExtra(Intent.EXTRA_ORIGINATING_UID, mOriginatingUid);
        }
        if (installerPackageName != null) {
            newIntent.putExtra(Intent.EXTRA_INSTALLER_PACKAGE_NAME,
                    installerPackageName);
        }
        if (getIntent().getBooleanExtra(Intent.EXTRA_RETURN_RESULT, false)) {
            newIntent.putExtra(Intent.EXTRA_RETURN_RESULT, true);
            newIntent.addFlags(Intent.FLAG_ACTIVITY_FORWARD_RESULT);
        }
        if (localLOGV)
            Log.i(TAG, "downloaded app uri=" + mPackageURI);
        startActivity(newIntent);
        finish();
    }
```

15. 去除挂断电话声音
    packages/services/Telephony/src/com/android/phone/CallNotifier.java

switch语句中type为TONE_CALL_ENDED时：
             

```java
       //toneType = ToneGenerator.TONE_PROP_PROMPT;
                    //toneVolume = TONE_RELATIVE_VOLUME_HIPRI;
                    toneVolume=0;
                    /* fleety John Fan delete end */
                    toneLengthMillis = 200;
```


16. 关机接口（有问题）
/frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java

方法systemReady(IAppOpsService appOps)中所有registerReceiver（）下面添加如下代码：
            

```java
filter = new IntentFilter();
            filter.addAction("com.fleety.android.shutdown");
            filter.addAction("com.fleety.android.sleep");
            filter.addAction("com.fleety.android.wakeup");
            filter.addAction("com.fleety.android.reboot");
            mContext.registerReceiver(new BroadcastReceiver() {
                @Override
                public void onReceive(Context context, Intent intent) {
                    String action = intent.getAction();
                    System.out.println("receive action: " + action);
                    if (action.equals("com.fleety.android.shutdown"))
                        shutdownOrRebootInternal(true, false, null, false);
                    // shutdown(false, false);
                    else if (action.equals("com.fleety.android.sleep")) {
                        long time = intent.getLongExtra("time", 0);
                        int reason = intent.getIntExtra("reason", 0);
                        System.out.println("goToSleep(" + time + ", " + reason
                                + ")");
                        goToSleepInternal(time, reason,
                                PowerManager.GO_TO_SLEEP_FLAG_NO_DOZE,
                                Process.SYSTEM_UID);
                        // goToSleep(time, reason);
                    } else if (action.equals("com.fleety.android.wakeup")) {
                        long time = intent.getLongExtra("time", 0);
                        System.out.println("wakeUp(" + time + ")");
                        wakeUpInternal(time, "android.server.power:POWER",
                                Process.SYSTEM_UID,
                                mContext.getOpPackageName(), Process.SYSTEM_UID);
                        // wakeUp(time);
                    } else if (action.equals("com.fleety.android.reboot")) {
                        long time = intent.getLongExtra("time", 0);
                        System.out.println("reboot(" + time + ")");
                        
                        shutdownOrRebootInternal(false, false, null, false);
                    }
                }
            }, filter, null, mHandler);
            // fleety min.chu add end
```

17. 去除插入U盘和TF卡时弹出的“修改默认存储器”对话框
    /frameworks/base/services/core/java/com/android/server/MountService.java
注释掉mContext.startActivity(intent);如下：

```java
Intent intent = new Intent("com.mediatek.storage.StorageDefaultPathDialog");
                                        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                                        if (path.equals(EXTERNAL_OTG)) {
                                            intent.putExtra(INSERT_OTG, true);
                                        }
                    // fleety min.chu comment (next one line)
                                        // mContext.startActivity(intent);
```

18. android按键广播(在任何界面，R600都可以接受到按键事件)
    frameworks/base/core/java/android/view/ViewRootImpl.java
processKeyEvent()方法中添加fleetyKeyEventBroadcaster(event)方法

```java
private int processKeyEvent(QueuedInputEvent q) {
            final KeyEvent event = (KeyEvent)q.mEvent;
          //add by fleety John Fan
            fleetyKeyEventBroadcaster(event);
            
            // If the key's purpose is to exit touch mode then we consume it
            // and consider it handled.
            if (checkForLeavingTouchModeAndConsume(event)) {
                return FINISH_HANDLED;
            }

            // Make sure the fallback event policy sees all keys that will be
            // delivered to the view hierarchy.
            mFallbackEventHandler.preDispatchKeyEvent(event);
            return FORWARD;
        }
        
        /**
        Add by fleety John Fan.
        */
        private void fleetyKeyEventBroadcaster(KeyEvent event){
            //android.util.Log.d("fleety_keyevent", "sendBroadcast event="+event);
            android.content.Intent intent=new android.content.Intent();
                intent.setAction("com.fleety.android.ACTION_KEYEVENT");
                android.os.Bundle extras=new Bundle();
                extras.putParcelable("R600UTIL_DATA_KEYEVENT", event);
                intent.putExtras(extras);
                mContext.sendBroadcast(intent);
            
        }
```

19. 屏蔽Home键功能
/frameworks/base/services/core/java/com/android/server/policy/PhoneWindowManager.java
handleShortPressOnHome()方法中注释掉launchHomeFromHotKey()方法


20. 增加TF卡、U盘写权限
    frameworks/base/data/etc/platform.xml
添加

```xml
    <!-- fleety min.chu add(media_rw)-->
    <permission name="android.permission.WRITE_EXTERNAL_STORAGE" >
        <group gid="sdcard_r" />
        <group gid="sdcard_rw" />
        <group gid="media_rw" />
    </permission>

    <!-- fleety min.chu add(media_rw)-->
    <permission name="android.permission.ACCESS_ALL_EXTERNAL_STORAGE" >
        <group gid="sdcard_r" />
        <group gid="sdcard_rw" />
        <group gid="sdcard_all" />
        <group gid="media_rw" />
    </permission>
```

21. 屏蔽系统电量低相关处理
frameworks/base/services/core/java/com/android/server/BatteryService.java
frameworks/base/core/res/res/values/config.xml

BatteryService.java中注释掉mContext.sendBroadcastAsUser(statusIntent, UserHandle.ALL);

```java
 if (shouldSendBatteryLowLocked()) {
                mSentLowBatteryBroadcast = true;
                mHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        Intent statusIntent = new Intent(Intent.ACTION_BATTERY_LOW);
                        statusIntent.setFlags(Intent.FLAG_RECEIVER_REGISTERED_ONLY_BEFORE_BOOT);
                        //mContext.sendBroadcastAsUser(statusIntent, UserHandle.ALL);
                    }
                });


    <!-- fleety min.chu modify next line(value to 0) -->
    <integer name="config_criticalBatteryWarningLevel">0</integer>

    <!-- fleety min.chu modify next line(value to 0) -->
    <integer name="config_lowBatteryWarningLevel">0</integer>

    <!-- Close low battery warning when battery level reaches the lowBatteryWarningLevel
         plus this -->
    <integer name="config_lowBatteryCloseWarningBump">0</integer>
```

22、强制横屏设置
frameworks/base/services/core/java/com/android/server/wm/WindowManagerService.java
updateOrientationFromAppTokensLocked(boolean inTransaction)方法中，

```java
 if (req != mForcedAppOrientation) {
                mForcedAppOrientation = req;
                //send a message to Policy indicating orientation change to take
                //action like disabling/enabling sensors etc.,
                mPolicy.setCurrentOrientationLw(req);
                if (updateRotationUncheckedLocked(inTransaction)) {
                    // changed
                    return true;
                }
            }
在上述代码的前面添加
    // fleety chang add start
        req = ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE;
    // fleety chang add end
```

23、APK可以降版本升级
frameworks/base/services/core/java/com/android/server/pm/PackageManagerService.java
installLocationPolicy(PackageInfoLite pkgLite)方法中
    

```java
        // Check for downgrading.
                        if ((installFlags & PackageManager.INSTALL_ALLOW_DOWNGRADE) == 0) {
            
                // Fleety chang modify start <upgrade ignore apk version>
                if(false)
                {
                                    try {
                                      checkDowngrade(pkg, pkgLite);
                                    } catch (PackageManagerException e) {
                                        Slog.w(TAG, "Downgrade detected: " + e.getMessage());
                                        return PackageHelper.RECOMMEND_FAILED_VERSION_DOWNGRADE;
                                }
                }
                // Fleety chang modify start
                       
```

 }
24. 隐藏/显示状态栏接口

新方法：
屏蔽系统底部虚拟按键（参考http://blog.csdn.net/qq_35981307/article/details/73843704）

frameworks/base/services/core/Java/com/android/server/policy/PhoneWindowManager.java
在PhoneWIndowManager.java文件中有如下代码：

```java
String navBarOverride = SystemProperties.get("qemu.hw.mainkeys");
if ("1".equals(navBarOverride)) {
    mHasNavigationBar = false;
} else if ("0".equals(navBarOverride)) {
    mHasNavigationBar = true;
}
```

mHasNavigationBar的值即是否隐藏底部虚拟按键，false为隐藏，所以在此处我采取的操作是mHasNavigationBar不执行判断，默认执行false 

```java
    //fleety chang modify  start <hide virtual buttons at the bottom of the system>
        //if ("1".equals(navBarOverride)) {
            mHasNavigationBar = false;
       // } else if ("0".equals(navBarOverride)) {
           // mHasNavigationBar = true;
        //}

    //fleety chang modify end
```

2、屏蔽系统顶部的状态栏和通知栏
主要两部分：上方的状态栏不显示、下拉不进行操作
（1）、屏蔽手势监听事件
frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/phone/PhoneStatusBarView.java
在以上路径下的PhoneStatusBarView.java文件中进行修改 

```java
 @Override
    public boolean onTouchEvent(MotionEvent event) {
        boolean barConsumedEvent = mBar.interceptTouchEvent(event);
    //fleety chang modify start <hide status_bar>
        /*if (DEBUG_GESTURES) {
            if (event.getActionMasked() != MotionEvent.ACTION_MOVE) {
                EventLog.writeEvent(EventLogTags.SYSUI_PANELBAR_TOUCH,
                        event.getActionMasked(), (int) event.getX(), (int) event.getY(),
                        barConsumedEvent ? 1 : 0);
            }
        }

        return barConsumedEvent || super.onTouchEvent(event);*/
    return false;
    }

 @Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
    //fleety chang modify <hide status_bar>
       // return mBar.interceptTouchEvent(event) || super.onInterceptTouchEvent(event);
    return false;
    }
```

将上面两个事件传递的方法中进行操作，全部返回false，当手势事件传递全部被屏蔽掉后，通知栏里面的内容也就不会被显示（没有下拉事件） 


（2）、状态栏不显示
在frameworks/base/core/res/res/values/dimens.xml文件中修改以下内容
这里写图片描述

```
  <!-- Height of the status bar -->
    <!-- fleety chang modify <24 to 0  hide status_bar> -->
    <dimen name="status_bar_height">0dp</dimen>
```

将高度改为0dp
此时系统上方的状态不显示，但是需要将其布局也置为gone，不占地方
在frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/phone/PhoneStatusBar.java文件中修改一下内容，将布局置为GONE 

```
 private void addStatusBarWindow() {
        makeStatusBarView();

        mStatusBarWindowManager = new StatusBarWindowManager(mContext);
        mStatusBarWindowManager.add(mStatusBarWindow, getStatusBarHeight());
    //fleety chang add only one line <hide status_bar>
    mStatusBarView.setVisibility(View.GONE);
    }
```

至此系统上方的状态栏以及下方的虚拟按键栏都已经屏蔽




25. 默认输入法  

26. 拷贝文件到系统中
    mediatek/config/mt6582/Init_Config.mk


32、内置APP
已百度输入发为例,
在packages/apps/下面创建文件夹BaiduInput,将BaiduInput.apk和Android.mk放入,Android.mk的内容为:

```
# This Android.mk is empty, and >> does not build subdirectories <<.
# Individual packages in experimental/ must be built directly with "mmm".
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

# Module name should match apk name to be installed.
LOCAL_MODULE := BaiduInput
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)

#you can choose apk's diff location 
LOCAL_MODULE_PATH := $(TARGET_OUT_APPS)
#LOCAL_MODULE_PATH := $(TARGET_OUT_DATA_APPS)
LOCAL_CERTIFICATE := PRESIGNED
LOCAL_PACKAGE_NAME := BaiduInput

include $(BUILD_PREBUILT)
```

然后在device/mediatek/mt6735/device.mk中添加PRODUCT_PACKAGES += BaiduInpput
如下面代码:

```

ifneq ($(strip $(MTK_A1_FEATURE)),yes)
  PRODUCT_PACKAGES += FleetyFMRadioService
endif
PRODUCT_PACKAGES += FleetyDemo
PRODUCT_PACKAGES += r600util
PRODUCT_PACKAGES += BaiduInpput
PRODUCT_PACKAGES += GaodeCheji
PRODUCT_PACKAGES += XunFeiTTS 
```

33、添加第三方输入法 
如32所述,将百度输入法内置进来,
frameworks/base/packages/SettingsProvider/src/com/android/providers/settings/DatabaseHelper.java

loadSecureSettings(SQLiteDatabase db)中添加如下代码:

    

```
    //FLEETY ADD start <setting baidu input method>
        loadStringSetting(stmt, Settings.Secure.DEFAULT_INPUT_METHOD, R.string.default_input_method);
        loadStringSetting(stmt, Settings.Secure.ENABLED_INPUT_METHODS, R.string.default_input_method);
        //FLEETY ADD end
```

因为需要R.string.default_input_method所以在
frameworks/base/packages/SettingsProvider/res/values/defaults.xml中添加 

    

```
<!-- fleety chang add <input method>  -->
    <string name="default_input_method" translatable="false">com.baidu.input/.ImeService</string>
```

33、数据网络开关
frameworks/base/packages/SettingsProvider/src/com/android/providers/settings/DatabaseHelper.java

```
将loadSetting(stmt, Settings.Global.MOBILE_DATA,
                    mUtils.getValue(Settings.Global.MOBILE_DATA,
                    SystemProperties.get("ro.mtk_gemini_support").equals("1") ? 0 : 1));
修改为:
    //fleety modify
            loadSetting(stmt, Settings.Global.MOBILE_DATA,
                    mUtils.getValue(Settings.Global.MOBILE_DATA, 1));
```


34、修改手机型号
build/tools/buildinfo.sh

```
#fleety modity
#echo "ro.product.model=$PRODUCT_MODEL"
echo "ro.product.model=`762LM`"
#fleety modity
#echo "ro.product.name=$PRODUCT_NAME"
echo "ro.product.name=`Fleety`"
```

35、第一次启动时,会弹出Update preferred SIM card?对话框,取消方法:

packages/apps/Settings/src/com/android/settings/sim/SimDialogActivity.java

中添加方法:

```
 /**
    fleety chang add method <do not show sim dialog(Update preferred SIM card?)>
      */
    private void donotDisplayPreferredDialog(int slotId) {
        Context context = getApplicationContext();
        SubscriptionInfo sir = SubscriptionManager.from(context)
                .getActiveSubscriptionInfoForSimSlotIndex(slotId);

        if (sir != null) {
          int subId = sir.getSubscriptionId();
          PhoneAccountHandle phoneAccountHandle =
                            subscriptionIdToPhoneAccountHandle(subId);
                    setDefaultDataSubId(context, subId);
                    setDefaultSmsSubId(context, subId);
                    setUserSelectedOutgoingPhoneAccount(phoneAccountHandle);
        } 
        finish();
    }
```

然后在onCreate(Bundle savedInstanceState)方法switch语句的case PREFERRED_PICK中,注释掉
displayPreferredDialog(extras.getInt(PREFERRED_SIM)),添加donotDisplayPreferredDialog(extras.getInt(PREFERRED_SIM));
如下:

        

```
//fleety chang modify <do not show sim dialog(Update preferred SIM card?)>
                //displayPreferredDialog(extras.getInt(PREFERRED_SIM));
                donotDisplayPreferredDialog(extras.getInt(PREFERRED_SIM));
```

36、插入TF卡时,会弹出SD card ready对话框,取消方法:

frameworks/base/services/core/java/com/android/server/MountService.java
 private void showDefaultPathDialog(VolumeInfo vol)方法中,注释掉

```
//fleety chang annotate <do not show dialog(SD card ready or External USB storage ready)>
        //mContext.startActivity(intent);
```

vendor/mediatek/proprietary/frameworks/base/res/res/values-zh-rCN/strings.xml


37、显示外部存储卡插入提示
 

```
   private void showDefaultPathDialog(VolumeInfo vol) {
        Slog.i(TAG, "showDefaultPathDialog, vol=" + vol);
        int userId = mCurrentUserId;
        if (!vol.isVisibleForWrite(userId)) {
            Slog.i(TAG, "showDefaultPathDialog,but vol is not visible to userID="
                    + userId + ", volumeInfo=" + vol);
            return;
        }

        Intent intent = new Intent("com.mediatek.storage.StorageDefaultPathDialog");
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        if (vol.isUSBOTG()) {
            intent.putExtra(INSERT_OTG, true);
        }
                //fleety chang annotate <do not show dialog(SD card ready or External USB storage ready)>
        //mContext.startActivity(intent);
        //fleety add <show externel storage insert>
        showSDCardInsert();
    }
//fleety add
private void showSDCardInsert() {
        mHandler.post(new Runnable() {
            public void run() {
                Toast.makeText(mContext, "外部存储插入",
                        Toast.LENGTH_LONG).show();
            }
        });
    }
```

38、设置短信默认通知提示音为静音
frameworks/base/core/java/android/preference/RingtonePreference.java

在RingtonePreference.java中

```
//fleety add
import android.content.SharedPreferences;
```

添加成员变量并初始化

```
//fleety add
private Context mContext;

public RingtonePreference(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
    super(context, attrs, defStyleAttr, defStyleRes);
        //fleety add
        mContext = context;
```

在onSetInitialValue()方法中:

```
@Override
    protected void onSetInitialValue(boolean restorePersistedValue, Object defaultValueObj) {
        String defaultValue = (String) defaultValueObj;
        
                //fleety add start <setting mms notification silent>
                if(isFirstStart() && getRingtoneType() == RingtoneManager.TYPE_NOTIFICATION){
                    onSaveRingtone(null);
            return;
                    
                }
                //fleety add end 

        if (restorePersistedValue) {
            return;
        }
        
        // If we are setting to the default value, we should persist it.
        if (!TextUtils.isEmpty(defaultValue)) {
            onSaveRingtone(Uri.parse(defaultValue));
        }
    }
```

添加方法

```
//fleety add <setting mms notification silent>
private boolean isFirstStart() {
                SharedPreferences sp = PreferenceManager.getDefaultSharedPreferences(mContext);
        boolean isStart = sp.getBoolean("fleety_isFirstStart", true);
        if(isStart) {
            SharedPreferences.Editor editor = sp.edit();
            editor.putBoolean("fleety_isFirstStart", false);
            editor.commit();
            return true;
         } 
        return false;
    }
```

39、取消收到消息是震动
vendor/mediatek/proprietary/packages/apps/Mms/res/xml/notificationpreferences.xml

```
<CheckBoxPreference android:layout="?android:attr/preferenceLayoutChild"
            android:defaultValue="true" android:key="pref_key_vibrate"
            android:dependency="pref_key_enable_notifications" android:title="@string/pref_vibrate"
            android:summary="@string/pref_summary_vibrate" />
```

中android:defaultValue="true"改为false

40、设置音量等级
vendor/mediatek/proprietary/packages/apps/EngineerMode/src/com/mediatek/engineermode/audio/AudioModeSetting.java

packages/apps/Launcher3/src/com/android/launcher3/Launcher.java


```
//fleety add <set volue grade>
import java.util.Arrays;
import android.media.AudioSystem;

//fleety add start <set volue grade>
private byte[] mData = null;
    private void setAudioData(int start, int max){
        byte normalMode = 55;
                for(int i = start; i < max; i++){
                    if(i == start){
                        mData[i] = (byte)10;
                    }else if(i == (start+1)){
                        mData[i] = (byte)20;
                    }else if(i == (start+2)){
                        mData[i] = (byte)35;
                    }else{
                        mData[i] = normalMode;
                        normalMode += (byte)20;
                    }
                }
    }

    private void operateAudioData(){
        SharedPreferences sp = getSharedPrefs();  
        boolean isStart = sp.getBoolean("fleety_isFirstStart", true);
    if(!isStart) {
        return;
    } 
    SharedPreferences.Editor editor = sp.edit();
    editor.putBoolean("fleety_isFirstStart", false);
    editor.commit();
    
        mData = new byte[581];
        Arrays.fill(mData, 0, 581, (byte) 0);
        AudioSystem.getAudioData(0x100, 581, mData);
        setAudioData(420, 433);
        setAudioData(435, 448);
        AudioSystem.setAudioData(0x101, 581, mData);
    }

//fleety add end
```


oncreate()中添加

```
//fleety add <set volue grade>
operateAudioData();
```

41、设置中,将默认通知提示音设置成无
packages/apps/Settings/src/com/mediatek/audioprofile/Editprofile.java
中添加一行:

```
  private void initNotification(PreferenceScreen parent) {
        mNotify = (DefaultRingtonePreference) parent.findPreference(KEY_NOTIFY);
        if (mNotify != null) {
            mNotify.setStreamType(DefaultRingtonePreference.NOTIFICATION_TYPE);
            mNotify.setProfile(mKey);
            mNotify.setRingtoneType(AudioProfileManager.TYPE_NOTIFICATION);
            mNotify.setNoNeedSIMSelector(true);
//fleety add only one line <setting notification cue tone to mute>
mNotify.onSaveRingtone(null);
        }
    }
```
































