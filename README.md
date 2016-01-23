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
  - [AdFrescaViewDelegate](#adfrescaviewdelegate) 
  - [Timeout Interval](#timeout-interval) 
- [Reference](#reference)
  - [Deep Link](#deep-link)
  - [Cross Promotion Configuration](#cross-promotion-configuration)
  - [IFV Only Option](#ifv-only-option)
- [Troubleshooting](#troubleshooting)
- [Release Notes](#release-notes)

* * *

## Basic Integration

### Installation

아래 링크를 통해 SDK 파일을 다운로드 합니다.

[iOS SDK Download](http://file.adfresca.com/distribution/sdk-for-iOS.zip) 

SDK를 프로젝트에 추가하기 위해 아래의 절차가 필요합니다.

1) 제공되는 AdFresca 폴더를 Xcode 프로젝트에 Drag & Drop 하여 추가합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/4uzya7c9rw4twus/?name=Screen+Shot+2013-03-27+at+8.22.04+PM.png" width="600" />

2) System Configuration.framework, StoreKit.framework, AdSupport.framework(선택)를 Xcode 프로젝트에 추가합니다.
  
  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />
  
  - AdSupport.framework를 추가할 경우, SDK는 [IFA(Identifier For Advertisers)](https://developer.apple.com/library/ios/documentation/AdSupport/Reference/ASIdentifierManager_Ref/ASIdentifierManager.html#jumpTo_3) 값을 수집하여 디바이스(=앱 사용자) 구분에 사용합니다. Nudge SDK는 IFA 값을 사용하여 크로스 프로모션 캠페인 기능을 제공하고 캠페인 노출 이후 사용자의 앱 설치 및 액션 트랙킹을 위해 사용하고 있습니다. 
  - AdSupport.framework를 제외할 경우, [IFV(Identifier For Vendor)](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDevice_Class/Reference/UIDevice.html#jumpTo_7) 값을 사용합니다. 이 경우 크로스 프로모션 캠페인 기능을 이용할 수 없으며 IFV의 특성상 사용자가 앱을 삭제하고 재설치할 때 새로운 디바이스(=앱 사용자)로 인식될 수 있습니다. 

  만약, 앱 업데이트 과정에서 AdSupport.framework를 제외하거나 새로 추가하는 경우 [IFV Only Option](#ifv-only-option) 항목의 내용을 참고하여 주시기 바랍니다.

3) Build Setting의 Other Linker Flags 값을 –ObjC로 설정 혹은 추가합니다. 

  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />

4) Info.plist 파일의 'aps-environment' 값을 'production' 으로 설정합니다. (Push Notification 적용 시 반드시 확인해주시기 바랍니다.)

  <img src="https://adfresca.zendesk.com/attachments/token/bd7oz41zoh5zjs4/?name=Screen+Shot+2013-02-07+at+5.22.50+PM.png" width="600" />

  만약 앱이 가로 방향만을 지원한다면 'Initial interface orientation' 값을 'Landscape (right home button)' 으로 설정합니다.

  그리고, URL Scheme 값을 지정합니다. 아래의 예제는 'myapp' 이라는 스키마 값을 지정한 예제입니다. 해당 값은 크로스 프로모션 기능을 이용하기 위하여 사용됩니다.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

5) iOS 9 및 Xcode 7 이상 버전에서는 [App Transport Security](https://developer.apple.com/library/prerelease/ios/technotes/App-Transport-Security-Technote/) 기능이 기본적으로 활성화되어 있습니다. 때문에 SDK가 넛지 서버와 통신할 수 있도록 도메인 예외 설정을 해주어야 합니다. [Info.plist 예제](https://gist.github.com/sunku/2dba02239f168dfec5d9#file-nsapptransportsecurity-plist)를 확인하여 Xcode 설정을 수정합니다.

아무런 에러 없이 빌드가 성공헀다면 모든 설치가 정상적으로 완료된 것입니다. 만약 Duplicate Symbol 등의 Linking Error 가 발생하였다면 아래의 '[Troubleshooting](#troubleshooting)' 항목을 확인해주시기 바랍니다

### Start Session

이제 SDK 적용을 시작하기 위해 몇 가지 간단한 코드를 적용합니다. 첫 번째로 API Key를 설정하고 앱의 실행을 기록하는 startSession() 메소드를 적용합니다. API Key는 [Dashboard](https://dashboard.nudge.do) 사이트에서 등록한 앱을 선택한 후 Overview 메뉴의 Settings - API Keys 버튼을 클릭하여 확인이 가능합니다.

startSession() 메소드를 적용하면 앱이 최초로 실행되거나, 백그라운드에서 재실행될 때 자동으로 앱의 실행을 기록합니다.

```objective-c
// AppDelegate.m
#import <AdFresca/AdFrescaView.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  ....
} 
```

### Sign In

Sign In 기능은 사용자의 로그인 액션을 트랙킹합니다. 넛지는 이 메소드를 통해 전달된 회원 ID를 이용하여 사용자를 구분합니다. 회원 ID를 이용하여 1명의 회원이 복수 개의 디바이스를 사용하는 경우에도 중복으로 캠페인에 노출되거나 리워드를 지급 받는 경우를 방지할 수 있습니다. 또한 로그인한 사용자, 로그인하지 않은 사용자 등을 대상으로 캠페인을 실행할 수 있습니다.

사용자는 반드시 회원 또는 비회원으로 Sign In 되어야 하며 다른 사용자가 로그인을 하면 이전의 사용자는 자동으로 로그아웃됩니다. (따라서 별도의 Sign Out은 필요하지 않습니다.) 회원의 경우 로그인 이벤트 (자동 로그인 포함) 발생 시 **signIn(string)** 메소드에 회원 ID를 인자로 넘겨 호출하며 비회원의 경우 **signInAsGuest(string)** 메소드에 게스트 ID를 인자로 넘겨 호출합니다. 비회원을 트랙킹하기 위해 별도의 게스트ID를 사용하지 않는다면 인자 없이 **signInAsGuest()** 메소드를 호출하면 됩니다.

```objective-c
- (void)onAppStart {
  if(isSignedIn) {
    // 직접 로그인하거나, 자동로그인 모두 호출되도록 함
    [[AdFrescaView shared] signIn:@"user_id"];
  } else {
    // 비회원을 별도의 guest_id로 트랙킹하고 있다면 인자로 설정 가능
    // 비회원을 별도의 ID로 트랙킹하지 않는다면 인자를 설정하지 않아야 함
    [[AdFrescaView shared] signInAsGuest:@"guest_user_id"];
  }
}

signedUserId() 메소드를 사용하면 현재 로그인되어 있는 사용자의 ID를 리턴합니다 (게스트 ID를 지정하지 않은 비로그인 사용자의 경우는 디바이스 ID 값이 리턴). 이 메소드를 사용하여 정상적으로 로그인이 기록되어 있는지 테스트할 수 있습니다.

### In-App Messaging

인-앱 메시징 기능을 이용하여, 사용자에게 원하는 메시지를 실시간으로 전달할 수 있습니다. 메시지를 전달하고자 하는 시점에 load(), show() 메소드만을 호출하여 적용이 가능합니다. 메시지는 전면 interstitial 이미지, 텍스트, 혹은 iframe 웹페이지 형태로 화면에 표시될 수 있습니다. 메시지는 현재 플레이 중인 사용자가 인-앱 메시징 캠페인의 조건과 매칭된 경우에만 화면에 표시됩니다. 조건에 만족하는 캠페인이 없다면 사용자는 아무런 화면을 보지 않고 자연스럽게 플레이를 이어갑니다. 매칭과 관련한 인-앱 메시징의 다이나믹 타겟팅 기능은 아래의 [Dynamic Targeting](#dynamic-targeting) 항목에서 보다 자세히 설명하고 있습니다.

```objective-c
- (void)applicationDidBecomeActive:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView sharedAdView]; 
  [fresca load]; 
  [fresca show]; 
} 
```

첫 번째로 인-앱 메시징 코드를 적용한 경우, 아래와 같이 테스트 이미지 메시지가 표시됩니다. 해당 이미지를 터치하면 앱스토어 페이지로 이동합니다. 현재 보고 있는 테스트 메시지는 이후 테스트 모드 설정을 변경하여 더이상 보이지 않도록 설정하게 됩니다.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

### Push Messaging

푸시 메시징 기능을 이용하여 사용자가 앱을 실행하지 않을 때에도 언제든 메시지를 전달할 수 있습니다. 아래의 과정을 통하여 푸시 메시징 기능을 적용합니다.

1) APNS 인증서 파일(.p12)을 Dashboard에 등록하기
  - Keychain 툴을 이용하여 .cer 인증서 파일을 .p12로 변환하고 [Dashboard](https://dashboard.nudge.do) 사이트에 등록합니다.
  - 보다 자세한 설명은 [iOS Push Notification 인증서 설정 및 적용하기](https://adfresca.zendesk.com/entries/21714780) 가이드를 통하여 확인이 가능합니다.

2) Info.plist 확인하기 / Provision 확인하기
- Nudge는 APNS의 Production 환경만을 지원합니다. 때문에 빌드가 production으로 빌드되어야 정상적인 서비스 이용이 가능합니다.
- Info.plist 파일의 'aps-environment' 값을 'production' 으로 설정되어 있어야 합니다.
- App Store / Ad Hoc release에 사용하는 Provision 인증서를 사용하여 빌드되어야 합니다.

3) AppDelegate 코드 적용하기 

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  ...
  NSDictionary* userInfo = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
  if (userInfo != nil) [self application:application didReceiveRemoteNotification:userInfo];
} 

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
  // 만약 iOS Notification 설정에서 푸시를 거부한 경우 문자열 "(null)" 값이 전달되고 Nudge에서는 푸시를 발송하지 않습니다. 
  // 앱 내에 Push On/Off 기능이 있는 경우, off 시 deviceToken 값을 nil로 지정합니다. 
  [AdFrescaView registerDeviceToken:deviceToken];
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
  if ([AdFrescaView isFrescaNotification:userInfo]) {
    [AdFrescaView handlePushNotification:userInfo];
  }  
} 
```

4) 사용자가 앱 내에 Push On/Off 기능을 사용하여 설정을 변경한 경우 APNS 토큰 값을 업데이트합니다. 

```objective-c
-(void)didPushConfigChange:(BOOL)pushEnabled {
  if (pushEnabled) {
    [AdFrescaView registerDeviceTokenString:@"YOUR_APNS_DEVICE_TOKEN"];
  } else {
    [AdFrescaView registerDeviceTokenString:nil];
  }
} 
```

만약 푸시 노티피케이션 기능을 처음 적용하는 경우 [샘플 코드](https://gist.github.com/sunku/791f1ff2d7d1b37ca9f8#file-gistfile1-m)를 확인하여 필요한 모든 코드를 확인합니다.

### Test Device Registration

Nudge는 테스트 모드 기능을 지원하여 테스트를 원하는 디바이스에만 원하는 메시지를 전달할 수 있습니다. 이로 인해 SDK가 적용된 앱이 이미 앱스토어에 출시된 경우, 게임 운영팀 혹은 개발팀에게만 새로운 메시지를 전달하여 테스트할 수 있도록 지원합니다.

테스트 기기 등록을 위한 아이디 값은 SDK를 통해 추출이 가능하며 2가지 방법을 지원 합니다.
 
1. testDeviceId Property를 사용하여 로그로 출력하는 방법
  - 테스트에 사용할 기기를 개발PC에 연결한 후 로그를 통해 해당 아이디 값을 출력하여 확인 합니다. 

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  NSLog(@"Nudge Test Device ID = %@", fresca.testDeviceId); 
```

2. printTestDeviceId Property를 설정하여 뷰에 기기 아이디를 화면에 표시하는 방법
  - 개발자가 기기를 직접 연결할 수 없는 경우, 설정을 활성화 한 상태로 앱 빌드를 전덜하여 설치합니다. 화면에 표시된 기기 아이디를 직접 기록하여 등록할 수 있습니다.
  - 담당 마케터가 원격에서 근무하는 경우 해당 기능을 유용하게 사용할 수 있습니다.
  - 설정이 활성화된 상태로 앱이 배포되지 않도록 주의해야 합니다.

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  fresca.printTestDeviceId = YES;
  [fresca load];
  [fresca show];
  ```

테스트 디바이스 아이디를 확인한 이후에는, [Dashboard](https://dashboard.nudge.do)를 접속하여 'Test Device' 메뉴를 통해 디바이스 등록이 가능합니다.

### Test Mode

Nudge SDK는 테스트 모드 기능을 지원합니다. 테스트 모드를 활성화하면 현재 실행되는 SDK 메소드와 그 실행 결과가 로그 메시지로 출력됩니다. 이를 통하여 본인이 올바른 코드와 인자 값을 설정하고 있는지 검증할 수 있습니다.

  ```objective-c
  [AdFrescaView setTestMode:YES];
  ```

<img src="http://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/ios_sdk_test_mode.png" width="900" />

현재 테스트 모드는 'Start Session', 'Push Messaging', 'In-App Purchase Tracking', 'Custom Parameter', 'Stickiness Custom Parameter' 항목에 대한 로그를 지원합니다. 다른 항목의 경우는 추후 제공할 예정에 있습니다.

* * *

## IAP, Reward and Sales Promotion

### In-App Purchase Tracking

_In-App-Purchase Tracking_ 기능을 통하여 현재 앱에서 발생하고 있는 모든 인-앱 결제를 분석하고 캠페인 타겟팅에 이용할 수 있습니다.

Nudge의 In-App-Purchase Tracking은 2가지 유형이 있습니다.

1. 실제 화폐를 통해 결제되는 Hard Currency Item Purchase Tracking (예: USD $1.99를 결제하여 Gold 100개 아이템을 구입)
2. 가상 화폐를 통해 결제되는 Soft Currency Item Purchase Tracking (예: Gold 10개를 이용하여 포션 아이템을 구입)

위 2가지 유형의 데이터를 모두 Tracking 함으로써 앱의 매출뿐만 아니라 인-앱 사용자들의 아이템 구매 추이 분석까지 가능합니다.

아이템 정보 등록을 위한 별도의 작업은 필요하지 않으며, 클라이언트에서 결제된 아이템 정보가 자동으로 대쉬보드에 등록되는 방식입니다. (아이템 리스트 확인은 대쉬보드 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.)

아래의 적용 예제를 참고하여 간단히 In-App-Purchase Tracking 기능을 적용합니다.

#### Hard Currency Item Tracking

Hard Currency Item의 결제는 각 앱스토어별 인-앱 결제 라이브러리를 통해 이루어집니다. iOS의 경우 Storekit 결제 라이브러리에서 _'결제 성공'_ 이벤트가 발생 할 시에 AFPurchase 객체를 생성하고 logPurchase(purchase) 메소드를 호출합니다. 그리고 _'결제 실패'_ 이벤트가 발생 할 시에는 cancelPromotionPurchase() 메소드를 호출합니다.

적용 예제: 
```objective-c
- (void)completeTransaction:(SKPaymentTransaction *)transaction
{
  // productInfo is NSMutableDictionary object to store fetched SKProduct object with its productIdentifier as a key.
  SKProduct *product = [productInfo objectForKey:transaction.payment.productIdentifier];

  NSString *itemId = transaction.payment.productIdentifier;
  NSString *currencyCode = [product.priceLocale objectForKey:NSLocaleCurrencyCode];
  NSNumber *price = product.price;
  NSDate *transactionDate = transaction.transactionDate;
  NSData *transactionReceiptData = transaction.transactionReceipt;

  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeHardItem
                                                    itemId:itemId
                                              currencyCode:currencyCode
                                                     price:[price doubleValue]
                                              purchaseDate:transactionDate
                                    transactionReceiptData:transactionReceiptData];

  [[AdFrescaView shardAdView] logPurchase:purchase];
  ......
}

- (void)failedTransaction:(SKPaymentTransaction *)transaction 
{
  [[AdFrescaView shardAdView] cancelPromotionPurchase];
  ....
}
```

Hard Currency Item을 위한 AFPurchase 객체 생성의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
itemId(string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. Nudge 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
currencyCode(string) | ISO 4217 표준 코드를 설정합니다. SKProduct 객체의 값을 이용하거나, 자체 백엔드 서버에서 가격을 내려받아 설정할 수 있습니다. 
price(double) | 아이템의 가격을 설정합니다. SKProduct 객체의 값을 이용하거나, 자체 백엔드 서버에서 가격을 내려받아 설정할 수 있습니다. 
purchaseDate(date) | 결제된 시간을 NSDate 객체 형태로 설정합니다. 값이 설정되지 않은 경우 Nudge 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.
transactionReceiptData(nsdata| SKPaymentTransaction 객체의 transactionReceipt 값을 지정합니다. 추후 Receipt Verficiation 기능을 위해 필요한 데이터를 설정합니다. 

#### Soft Currency Item Tracking

Soft Currency Item의 결제는 앱 내의 가상 화폐로 아이템을 결제한 경우를 의미합니다. 앱 내에서 가상 화폐를 이용한 결제 이벤트가 성공한 경우 아래 예제와 같이 AFPurchase 객체를 생성하고 logPurchase(purchase) 메소드를 호출합니다. 그리고 _'결제 실패'_ 이벤트가 발생 할 시에는 cancelPromotionPurchase() 메소드를 호출합니다.


적용 예제: 
```objective-c
- (void)didPurchaseSoftItem {
  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeSoftItem
                                                    itemId:@"gun_001"
                                              currencyCode:@"gold"
                                                     price:100
                                              purchaseDate:nil
                                    transactionReceiptData:nil]; 

  [[AdFrescaView shardAdView] logPurchase:purchase];
}

- (void)didFailToPurchaseSoftItem {
  [[AdFrescaView shardAdView] cancelPromotionPurchase];
}
```

Soft Currency Item을 위한 AFPurchase 객체 생성의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
itemId(string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. Nudge 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
currencyCode(string) | 결제에 사용한 가상화폐 고유 코드를 설정합니다. (예: gold)
price(double) | 가상 화폐로 결제한 가격 정보를 설정합니다. (예: gold 10개의 경우 10 값을 설정)
purchaseDate(date) | 결제된 시간을 NSDate 객체 형태로 설정합니다. 값이 설정되지 않은 경우 Nudge 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.
transactionReceiptData(nsdata| Soft 아이템의 경우는 값을 지정하지 않습니다.

#### IAP Trouble Shooting

logPurchase() 메소드를 통해 기록된 AFPurchase 객체는 Nudge 서비스에 업데이트되어 실시간으로 대쉬보드에 반영됩니다. 현재까지 등록된 아이템 리스트는 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.

만약 아이템 리스트가 새로 갱신되지 않는 경우, AFPurchaseDelegate를 구현하여 혹시 에러가 발생하고 있지 않은지 확인해야 합니다. 

만약 AFPurchase 객체의 값이 제대로 설정되지 않은 경우, didFailToLogWithException 이벤트를 통하여 에러 메시지를 표시하고 있으니 아래와 같이 코드를 적용하여 로그를 확인합니다.

```objective-c
// AppDelegate.h

@interface AppDelegate : UIResponder <UIApplicationDelegate, AFPurchaseDelegate> {
  ...
}

// AppDelegate.m
- (void)didPurchaseSoftItem {
  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeSoftItem
                                               itemId:@"gun_001"
                                              currencyCode:@"gold"
                                                     price:100
                                              purchaseDate:nil
                                    transactionReceiptData:nil];
  [[AdFrescaView shardAdView] logPurchase:purchase, self];
}

- (void)purchase:(AFPurchase *)purchase didFailToLogWithException:(AdFrescaException *)exception {
  NSLog(@"AFPurchase didFailToLogWithException :: purchase = %@, exception = %@", [purchase JSONRepresentation], [exception description]);
}
```

* * *

### Give Reward

리워드 지급 기능을 이용하여 사용자에게 인앱 아이템을 보상으로 지급할 수 있습니다.

리워드 지급에는 2가지 과정이 필요합니다.

1. 리워드 지급 요청: 사용자에게 지급해야 할 리워드가 있는 경우 SDK에서 리워드 지급 요청 이벤트를 발생시킵니다. 해당 이벤트 발생 시에 게임 서버를 통해 사용자에게 리워드를 지급합니다.
2. 리워드 지급 완료: 사용자에게 리워드를 지급한 후 SDK에서 지급 완료 사실을 알립니다.

먼저 AFRewardClaimListener를 구현합니다. 사용자에게 지급해야 할 리워드가 있는 경우 onReward 이벤트가 발생됩니다. 인자로 넘어온 아이템 정보를 이용하여 사용자에게 아이템을 지급합니다.


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
  NSString *logMessage = [NSString stringWithFormat:@"You got the reward item! (%@)", item.name];
  NSLog(@"%@", logMessage);
  
  // Give an item to users.  
  [self sendItemToUser:currentUserId itemId:item.uniqueValue quantity:item.quantity securityToken:item.securityToken];
}
```

사용자에게 아이템을 지급한 후 finishRewardClaim() 메소드를 호출하여 넛지에 리워드 지급 완료를 통보해야 합니다. 넛지는 리워드 지급 완료 기록을 전달 받아야만 리워드가 정상적으로 지급된 것으로 처리합니다. 즉 게임서버나 클라이언트에서 에러가 발생하여 리워드 지급이 실패한 경우 넛지 SDK에서는 다시 리워드 지급 요청을 합니다. 넛지 SDK에서는 3분 이상 지급 확인 기록이 전달되지 않은 경우 다음 번 마케팅 모멘트가 실행될 때 다시 리워드 지급 요청을 합니다. 이는 지급 처리 중에 다시 지급요청을 해서 중복 지급되는 것을 막기 위함입니다.

```objective-c
- (void)onRewardClaimSuccess:(AFRewardItem *)item {
  [[AdFrescaView shared] finishRewardClaim:item.rewardClaimToken];
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

프로모션 기능을 적용하기 위해서 AFPromotionDelegate를 구현합니다. 프로모션 캠페인이 노출된 후 사용자가 이미지 메시지의 액션 영역을 탭하면 onPromotion() 이벤트가 발생합니다. 이벤트에 넘어오는 promotionPurchase 객체 정보를 이용하여 사용자에게 아이템 결제 UI를 표시하도록 코드를 적용합니다.

Hard Currency 아이템의 경우 인-앱 결제 라이브러리를 이용하여 결제 UI를 표시합니다. promotionPurchase 객체의 ItemId 값이 아이템의 SKU 값에 해당됩니다. 아래의 예제는 구글 플레이의 결제 라이브러리 코드를 이용하고 있습니다.

Soft Currency 아이템의 경우는 앱이 기존에 사용하고 있는 상점 내 아이템 결제 UI를 표시하도록 코드를 작성합니다. Soft Currency 프로모션의 경우는 2가지 가격 할인 옵션을 제공하고 있습니다. discountType 프로퍼티를 이용하여 할인 옵션을 확인할 수 있습니다.

1. **Discount Price**: 캠페인에 직접 지정된 가격으로 아이템을 판매합니다. price 프로퍼티 값을 이용하여 가격 정보를 얻습니다.
2. **Discount Rate**: 캠페인에 지정된 할인율을 적용하여 아이템을 판매합니다. discountRate 프로퍼티 값을 이용하여 할인율 정보를 받아옵니다.

```objective-c
// AppDelegate.h
@interface AppDelegate : UIResponder <UIApplicationDelegate, AFPromotionDelegate> {

}
....

// AppDelegate.m
- (void)applicationDidBecomeActive:(UIApplication *)application 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setPromotionDelegate:self];
}

- (void)onPromotion:(AFPurchase *)promotionPurchase {
  NSString *itemId = promotionPurchase.itemId;
  NSString *logMessage = @"onPromotion: no logMessage";
  
  if (promotionPurchase.type == AFPurchaseTypeHardItem) {
    // Use SKPaymentQueue to show the purchase ui of this item.
    SKProduct *product = [self paymentWithProductIdentifier:itemId];
    SKPayment *payment = [SKPayment paymentWithProduct:product];
    [[SKPaymentQueue defaultQueue] addPayment:payment];
    
    logMessage = [NSString stringWithFormat:@"on HARD_ITEM Promotion (%@)", itemId];
    
  } else if (promotionPurchase.type == AFPurchaseTypeSoftItem) {
    NSString *currencyCode = promotionPurchase.currencyCode;
    
    if (promotionPurchase.discountType == AFDiscountTypePrice) {
      // Use a discounted price
      double discountedPrice = promotionPurchase.price;
      [self showPurchaseUIWithItemId:itemId withDiscountedPrice:discountedPrice];
      logMessage = [NSString stringWithFormat:@"on SOFT_ITEM Promotion (%@) with %.2f %@", itemId, discountedPrice, currencyCode];

    } else if (promotionPurchase.discountType == AFDiscountTypeRate) {
      // Use this rate to calculate a discounted price of item. discountedPrice = originalPrice - (originalPrice * discountRate)
      double discountRate = promotionPurchase.discountRate;
      [self showPurchaseUIWithItemId:itemId withDiscountRate:discountRate];
      logMessage = [NSString stringWithFormat:@"on SOFT_ITEM Promotion (%@) with %.2f %%", itemId, discountRate * 100.0];
    }
    
    NSLog(@"%@", logMessage);
  }
}
```

## Dynamic Targeting

### Custom Parameter

커스텀 파라미터는 마케팅 목적으로 사용자를 분류하기 위해 사용하는 속성을 말하며 마케터가 임의로 정의할 수 있습니다. (예. 사용자의 레벨, 스테이지, 플레이 횟수 등) 커스텀 파라미터를 이용하면 사용자의 특정 속성에 따라 세그먼트를 정의하고 실시간으로 모니터링할 수 있습니다. 또한 캠페인 실행 시에는 보다 더 정교한 타겟팅을 통해 높은 성과를 거둘 수 있습니다. (Nudge SDK에서 자동적으로 수집하는 단말 ID, 기본 언어, 국가, 앱 버전 등의 정보는 커스텀 파라미터로 설정할 필요가 없습니다.)

Nudge SDK는 트랙킹하려고 하는 커스텀 파라미터의 유형에 따라 아래 2가지 방법을 제공합니다.

- 현재 상태 값을 트랙킹하는 경우
  - 특정 속성의 현재 상태 값을 트랙킹할 때 사용합니다. 
  - 예) 레벨, 최종 스테이지, 친구수 등
  - 사용 코드: **setCustomParameterWithValue** 메소드를 이용하여 현재 상태 값 (Integer, Boolean 지원)을 SDK에 전달합니다.

- 특정 이벤트의 횟수를 트랙킹하는 경우
  - 특정 이벤트마다 증가하는 횟수를 트랙킹할 때 사용합니다.
  - 예) 플레이 횟수, 가챠 이용 횟수 등
  - 사용 코드: **incrCustomParameterWithAmount** 메소드를 이용하여 이벤트 발생시 증가된 횟수 (Integer)을 SDK에 전달합니다.

먼저 커스텀 파라미터를 특정할 수 있는 문자열 형태의 Unique Key 값을 정합니다. (예: "level", "facebook_flag", "play_count") 앱을 처음 실행하는 시점 또는 사용자가 로그인하는 시점에 커스텀 파라미터의 현재 상태 값을 설정합니다.

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions 
{
  ...
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:User.level] forKey:@"level"];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithBool:User.hasFacebookAccount] forKey:"facebook_flag"];
}
```

커스텀 파라미터의 값이 변경되는 시점 (이벤트 발생 시)에 상태 값을 갱신하거나 횟수를 증가시킵니다.

```objective-c
- (void)levelDidChange:(int)level 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:level] forKey:"level"];
}   

- (void)didFinishGame
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca incrCustomParameterWithAmount:[NSNumber numberWithInt:1] forKey:"play_count"];
}
```

위와 같이 코딩을 마치고 앱을 실행하면 SDK는 저장한 커스텀 파라미터 값을 Nudge 서버로 전송합니다.  Nudge 서버는 활성화된 커스텀 파라미터의 값만을 저장하기 때문에 반드시 [Dashboard](https://dashboard.nudge.do)에 접속하여 해당 커스텀 파라미터를 활성화 (Activate)해야 합니다.

<img src="https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/custom_parameter_index.png">

 Overview 메뉴 -> Settings - Custom Parameters 버튼을 클릭하면 커스텀 파라미터 목록이 표시됩니다. 해당 커스텀 파라미터의 Unique Key를 찾은 다음, 이름 ('Name')을 입력하고 활성화 (Activate)합니다.

#### Stickiness Custom Parameter

Stickiness 커스텀 파라미터는 사용자의 stickiness를 측정하기 위해 사용하는 특수한 형태의 커스텀 파라미터입니다. 예를 들어 사용자의 플레이 횟수를 커스텀 파라미터로 지정하고 Stickiness 커스텀 파라미터로 설정하면 해당 사용자의 '오늘 총 플레이 횟수', '최근 7일간의 총 플레이 횟수', '최근 7일간의 일평균 플레이 횟수' 등을 세그먼트 필터로 사용할 수 있습니다. Stickiness 커스텀 파라미터를 이용하면 사용자들의 충성도 (또는 몰입도)에 따라 세그먼트를 나누고 모니터링할 수 있습니다.

Stickiness 커스텀 파라미터로 사용하고자 하는 커스텀 파라미터는 반드시 **incrCustomParameterWithAmount** 메소드를 이용해야 합니다.  Stickiness 커스텀 파라미터를 사용하고자 하는 경우 해당 커스텀 파라미터를 활성화한 후 support@nudge.do 로 메일 주시기 바랍니다.

* * *

### Marketing Moment

마케팅 모멘트는 유저에게 메세지를 전달하고자 하는 상황을 의미합니다. (예: 캐릭터 레벨 업, 퀘스트 달성, 스토어 페이지 진입)

마케팅 모멘트 기능을 사용하여 지정된 상황에 알맞는 캠페인이 노출되도록 할 수 있습니다.

마케팅 모멘트 설정은 [Dashboard](https://dashboard.nudge.do) 사이트를 접속하여 앱의 Overview 메뉴 -> Settings - Marketing Moments 버튼을 클릭하여 확인할 수 있습니다.

SDK 적용을 위해서는 Dashboard에서 지정된 각 마케팅 모멘트의 '인덱스' 값이 필요합니다. 인덱스 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 지정하여 이용하는 것을 권장합니다.

각 모멘트 발생 시, load() 메소드에 원하는 모멘트 인덱스 값을 인자로 넘겨주시면 간단히 적용이 완료됩니다.

(load() 메소드에 인덱스를 설정하지 않은 경우, 인덱스 값은 '1' 값이 자동으로 지정됩니다.)

```objective-c
- (void)userDidEnterItemStore {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca load:EVENT_INDEX_STORE_PAGE];    
  [fresca show];
} 

