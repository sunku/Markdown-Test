## Contents
- [Basic Integration](#basic-integration)
  - [Installation](#installation)
  - [Start Session](#start-session)
  - [Sign In](#sign-in)     
  - [In-App Messaging](#in-app-messaging)
  - [Push Messaging](#push-messaging)
  - [Test Device Registration](#test-device-registration)
  - [Test Mode](#test-mode)
- [IAP, Reward and Sales Promotion](#iap-reward-and-sales-promotion)
  - [In-App Purchase Tracking](#in-app-purchase-tracking)
  - [Give Reward](#give-reward)
  - [Sales Promotion](#sales-promotion)
- [Dynamic Targeting](#dynamic-targeting)
  - [Custom Parameter](#custom-parameter)
  - [Marketing Moment](#marketing-moment)
- [Advanced](#advanced)
  - [Timeout Interval](#timeout-interval) 
- [Reference](#reference)
  - [Deep Link](#deep-link)
  - [Cross Promotion Configuration](#cross-promotion-configuration)
  - [Image Push Notification](#image-push-notification)
  - [Proguard Configuration](#proguard-configuration)
- [Troubleshooting](#troubleshooting)
- [Release Notes](#release-notes)

* * *

## Basic Integration

### Installation

아래 링크를 통해 _Unity Plugin_을 다운로드 합니다.

[Unity Plugin 다운로드](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-Unity.zip) 

Unity 프로젝트를 열고 AdFrescaUnityPlugin.package 파일을 실행합니다.

아래의 구성 요소들이 Assets 폴더 아래에 복사되어야 합니다.

Assets/

    LitJson.dll

Assets/AdFresca/

    Plugin.cs 
    AndroidPlugin.cs
    IOSPlugin.cs
    RewardItem.cs
    Purchase.cs 

Assets/Plugins/Android/

    AdFresca.jar 
    AdFrescaPlugin.jar 
    gcm.jar 
    assets 

Assets/Plugins/iOS/

    AdFrescaPlugin.h
    AdFrescaPlugin.mm

이제 각 플랫폼에 맞게 설치 작업을 진행합니다.

#### Android

Android 플랫폼의 대부분의 설치 및 적용 작업이 플러그인에 이미 구현되어 있습니다. 기본적으로 AndroidManifest.xml 수정 작업만 진행하여 주시면 됩니다.

##### AndroidManifest.xml 적용하기

```xml
<manifest package="your.app.package">
  <application>
    <activity>
    <!-- Enable ForwardNativeEventsToDalvik -->
    <meta-data android:name="unityplayer.ForwardNativeEventsToDalvik" android:value="true" />
    </activity>
    
    <!-- Device ID 수집을 위한 OpenUDID 서비스 등록 -->
    <service android:name="org.openudid.OpenUDID_service">
      <intent-filter>
        <action android:name="org.openudid.GETUDID" />
      </intent-filter>
    </service>

    <!-- Reward 기능을 위한 액티비티 등록 -->
    <activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
   
    <!-- Google Referrer Tracking 을 위한 Boradcast Receiver 등록 -->
    <receiver android:name="com.adfresca.sdk.referer.AFRefererReciever" android:exported="true">
      <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
      </intent-filter>
    </receiver>
  </application>
  
  <!-- Permission 추가 -->
  <uses-permission android:name="android.permission.INTERNET"/>
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
</manifest>
```

#### iOS

iOS 플랫폼의 경우는 Native SDK와 동일한 설치 작업을 진행합니다. 모든 플러그인 구성 요소가 Import 되었는지 확인 후, Unity에서 Xcode 프로젝트를 빌드합니다. 

빌드된 Xocde 프로젝트를 실행한 후 iOS SDK 설치 가이드의 ['Installation'](https://github.com/adfresca/sdk-ios/blob/master/README.kor.md#installation) 항목을 따라서 설치 작업을 완료합니다.

### Start Session

이제 플러그인을 적용을 시작하기 위해 몇 가지 간단한 코드를 적용합니다. 첫 번째로 API Key를 설정하고 앱의 실행을 기록하는 StartSession() 메소드를 적용합니다. API Key는 [Dashboard](https://dashboard.nudge.do) 사이트에서 등록한 앱을 선택한 후 Overview 메뉴의 Settings - API Keys 버튼을 클릭하여 확인이 가능합니다. 

#### Android

Android 플랫폼 경우는 유니티 스크립트로 직접 사용자의 게임 실행을 기록할 수 있습니다. 게임이 실행되는 시점에서 StartSession() 메소드를 실행합니다.

```cs
#if UNITY_ANDROID
private static string API_KEY = "YOUR_ANDROID_API_KEY";
#elif UNITY_IPHONE
private static string API_KEY = "YOUR_IOS_API_KEY";
#endif

void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  plugin.StartSession();
}
```

#### iOS

iOS 플랫폼의 경우는 Xocde에서 메소드를 실행해야 합니다. AppController.mm 파일을 열고 didFinishLaunchingWithOptions() 이벤트에 아래와 같이 코드를 적용합니다.

```objective-c
#import <AdFresca/AdFrescaView.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  ....
} 
```

### Sign In

Sign In 기능은 사용자의 로그인 액션을 트랙킹합니다. 넛지는 이 메소드를 통해 전달된 회원 ID를 이용하여 사용자를 구분합니다. 회원 ID를 이용하여 1명의 회원이 복수 개의 디바이스를 사용하는 경우에도 중복으로 캠페인에 노출되거나 리워드를 지급 받는 경우를 방지할 수 있습니다. 또한 로그인한 사용자, 로그인하지 않은 사용자 등을 대상으로 캠페인을 실행할 수 있습니다.

사용자는 반드시 회원 또는 비회원으로 Sign In 되어야 하며 다른 사용자가 로그인을 하면 이전의 사용자는 자동으로 로그아웃됩니다. (따라서 별도의 Sign Out은 필요하지 않습니다.) 회원의 경우 로그인 이벤트 (자동 로그인 포함) 발생 시 **signIn(string)** 메소드에 회원 ID를 인자로 넘겨 호출하며 비회원의 경우 **signInAsGuest(string)** 메소드에 게스트 ID를 인자로 넘겨 호출합니다. 비회원을 트랙킹하기 위해 별도의 게스트ID를 사용하지 않는다면 인자 없이 **signInAsGuest()** 메소드를 호출하면 됩니다.

```cs
void onAppStart() {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;

  if(isSignedIn) {
    // 직접 로그인하거나, 자동로그인 모두 호출되도록 함
    plugin.SignIn(“user_id”);
  } else {
    // 비회원을 별도의 guest_id로 트랙킹하고 있다면 인자로 설정 가능
    // 비회원을 별도의 ID로 트랙킹하지 않는다면 인자를 설정하지 않아야 함
    plugin.SignInAsGuest("guest_user_id");
  }
}
```

GetSignedUserId() 메소드를 사용하면 현재 로그인되어 있는 사용자의 ID를 리턴합니다 (게스트 ID를 지정하지 않은 비로그인 사용자의 경우는 디바이스 ID 값이 리턴). 이 메소드를 사용하여 정상적으로 로그인이 기록되어 있는지 테스트할 수 있습니다.

### In-App Messaging

인-앱 메시징 기능을 이용하여, 사용자에게 원하는 메시지를 실시간으로 전달할 수 있습니다. 메시지를 전달하고자 하는 시점에 유니티 스크립트로 제공되는 Load(), Show() 메소드만을 호출하여 적용이 가능합니다. 메시지는 전면 interstitial 이미지, 텍스트, 혹은 iframe 웹페이지 형태로 게임 화면에 표시될 수 있습니다. 메시지는 현재 게임을 플레이 중인 사용자가 인-앱 메시징 캠페인의 조건과 매칭된 경우에만 화면에 표시됩니다. 조건에 만족하는 캠페인이 없다면 사용자는 아무런 화면을 보지 않고 자연스럽게 게임 플레이를 이어갑니다. 매칭과 관련한 인-앱 메시징의 다이나믹 타겟팅 기능은 아래의 [Dynamic Targeting](#dynamic-targeting) 항목에서 보다 자세히 설명하고 있습니다.

```cs
void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Load();
  plugin.Show();
}
```

첫 번째로 인-앱 메시징 코드를 적용한 경우, 아래와 같이 테스트 이미지 메시지가 표시됩니다. 해당 이미지를 터치하면 앱스토어 페이지로 이동합니다. 현재 보고 있는 테스트 메시지는 이후 테스트 모드 설정을 변경하여 더이상 보이지 않도록 설정하게 됩니다.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

### Push Messaging

푸시 메시징 기능을 이용하여 사용자가 게임을 플레이하지 않을 때에도 언제든 메시지를 전달할 수 있습니다. 아래의 플랫폼 별 적용 과정을 통하여 푸시 메시징 기능을 적용합니다.

#### Android

SDK를 적용하기 이전에 [Google API Console](https://cloud.google.com/console) 사이트에서 프로젝트를 생성하고, [Dashboard](https://dashboard.nudge.do) 사이트에 설정할 GCM API Key 및 SDK 적용에 필요한 GCM_SENDER_ID (Project Number) 값을 얻어야 합니다.

'[Android Push Notification 설정 및 적용하기 (GCM)](https://adfresca.zendesk.com/entries/28526764)' 가이드를 참고하여 필요한 값들을 얻습니다.

이제 SDK 적용을 시작합니다.

1) AndroidManifest.xml 내용 추가하기

```xml
<manifest>   
  <application>
      .........
      <receiver android:name="YOUR.PACKAGE.NAME.GCMReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">  
        <intent-filter>
          <action android:name="com.google.android.c2dm.intent.RECEIVE" />
          <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
          <category android:name="YOUR.PACKAGE.NAME" />
         </intent-filter>
      </receiver>
      <service android:name="YOUR.PACKAGE.NAME.GCMIntentService" />  

      <activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
      ..........
   </application>
    ..........
    <permission android:name="YOUR.PACKAGE.NAME.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="YOUR.PACKAGE.NAME.permission.C2D_MESSAGE" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.VIBRATE" />
    ..........
</manifest>
```

- CustomGCMReceiver 클래스와 CustomGCMIntentService 클래스는 이미 적용 중인 내용이 있다면 그대로 사용하여 필요한 SDK 코드만 추가합니다. 
- 만약 기존에 사용 중인 GCM 클래스가 없다면, 직접 안드로이드 자바 클래스를 작성해야 합니다. 빠른 진행을 위하여 유니티 플러그인에 포함된 'Android Plugin Project'를 이용합니다. Eclipse ADT에서 해당 샘플 프로젝트를 추가한 후 /src 및 /gen 아래에 있는 패키지를 모두 현재 적용 중인 게임의 패키지로 변경합니다. 그리고 위의 AndroidManifest.xml 파일 내용 중 'YOUR.PACKAGE.NAME' 표시된 패키지명도 함께 변경해야 합니다.

2) CustomGCMIntentService 클래스 구현하기

```java
public CustomGCMIntentService() {
  super();
}

@Override
protected String[] getSenderIds (Context context) {
  String[] ids = {AdFrescaPlugin.gcmSenderId};
  return ids;
}

@Override
protected void onRegistered(Context context, String registrationId) {
  AdFresca.handlePushRegistration(registrationId);
}

@Override
protected void onUnregistered(Context context, String registrationId) {
  AdFresca.handlePushRegistration(null);
}

@Override
protected void onMessage(Context context, Intent intent) {
  // Check Nudge notification
  if (AdFresca.isFrescaNotification(intent)) {    

    String title = AdFrescaPlugin.getAppName(context);
    int icon = com.MyCompany.ProductName.R.drawable.app_icon;
    long when = System.currentTimeMillis();
    Class<?> targetActivityClass = null;
    
    if (UnityPlayer.currentActivity != null) {
      targetActivityClass = UnityPlayer.currentActivity.getClass();
    } else {
      targetActivityClass = UnityPlayerProxyActivity.class; // or YourUnityPlayerProxyActivity.class
    }
    
    AFPushNotification notification = AdFresca.generateAFPushNotification(context, intent, targetActivityClass, appName, icon, when);
    notification.setDefaults(Notification.DEFAULT_ALL); 
    AdFresca.showNotification(notification);
  }                
}  
```

3) CustomGCMReceiver 클래스 구현하기

```java
@Override
protected String getGCMIntentServiceClassName(Context context) { 
  return "YOUR.PACKAGE.NAME.CustomGCMIntentService"; 
} 
```

4) Jar 파일 저장하기

위 2개 클래스의 수정한 후 유니티 플러그인 폴더에 Export 합니다. 유니티 프로젝트 폴더의 /Assets/Plugins/Android 폴더 아래에 파일을 저장합니다. 다른 불필요한 클래스가 함께 첨부되지 않도록 주의하여 진행합니다.

<img src="https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/unity/unity-android-gcm-export.png"/>

5) Unity 환경에서 GCM 적용하기

이제 유니티 클래스에서 GCM Sender ID 값을 플러그인에 설정하여 적용을 완료합니다. 만약 기존에 사용 중인 GCM 서비스가 있어 이미 디바이스의 GCM 푸시 고유 아이디를 알고 있는 경우, SetGCMPushRegistrationIdentifier(string) 메소드를 호출하여 값만 설정하는 것도 가능합니다.

```cs
#if UNITY_ANDROID
private static string GCM_SENDER_ID = "12345678"; // Google API Proejct Number (ex: 12345678)
#endif

void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  
  plugin.SetGCMSenderId(GCM_SENDER_ID);
  // plugin.SetGCMPushRegistrationIdentifier("DEVICE_GCM_PUSH_REG_ID"); // if you already have it

  plugin.StartSession();
}
```

#### iOS

1) APNS 인증서 파일(.p12)을 Dashboard에 등록하기
  - Keychain 툴을 이용하여 .cer 인증서 파일을 .p12로 변환하고 [Dashboard](https://dashboard.nudge.do) 사이트에 등록합니다.
  - 보다 자세한 설명은 [iOS Push Notification 인증서 설정 및 적용하기](https://adfresca.zendesk.com/entries/21714780) 가이드를 통하여 확인이 가능합니다.

2) Info.plast 확인하기 / Provision 확인하기
- Nudge는 APNS의 Production 환경만을 지원합니다. 때문에 게임 빌드가 production으로 빌드되어야 정상적인 서비스 이용이 가능합니다.
- Info.plst 파일의 'aps-environment' 값을 'production' 으로 설정되어 있어야 합니다.
- App Store / Ad Hoc release에 사용하는 Provision 인증서를 사용하여 빌드되어야 합니다.

3) AppController.mm 코드 적용하기 

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  ...
  NSDictionary* userInfo = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
  if (userInfo != nil) [self application:application didReceiveRemoteNotification:userInfo];
} 

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
  [AdFrescaView registerDeviceToken:deviceToken];
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
  if ([AdFrescaView isFrescaNotification:userInfo]) {
    [AdFrescaView handlePushNotification:userInfo];
  }  
} 
```

만약 푸시 노티피케이션 기능을 처음 적용하는 경우 [샘플 코드](https://gist.github.com/sunku/791f1ff2d7d1b37ca9f8#file-gistfile1-m)를 확인하여 필요한 모든 코드를 확인합니다.

이로써 푸시 메시징 기능을 위한 적용 작업이 모두 완료되었습니다.

### Test Device Registration

Nudge는 테스트 모드 기능을 지원하여 테스트를 원하는 디바이스에만 원하는 메시지를 전달할 수 있습니다. 이로 인해 SDK가 적용된 앱이 이미 앱스토어에 출시된 경우, 게임 운영팀 혹은 개발팀에게만 새로운 메시지를 전달하여 테스트할 수 있도록 지원합니다.

테스트 기기 등록을 위한 아이디 값은 SDK를 통해 추출이 가능하며 2가지 방법을 지원 합니다.
 
1. testDeviceId 값을 직접 얻어와서 로그로 출력하는 방법

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance
plugin.Init(API_KEY);
plugin.StartSession();

if(Application.platform == RuntimePlatform.Android) {
  plugin.PrintTestDeviceIdByLog();
} else {
  string testDeviceId = plugin.TestDeviceId();
  Debug.Log("testDeviceId = " + testDeviceId);
}
```

2. SetPrintTestDeviceId() 메소드를 사용하여 화면에 Device ID를 표시하는 방법
 
```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance
plugin.Init(API_KEY);
plugin.StartSession();
plugin.SetPrintTestDeviceId(true);
plugin.Load();
plugin.Show();
```

테스트 디바이스 아이디를 확인한 이후에는, [Dashboard](https://dashboard.nudge.do)를 접속하여 'Test Device' 메뉴를 통해 디바이스 등록이 가능합니다.

### Test Mode

Nudge SDK는 테스트 모드 기능을 지원합니다. 테스트 모드를 활성화하면 현재 실행되는 SDK 메소드와 그 실행 결과가 로그 메시지로 출력됩니다. 이를 통하여 본인이 올바른 코드와 인자 값을 설정하고 있는지 검증할 수 있습니다.

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance
plugin.SetTestMode(true);
```

<img src="https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/android_sdk_test_mode.png" width="900" />
<img src="http://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/ios_sdk_test_mode.png" width="900" />

현재 테스트 모드는 'Start Session', 'Push Messaging', 'In-App Purchase Tracking', 'Custom Parameter', 'Stickiness Custom Parameter' 항목에 대한 로그를 지원합니다. 다른 항목의 경우는 추후 제공할 예정에 있습니다.

* * *

## IAP, Reward and Sales Promotion

### In-App Purchase Tracking

_In-App-Purchase Tracking_  기능을 통하여 현재 앱에서 발생하고 있는 모든 인-앱 결제를 분석하고 캠페인 타겟팅에 이용할 수 있습니다.

Nudge의 In-App-Purchase Tracking은 2가지 유형이 있습니다.

1. 실제 화폐를 통해 결제되는 Hard Currency Item Purchase Tracking (예: USD $1.99를 결제하여 Gold 100개 아이템을 구입)
2. 가상 화폐를 통해 결제되는 Soft Currency Item Purchase Tracking (예: Gold 10개를 이용하여 포션 아이템을 구입)

위 2가지 유형의 데이터를 모두 Tracking 함으로써 앱의 매출뿐만 아니라 인-앱 사용자들의 아이템 구매 추이 분석까지 가능합니다.

아이템 정보 등록을 위한 별도의 작업은 필요하지 않으며, 클라이언트에서 결제된 아이템 정보가 자동으로 대쉬보드에 등록되는 방식입니다. (아이템 리스트 확인은 대쉬보드 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.)

아래의 적용 예제를 참고하여 간단히 In-App-Purchase Tracking 기능을 적용합니다.

### Hard Currency Item Tracking

Hard Currency Item의 결제는 각 앱스토어별 인-앱 결제 라이브러리를 통해 이루어집니다. 각 결제 라이브러리에서 _'결제 성공'_ 이벤트가 발생 할 시에 Purchase 객체를 생성하고 LogPurchase(purchase) 메소드를 호출합니다. 그리고 _'결제 실패'_ 이벤트가 발생 할 시에는 CancelPromotionPurchase() 메소드를 호출합니다.

적용 예제: 유니티 환경에서 결제 성공 이벤트 발생 시
```cs
private void OnHardItemPurchased() 
{
    AdFresca.Purchase purchase = new AdFresca.PurchaseBuilder(AdFresca.Purchase.Type.HARD_ITEM)
      .WithItemId("gold100")
      .WithItemName("100 Gold")
      .WithCurrencyCode("USD") // The currencyCode must be specified in the ISO 4217 standard. (ex: USD, KRW, JPY)
      .WithPrice(0.99)
      .WithPurchaseDate(purchaseDateTime) // purchaseDateTime from In-app billing library
      .WithReceipt("google_play_order_id", "google_play_receipt_json", "google_play_signature"); // Optional
          
    AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
    plugin.LogPurchase(purchase);
}

private void OnPurchaseHardItemFailure() 
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.CancelPromotionPurchase();
}
```

Hard Currency Item을 위한 PurchaseBuilder의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
WithItemId(string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. Nudge 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
WithItemName(string) | 결제한 아이템의 이름을 설정합니다. 이 값은 대쉬보드에서 이름을 표시하기 위한 용도로만 이용됩니다. 등록된 이름은 언제든지 대쉬보드에서 변경이 가능합니다. 
WithCurrencyCode(string) | ISO 4217 표준 코드를 설정합니다. Google Play의 경우 'Default price' 에 설정되는 Currency Code 값을 이용하며 타 결제 라이브러리의 경우는 보통 이용 가능한 Currency Code가 고정되어 있습니다 (예: 아마존은 USD, 티스토어는 KRW). 또는 자체 백엔드 서버에서 결제하는 아이템의 Currency Code를 내려받아 설정할 수 있습니다.
WithPrice(double) | 아이템의 가격을 설정합니다. 결제 라이브러리에서 주는 값을 이용하거나, 자체 백엔드 서버에서 가격을 내려받아 설정할 수 있습니다. 
WithPurchaseDate(datetime) | 결제된 시간을 DateTime 객체 형태로 설정합니다. 값이 설정되지 않은 경우 Nudge 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.
WithReceipt(string, string, string) | 추후 Receipt Verficiation 기능을 위해 필요한 데이터를 설정합니다. 현재 버전의 SDK는 Google Play만 지원하며 타 결제 라이브러리의 경우는 값을 설정하지 않습니다.

### Soft Currency Item Tracking

Soft Currency Item의 결제는 앱 내의 가상 화폐로 아이템을 결제한 경우를 의미합니다. 앱 내에서 가상 화폐를 이용한 결제 이벤트가 성공한 경우 아래 예제와 같이 Purchase 객체를 생성하고 LogPurchase(purchase) 메소드를 호출합니다. 그리고 _'결제 실패'_ 이벤트가 발생 할 시에는 CancelPromotionPurchase() 메소드를 호출합니다.

적용 예제: 
```cs
private void OnSoftItemPurchased() 
{
    AdFresca.Purchase purchase = new AdFresca.PurchaseBuilder(AdFresca.Purchase.Type.SOFT_ITEM)
      .WithItemId("long_sword")
      .WithItemName("Long Sword")
      .WithCurrencyCode("gold") 
      .WithPrice(100)
      .WithPurchaseDate(purchaseDateTime);
    
    AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
    plugin.LogPurchase(purchase);
}

private void OnPurchaseSoftItemFailure() 
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.CancelPromotionPurchase();
}
```

Soft Currency Item을 위한 PurchaseBuilder의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
WithItemId(string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. Nudge 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
WithItemName(string) | 결제한 아이템의 이름을 설정합니다. 이 값은 대쉬보드에서 이름을 표시하기 위한 용도로만 이용됩니다. 등록된 이름은 언제든지 대쉬보드에서 변경이 가능합니다. 
WithCurrencyCode(string) | 결제에 사용한 가상화폐 고유 코드를 설정합니다. (예: gold)
WithPrice(double) | 가상 화폐로 결제한 가격 정보를 설정합니다. (예: gold 10개의 경우 10 값을 설정)
WithPurchaseDate(datetime) | 결제된 시간을 DateTime 객체 형태로 설정합니다. 값이 설정되지 않은 경우 Nudge 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.

### IAP Trouble Shooting

LogPurchase() 메소드를 통해 기록된 Purchase 객체는 Nudge 서비스에 업데이트되어 실시간으로 대쉬보드에 반영됩니다. 현재까지 등록된 아이템 리스트는 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.

만약 아이템 리스트가 새로 갱신되지 않는 경우, Android SDK의 AFPurchaseExceptionListener 구현하여 혹시 에러가 발생하고 있지 않은지 확인해야 합니다. Purchase 객체의 값이 제대로 설정되지 않은 경우, AFPurchaseExceptionListener 통하여 에러 메시지가 표시됩니다.

Unity Plugin의 경우는 이미 아래와 같은 코드가 적용되어 있습니다. 따라서 Purchase 객체가 제대로 생성되지 않은 경우 "AFPurchaseExceptionListener.onException = {error message}" 형태의 로그가 콘솔에 출력됩니다.

```java
......
AdFresca.getInstance(UnityPlayer.currentActivity).logPurchase(purchase, new AFPurchaseExceptionListener(){
  public void onException(AFPurchase purchase, AFException e) {
    Log.e("AdFresca", "AFPurchaseExceptionListener.onException = " + e.getMessage());
  }
});
```

* * *

### Give Reward

리워드 지급 기능을 이용하여 사용자에게 인앱 아이템을 보상으로 지급할 수 있습니다.

리워드 지급에는 2가지 과정이 필요합니다.

1. 리워드 지급 요청: 사용자에게 지급해야 할 리워드가 있는 경우 SDK에서 리워드 지급 요청 이벤트를 발생시킵니다. 해당 이벤트 발생 시에 게임 서버를 통해 사용자에게 리워드를 지급합니다.
2. 리워드 지급 완료: 사용자에게 리워드를 지급한 후 SDK에서 지급 완료 사실을 알립니다.

먼저 지급 요청을 위한 코드를 구현합니다. 사용자에게 지급해야 할 리워드가 있는 경우 onRewardClaim 이벤트가 발생됩니다. 인자로 넘어온 아이템 정보를 이용하여 사용자에게 아이템을 지급합니다.

**iOS에서 AFRewardClaimDelegate 구현하기**

```objective-c
// AppDelegate.h

@interface AppDelegate : UIResponder <UIApplicationDelegate, AFRewardClaimDelegate> {
  ...
}


// AppDelegate.m

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [[AdFrescaView shardAdView] setRewardClaimDelegate:self];
}

- (void)onRewardClaim:(AFRewardItem *)item {
  UnitySendMessage("YourGameObject", "onRewardClaim", [[item JSON] UTF8String]);
}
```

**Unity 코드 적용하기**

```cs
void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  plugin.SetGCMSenderId(GCM_SENDER_ID);    
  plugin.StartSession();

  plugin.SetAndroidRewardItemListener("YourGameObject", "onRewardClaim");
  plugin.CheckRewardItems();
}

public void onRewardClaim(string json)
{
  RewardItem rewardItem = LitJson.JsonMapper.ToObject<RewardItem>(json);
  
  Debug.Log ("rewardItem.name: " + rewardItem.name);
  Debug.Log ("rewardItem.quantity: " + rewardItem.quantity);
  Debug.Log ("rewardItem.uniqueValue: " + rewardItem.uniqueValue);
  Debug.Log ("rewardItem.securityToken: " + rewardItem.securityToken);
  Debug.Log ("rewardItem.rewardClaimToken: " + rewardItem.rewardClaimToken);
  
  SendItemToUser("USER_ID", rewardItem);
}
```

사용자에게 아이템을 지급한 후 finishRewardClaim() 메소드를 호출하여 넛지에 리워드 지급 완료를 통보해야 합니다. 넛지는 리워드 지급 완료 기록을 전달 받아야만 리워드가 정상적으로 지급된 것으로 처리합니다. 즉 게임서버나 클라이언트에서 에러가 발생하여 리워드 지급이 실패한 경우 넛지 SDK에서는 다시 리워드 지급 요청을 합니다. 넛지 SDK에서는 3분 이상 지급 확인 기록이 전달되지 않은 경우 다음 번 마케팅 모멘트가 실행될 때 다시 리워드 지급 요청을 합니다. 이는 지급 처리 중에 다시 지급요청을 해서 중복 지급되는 것을 막기 위함입니다.

```cs
public void onRewardClaimSuccess(RewardItem rewardItem)
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.FinishRewardClaim(rewardItem.rewardClaimToken);
}
```

##### sendItemToUser() 메소드의 구현

SDK에서 요청한 아이템을 사용자에게 지급해야 합니다. 클라이언트에서 직접 지급하거나, 백엔드 서버와 통신하여 사용자의 선물함으로 보상을 지급하는 방식 등을 이용할 수 있습니다. 아이팀의 고유 값, 수량, 그리고 시큐리티 토큰값을 이용하여 현재 로그인된 사용자에게 아이템을 지급합니다.

##### 백엔드 서버를 통해 아이템을 지급할 경우 보안 이슈 해결하기

저희 SDK에서는 특정 사용자가 동일한 캠페인에서 1회 이상 아이템이 지급되지 않도록 처리하고 있습니다. 하지만 클라이언트에서 백엔드 서버와 통신하는 과정이 노출된다면 해킹외부 공격에 의해 아이템이 중복으로 지급되는 보안 이슈가 발생할 수도 있습니다. 해당 문제를 방지하기 위하여 시큐리티 토큰값을 제공하고 있습니다. 시큐리티 토큰값은 대쉬보드에서 캠페인 생성 시에 자동으로 생성되며 필요에 따라 직접 지정할 수도 있습니다. 시큐리티 토큰값은 다음과 같이 이용할 수 있습니다.

1. 게임 서버에서는 리워드 캠페인의 시큐리티 토큰값을 데이터베이스에 미리 저장하고 있다가 클라이언트에서 리워드 지급 요청시에 올라온 토큰값과 비교하여 정상적인 요청인지를 검증할 수 있습니다.
2. 특정 사용자가 동일한 토큰값으로 1회 이상 지급 요청을 하는 경우 요청을 거절합니다. 
3. 만약 토큰값이 외부에 노출되었다고 판단될 경우, 대쉬보드에서 토큰값을 새로 생성하거나 수정합니다.

* * *

### Sales Promotion

Sales Promotion 캠페인을 이용하여 특정 아이템의 구매를 유도할 수 있습니다. 사용자가 캠페인에 노출된 이미지 메시지를 클릭할 경우 해당 아이템의 결제 UI가 표시됩니다. SDK는 사용자의 실제 결제 여부까지 자동으로 트랙킹하여 대쉬보드에서 실시간으로 통계를 제공합니다. 

프로모션 기능을 적용하기 위해서 OnPromotion 이벤트를 구현합니다. 프로모션 캠페인이 노출된 후 사용자가 이미지 메시지의 액션 영역을 탭하면 onPromotion() 이벤트가 발생합니다. 이벤트에 넘어오는 PromotionPurchase 객체 정보를 이용하여 사용자에게 아이템 결제 UI를 표시하도록 코드를 적용합니다.

Hard Currency 아이템의 경우 인-앱 결제 라이브러리를 이용하여 결제 UI를 표시합니다. PromotionPurchase 객체의 ItemId 값이 아이템의 SKU 값에 해당됩니다. 

Soft Currency 아이템의 경우는 앱이 기존에 사용하고 있는 상점 내 아이템 결제 UI를 표시하도록 코드를 작성합니다. Soft Currency 프로모션의 경우는 2가지 가격 할인 옵션을 제공하고 있습니다. PromotionDiscountType 값을 이용하여 할인 옵션을 확인할 수 있습니다.

1. **Discount Price**: 캠페인에 직접 지정된 가격으로 아이템을 판매합니다. Price 값을 이용하여 가격 정보를 얻습니다.
2. **Discount Rate**: 캠페인에 지정된 할인율을 적용하여 아이템을 판매합니다. PromotionDiscountRate 값을 이용하여 할인율 정보를 받아옵니다.

**iOS에서 AFPromotionDelegate 구현하기**

```objective-c
// UnityAppController.h
@interface UnityAppController : NSObject<UIApplicationDelegate, AFPromotionDelegate>
{
  ...
}

...

// UnityAppController.mm
- (void)applicationDidBecomeActive:(UIApplication *)application 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setPromotionDelegate:self];
}

- (void)onPromotion:(AFPurchase *)promotionPurchase
{
  UnitySendMessage("YourGameObject", "OnPromotion", [[promotionPurchase JSONForUnity] UTF8String]);
}
```

**Unity 코드 적용하기**

```cs
void Start ()
{
	AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
	plugin.Init(API_KEY);
	plugin.StartSession();
	
	....
	
	plugin.SetPromotionListener("YourGameObject", "OnPromotion"); // for Android only
}
	
public void OnPromotion(string json)
{
	Debug.Log("OnPromotion = " + json);		
	Purchase PromotionPurchase = LitJson.JsonMapper.ToObject<Purchase>(json);

	string ItemId = PromotionPurchase.ItemId;
	string LogMessage = "no logMessage";

	if (PromotionPurchase.PurchaseType == Purchase.Type.HARD_ITEM)
	{
		// 인-앱 결제 라이브러리를 호출.
		ShowHardItemPurchaseUI(ItemId);
		LogMessage = String.Format("on HARD_ITEM Promotion ({0})", ItemId);  
	}
	else if (PromotionPurchase.PurchaseType == Purchase.Type.SOFT_ITEM)
	{
		String CurrencyCode = PromotionPurchase.CurrencyCode;
		if (PromotionPurchase.PromotionDiscountType == Purchase.DiscountType.DISCOUNTED_TYPE_PRICE) 
		{
      // 할인된 가격을 이용하여 앱-내 아이템 구매 UI를 표시.
			double DiscountedPrice = PromotionPurchase.Price;
			ShowSoftItemPurchaseUIWithDiscountedPrice(ItemId, CurrencyCode, DiscountedPrice);
			LogMessage = String.Format("on SOFT_ITEM Promotion ({0}) with {1} {2}", ItemId, DiscountedPrice, CurrencyCode);
		}
		else if (PromotionPurchase.PromotionDiscountType == Purchase.DiscountType.DISCOUNT_TYPE_RATE)
		{
      // 할인율 값을 이용하여 앱-내 아이템 구매 UI를 표시. discountedPrice = originalPrice - (originalPrice * discountRate)
			double DiscountRate = PromotionPurchase.PromotionDiscountRate;
			ShowSoftItemPurchaseUIWithDiscountRate(ItemId, CurrencyCode, DiscountRate);
			LogMessage = String.Format("on SOFT_ITEM Promotion ({0}) with {1}% discount", ItemId, DiscountRate * 100.0);
		}
	}

	Debug.Log(LogMessage);
}
```

SDK가 사용자의 실제 구매 여부를 트랙킹하기 위해서는 [In-App Purchase Tracking](#in-app-purchase-tracking) 기능이 미리 구현되어 있어야 합니다. 특히 사용자가 아이템을 구매를 하지 않거나 실패한 경우를 트랙킹 하기 위하여 CancelPromotionPurchase() 메소드가 반드시 적용되어 있어야 합니다.

* * *

## Dynamic Targeting

### Custom Parameter

커스텀 파라미터는 마케팅 목적으로 사용자를 분류하기 위해 사용하는 속성을 말하며 마케터가 임의로 정의할 수 있습니다. (예. 사용자의 레벨, 스테이지, 플레이 횟수 등) 커스텀 파라미터를 이용하면 사용자의 특정 속성에 따라 세그먼트를 정의하고 실시간으로 모니터링할 수 있습니다. 또한 캠페인 실행 시에는 보다 더 정교한 타겟팅을 통해 높은 성과를 거둘 수 있습니다. (Nudge SDK에서 자동적으로 수집하는 단말 ID, 기본 언어, 국가, 앱 버전 등의 정보는 커스텀 파라미터로 설정할 필요가 없습니다.)

Nudge SDK는 트랙킹하려고 하는 커스텀 파라미터의 유형에 따라 아래 2가지 방법을 제공합니다.

- 현재 상태 값을 트랙킹하는 경우
  - 특정 속성의 현재 상태 값을 트랙킹할 때 사용합니다. 
  - 예) 레벨, 최종 스테이지, 친구수 등
  - 사용 코드: **SetCustomParameter** 메소드를 이용하여 현재 상태 값 (Integer, Boolean 지원)을 SDK에 전달합니다.

- 특정 이벤트의 횟수를 트랙킹하는 경우
  - 특정 이벤트마다 증가하는 횟수를 트랙킹할 때 사용합니다.
  - 예) 플레이 횟수, 가챠 이용 횟수 등
  - 사용 코드: **IncrCustomParameter** 메소드를 이용하여 이벤트 발생시 증가된 횟수 (Integer)을 SDK에 전달합니다.

먼저 커스텀 파라미터를 특정할 수 있는 문자열 형태의 Unique Key 값을 정합니다. (예: "level", "facebook_flag", "play_count") 앱을 처음 실행하는 시점 또는 사용자가 로그인하는 시점에 커스텀 파라미터의 현재 상태 값을 설정합니다.

```cs
void Start() {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  plugin.SetCustomParameter("level", User.Level);
  plugin.SetCustomParameter("facebook_flag", User.HasFacebookAccount);
  plugin.StartSession();
}
```

커스텀 파라미터의 값이 변경되는 시점 (이벤트 발생 시)에 상태 값을 갱신하거나 횟수를 증가시킵니다.
  
```cs
void onUserLevelChanged(int level) {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.SetCustomParameter("level", level);
}

void OnFinishGame() {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.IncrCustomParameter("play_count", 1);
}
```

위와 같이 코딩을 마치고 앱을 실행하면 SDK는 저장한 커스텀 파라미터 값을 Nudge 서버로 전송합니다.  Nudge 서버는 활성화된 커스텀 파라미터의 값만을 저장하기 때문에 반드시 [Dashboard](https://dashboard.nudge.do)에 접속하여 해당 커스텀 파라미터를 활성화 (Activate)해야 합니다.

<img src="https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/custom_parameter_index.png">

Overview 메뉴 -> Settings - Custom Parameters 버튼을 클릭하면 커스텀 파라미터 목록이 표시됩니다. 해당 커스텀 파라미터의 Unique Key를 찾은 다음, 이름 ('Name')을 입력하고 활성화 (Activate)합니다.

#### Stickiness Custom Parameter

Stickiness 커스텀 파라미터는 사용자의 stickiness를 측정하기 위해 사용하는 특수한 형태의 커스텀 파라미터입니다. 예를 들어 사용자의 플레이 횟수를 커스텀 파라미터로 지정하고 Stickiness 커스텀 파라미터로 설정하면 해당 사용자의 '오늘 총 플레이 횟수', '최근 7일간의 총 플레이 횟수', '최근 7일간의 일평균 플레이 횟수' 등을 세그먼트 필터로 사용할 수 있습니다. Stickiness 커스텀 파라미터를 이용하면 사용자들의 충성도 (또는 몰입도)에 따라 세그먼트를 나누고 모니터링할 수 있습니다.

Stickiness 커스텀 파라미터로 사용하고자 하는 커스텀 파라미터는 반드시 **IncrCustomParameter** 메소드를 이용해야 합니다.  Stickiness 커스텀 파라미터를 사용하고자 하는 경우 해당 커스텀 파라미터를 활성화한 후 support@nudge.do 로 메일 주시기 바랍니다.

* * *

### Marketing Moment

마케팅 모멘트는 유저에게 메세지를 전달하고자 하는 상황을 의미합니다. (예: 캐릭터 레벨 업, 퀘스트 달성, 스토어 페이지 진입)

마케팅 모멘트 기능을 사용하여 지정된 상황에 알맞는 캠페인이 노출되도록 할 수 있습니다.

마케팅 모멘트 설정은 [Dashboard](https://dashboard.nudge.do) 사이트를 접속하여 앱의 Overview 메뉴 -> Settings - Marketing Moments 버튼을 클릭하여 확인할 수 있습니다.

SDK 적용을 위해서는 Dashboard에서 지정된 각 마케팅 모멘트의 '인덱스' 값이 필요합니다. 인덱스 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 지정하여 이용하는 것을 권장합니다.

각 모멘트 발생 시, Load() 메소드에 원하는 모멘트 인덱스 값을 인자로 넘겨주시면 간단히 적용이 완료됩니다.

(Load() 메소드에 인덱스를 설정하지 않은 경우, 인덱스 값은 '1' 값이 자동으로 지정됩니다.)

**Example**:  사용자가 메인 페이지로 이동할 시에 설정한 컨텐츠를 노출

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.Load(EVENT_INDEX_MAIN_PAGE);  // 메인 페이지에 설정한  컨텐츠를 노출
plugin.Show();
```

**Example**: 사용자의 게임 캐릭터가 레벨업을 했을 때 설정한 컨텐츠를 노출

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, level); // 사용자 level 정보를 가장 최신으로 업데이트
plugin.Load(EVENT_INDEX_LEVEL_UP); // 레벨업 이벤트에 설정한 컨텐츠를 노출
plugin.Show();
```

## Advanced

### Timeout Interval

메시지의 최대 로딩 시간을 직접 지정하실 수 있습니다. 지정된 시간 내에 메시지가 로딩되지 못한 경우, 사용자에게 메시지를 표시하지 않습니다.

최소 1초 이상 지정이 가능하며, 지정하지 않을 시 기본 값으로 5초가 지정 됩니다.

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.SetTimeoutInterval(5);
plugin.Load();
plugin.Show();
```

* * *

## Reference

### Deep Link

캠페인의 Deep Link 설정 시에 Custom URL Schema를 지정할 수 있습니다. 이를 통해 사용자가 콘텐츠를 클릭할 경우, 자신이 원하는 특정 앱 페이지로 이동하는 등의 액션을 지정할 수 있습니다.

#### Android 환경에서 Custom URL 적용하기

먼저 Eclipse에서 Android Project를 생성하여 UnityPlayerActivity를 상속받은 'MainActvity' 클래스를 생성합니다. 그리고 AndroidMenefest.xml 파일을 아래와 같이 수정합니다.

```xml
  <activity android:name="com.MyCompany.ProductName.MainActivity">
      <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
      <intent-filter> 
             <action android:name="android.intent.action.VIEW" /> 
             <category android:name="android.intent.category.DEFAULT" /> 
             <category android:name="android.intent.category.BROWSABLE" /> 
             <data android:scheme="your_scheme" android:host="your_action" />
        </intent-filter> 
  </activity>
```

MainActvity에 아래와 같은 코드를 추가하여 Url 값을 유니티 엔진으로 넘겨줍니다.

```java
@Override
public void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);

  Uri uri = getIntent().getData();
  handleUri(uri);
}

@Override
protected void onNewIntent(Intent intent) {
  super.onNewIntent(intent);
  setIntent(intent);

  Uri uri = intent.getData();
  handleUri(uri);
}

@Override 
public void startActivity(Intent intent) { 
  boolean isStartActivity = true;

  // Check intent 
  Uri uri = intent.getData(); 
  if (uri != null && uri.getScheme().equals("your_scheme")) { 
    isStartActivity = false; 
  }

  if (isStartActivity) { 
    super.startActivity(intent); 
  } else { 
    handleUri(uri);
  } 
}

private handleUri(Uri uri) {
  if (uri != null && uri.getScheme().equals("your_scheme")) { 
    UnityPlayer.UnitySendMessage("Nudge", "OnCustomURL", uri.toString());
  }
}
```

#### iOS 환경에서 Custom URL 적용하기

iOS의 경우 1개의 이벤트체서 모든 URL 처리가 가능하기 때문에 비교적 간단하게 적용할 수 있습니다.

1) Info.plst 파일을 열어 사용할 URL Schema 정보를 설정 합니다.

<img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png" />

2) AppController.mm 파일을 열어 handleOpenURL 메소드를 구현합니다. 호출되는 URL 값을 유니티 게임 오브젝트에 전달합니다. 

```mm
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url 
{  
  if ([url.scheme isEqualToString:@"your_scheme"])
  {
        NSString *frescaURL = [url absoluteString];
        UnitySendMessage("Nudge", "OnCustomURL", [frescaURL UTF8String]);
  }
  return YES;
}
```

#### Unity에서 url 값 확인 및 처리하기

위 예제에서는 모두 'Fresca' 게임 오브젝트의 'OnCustomURL' 이벤트 메소드를 호출하여 값을 전달하였습니다. 이제 Unity 게임 상에서 그 값을 직접 받아 확인하고 처리할 수 있습니다.

```java
public void OnCustomURL(string url)
{
  Debug.Log("OnCustomURL = " + url); 
  // URL 값을 파싱하여 지정된 액션을 실행
}
```

* * *

### Cross Promotion Configuration

Incentivized CPI & CPA 캠페인 기능을 사용하여, 사용자가 Media App에서 Advertising App의 광고를 보고 앱을 설치하였을 때 보상으로 Media App의 아이템을 지급할 수 있습니다.

- Medial App: 다른 앱의 광고를 노출하고, 광고 대상의 앱을 설치한 사용자들에게 보상을 지급하는 앱
- Advertising: Media App에 광고가 노출되는 앱.

Incentivized CPI & CPA 캠페인에 대한 보다 자세한 설명 및 [Dashboard](https://dashboard.nudge.do) 사이트에서의 설정 방법은 [크로스 프로모션 캠페인 이해하기](https://adfresca.zendesk.com/entries/22033960) 가이드를 참고하여 주시기 바랍니다.

SDK 적용을 위해서는 Advertising App에서의 URL Schema 설정 및 Media App에서의 Reward Item 지급 기능을 구현해야 합니다.

#### Advertising App 설정하기:
  1. Android

  Android 플랫폼의 경우 앱의 패키지 이름을 이용하여 광고를 노출한 앱이 실제로 디바이스에 설치되었는지 검사하게 됩니다. 따라서 Advertising App 앱의 패키지 이름을 확인하고 CPI Identifier로 사용합니다.

  AndroidManifest.xml 파일을 열어 패키지 이름을 확인합니다.

  ```xml
  <?xml version="1.0" encoding="utf-8"?>

  <manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.adfresca.demo">
    <application>
    .....
    </application>
  </manifest>
  ```

  위 경우 [Dashboard](https://dashboard.nudge.do) 사이트에서 Advertising App의 CPI Identifier 값을 'com.adfresca.demo' 으로 설정하게 됩니다. 

  2. iOS

  iOS 플랫폼의 경우 URL Schema 값을 이용하여 광고를 노출한 앱이 실제로 디바이스에 설치되었는지 검사하게 됩니다. 따라서 Advertising App 앱의 URL Schema을 설정하고 CPI Identifier로 사용합니다.

  Xcode 프로젝트의 Info.plst 파일을 열어 사용할 URL Schema 정보를 설정 합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  위 경우 [Dashboard](https://dashboard.nudge.do) 사이트에서 Advertising App의 CPI Identifier 값을 'myapp://' 으로 설정하게 됩니다. 
  iOS 플랫폼의 경우 URL Schema 값이 다른 앱과 중복될 수 있습니다. 정상적인 캠페인 진행을 위해서는 최대한 Unique한 값을 선택해야 합니다.
  
  마지막으로, Incentivized CPI 캠페인을 진행할 경우, Advertising App의 SDK 설치는 필수가 아니며 CPI Identifier 설정만 진행되면 됩니다. 하지만 Incentivized CPA 캠페인을 진행할 경우 반드시 SDK 설치가 필요하며 보상 조건으로 지정한 마케팅 모멘트르 발생되어야 합니다. 사용자가 보상 조건을 완료한 이후 아래와 같이 유니티 코드로 지정한 마케팅 모멘트 호출합니다.

  ```cs
  // 튜토리얼 완료 이벤트를 보상 조건으로 지정한 경우
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Load(EVENT_INDEX_TUTORIAL);  
  plugin.Show(EVENT_INDEX_TUTORIAL);
  ```

#### Media App SDK 적용하기:

  Media App에서 보상 지급 여부를 확인하고, 사용자에게 아이템을 지급하기 위해서는 SDK 가이드의 [Give Reward](#give-reward) 항목의 내용을 구현합니다.

* * *

### Image Push Notification

Image Push Notification 기능 적용에 대한 내용은 Android SDK 가이드의 ["Image Push Notification"](https://github.com/nudge-now/sdk-android/blob/master/README.kor.md#image-push-notification) 내용을 참고하여 진행이 가능합니다.

* * *

### Proguard Configuration

안드로이드에서 Proguard 툴을 이용하여 APK 파일을 보호하는 경우 몇 가지 예외 처리 작업을 진행해야 합니다. Nudge SDK와 SDK에 포함된 OpenUDID 및 Google Gson에 대한 예외 처리를 아래와 같이 적용합니다.

```java
-keep class com.adfresca.** {*;} 
-keep class com.google.gson.** {*;} 
-keep class org.openudid.** {*;} 
-keep class sun.misc.Unsafe { *; }
-keepattributes Signature 
```

* * *

## Troubleshooting

특정 개발 환경의 안드로이드 플랫폼에서 간혹 인-앱 메시징 뷰 터치 시 뷰 뒷면의 게임 UI가 터치되는 사례가 있습니다. 이 경우 IsVisible() 메소드를 이용하여 현재 뷰가 보여지는 상태인지 검사한 후 터치 이벤트를 무시하도록 적용하여 문제를 해결할 수 있습니다.

그 외에 콘텐츠가 화면에 제대로 출력되지 않거나, 에러가 발생하는 경우 SDK에서 에러 로그를 출력할 수 있습니다.

**Android**

안드로이드 플러그인의 경우는 exception 로그를 자동으로 출력하도록 구현되어 있습니다. "AdFresca" 라는 tag가 설정된 로그를 검색하여 확인이 가능합니다.

**iOS**

iOS의 경우 Xcode 프로젝트에서 AdFrescaViewDelegate를 구현하여 로그를 출력할 수 있습니다. (혹은 UnitySendMessage 메소드를 이용하여 유니티로 이벤트를 전달할 수 있습니다.)

```objective-c
// UnityAppController.h
@interface UnityAppController : NSObject<UIApplicationDelegate, AdFrescaViewDelegate>
{
  .....
}
```

```objective-c
// UnityAppController.mm

- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions
{
  ....
  AdFrescaView *view = [AdFrescaView shardAdView];
  view.delegate = self;
}

- (void)fresca:(AdFrescaView *)fresca didFailToReceiveAdWithException:(AdException *)error {  
  NSLog(@"AdException message : %@", [error message]);
}
```

* * *

## Release Notes
- **v2.3.0 _(2016/01/23 Updated)_**
  - [리워드 지급 기능](#give-reward)이 개선되어 지급 완료 확인이 가능해졌습니다. 기존의 메소드가 삭제 되었기 때문에 반드시 새로운 가이드를 참고하여 코드를 변경해야 합니다.
- 2.2.8
    - [iOS SDK 1.5.6](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
- **v2.2.6 _(2015/03/20 Updated)_**
    - [Android SDK 2.4.8](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- **v2.2.5 _(2015/02/13 Updated)_**
    - [Android SDK 2.4.7](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - [iOS SDK 1.5.3](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
    - IsVisible() 메소드가 추가되었습니다.
- v2.2.4 _(2015/01/29 Updated)
    - [Android SDK 2.4.6](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - [iOS SDK 1.5.2](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.2.3 _(2014/12/05 Updated)_
  - Purchase 객체에 HARD_ITEM, SOFT_ITEM purchase type이 추가되고 ACTUAL_ITEM, SOFT_ITEM 값이 deprecated 되었습니다. 자세한 내용은 [In-App Purchase Tracking](#in-app-purchase-tracking) 항목을 참고하여 주세요.
- v2.2.3 (2014/11/11 Updated)
    - iOS 플랫폼에서의 [In-App Purchase Tracking](#in-app-purchase-tracking) 기능을 지원합니다.
    - iOS 플랫폼에서의 [Sales Promotion](#sales-promotion) 기능을 지원합니다.
    - 안드로이드 플랫폼에서 GCM Push Registration ID를 직접 지정할 수 있는 SetGCMPushRegistrationIdentifier() 메소드가 추가되었습니다.   
    - [iOS SDK 1.4.8](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.2.2 (2014/09/30 Updated)
    - iOS 플랫폼의 A/B 테스트 기능을 지원합니다. 해당 기능은 별도의 코딩 작업 없이 이용 가능합니다.
    - [Android SDK 2.4.4](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - [iOS SDK 1.4.6](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.2.1 _(8/15/2014 Updated)_
    - 리워드 지급 시에 시큐리티 토큰값을 이용하여 보안 이슈를 해결할 수 있습니다. 자세한 내용은 [Give Reward](#give-reward) 항목을 참고하여 주세요.
    - Sales Promotion 캠페인 기능을 이용하여 아이템의 프로모션 기능을 지원합니다. 자세한 내용은 [Sales Promotion](#sales-promotion) 항목을 참고하여 주세요.
    - [In-App Purchase Tracking](#in-app-purchase-tracking) 기능에서 cancelPromotionPurchase() 메소드가 추가되었습니다. 
    - 이미지 메시지의 **Tap Area** 기능을 지원합니다.
    - Android SDK가 캠페인 매칭 시에 여러 개의 캠페인이 동시에 매칭될 수 있도록 지원합니다.. 새로운 SDK는 순차적으로 매칭된 캠페인들의 메시지를 표시합니다.
    - iap beta 버전이 2.2.1부터 통합되었습니다. 
    - [Android SDK 2.4.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.2.0-beta3 _(4/6/2014 Updated)_
    - iOS SDK 설치 과정에서 AdSupport framework 추가가 필수항목에서 제외됩니다. IFA 수집을 하지 않아도 SDK 이용이 가능하도록 수정되었습니다. 보다 자세한 내용은 [iOS SDK - Installation](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#installation) 항목을 참고하여 주세요.
    - v2.1.8에서 적용된 '인-앱 메시징 캠페인을 통한 Reward Item 지급 기능'을 지원합니다.
    - v2.1.8에서 적용된 Incentivized CPA 캠페인 기능을 지원합니다. 자세한 내용은 [CPI Identifier](#cpi-identifier) 항목을 참고하여 주세요
    - v2.1.8에서 개선된 [Reward Item](#reward-item) 기능이 적용되었습니다. 
    - [Android SDK 2.4.0-beta4](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - [iOS SDK 1.3.5](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.2.0-beta2 _(1/31/2014 Updated)_ 
    - [Android SDK 2.4.03](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.2.0-beta1 _(1/14/2014 Updated)_ 
    - 앱 내에서 발생하는 In-App Purchase 데이터를 트랙킹할 수 있는 기능이 추가되었습니다. 자세한 내용은 [In-App Purchase Tracking](#in-app-purchase-tracking) 항목을 참고하여 주세요.
    - [Android SDK 2.4.02](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.8 _(4/6/2014 Updated)_
   - iOS SDK 설치 과정에서 AdSupport framework 추가가 필수항목에서 제외됩니다. IFA 수집을 하지 않아도 SDK 이용이 가능하도록 수정되었습니다. 보다 자세한 내용은 [iOS SDK - Installation](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#installation) 항목을 참고하여 주세요.
   - 인-앱 메시징 캠페인을 통한 Reward Item 지급 기능을 지원합니다.
   - Incentivized CPA 캠페인 기능을 지원합니다. 자세한 내용은 [CPI Identifier](#cpi-identifier) 항목을 참고하여 주세요
   - SetAndroidRewardItemListener 구현 기능이 추가되어, 지급 가능한 아이템이 발생할 시에 자동으로 이벤트가 발생합니다. 보다 자세한 내용은 [Reward Item](#reward-item) 항목을 참고하여 주세요.
    - [Android SDK 2.3.4](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - [iOS SDK 1.3.5](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.7 _(1/31/2014 Updated)_
    - [Android SDK 2.3.3](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.6 _(1/10/2014 Updated)_ 
    - [Android SDK 2.3.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - Unity 4.3.x for Android 버전에서 ForwardNativeEventsToDalvik 옵션이 설정되지 않은 경우 터치 이벤트가 동작하지 않습니다. 이를 해결하기 위한 자세한 적용 방법은 [Installation](#installation) 항목을 참고하여 주세요.
- v2.1.5 _(12/01/2013 Updated)_ 
    - [iOS SDK 1.3.4](https://adfresca.zendesk.com/entries/21346861#release-notes) 버전을 지원합니다.
- v2.1.4 _(11/27/2013 Updated)_ 
    - [iOS SDK 1.3.3](https://adfresca.zendesk.com/entries/21346861#release-notes) 버전을 지원합니다.
    - [Android SDK 2.3.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.3 _(10/01/2013 Updated)_ 
    - [Android SDK 2.2.3](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.2 _(08/19/2013 Updated)_ 
    - [iOS SDK 1.3.2](https://adfresca.zendesk.com/entries/21346861#release-notes) 버전을 지원합니다.
- v2.1.1
    - [Android SDK 2.2.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.0 _(08/08/2013 Updated)_
    - [Android SDK 2.2.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - Android Platform 에서는 TestDeviceId() 메소드 대신 PrintTestDeviceIdByLog() 메소드를 사용하여 연결된 디바이스의 아이디를 확인하도록 변경 되었습니다.
- v2.0.1 _(07/26/2013 Updated)_
    - Plugin에 포함된 GCMIntentService 클래스를 이용하는 경우, 앱이 완전히 종료된 상황에서 푸시 메시지 수신 시 에러 메시지가 발생하는 버그를 수정하였습니다.
    - AndroidPlugin.cs 파일의 기본 매개변수 설정을 삭제하였습니다. 
    - 포함된 Android SDK를 2.1.3 버전으로 업데이트하였습니다.
- v2.0.0 _(07/10/2013 Updated)_
    - _Incentivized CPI_캠페인을 위한 API 가 추가되었습니다.
