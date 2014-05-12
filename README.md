## Contents
- [Basic Integration](#basic-integration)
    - [Installation](#installation)
    - [Start-Session](#start-session)
    - [In-App Messaging](#in-app-messaging)
    - [Push Messaging](#push-messaging)
    - [Test Device Registration](#test-device-registration)
- [IAP & Reward](#iap-&-reward)
  - [In-App Purchase Tracking (Beta)](#in-app-purchase-tracking-beta)
  - [In-App-Purchase Count](#in-app-purchase-count)
  - [Give Reward](#give-reward)
- [Dynamic Targeting](#dynamic-targeting)
  - [Custom Parameter](#custom-parameter)
  - [Marketing Moment](#marketing-moment)
- [Advanced](#advanced)
  - [Custom Banner (Android Only)](#custom-banner)
  - [Baidu Push Service Integration](#baidu-push-service-integration)
  - [AFShowListener](#afshowlistener)
  - [Timeout Interval](#timeout-interval)
- [Reference](#reference)
  - [Custom URL Schema](#custom-url-schema)
  - [Cross Promotion Configuration](#cross-promotion-configuration)
  - [Google Referrer Tracking](#google-referrer-tracking)
  - [Image Push Notification](#image-push-notification)
  - [Proguard Configuration](#proguard-configuration)
- [Trouble Shooting](#trouble-shooting)
- [Release Notes](#release-notes)

* * *

## Basic Integration

### Installation

아래 링크를 통해 SDK 파일을 다운로드 합니다.

[Android SDK Download](http://file.adfresca.com/distribution/sdk-for-Android.zip) (v2.3.4)

[Android SDK Download without Gson Library](http://file.adfresca.com/distribution/sdk-for-Android-wihtout-gson.zip) (v2.3.4)

[Android SDK with IAP Tracking Beta Download](http://file.adfresca.com/distribution/sdk-for-Android-iap-beta.zip) (v.2.4.0-beta4)

**AdFresca.jar** 파일은 **lib** 폴더에, **adfresca_attr.xml** 파일은 **res/values** 폴더에 각각 복사합니다.

<img src="https://adfresca.zendesk.com/attachments/token/bja88u9zake4knm/?name=add_adfresca_jar_and_attr_xml.png" width="300"/>

프로젝트를 오른쪽 클릭 후, **Properties** 메뉴를 클릭합니다.

좌측 **Java Build Path** 메뉴에서 **Libraries** 탭을 들어간 후, **Add JARs** 버튼을 클릭하여 **AdFresca.jar** 파일을 추가합니다.

<img src="https://adfresca.zendesk.com/attachments/token/ogcnzf3kmyzbcvg/?name=add_jar.png" width="600" />

**AndroidManifest.xml** 내용 추가하기

마지막으로 아래와 같이 필요한 퍼미션, 서비스, 리시버 등을 추가합니다.

```xml
<manifest package="your.app.package">
  <application>
    <activity/>

    <!-- Device ID 수집을 위한 OpenUDID 서비스 등록 -->
    <service android:name="org.openudid.OpenUDID_service">
      <intent-filter>
        <action android:name="org.openudid.GETUDID" />
      </intent-filter>
    </service>

    <!-- Push Messaging 기능을 사용하기 위한 액티비티 등록 -->
    <activity android:name="com.adfresca.ads.AdFrescaPushActivity" />

    <!-- Cross Promotion 및 Reward 기능을 위한 액티비티 등록 -->
    <activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
   
    <!-- Google Referrer Tracking 을 위한 Boradcast Receiver 등록 -->
    <receiver android:name="com.adfresca.sdk.referer.AFRefererReciever" android:exported="true">
      <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
      </intent-filter>
    </receiver>
  </application>

  <!-- Permission 등록 -->
  <uses-permission android:name="android.permission.INTERNET"/>
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
  <uses-permission android:name="android.permission.READ_PHONE_STATE" /> 
  <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
  <uses-permission android:name="android.permission.VIBRATE" />
</manifest>
```

### Start Session

앱이 최초로 실행될 때 SDK에 API Key를 설정하고, 앱이 실행되었음을 기록합니다.

```java
protected void onCreate(Bundle savedInstanceState) {
  ....
  AdFresca.setApiKey(API_KEY);
  AdFresca.getInstance(this).startSession();
}
```

- API Key는 [Dashboard](https://admin.adfresca.com) 사이트에서 앱 추가 후 Overview 메뉴의 Settings - API Keys 버튼을 클릭하여 확인이 가능합니다.
- startSession() 메소드는 **한 번만** 실행되도록 해야 합니다.

### In-App Messaging

인-앱 메시징 기능을 통하여 앱 사용자에게 실시간으로 원하는 콘텐츠를 표시할 수 있습니다. 콘텐츠의 노출을 원하는 위치에 아래와 같이 코드를 적용합니다.

```java
public void onResume(Bundle savedInstanceState) {
  ...
  AdFresca fresca = AdFresca.getInstance(this);
  fresca.load();
  fresca.show();
}
```

코드가 실행되면 다음과 같은 화면이 보여집니다. 정상적으로 콘텐츠 뷰가 화면에 표시되고, 터치 시 앱스토어 페이지로 이동하는지 확인합니다.

<img src="https://adfresca.zendesk.com/attachments/token/zngvftbmcccyajk/?name=device-2013-03-18-133517.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/phn4fcpvbi2damx/?name=device-2013-03-18-133443.png" height="240" />

### Push Messaging

푸시 메시징 기능을 적용하여 사용자들에게 손쉽게 메시지를 전송할 수 있습니다.

SDK를 적용하기 이전에 [Google API Console](https://cloud.google.com/console) 사이트에서 프로젝트를 생성하고, [Dashboard](https://admin.adfresca.com) 사이트에 설정할 GCM API Key 및 SDK 적용에 필요한 GCM_SENDER_ID (Project Number) 값을 얻어야 합니다.

'[Android Push Notification 설정 및 적용하기 (GCM)](https://adfresca.zendesk.com/entries/28526764)' 가이드를 참고하여 필요한 값들을 얻습니다.

이제 SDK 적용을 시작합니다.

1) GCM Helper Library 설치 확인하기
  - GCM 서비스를 이용하기 위해서는 구글에서 제공하는 GCM Client 라이브러리가 설치되어 있어야 합니다. 
  - 기존에 GCM 라이브러리가 설치되어 있지 않는 경우, 구글에서 제공하는 [GCM Helper Library](http://code.google.com/p/gcm/source/browse/) 를 다운로드 받습니다. /gcm-client/dist 폴더에 포함된 **gcm.jar** 파일을 프로젝트에 복사합니다.
    
2) AndroidManifest.xml 확인하기

```xml
<application>
  <activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
</application>

<uses-permission android:name="android.permission.READ_PHONE_STATE" /> 
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
<uses-permission android:name="android.permission.VIBRATE" />
```

[Installation](#installation) 과정에서 위 항목이 정상적으로 추가되었는지 확인합니다. 만약 GCM 기능을 처음 사용하는 경우, 여기를 클릭하여 전체 xml 내용을 확인합니다. 

3) GCM Registration ID 값을 SDK에 등록하기

```java
AdFresca.setPushRegistrationId("GCM_REGISTRATION_ID_OF_THIS_DEVICE");
```

GCM Registration ID를 추출하는 코드는 여기 링크를 통하여 확인할 수 있습니다.

4) GCMIntentService 클래스의 이벤트 코드 구현하기

```java
protected void onRegistered(Context context, String registrationId) {
 AdFresca.handlePushRegistration(registrationId);
}

protected void onUnregistered(Context context, String registrationId) {
  AdFresca.handlePushRegistration(null);
}

protected void onMessage(Context context, Intent intent) {

  // AD fresca를 통해서 수신한 notification인지 확인
  if (AdFresca.isFrescaNotification(intent)) { 
    Class<?> targetActivityClass = YourMainActivity.class;
    String appName = context.getString(R.string.app_name);
    int icon = R.drawable.icon;
    long when = System.currentTimeMillis();

    // 푸시 메시지를 표시합니다. 
    AFPushNotification notification = AdFresca.generateAFPushNotification(context, intent, targetActivityClass, appName, icon, when);
    notification.setDefaults(Notification.DEFAULT_ALL); 
    AdFresca.showNotification(notification);
  } 

}
```

- 푸시 메시지를 터치하면 targetActivityClass로 지정한 액티비티를 기본적으로 실행합니다. 단, 푸시 메시지에 Click URl 값을 실행한 경우는 해당 URL이 실행됩니다.
- 예제와 같이 AFPushNotification 객체의 setDefaults(), setSound() 메소드를 호출하면 메시지 수신 시에 알림음, 사운드 재생 등을 설정할 수 있습니다.

### Test Device Registration

AD fresca는 테스트 모드 기능을 지원하여 테스트를 원하는 디바이스에만 지정한 캠페인의 콘텐츠를 화면에 표시하고 푸시 메시지를 전송할 수 있습니다. 이로 인해 SDK가 적용된 앱이 이미 앱스토어에 출시된 경우, 게임 운영팀 혹은 개발팀에게만 새로운 메시지를 전달할 수 있도록 지원합니다.

테스트 기기 등록을 위한 아이디 값은 SDK를 통해 추출이 가능하며 2가지 방법을 지원 합니다.
 
1. getTestDeviceId() 메소드를 사용하여 로그로 출력하는 방법
  - 테스트에 사용할 기기를 개발PC에 연결한 후 로그를 통해 해당 아이디 값을 출력하여 확인 합니다. 

  ```java
  AdFresca fresca = AdFresca.getInstance(this);
  Log.d(TAG, "AD fresca Test Device ID is = " + fresca.getTestDeviceId());
  ```

2. setPrintTestDeviceId() 메소드를 사용하여 콘텐츠 뷰에 기기 아이디를 화면에 표시하는 방법
  - 개발자가 기기를 직접 연결할 수 없는 경우, 설정을 활성화 한 상태로 앱 빌드를 전덜하여 설치합니다. 화면에 표시된 기기 아이디를 직접 기록하여 등록할 수 있습니다.
  - 담당 마케터가 원격에서 근무하는 경우 해당 기능을 유용하게 사용할 수 있습니다.
  - 설정이 활성화된 상태로 앱이 배포되지 않도록 주의해야 합니다.

  ```java
    AdFresca fresca = AdFresca.getInstance(this);
    fresca.setPrintTestDeviceId(true);
    fresca.load();
    fresca.show();
  ```

* * *

## IAP & Reward

### In-App Purchase Tracking (Beta)

_**(현재 In-App-Purchase Tracking 기능은 SDK 2.4.0-beta 버전에서만 지원됩니다.)**_

_In-App-Purchase Tracking_  기능을 통하여 현재 앱에서 발생하고 있는 모든 인-앱 결제를 분석하고 캠페인 타겟팅에 이용할 수 있습니다.

AD fresca의 In-App-Purchase Tracking은 2가지 유형이 있습니다.

1. 실제 화폐를 통해 결제되는 Actual Item Purchase Tracking (예: USD $1.99를 결제하여 Gold 100개 아이템을 구입)
2. 가상 화폐를 통해 결제되는 Virtual Item Purchase Tracking (예: Gold 10개를 이용하여 포션 아이템을 구입)

위 2가지 유형의 데이터를 모두 Tracking 함으로써 앱의 매출뿐만 아니라 인-앱 사용자들의 아이템 구매 추이 분석까지 가능합니다.

아이템 정보 등록을 위한 별도의 작업은 필요하지 않으며, 클라이언트에서 결제된 아이템 정보가 자동으로 대쉬보드에 등록되는 방식입니다. (아이템 리스트 확인은 대쉬보드 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.)

아래의 적용 예제를 참고하여 간단히 In-App-Purchase Tracking 기능을 적용합니다.

#### Actual Item Tracking

Actual Item의 결제는 각 앱스토어별 인-앱 결제 라이브러리를 통해 이루어집니다. 각 결제 라이브러리에서 _'결제 성공'_ 이벤트가 발생 할 시에 AFPurchase 객체를 생성하고 logPurchase(purchase) 메소드를 호출합니다.

적용 예제: Google Play 결제 
```java
// Callback for when a purchase is finished
IabHelper.OnIabPurchaseFinishedListener mPurchaseFinishedListener = new IabHelper.OnIabPurchaseFinishedListener() {
  public void onIabPurchaseFinished(IabResult result, Purchase purchase) {
    Log.d(TAG, "Purchase finished: " + result + ", purchase: " + purchase);

    if (mHelper == null || result.isFailure() || !verifyDeveloperPayload(purchase)) {
      ......
      return;
    }

    Log.d(TAG, "Purchase successful.");
    if (purchase.getPurchaseState() == 0) {
      SkuDetails detail = currentInventory.getSkuDetails(purchase.getSku());
            
      String itemId = purchase.getSku(); // Sku value or any unique value of purchased item
      String currencyCode = "KRW"; // The currencyCode must be specified in the ISO 4217 standard. (ex: USD, KRW, JPY)
      Double price =  parsePrice(detail.getPrice()); // For Google Play, you can get the price value from SkuDetails
      Date purhcaseDate = new Date(purchase.getPurchaseTime());
      String orderId = purchase.getOrderId();
      String receiptData = purchase.getOriginalJson();
      String signature = purchase.getSignature();

      AFPurchase actualPurchase = new AFPurchase.Builder(AFPurchase.Type.ACTUAL_ITEM)
                            .setItemId(itemId)
                            .setCurrencyCode(currencyCode)
                            .setPrice(price)
                            .setPurchaseDate(purhcaseDate)
                            .setReceipt(orderId, receiptData, signature)
                            .build();

      AdFresca.getInstance(MainActivity.this).logPurchase(actualPurchase);
    }
    
    ......
    }
};
```

위 예제는 Google Play 결제 라이브러리를 기준으로 작성되었지만 아마존이나 티스토어 등 모든 결제 라이브러리에서도 AFPurchase 객체에 필요한 값을 얻어올 수 있습니다.

Actual Item을 위한 AFPurchase.Builder의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
setItemId(string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. AD fresca 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
setCurrencyCode(string) | ISO 4217 표준 코드를 설정합니다. Google Play의 경우 'Default price' 에 설정되는 Currency Code 값을 이용하며 타 결제 라이브러리의 경우는 보통 이용 가능한 Currency Code가 고정되어 있습니다 (예: 아마존은 USD, 티스토어는 KRW). 또는 자체 백엔드 서버에서 결제하는 아이템의 Currency Code를 내려받아 설정할 수 있습니다.
setPrice(double) | 아이템의 가격을 설정합니다. 결제 라이브러리에서 주는 값을 이용하거나, 자체 백엔드 서버에서 가격을 내려받아 설정할 수 있습니다. 
setPurchaseDate(date) | 결제된 시간을 Date 객체 형태로 설정합니다. 값이 설정되지 않은 경우 AD fresca 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.
setReceipt(string, string, string) | 추후 Receipt Verficiation 기능을 위해 필요한 데이터를 설정합니다. 현재 버전의 SDK는 Google Play만 지원하며 타 결제 라이브러리의 경우는 값을 설정하지 않습니다.

#### Virtual Item Tracking

Virtual Item의 결제는 앱 내의 가상 화폐로 아이템을 결제한 경우를 의미합니다. 앱 내에서 가상 화폐를 이용한 결제 이벤트가 성공한 경우 아래 예제와 같이 AFPurchase 객체를 생성하고 logPurchase(purchase) 메소드를 호출합니다.

적용 예제: 
```java
public void onVirtualItemPurchased(Item item, Date purchasedDate) {
  AFPurchase virtualPurchase = new AFPurchase.Builder(AFPurchase.Type.VIRTUAL_ITEM)
                  .setItemId(item.getId()) // "long_sword"
                  .setCurrencyCode(item.getCurrencyCode()) // "gold"
                  .setPurchaseDate(purchaseDate) // Date object or null
                  .setPrice(item.getPrice()) // 10
                  .build();
  
  AdFresca.getInstance(this).logPurchase(virtualPurchase);
}
```

Virtual Item을 위한 AFPurchase.Builder의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
setItemId(string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. AD fresca 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
setCurrencyCode(string) | 결제에 사용한 가상화폐 고유 코드를 설정합니다. (예: gold)
setPrice(double) | 가상 화폐로 결제한 가격 정보를 설정합니다. (예: gold 10개의 경우 10 값을 설정)
setPurchaseDate(date) | 결제된 시간을 Date 객체 형태로 설정합니다. 값이 설정되지 않은 경우 AD fresca 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.

#### IAP Trouble Shooting

logPurchase() 메소드를 통해 기록된 AFPurchase 객체는 AD fresca 서비스에 업데이트되어 실시간으로 대쉬보드에 반영됩니다. 현재까지 등록된 아이템 리스트는 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.

만약 아이템 리스트가 새로 갱신되지 않는 경우, AFPurchaseExceptionListener 구현하여 혹시 에러가 발생하고 있지 않은지 확인해야 합니다. 

만약 AFPurchase 객체의 값이 제대로 설정되지 않은 경우, AFPurchaseExceptionListener 통하여 에러 메시지를 표시하고 있으니 아래와 같이 코드를 적용하여 로그를 확인합니다.

```java
......
AdFresca.getInstance(this).logPurchase(purchase, new AFPurchaseExceptionListener(){
  public void onException(AFPurchase purchase, AFException e) {
    Log.e(TAG, (purchase == null ? "purchase=null" : purchase.toString()));
    Log.e(TAG, e.getMessage());
  }
});
```

### In-App Purchase Count

앱에서 IAP 기능을 사용하는 경우, 현재까지 사용자가 구매한 누적 횟수를 SDK에 설정하여 분석 및 타겟팅에 이용할 수 있습니다. 

**setNumberOfInAppPurchases(int)** 메소드를 사용하여 현재까지 사용자가 구매한 누적 횟수 값을 SDK에 설정합니다. 커스텀 파라미터와 마찬가지로 앱 실행 혹은 사용자 로그인 이후에 값을 지정하고, IAP 결제가 일어난 직후에 갱신된 누적 구매 횟수 값을 설정합니다.

```java
  AdFresca adfresca = AdFresca.getInstance(this);
  
  public void onCreate() {
    AdFresca fresca = AdFresca.getInstance(this);     
    fresca.setNumberOfInAppPurchases(User.inAppPurchaseCount);
    fresca.startSession();
  }
  
  .....
  
  public void onUserPurchasedItem(int totalPurchaseCount) {
    User.inAppPurchaseCount = totalPurchaseCount;
    
    AdFresca fresca = AdFresca.getInstance(this);     
    fresca.setNumberOfInAppPurchases(User.inAppPurchaseCount);
  }
```

(Advanced) SDK는 현재 설정한 inAppPurchaseCount 값을 로컬에 저장하여 두고 있습니다. 특정 이슈가 발생하여 해당 값을 확인 및 초기화 시키고 싶은 경우 getNumberOfInAppPurchases(), resetNumberOfInAppPurchases() 메소드를 사용할 수 있습니다.


### Give Reward

Reward 지급 기능을 적용하여 현재 사용자에게 지급 가능한 보상 아이템이 있는지 검사하고, 보상 아이템을 사용자에게 지급할 수 있습니다.

Annoucnement 캠페인의 'Reward Item' 항목을 설정했거나, Incentivized CPI & CPA 캠페인의 'Incentive Item' 을 설정한 경우 사용자에게 보상 아이템이 지급됩니다.

먼저 AndroidManifest.xml 내용을 확인합니다.

```xml
<manifest>   
  <application>
      <activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
   </application>
</manifest>
```

이제 구현을 위해서 아래 2가지 코드를 이용합니다.
- checkRewardItems 메소드 호출: 현재 지급 가능한 보상 아이템이 있는지 검사합니다. 사용자가 앱을 실행할 호출하는 것을 권장합니다.
- AFRewardItemListener 구현: 아이템 지급 조건이 만족되면 onReward 이벤트가 발생됩니다. 인자로 넘어온 아이템 정보를 이용하여 사용자에게 아이템을 지급합니다.

```java
@Override
public void onResume() {
  super.onResume();

  AdFresca.setRewardItemListener(new AFRewardItemListener(){
      @Override
      public void onReward(AFRewardItem item) {
        String logMessage = String.format("You got the reward item! (%s)", item.getName());
        Log.d(TAG, logMessage);
        
        // 아이템 고유 값 'uniqueValue'을 이용하여 사용자에게 아이템 지급
        sendItemToUser(item.getUniqueValue());  
      }});
          
  AdFresca adfresca = AdFresca.getInstance(this);
  adfresca.checkRewardItems();
}
```

캠페인 종류에 따라 onReward 이벤트의 발생 조건이 다릅니다.

- Annoucnement 캠페인: 캠페인이 앱 사용자에게 매칭되어 노출될 때 이벤트가 발생합니다
- Incentivized CPI 캠페인: 사용자의 Advertising App 설치가 확인된 후 이벤트가 발생합니다.
- Incentivized CPA 캠페인: 사용자의 Advertising App 설치가 확인되고 보상 조건으로 지정된 마케팅 이벤트가 호출된 후에 발생합니다.

만일 디바이스의 네트워크 단절이 발생한 경우 SDK는 데이터를 로컬에 보관하여 다음 앱 실행에서 아이템 지급이 가능하도록 구현되어 있기 때문에 항상 100% 지급을 보장합니다.

(기존의 getAvailableRewardItems 메소드는 Deprecated 상태로 변경되었지만, 호환성을 보장하여 정상적으로 동작하고 있습니다.)

* * *