- (void)levelDidChange:(int)level {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca setCustomParameterWithValue:[NSNumber numberWithInt:level] forKey:"level"]; 
  [fresca load:EVENT_INDEX_LEVEL_UP]; 
  [fresca show];
}  
```

## Advanced

### AdFrescaViewDelegate
AdFrescaViewDelegate 를 직접 구현함으로써, 콘텐츠 뷰에서 발생하는 이벤트를 확인 할 수 있습니다. 

```objective-c
// ViewController.h
@interface MainViewController : UIViewController<AdFrescaViewDelegate> {
  .......
@end

// ViewController.m

- (void)viewDidLoad {
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  fresca.delegate = self;
  [fresca load];
  [fresca show];
}

#pragma mark – AdFrescaViewDelegate

// 콘텐츠를 요청하기 직전에 호출됩니다.
- (void)frescaWillReceiveAd:(AdFrescaView *)theAdView {}

// 콘텐츠를 정상적으로 불러온 후 발생하는 이벤트입니다.
- (void)frescaDidReceiveAd:(AdFrescaView *)theAdView {}

// 콘텐츠를 불러오지 못한 경우 발생됩니다. 에러 정보를 확인 할 수 있습니다.
- (void)fresca:(AdFrescaView *)view didFailToReceiveAdWithException:(AdException *)error {}

// 사용자가 뷰를 종료한 이후 발생하는 이벤트입니다. 콘텐츠 불러오지 못해 에러가 발생한 경우에도 해당 이벤트가 발생됩니다.
- (void)frescaClosed:(AdFrescaView *)fresca {}
```

위의 이벤트 메소드 내용을 직접 구현함으로써 다양한 응용이 가능해집니다. 

예를 들면

- 앱의 인트로 화면에서 콘텐츠를 표시한 후, 사용자가 콘텐츠 뷰를  닫으면 메인 페이지로 이동하고 싶은 경우
- 게임 도중 ‘Next Stage” 버튼을 눌러 콘텐츠를 표시한 후, 사용자가 콘텐츠를  닫으면 스테이지가 넘어가는 경우  
위 경우는 frescaClosed 함수 내용을 구현함으로써 적용이 가능합니다.

```objective-c
// Example: FirstViewController.m
#pragma mark – AdFrescaViewDelegate

- (void)frescaClosed:(AdFrescaView *)fresca {
  // 다음 페이지로 이동

  NextViewController *vc = [[NextViewController alloc] init];
  [self.navigationController pushViewController:vc animated:YES];  
  [vc release];
}
```

주의사항:

사용자가 마켓이나 다른 애플리케이션의 URI가 설정된 콘텐츠를 클릭한 경우, 화면이 다른 애플리케이션으로 이동할 수 있습니다. 
이 때 frescaClosed 에 다른 페이지로 이동하도록 구현하였다면, 사용자가 다른 화면에 있는 동안 앱의 페이지가 가 미리 이동해버리거나, 페이지 애니메이션이 비정상적으로 실행될 수 있습니다.
아래와 같은 방법으로 해당 문제를 해결할 수 있습니다.

1. Dashboard 에서 해당 Event 의 Close Mode 를 Override 로 변경 합니다.(콘텐츠 이미지를 클릭해도 뷰가 닫히지 않습니다..)
2. AppDelegate의 applicationWillEnterForeground() 이벤트를 아래와 같이 구현합니다.

```objective-c
#pragma mark – AdFrescaViewDelegate

- (void)applicationWillEnterForeground:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView shardAdView];
  if (!fresca.hidden && fresca.userClicked) {
    [fresca closeAd];
  }
}
```

### Timeout Interval

load() 메소드의 최대 로딩 시간을 직접 지정하실 수 있습니다. 지정된 시간 내에 데이터가 로딩되지 못한 경우, 사용자에게 콘텐츠를 노출하지 않습니다.

최소 1초 이상 지정이 가능하며, 지정하지 않을 시 기본 값으로 5초가 지정 됩니다.

```objective-c
AdFrescaView *fresca = [AdFrescaView sharedAdView];  
fresca.timeoutInterval = 3 // # secs  
[fresca load];
[fresca show];
```

* * *

## Reference

### Deep Link

캠페인의 Deep Link 설정 시에 Custom URL Schema를 지정할 수 있습니다.

이를 통해 사용자가 콘텐츠를 클릭할 경우, 자신이 원하는 특정 앱 페이지로 이동하는 등의 액션을 지정할 수 있습니다.

1. Info.plist 파일을 열어 사용할 URL Schema 정보를 설정 합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png" />

2. AppDelegate.m 파일을 열어 handleOpenURL 메소드를 구현합니다. 호출되는 URL 값에 따라 다른 페이지를 호출하도록 설정할 수 있습니다. 
  ```objective-c
  - (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {  
    if ([url.scheme isEqualToString:@"myapp"]) {
      if ([url.host isEqualToString:@"item"]) {
        ItemViewController *vc = [[ItemViewController alloc] init];
        [navigationController pushViewController:vc animated:YES];
        [vc release];
        return YES;
      }
    }
    return NO;
  }
```
  위와  같이 구현한 경우, 캠페인의 Deep Link를 'myapp://item' 으로 설정하여 전송하면, ItemViewController 페이지가 실행됩니다.

* * *

### Cross Promotion Configuration

Incentivized CPI & CPA 캠페인 기능을 사용하여, 사용자가 Media App에서 Advertising App의 광고를 보고 앱을 설치하였을 때 보상으로 Media App의 아이템을 지급할 수 있습니다.

- Medial App: 다른 앱의 광고를 노출하고, 광고 대상의 앱을 설치한 사용자들에게 보상을 지급하는 앱
- Advertising: Media App에 광고가 노출되는 앱.

Incentivized CPI & CPA 캠페인에 대한 보다 자세한 설명 및 [Dashboard](https://dashboard.nudge.do) 사이트에서의 설정 방법은 [크로스 프로모션 이해하기](https://adfresca.zendesk.com/entries/22033960) 가이드를 참고하여 주시기 바랍니다.

SDK 적용을 위해서는 Advertising App에서의 URL Schema 설정 및 Media App에서의 Reward Item 지급 기능을 구현해야 합니다.

#### Advertising App 설정:

  iOS 플랫폼의 경우 URL Schema 값을 이용하여 광고를 노출한 앱이 실제로 디바이스에 설치되었는지 검사하게 됩니다. 따라서 Advertising App 앱의 URL Schema을 설정하고 CPI Identifier로 사용합니다.

  (현재 Incentivized CPI 캠페인을 진행할 경우, Advertising App의 SDK 설치는 필수가 아니며 URL Schema 설정만 진행되면 됩니다. 하지만 Incentivized CPA 캠페인을 진행할 경우 반드시 SDK 설치 및 [Marketing Moment](#marketing-moment) 기능이 적용되어야 합니다.)

  먼저 Xcode 프로젝트의 Info.plist 파일을 열어 사용할 URL Schema 정보를 확인합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  위 경우 [Dashboard](https://dashboard.nudge.do) 사이트에서 Advertising App의 CPI Identifier 값을 'myapp://' 으로 설정하게 됩니다.
  
  iOS 플랫폼의 경우 URL Schema 값이 다른 앱과 중복될 수 있습니다. 정상적인 캠페인 진행을 위해서는 고유한 값을 사용해야 합니다.

  마지막으로, Incentivized CPA 캠페인을 진행할 경우는 보상 조건으로 지정한 마케팅 모멘트가 발생되어야 합니다. 사용자가 보상 조건을 완료한 이후 아래와 같이 지정한 마케팅 모벤트 코드를 실행합니다.
  
  ```objective-c
  // 튜토리얼 완료 모멘트를 보상 조건으로 지정한 경우
  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  [fresca load:MOMENT_INDEX_TUTORIAL];     
  [fresca show];
  ```

#### Media App SDK 적용하기:

  Media App에서 보상 지급 여부를 확인하고, 사용자에게 아이템을 지급하기 위해서는 SDK 가이드의 [Give Reward](#give-reward) 항목이 적용되어 있어야 합니다. 앞서 리워드 지급 기능을 적용하지 않았다면, 가이드에 따라 기능을 적용합니다.

* * *

### IFV Only Option

[SDK 설치 과정](#installation)에서 AdSupport.framework 를 추가한 경우 SDK는 IFA 값을 이용하여 디바이스를 구분하며, AdSupport.framework를 제외한 경우 IFV 값을 이용하여 디바이스를 구분하게 됩니다.

만약 앱스토어 출시 이후 앱을 업데이트 하는 과정에서 AdSupport.framework를 제외시키거나, 추가하는 경우 아래와 같은 상황이 발생합니다.

1. 기존에 사용하던 AdSupport.framework를 제외시키는 경우:
  - Nudge API 서버는 기존에 함께 수집한 IFV 값을 이용하여, 기존의 앱 사용자들이 새로운 사용자로 인식 되지 않도록 자동으로 처리합니다. 따라서 아무런 문제가 발생하지 않습니다. 단, iOS SDK 1.3.3 (2013년 11월 26일 출시) 이상의 버전이 탑재되었던 앱에 한해서만 처리됩니다.
2. AdSupport.framework를 새로 추가하는 경우:
  - 이 경우는 SDK가 기존에 IFA 값을 수집하지 못하였기 때문에, 앱을 그대로 릴리즈하면 기존 사용자들이 모두 새로운 사용자로 인식되는 문제가 발생합니다. iOS SDK에서는 이 문제를 해결하기 위하여 **setUseIFVOnly** 메소드를 제공합니다. didFinishLaunchingWithOptions 이벤트에서 **setUseIFVOnly** 메소드에 'YES' 값을 설정하면 AdSupport.framework가 추가되었더라도 기존의 IFV 값을 가지고 디바이스를 구분하도록 합니다. 이로 인해 기존의 IFV 값을 이용하여 등록된 사용자들이 새로운 사용자로 인식되는 것을 방지할 수 있습니다.

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [AdFrescaView startSession:API_KEY];
  [[AdFrescaView shardAdView] setUseIFVOnly:YES];
}
```

위 내용이 제대로 확인되지 않을 시에는 사용자 통계나 타겟팅 기능에 큰 문제를 일으킬 수 있습니다. 개발팀에서는 해당 내용을 자세히 확인하고 대응해야 하며, 보다 자세한 문의는 support@adfresca.com 메일을 통해 연락을 부탁드립니다.

* * *

## Troubleshooting

SDK 설치시에 SBJson의 Duplicate Symbol 에러가 발생하여 빌드가 되지 않을 수 있습니다.

<img src="https://adfresca.zendesk.com/attachments/token/ikafbcqjnj9jbak/?name=6666.png">

위와 같은 에러가 발생하며 빌드가 실패하게 됩니다.

현재 개발 중인 프로젝트내에 이미 SBJson을 사용중인 경우에 발생할 수 있으며, AdFresca SDK에 포함된 SBJson을 제거함으로써 해결이 가능합니다. 현재 SDK에 포함된 SBJson은 [3.1 release](https://github.com/stig/json-framework/tree/v3.1) 버전이며, 프로젝트에서 이보다 하위 버전을 사용할 시에 문제가 발생할 수 있습니다.

그 외에 콘텐츠가 제대로 출력되지 않거나, 에러가 발생한다면 AdFrescaViewDelegate의 didFailToReceiveAdWithException 이벤트 함수를 구현하여, 에러 정보를 확인 할 수 있습니다. 

```objective-c
- (void)fresca:(AdFrescaView *)fresca didFailToReceiveAdWithException:(AdException *)error {  
  NSLog(@"AdException message : %@", [error message]);
}
```

* * *

## Release Notes
- **v1.6.2 _(2016/01/23 Updated)_**
  - [리워드 지급 기능](#give-reward)이 개선되어 지급 완료 확인이 가능해졌습니다. 기존의 메소드가 삭제 되었기 때문에 반드시 새로운 가이드를 참고하여 코드를 변경해야 합니다.
- v1.5.6
  - Push Reward Campaign 기능을 지원합니다. [Push Messaging](#push-messaging) 항목을 참고하여 didReceiveRemoteNotification 이벤트의 분기문을 예제 코드와 같이 수정해야 합니다.
- v1.5.5
  - [In-App Purchase Tracking](#in-app-purchase-tracking) 기능에서 '%' 문자가 포함된 아이템 이름을 입력받을 수 있도록 개선되었습니다.
- v1.5.4
  - [Test Mode](#test-mode) 기능이 추가되었습니다.
- v1.5.3
  - [Custom Parameter](#custom-parameter) 설정 시 정수 형태의 고유 인덱스 값이 아닌 문자열 형태의 고유 키 값을 사용할 수 있도록 변경되었습니다. (인덱스를 이용하는 기존 방식도 그대로 지원합니다.)
- v1.5.2
  - [Stickiness Custom Parameter](#stickiness-custom-parameter)를 이용한 인앱 메시징 매칭 시 값 변경이 바로 적용되지 않던 문제를 해결하였습니다.
- v1.5.1
  - hasCustomParameterWithIndex 메소드가 추가되었습니다.
- v1.5.0
  - [Stickiness Custom Parameter](#stickiness-custom-parameter)을 지원합니다.
- v1.4.9
  - AFPurchase 객체에 AFPurchaseTypeHardItem, AFPurchaseTypeSoftItem purchase type이 추가되고 AFPurchaseTypeActualItem, AFPurchaseTypeVirtualItem 값이 deprecated 되었습니다. 자세한 내용은 [In-App Purchase Tracking](#in-app-purchase-tracking) 항목을 참고하여 주세요.
- v1.4.8
  - 유니티 플러그인에서의 In-App Purchase Tracking 기능을 지원합니다.
- v1.4.7
  - 아이폰6 모델에서의 가로형 이미지 표시 문제를 해결하였습니다.
- v1.4.6
  - A/B 테스트 기능을 지원합니다. 해당 기능은 별도의 코딩 작업 없이 이용 가능합니다.
- v1.4.5
  - Nudge SDK는 iOS 8 버전에서 정상적으로 동작합니다. 최신 버전이 아닌 기존 버전들도 아무런 호환 이슈 없이 동작됩니다.
  - iOS 8에서 푸시 연동을 위한 가이드 내용이 수정되었습니다. [Push Messaging](#push-messaging) 항목을 참고하여 푸시 서비스 사용 요청 코드를 수정합니다.
  - Deep Link 관련한 마이너 버그가 수정되었습니다.
- v1.4.4  
  - 프로모션 기능 관련하여 마이너 버그가 수정되었습니다.
- v1.4.3
  - 세일즈 프로모션 캠페인 기능을 지원합니다. 자세한 내용은 [Sales Promotion](#sales-promotion) 항목을 참고하여 주세요.
  - 리워드 지급 시에 시큐리티 토큰값을 이용하여 보안 이슈를 해결할 수 있습니다. 자세한 내용은 [Give Reward](#give-reward)   항목을 참고하여 주세요.
  - [In-App Purchase Tracking](#in-app-purchase-tracking) 기능에서 cancelPromotionPurchase() 메소드가 추가되었습니다. 
  - 이미지 메시지의 Tap Area 기능을 지원합니다.
- v1.4.2
	- 1개의 마케팅 모멘트에서 복 수 개의 캠페인이 매칭되어 표시가 가능하도록 변경되었습니다.
- v1.4.1
  - Xcode의 64-bit 아키텍쳐 설정을 지원합니다.
  - 1.4.0에서 지원하는 [In-App Purchase Tracking](#in-app-purchase-tracking) 기능을 통합하여 제공합니다.
  - 몇몇 메소드의 이름이 변경되었습니다. (load -> load, show -> show, closeAd -> close) 기존에 제공하던 메소드도 호환성을 위하여 정상적으로 지원합니다.
- 1.4.01
  - 앱 내에서 발생하는 In-App Purchase 데이터를 트랙킹할 수 있는 기능이 추가되었습니다. 자세한 내용은 [In-App Purchase Tracking](#in-app-purchase-tracking) 항목을 참고하여 주세요. [In-App Purchase Tracking](#in-app-purchase-tracking) 항목을 참고하여 주세요.
- v1.3.5
  - SDK 설치 과정에서 AdSupport framework 추가가 필수항목에서 제외됩니다. IFA 수집을 하지 않아도 SDK 이용이 가능하도록 수정되었습니다. 보다 자세한 내용은 [Installation](#installation) 항목을 참고하여 주세요.
  - 인-앱 메시징 캠페인 캠페인을 통한 Reward Item 지급 기능을 지원합니다. 
  - Incentivized CPA 캠페인 기능을 지원합니다. 자세한 내용은 [CPI Identifier](#cpi-identifier) 항목을 참고하여 주세요.
  - AFRewardItemDelegate가 구현 기능이 추가되어, 지급 가능한 아이템이 발생할 시에 자동으로 itemRewarded 이벤트가 발생합니다. 보다 자세한 내용은 [Reward Item](#reward-item) 항목을 참고하여 주세요.
- v1.3.4
  - testDeviceId property 값이 각 iOS  버전에 맞는 값으로 출력되도록 변경되었습니다. 
- v1.3.3 
  - APNS 디바이스 토큰이 새로 생성되거나 변경 시, SDK가 토큰 값을 실시간으로 Nudge  서비스에 업데이트하도록 개선되었습니다. (기존에는 앱 실행 시에만 업데이트하였습니다.)
- v1.3.2 
  - 커스텀 파라미터 설정 시 'long long' 타입까지 확장하여 지원합니다.
customParameterWithIndex 호출 시 설정된 값이 없는 경우 nil 값을 리턴하도록 변경되었습니다.
- v1.3.1 
  - 'Close Mode' 기능을 지원합니다. Dashboard에서 Interstitial View의 닫힘 설정을 제어할 수 있습니다.
  - In-App-Purchase Count, Custom Parameter 정보를 로컬에 캐싱하여 사용합니다. 
  - iOS 4.3 버전에서 가로모드의 뷰가 비정상적으로 표시되는 문제를 해결하였습니다.
  - 커스텀 파라미터 설정 시 long 타입을 지원합니다.
- v1.3.0 
  - Reward 아이템 지급 기능을 지원합니다. '10. Reward Item 지급하기' 항목을 참고하여 주세요.
- v1.2.1
  - numberOfInAppPurchases 설정 값이 추가 되었습니다. In-App Purchase를 구매한 횟수를 관리 할 수 있습니다. (적용 방법은 5. In-App Purchased Count 관리 항목 참고하여 주세요.)
  - isInAppPurchasedUser property가 Deprecated 되었습니다. 새로 추가된 numberOfInAppPurchases property를 사용하여 주세요.
- v1.2.0
  - Event 기능을 지원합니다. load() 메소드에 Event Index 값을 설정할 수 있습니다. 자세한 내용은 '7. Event 지정하기'를 참고해주세요.
  - AD Slot 기능이 Deprecated 되었습니다. 기존의 Default Slot은 '1'번 이벤트 인덱스,  AD Only Slot은 '2'번 이벤트 인덱스로 적용됩니다.
  - 콘텐츠 데이터를 요청하는 중에 새로 load()가 호출된 경우, 가장 최근에 요청된 콘텐츠 화면에 표시됩니다. (기존에는 콘텐츠 데이터를 요청 중에 새 요청을 할 수 없었습니다.)
  - 앱스토어로 연결되는 콘텐츠 경우, 앱스토어 페이지를 앱 안에서 표시합니다. 더이상 앱 밖으로 나가지 않습니다. (해당 기능을 위해서 반드시 StoreKit. framework를 추가하여 주세요.)
  - 콘텐츠 이미지 클릭 시 콘텐츠 뷰가 닫히도록 변경되었습니다.
  - 콘텐츠를 일정 시간 후 자동으로 닫을수 있는 Auto Close Timer 기능이 추가 되었습니다. Dashboard 에서 설정할 수 있습니다.
- v1.1.0 
  - Push Notification 기능이 적용되었습니다. 자세한 내용은 '8. Push Notification 설정하기'를 참고해주세요.
- v1.0.1 
  - Custom Parameter를 지원합니다.  (자세한 내용은 'Custom Parameter 관리하기'를 참고해주세요)
- v1.0.0
  - HTML5 형태의 View를 지원합니다. (SDK 적용 코드는 전혀 변경하지 않아도 됩니다.)
  - iOS6에서 추가된 'Advertising Identifier'를 추가로 수집 및 사용합니다. 이와 관련하여 AdSupport framework를 Xcode 프로젝트에 추가해야 합니다. (자세한 내용은 'SDK 설치'를 참고해주세요)
- v0.9.9
  - iOS 6  정식 버전 및 iPhone 5 모델을 지원합니다.
- v0.9.8
  - 테스트 모드 기능 지원을 위한 테스트 기기 ID 확인 기능을 지원 합니다. (자세한 내용은 '테스트 기기 ID 확인하기'를 참고해 주세요)
- v0.9.7
  - 공지사항 기능이 추가 되면서 AD Slot 관리 기능이 추가 되었습니다. (자세한 내용은 'AD Slot 관리하기' 를 참고해 주세요)
  - load() 메서드에서 에러가 발생 시, frescaClosed 이벤트가 강제로 발생하던 문제를 해결 하였습니다. frescaClosed 이벤트는 항상 show 메서드가 호출된 이후에 발생 됩니다.
  - 캐시 기능 및 퍼포먼스가 향상 되었습니다.
  - 몇몇 메서드 이름의 오타를 수정하였습니다. (sharedAdView, frescaClosed) SDK의 호환성 유지를 위하여 잘못된 이름의 메서드는 삭제되지 않았으며 추후 Depreciated 설정 될 예정 입니다.
- v0.9.6 
  - 콘텐츠 캐싱 기능이 향상 되었습니다.
- v0.9.5 
  - SDK가 콘텐츠 데이터를 캐싱하여 보여 줍니다. 콘텐츠를 1회 이상 노출 시 캐시가 자동으로 적용되어 빠른 노출이 가능하여 졌습니다.
  - timeoutInterval 설정 값이 추가 되었습니다. 지정된 시간 내에 데이터를 로딩하지 못한 경우, 사용자에게 콘텐츠를 노출하지 않습니다. 최소 1초 이상 지정이 가능하며 기본 값은 기존의 5초로 설정 됩니다.
  - testModeEnabled 설정 값이 deprecated 되었습니다. 이후 모든 테스트 모드의 제어는 웹 Admin 페이지에서 가능합니다.
  - AdFrescaViewDelegate의 required 메소드 목록이 변경 되었습니다.
  - iOS 6 버전을 지원합니다.
- v0.9.4
  - startSession: 메소드가 추가 되었습니다. 보다 정확한 세션로깅을 위해 startSession 메소드를 didFinishLaunchingWithOptions 델리게이트 메소드에 구현해 주세요. (적용 방법은 4. Session Logging 항목 참고)
  - isInAppPurchasedUser 설정 값이 추가 되었습니다. In-App Purchase를 구매한 사용자들을 분류하여 관리 할 수 있습니다. (적용 방법은 5. In-App Purchased User 관리 항목 참고)
- v0.9.3
  - 세션 로깅 기능을 지원 합니다.  SDK가 자동으로 앱의 실행 이벤트를 감지하여 세션 정보를 기록 합니다. 
- v0.9.2
  - Performance가 개선 되었습니다.
  - 콘텐츠 클릭 시 앱스토어 이동 관련하여 일부 발생하던 버그를 수정 하였습니다.
- v0.9.1
  - UI가 개선 되었습니다.
- v0.9.0
  - Nudge iOS SDK가 출시 되었습니다. 기본적인 콘텐츠 출력 기능이 포함 됩니다.
