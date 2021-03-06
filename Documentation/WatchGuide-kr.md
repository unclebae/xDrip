xDrip+ Wear 설정 및 문제해결 가이드
======================================
**내용**
- [Enable xDrip+ Android Wear Integration](#enable-xdrip-android-wear-integration)
    - [Patching Watch Firmware](#patching-watch-firmware)
    - [Collector](#collector)
        - [xDrip+ System Status](#xdrip-system-status)
    - [Initial Wear Enablement Requests Location Permission](#initial-wear-enablement-requests-location-permission)
    - [Syncing Phone and Wear Preferences](#syncing-phone-and-wear-preferences)
    - [Syncing BGs and Wear Database](#syncing-bgs-and-wear-database)
    - [XDrip Prefs Overview](#xdrip-prefs-overview)
        - [XDrip BT Settings](#xdrip-bt-settings)
        - [XDrip Watchface Settings](#xdrip-watchface-settings)
        - [Watchface Tap Feature](#watchface-tap-feature)
        - [Battery Usage](#battery-usage)
    - [3-Dot Menu](#3-dot-menu)
    - [XDrip Treatments](#xdrip-treatments)
- [Troubleshooting xDrip+ Wear](#troubleshooting-xdrip-wear)
    - [Confirm Collector runs on the Phone with Wear Integration](#confirm-collector-runs-on-the-phone-with-wear-integration)
    - [Confirm Collector runs on the Watch with Wear Integration](#confirm-collector-runs-on-the-watch-with-wear-integration)
    - [Confirm the following in Android Wear app on phone](#confirm-the-following-in-android-wear-app-on-phone)
    - [Confirm Wear Integration preferences are consistent on both phone and watch](#confirm-wear-integration-preferences-are-consistent-on-both-phone-and-watch)
    - [Confirm Collection Method is consistent on both phone and watch](#confirm-collection-method-is-consistent-on-both-phone-and-watch)
    - [Confirm Collector device exists under Watch Settings->Bluetooth -> Devices](#confirm-collector-device-exists-under-watch-settings-bt-devices)
    - [Confirm Calibration Preferences are consistent on both phone and watch](#confirm-calibration-preferences-are-consistent-on-both-phone-and-watch)
    - [Confirm Noise Preferences are consistent on both phone and watch](#confirm-noise-preferences-are-consistent-on-both-phone-and-watch)
    - [Confirm Location Permission is enabled in Watch Settings](#confirm-location-permission-is-enabled-in-watch-settings)
    - [Debugging Android Wear](#debugging-android-wear)

# xDrip+ 안드로이드 Wear 통합 가능
xDrip+ 는 wear app 을 통해 wear 통합을 수행한다. xDrip+ wear app 은 표준 xDrip+ apk로 설치된다.
**Android Wear 1.x** 에서 wear 컴포넌트는 watch 와 자동적으로 동기화 된다.
**Android Wear 2.0** 에서는 xDrip+을 폰에 설치하고 난뒤에 Play-Store 에서 _watch_ 부분 (이는 "App on your phone" 하위에 있다) 에서 따로 컴포넌트를 설치해야한다.
최종 릴리즈는 스탠드얼론 모드를 지원하며 wear 앱이 직접적으로 Bluetooth Collector 와 통신한다. Bluetooth Collector 은 Dexcom G5, Dexcom G4 + xBridge, Dexcom Share 혹은 Libere LimitTTer 이다.

wear 스탠드 얼론 기능은 **Smart Watch Features** 기능 하위에 위치하는 xDrip+ 세팅을 통해서 가능하다. 그리고 watch **XDrip Prefs** 세팅으로 가능하다.

<img align="middle" src="./images/prefs-wear-sync.png" title="xDrip+ Wear Integration Preferences">

|Phone Settings                        | Watch Settings   | Description             |
| ------------------------------------ | ---------------- | ------------------------|
| Android Wear Integration             | NA               | Wear 통합이 가능하다.|
| Enable Wear Collection Service       | Enable Collector | BT Collector 이 스마트폰에서 멀어졌을때 wear device 에서 수행할 수 있도록 한다. |
| Force Wear Collection Service        | Force Collector  | 강제 BT Collector 은 스마트폰이 범위 내에 있을때도 wear device에서 수행되도록 한다.|
| Disable Force on Low Batterty        | NA               | 강제 BT Collector 을 Wear Low Battery Alert 인경우 끄도록 한다. Low Battery 는 Alarms, Alerts -> Extra Alerts (xDrip+) -> **Collector Battery Alerts** 를 통해서 설정된다. [XDrip BT Settings](#xdrip-bt-settings) 하위 섹션을 참고하자|
| Disable Force on Missed Readings     | NA               | 강제 BT Collector 서비스를 Wear Missed Reading 에서 끄도록 한다. ON 을 하면 **Force Wear Collection Service** 가 꺼지며 **Minutes since last Reading from Wear** 이후에 동작하며, 스마트폰 BT Collector 로 동작하게 된다.|
| Minutes since last Reading from Wear | NA               | 마지막 BG 읽기로부터 수분후에 **Disable Force Wear Collection Service** 이 트리거된다.|
| Device running Collection Service    | NA               | 읽기 전용; BT Collector가 수행중인 웨어 디바이스를 보여준다. 이것은 watch 디스플레이 이름 + uuid 가 될 것이며 Force Wear 가 가능하게 설정이 된다.|
| NA                                   | BT Collector     | 읽기 전용; xDrip+ Setting -> **Hardware Data Source** 로 수행된다. 예를 들어 만약 **Hardware Data Source** 가 **G5 Transmitter (test)** 로 설정이 되면 wear 앱 BT Collector 은 **DexcomG5**로 표시된다.|
| Sync Wear Logs                       | NA               | Wear 로그 엔트리를 폰과 동기화 되도록 트리거한다. xDrip+ 앱 위에서 로그를 볼 수 있으며 올바른 메뉴 아이템은 **View Events Log** 이다. xDrip+ Settings -> LCS -> Extra Logging Settings -> **Extra tags for logging** 메뉴에서 ExtraLogTags 를 활성화 한다 이는 서비스에서 로그를 생성하도록 한다. s.a. **G5CollectionService:v**.|
| Wear Log Prefix                      | NA               | wear 이벤트 로그들은 **Wear Log Prefix** 프리픽수가 붙는다. 만약 이를 탭하면 기본으로 **wear** 가 보일 것이다. 그러나 무언가를 기입할 수 있다. 이 프리픽스는 로그 엔트리가 두 장비들에서 유사하게 이름된 서비스들의 로그가 생성되었을때 구분지어주는 역할을 한다. 예를 들어 **wearG5CollectionService** 는 wear 장비에서 생성된 로그를 나타낸다.|

이 설정들은 의존성의 순서대로 리스트 된다. 중요, Watch의 **XDrip BT Settings** 는 오직 폰에서 Wear Integration 이 활성화 된경우에만 보입니다.

* **Wear 활성화하기**

  만약 폰이 범위 밖을 벗어났을때 오직 watch 만을 스탠드 얼론 모드로 이용하고자 한다면, 그리고 단지 Enable Wear checkbox 를 선택한다.
  이것은 단순히 폰이 범위 밖을 벗어났을때 BT collection 서비스를 가능하게 워치에서 수행한다.
  만약 폰이 다시 연결이 되는경우 워치는 BT 컬렉션 서비스를 정지할 것이다. 그리고 읽고 있는 BT 정볼르 전송한다.
  폰은 수신된 읽기 정보를 동기화 할 것이다. 그리고 자신의 BT 컬렉션 서비스를 시작한다. 이때 딜레이가 있을 것이다.
  약 14분 정도 소요될 수 있다. 몇몇 스마트폰에서 폰으로 스위치 되는 경우 초기 bg 읽기 정보를 획득한다. (스마트폰은 좋지 않은 연결 설정을 가진다.)

* **Wear 강제로 이용하기**

  **Force Wear** 를 활성화 하는 것은 xDrip+ BT 컬렉션 서비스를 수행하기 위해서 워치를 이용하게 한다.
  워치 BT 컬렉션 서비스는 매번 bg 읽기 값을 폰과 동기화 한다 컬렉터로 부터 수신된 읽은 정보 혹은 폰과 재연결된 정보를 읽는다.
  **Force Wear** 는 일상적으로 사용을 위해서 스마트폰에서 요청된 처리를 시계에 맡기는 이점이 있다. 그러므로 스마트폰의 배터리를 절약하고 CPU 사용량을 절약 할 수 있다.
  그러나, 이 오프로딩의 의미는 시계 배터리가 Force Wear 사용하지 않을때까지 지속되지 않을 수 있다는 것이다.
  예제와 같이 몇몇 사용자는 소니 스마트워치 3 (SW3) 가 플러그가 뽑히지 않고 G5 컬렉터가 수행되고있고, 항상 스크린에 떠 있을때 20% 오버나이트 (7-8시간) 이용할 수 있다.

  Force Wear 는 또한 더 낳은 BT 연결성을 제공한다. 이는 스마트폰에서 제공된 것이다. 예제에서와 같이 몇몇 사용자는 발견한다. SW3가 삼성 갤럭시 노트 4 스마트폰보다 더 낳은 연결성을 갖는다.

### FirmwareWatch 패치하기

  오직 Sony SW3 만이 G5 트랜스미트와 같이 컬렉터로 동작한다.
  대부분 다른 스마트 워치들은 버그를 가지고 있으며 이것은 그들의 불루투스가 올바르게 동작하는 것을 방해한다.
  이 버그는 워치 모델들이 나오고 나면 패치 되고 수정될 수 있다.
  자세한 내용은 https://github.com/NightscoutFoundation/xDrip/wiki/Patching-Android-Wear-devices-for-use-with-the-G5 을 살펴보자.

### 컬렉터
    **BT 컬렉터** 는 워치의 XDrip Pref 에서 읽기 전용으로 설정되어 있다. 이것은 xDrip+ Setting -> **Hardware Data Source** 에따라 동작한다.

    다음 이미지는 **Hardware Data Source** 의 설정을 보여준다. 이는 **LimiTTer** 그리고 워치에 설정된 설정에 따라 동작한다.

<img align="middle" src="./images/prefs-hardware-data-source-phone.png" title="xDrip+ Hardware Data Source">

    프리퍼런스는 BT Collector 를 가리킨다. 워치는 BG 트랜스미터와 커뮤니케이션 하며 동작할 것이며 워치는 스탠드 얼론 모드로 동작한다.

#### xDrip+ System Status
xDrip+ 시스템 상태 스크린은 BT 컬렉터를 위한 각각의 상태 페이지를 제공한다.
이것은 사용자가 쉽게 디바이스에서 BT 컬렉터가 수행되고 있는것을 쉽게 식별하게 해준다.

다음 이미지는 시스템 상태 페이지를 보여주며 G5 컬렉터도 포함하고 있다.

<img align="middle" src="./images/prefs-wear-g5systemstatus.png" title="xDrip+ System Status">


### Wear가 위치 퍼미션 요청을 가능하게 하기 위한 초기화
스탠드얼론 웨어의의 초기 enablement Enable Wear 설정을 워치나 폰에서 선택하면 트리거 된다.

이것은 **Location Permission is REquired** 다이얼로그를 디스플리에 되도록 트리거 한다.
안드로이드 웨어는 **Location Access** 를 사용자에 의해서 수동적으로 엑셉트 하도록 요구한다. 그러므로 사용자는 **반드시** 로케이션 퍼미션 쿼리를 스탠드 얼론 모드에서 수행될 수 있도록 하기 위해서
엑셉트 해야한다.
그렇지 않으면 사용자는 로케이션 허용을 Watch -> Settings -> Permissions -XDrip Prefs 에서 로케이션을 활성화 하도록 할 수 있따.

<img align="middle" src="./images/prefs-wear-permissions.png" title="xDrip+ Wear Integration Preferences">

### 폰과 웨어 동기화 설정
노트, XDrip+ 와 웨어는 그들이 둘다 존재하는 설정을 동기화 한다. 설정 동기화는 커넥션 상에서 다음과 같이 할 수 있다.

  1. xDrip+ 앱이 시작된다. xDrip+ 는 설정을 워치로 보내고 워치는 해당 값을 폰으로 업데이트 할 것이다.
  2. 재 연결에서 웨어 앱은 자신의 설정을 폰에 전송하고 폰은 값을 워치에 업데이트 한다.

예를 들어 만약 당신이 Force Wear 설정을 워치에서 변경했다면 이것은 즉시 폰으로 다시 연결하도록 전송한다. 그리고 폰은 세팅을 업데이트 할 것이다.

### Syncing BGs and Wear Database
* Sync DB - The watch data (BGs, Logs)are saved in the watch database.  The watch will attempt to sync its data with the phone upon connection until all delta data have been synced. So, for example, if you have 8 hours of overnight data generated while disconnected from the phone, the watch will attempt to send all data upon re-connection with the phone.
* Resetting the Wear DB - The watch data exists on the phone until you:

  1. **Reset Wear DB** on the phone via the xDrip+ upper right menu.  This task deletes the wear database, then initializes it with the most recent calibrations & BGs from the phone.
  This task may be useful if the wear app is not responding appropriately.  Refer to [Confirm Collector runs on the Watch with Wear Integration](#confirm-collector-runs-on-the-watch-with-wear-integration) below.
  2. **Sync Wear DB** is auto-executed on a daily basis at 4 am.  This task will removes all data from the database which have already been synced with the phone.
  3. The app is uninstalled.

* UserError Table - Similar to the xDrip+ phone app, UserError log messages are saved in the watch UserError table.  To access the watch log entries on the phone, enable the **Sync Wear Logs** preference shown in the above image.  The log entries will be prefixed with the **Wear Log Prefix**, which defaults to **wear**, but is user-configurable.  This allows users to identify which device generated the log entry.  The log entries can be viewed using the follwoing options:
  - Users can view log messages on the phone via the xDrip+ upper right menu item, **View Events Log**.
  - As with the xDrip+ phone app, specific log entries can be enabled by entering the extra log tag and severity level preference via the xDrip+ phone app settings, Less Common Settings (LCS) - Extra Logging Settings - **Extra tags for logging**.

The following image shows an example of the phone **View Events Log** containing phone and watch log entries.

<img align="middle" src="./images/prefs-wear-vieweventslog.png" title="xDrip+ Wear Integration Preferences">

### XDrip Prefs Overview
The watch XDrip Prefs app is used to set the xDrip+ wear app preferences.  In addition to the Wear Integration preferences mentioned above under [Enable xDrip+ Android Wear Integration](#enable-xdrip-android-wear-integration), XDrip Prefs provide the following new preferences used in the standalone version.

#### XDrip BT Settings

  - XDrip G5 Settings

    Wear provides G5 BT settings similar to those provided by the xDrip+ app, such as **Scan for G5 Contantly**, under **G5 Debug Settings**.  As with the xDrip+ app, they should only be enabled if the watch has connectivity issues.

    For example, many users find that the **Sony Smartwatch 3 (SW3)** does not require any of these settings enabled.
    While other SW3 users find enabling **Scan for G5** helpful.

    Some users of the **Moto 360 2nd Gen** watch report the **Unbond G5/Read** pref is required.

    There are the following two, known exceptions:
    - **Force Screen On** - Some watches, such as the **Moto 360 2nd Gen**, fall into deep sleep preventing the BT Collector from retrieving the transmitter data.
    Enabling this preference will trigger the watch to wakeup to read the transmitter data, then fall back to sleep.
    This preference is **experimental** but may be helpful for watches that do not support the **Always-on screen** feature.

      Rather than use this preferences, it is recommended that users enable **Always-on screen** on their watch or Android Wear app, when supported.  The **Moto 360 2nd Gen** is currently the only known watch that does not support **Always-on screen**.

      The following image shows an example of the **Always-on Screen** setting in Android Wear and on the watch settings.

      <img align="middle" src="./images/androidwear-always-on-screen.png" title="Android Wear Always-on Screen">

    - **Auth G5/Read** - This should be enabled if using the latest, Dexcom G5 transmitter firmware (released in November 2016, **firmware 1.0.4.10**, or newer than **firmware 1.0.0.17**).

  - Alerts

    Alerts can be enabled on the watch when in standalone mode (i.e., when Force Wear is enabled) by enabling **Enable Alerts** and **High Alerts** on the watch.  This will allow alerts to be triggered when disconnected from the phone.  Currently the following xDrip+ app alerts under **Alarms and Alerts** are supported on the watch:

    1. **Alert List** - All alerts.
    2. **Other Alerts** - **Bg falling fast** and **Bg risng fast**.
    3. **Extra Alerts (xDrip+)**
       1. **Persistent High Alert** - configured on watch by enabling **High Alerts**.
       2. **Collector Battery Alerts** - this alert applies to the BT Collector running on phone or watch.
       If the collector is running on the watch, battery refers to the watch battery for G4 and G5, for other non-G5 BT Collectors, battery refers to those devices communicating with the transmitter.  Battery level is checked upon each transmitter BG reading.

    **Glucose Alerts Settings** are currently not configurable for the watch.  Watch alerts use following defaults:

        1. Vibrate-only Profile
        2. Smart Snoozing.
        3. Smart Alerting.

The following image shows xDrip+ app alerts under **Alarms and Alerts** which are supported on the watch.

<img align="middle" src="./images/prefs-alerts-phone.png" title="xDrip+ Wear Alert Preferences on Phone">

The following images show watch alert preferences and example alerts.

<img align="middle" src="./images/alerts.png" title="xDrip+ Wear Alerts">

Users will continue to receive those phone alerts which are not supported on the watch.

Phone and watch alerts can be distinguished by their **Open** dialog.  The phone alert will display **Open on phone**.  Whereas the watch alert will display **Open** dialog.  Upon tapping Open, the Snooze dialog will be displayed on the watch.

The watch Snooze performs the same functionality that the phone Snooze performs.  Tapping the Snooze button and number from the NumberPicker, on either device, will snooze the alert on both the phone and the watch.  The snoozed active alert will trigger a toast message on the paired device to alert the user of the snooze.

Note, tapping the other Snooze buttons, such as **Disable All alerts** will only snooze the alarms on the active device.

The following image shows xDrip+ app Battery Alert under **Extra Alerts (xDrip+)** and example alerts on both devices.

<img align="middle" src="./images/prefs-alerts-battery.png" title="xDrip+ Battery Alert">

#### XDrip Watchface Settings
The following new preferences are supported:
  - Refresh On Change - Refresh face on each data change.  When disabled, face is refreshed only each minute, potentially saving battery usage.
  - Show Treatments - Show Treatment points on graph.  When disabled, treatments will not be displayed, potentially saving battery usage.
  - 24 HR Format - Format time using 24 Hour Format.  When enabled, uses less face real estate, potentially useful for circular faces.
  - Show Date - Show Date on face.
  - Override Locale - Override watch with phone locale, used to format date.
  - Show Toasts - Show Steps and Extra Status toast popup messages.
  - Show Steps - Show Step Counter on all XDrip watchfaces.  Steps reset to 0 at midnight. To enable xDrip+ synchronization, you must switch **Use Wear Health Data** on under xDrip+ **Smart Watch Features**. When enabled, wear steps will be synced to xDrip+.
  - Step Delay - Select time delay from the list preferences.  Step Delay is maximum reporting latency. Events are batched up until this "maximum" latency has lapsed. Once lapsed all batched up events will occur sequentially.
  This is a way to save power consumption so that each event does not interrupt the processer. If you find 10 seconds uses too much power you can select a larger latency. This will of course effect the frequency at which the display of the steps is updated on the watch. Defaults to 10 seconds.  To conserve wear battery, select 5 minutes.
  - Heart Rate - Show last BPM is supported by device.
  - Show Status - Show Loop Status on the XDrip, XDrip(Large) and XDrip (BigChart)watchfaces.  This will display the HAPP status message containing Basal%, IOB, COB.
  - Opaque Card - Show notifications cards with opaque background.  This will allow cards to be read more easily in ambient mode.
  - Small Font - Fontsize of small text in status and delta time fields on the XDrip and XDrip(Large) watchfaces.
  - Show Bridge Battery - Show bridge battery usage on the XDrip and XDrip(Large) watchfaces.  This setting will only be displayed when the BT Collector uses a battery, for example, LimiTTer or Wixel/xBridge.

The following images show some of the watchface preferences under XDrip Watchface Settings.

<img align="middle" src="./images/prefs-watchface-settings.png" title="XDrip Watchface Settings">

The following images show an example of an alert displayed with Opaque Card enabled and disabled in Ambient Mode and Interactive Mode.

<img align="middle" src="./images/prefs-watch-opaque.png" title="XDrip Watchface Opaque Card">

The following images show required xDrip+ preferences and examples of Watch Steps synced with xDrip+.

<img align="middle" src="./images/prefs-phone-sync-steps.png" title="XDrip Phone and Watchface Step Counter">

#### Use Android Wear complication since Wear 2.0

You can also install Watchfaces supporting ```complication```, for example:

https://play.google.com/store/apps/details?id=co.smartwatchface.watch.face.aviator.android.wear&hl=en_US

How-to use:
* long press on watchface
* open settings
* complication
* 3rd party
* select where to place complication
* select xdrip pref

If placing right or left, it looks like this:

<img align="middle" src="./images/wear-complication-short.png" title="Wear complication short">

* If under xdrip settings you have selected ```Loop status``` and placing up or bottom, it looks like this:

<img align="middle" src="./images/wear-complication-long.png" title="Wear complication long">

If getting "Null, Null" displayed, try to add a blood calibration (see https://github.com/NightscoutFoundation/xDrip/issues/651)


#### Watchface Tap Feature
Watchface tap feature is now implemented for the following preferences:
* Chart Timeframe - double tap on the chart in any of the watchfaces will toggle the chart timeframe allowing one to zoom in/out of a frame.
* Small Font - double tap on the delta time in the XDrip or XDrip(Large) watchface will toggle the fontsize of the delta and status line text for ease of viewing.
* Status Line - single tap on the status line in the XDrip or XDrip(Large) watchface will popup a toast message containing the full text for ease of viewing.

The following images show an example of the HAPP message and its integration with xDrip+ watchface.

<img align="middle" src="./images/prefs-wear-happ-status-line.png" title="XDrip Watchface HAPP Status Line">

#### Battery Usage
The wear app supports the display of two battery usage options:
* Bridge - displays the wixel or LimiTTer battery usage.  The Show Bridge Battery must be enabled to display the bridge battery usage.
* Uploader or Wear - will display the battery usage of the device running the collection service.  So, if Enable Wear and Force Wear prefs are enabled, it will display the **watch** battery usage.  If only Enable Wear is enabled, then it will display the battery usage of whichever device is actually running the collection service.  If neither prefs are enabled, it displays the phone's battery usage.  The label, **Uploader** or **Wear** corresponds to the device running the collector.  **Uploader** for phone which is the default, and **Wear** for the watch.  This will allow users to identify which device is running the collection service.

The following images show an example of BT Collector running under each device, with and without Show Status enabled, and Chart Timeframe toggled.

<img align="middle" src="./images/prefs-wear-battery-usage.png" title="XDrip Watchface Battery Usage">

For G4 and G5 BT Collectors, when BT Collector is running on the Phone, the battery usage displays the phone's battery usage as indicated by its label, "Uploader".
When BT Collector is running on the watch, battery usage shows the watches' battery usage as indicated by its label, "Wear".
When the BT Collector is connected to a bridge device, the battery usage displays that device's battery usage as indicated by its label, "Bridge".

## 3-Dot Menu
The 3-dot Menu icon is displayed in the upper, left corner on the XDrip watchface, below the Steps icon if enabled.  Tapping it will activate a new activity with two rows of 4 icons, each with a separate task as shown in the following screenshot.

<img align="middle" src="./images/3-dot-menu.png" title="XDrip Watchface 3-Dot Menu">

The second row of icons in the Menu allow the user to view data tables as shown in the following examples.

<img align="middle" src="./images/data-tables.png" title="XDrip Watchface 3-Dot Menu Data Tables">

## XDrip Treatments
Treatment data points can be displayed on the watchface graph by enabling the "Show Treatments" preference, either on the Phone under Wear Integration, or via the watch XDrip Prefs activity as shown in the following screenshot.

Once enabled, treatments entered via the phone or watch UI will be displayed on the watch.  Insulin, Carbs, Notes are displayed with a green diamond.  Bloodtests are displayed with a red square, and Calibrations are displayed with a red circle.  Each of these are outlined with a yellow rim for a 3-dimensional effect.  Treatments entered on the watch will be saved to the Wear database and posted to the phone upon next connection.  Likewise, treatments entered on the phone will be posted to the watch upon next connection so that data should remain in-sync when Show Treatments is enabled.

Note, this process only supports treatments entered by the Keyboard Input Activity.  Spoken Treatment is currently not supported since many watches do not support speech recognition well.  Treatments entered via the Spoken Treatment Activity continues to work as before and requires a connection to the phone for data to be processed.
Here is the typical setup and process on the watch for Spoken Treatments.

	1. Associate the Spoken Treatment app with "Take a note" Action in your phone's Android Wear for the watch. Your phone may also have the option to select Google Keep as well.
	2. Say "Ok Google" to your watch.  It is slow so you have to speak clearly. Then say, "Take a note". The watch should display the speak icon.
	3. Then speak your treatment, slowly and distinctly, for example, "1 carb".  This activates the Spoken Treatment activity with the mic icon. Press the mic icon, and the approval screen should be displayed.

<img align="middle" src="./images/prefs-show-treatments.png" title="XDrip Show Treatments Preference">

The following screenshot shows an example of entering Treatments on the watch.

<img align="middle" src="./images/enter-treatments.png" title="XDrip Enter Treatments">

## Troubleshooting xDrip Wear
The BT Collector connects to the transmitter every 5 mins by design. This is how the Collector's BLE works. The following provides some troubleshooting suggestions if readings are not being receiving every 5 minutes.

### Confirm Collector runs on the Phone with Wear Integration

  Ensure Wear Integration preferences are set as follows:
  - **Wear Integration** is enabled.
  - **Enable Wear** is selected.
  - **Force Wear** is **NOT** selected.

This will allow your phone to use the G5 collector as normal when both phone and watch are in-range. After receiving a reading on your phone, ensure it displays on your watch.

After you confirm that you are get a reading on your phone, enable **Force Wear**, either on the phone or watch XDrip Prefs.

This will force the watch to use its BT collector, and force the phone to stop its BT collector service discussed next.

### Confirm Collector runs on the Watch with Wear Integration
  Ensure Wear Integration preferences are set as follows on both phone and watch:
  - **Wear Integration** is enabled.
  - **Enable Wear** is selected.
  - **Force Wear** is selected.
  - **Device running Collection Service** corresponds to the watch display name + uuid.
  - XDrip Prefs **BT Collector** corresponds to phone's Hardware Data Source.

  Confirm Environment:
  - Disable engineering mode.
  - Disable Calibration plugin (incl. Adrian calibration mode).
  - Disable then re-enable Wear Integration.
  - Enable Force Wear.
  - Show raw values.
  - Smooth sensor noise off.
  - Reset Wear DB, restart Watch, restart phone.
  - Confirm phone and watch are connected via Android Wear.
  - Change Watchface to big chart and then back to standard xDrip.
  - Optionally, perform **Reset Wear DB** from xDrip+ upper, right menu.  Refer to [Syncing BGs and Wear Database](#syncing-bgs-and-wear-database) above.

### Confirm the following in Android Wear app on phone
- Watch is connected to the phone.
- Watch Settings **always-on screen** is enabled.  This will prevent watch doze mode from shutting down the BT Collector.
Refer to [XDrip BT Settings](#xdrip-bt-settings) above for additional details.

  To verify devices are connected, check the phone Android Wear app.  Android wear (on the watch) displays the **cloud** icon if the devices are not in-range, or if the user manually disconnects the devices in Android Wear.

  Similarly, some users have found it necessary to enable the **Stay awake while charging** setting under their watch Settings **Developer Options**.  In testing thus far, only the **Moto 360 2nd Gen** watch has required this option.

### Confirm Wear Integration preferences are consistent on both phone and watch

  **Enable Wear** and  **Force Wear** should have same settings on phone and watch.  If not, reset them accordingly.  The xDrip+ should sync these values whenever the user modifies them or at application startup, but both phone and watch must be connected and in-range for syncing to be performed.

### Confirm Collection Method is consistent on both phone and watch

  Confirm the phone's Hardware Data Source preference matches the watch's BT Collector preference.  The watch's BT Collector preference is a read-only preference.  It gets set based on the phone's Hardware Data Source preference. The following values correspond to the collectors:
   - BluetoothWixel("BluetoothWixel"),
   - DexcomShare("DexcomShare"),
   - DexbridgeWixel("DexbridgeWixel"),
   - LimiTTer("LimiTTer"),
   - WifiWixel("WifiWixel"),
   - DexcomG5("DexcomG5"),
   - WifiDexBridgeWixel("WifiDexbridgeWixel"),
   - LibreAlarm("LibreAlarm")

Refer to [Collector](#collector) above for additional details.

### Confirm Collector device exists under Watch Settings BT Devices

  Once the BT Collection Service executes it will perform a BT scan, and upon detecting the BT Collector device, will display under the Watch Settings -> Bluetooth Devices.  Typically it will show as disconnected as it only connects briefly to receive the BG reading.

### Confirm Calibration Preferences are consistent on both phone and watch

  The watch app does not yet support Calibration Plugins.  Therefore, to confirm BG readings are consistently calculated on both phone and watch, it is best to turn off Calibration Plugins on the phone.
  - LCS - **Advanced Calibration** - all should be off, including **Adrian calibration mode**.

### Confirm Noise Preferences are consistent on both phone and watch

  The watch app does not yet support Noise smoothing.  Therefore, to confirm BG readings are consistently calculated on both phone and watch, it is best to turn off Noise Smoothing on the phone.

  - xDrip+ Display Settings - **Show Noise workings** should be disabled.

    When Show Noise Workings is enabled, **BG Original** and **BG Estimate** will display on the home screen.
    - BG Original should correspond to your watch value, and
    - BG Estimate should correspond to your phone value.

  - xDrip+ Display Settings - **Smooth Sensor Noise** should be disabled.

### Confirm Location Permission is enabled in Watch Settings

  Android Wear requires Location Access to be manually accepted by the user.

  However, sometimes the user overlooks this dialog.

  Further, this preference may appear to be enabled under the Watch -> Settings -> Permissions - XDrip Prefs, when it actually is DISABLED, perhaps an Android Wear bug.
  Evidence of this can be seen in the Events log (refer to [Syncing BGs and Wear Database](#syncing-bgs-and-wear-database) UserError Table) with the following log entry:
  ```
    E/jamorham JoH: Could not set pairing confirmation due to exception: java.lang.SecurityException: Need BLUETOOTH PRIVILEGED permission: Neither user 10035 nor current process has android.permission.BLUETOOTH_PRIVILEGED.
  ```
  This message appears to occur only once, so it is easily missed if logging is not enabled via the Sync Wear Logs preference.

  However, this Permission is required, and therefore, if not enabled, the Collection Service will not be able to connect to the transmitter via BLE.
  Connection errors such as the following (for G5CollectionService, DexcomG5 Hardware Service) will continued to be logged to the Events log:

  ```
  06-10 07:47:03.427 2162-2172/? E/G5CollectionService: Encountered 133: true

  06-10 07:57:14.936 2162-2173/? I/G5CollectionService: Read code: 7 - Transmitter NOT already authenticated?
  06-10 07:57:14.937 2162-2173/? E/G5CollectionService: Sending new AuthRequestTxMessage to Authentication ...
  06-10 07:57:14.958 2162-2173/? D/G5CollectionService: New AuthRequestTxMessage: 01346163356234373302
  06-10 07:57:14.959 2162-2173/? I/G5CollectionService: AuthRequestTX: 01346163356234373302
  06-10 07:57:14.961 2162-2173/? E/G5CollectionService: OnCharacteristic READ finished: status: Gatt Success
  06-10 07:57:14.984 2162-2173/? E/G5CollectionService: STATE_DISCONNECTED: Connection terminated by peer
  ```

  Causing the Collector to restart Bluetooth:
  ```
  06-10 07:57:15.183 2162-2751/? E/G5CollectionService: Cycling BT-gatt - disabling BT
  ...
  06-10 07:57:18.204 2162-2752/? E/G5CollectionService: Cycling BT-gatt - enabling BT
  ```

  The following steps are recommended for resolution:
  1. Toggle the ```Location ENABLED preference``` to disable it under Watch Settings (refer to [Initial Wear Enablement Requests Location Permission](#initial-wear-enablement-requests-location-permission)).
  2. Restart the Collector on Wear (refer to [3-Dot Menu](#3-dot-menu) Restart Collector or alternatively, Restart Collector in xDrip+ System Status).  This should force the Location Access Permission dialog to be displayed on the watch for user acceptance.
  3. Or, if the above fail to resolve the issue, uninstall then reinstall the Wear app.  This will force the Location Access Permission dialog to be displayed on the watch for user acceptance.

# ADB DEBUG
### Debugging Android Wear
[Howto Enable Debugging](http://www.androidpolice.com/2014/07/05/how-to-android-wear-enable-debugging-take-screenshots-unlock-the-bootloader-and-root-the-lg-g-watch/)

1. Open Settings.
	1. Tap on Wear's watch face. This will take you to the voice prompt. Be sure to hit the watch face instead of a notification card.
	2. Wear will wait up to 3 seconds for you to say something, then it'll change to a scrollable list of native actions. You can speed this up by swiping up or tapping on the voice prompt.
	3. Scroll down and select Settings.
2. Open About.
3. Find Build number and tap on it 7 times. You're done when a toast popup appears with the message, "You are now a developer!"
4. Swipe right (to go back) to the Settings menu.
5. Open Developer options.
6. Find and set ADB debugging to Enabled.
7. You'll be asked if you're sure you want to enable. Tap the checkmark button to confirm.
8. [Optional] If you want to also turn on debugging over Bluetooth, Find and set Debug over Bluetooth to Enabled.

At the terminal, issue:

```
D:\Android\sdk\platform-tools>adb devices
List of devices attached
14502D1AF252D74 device
D:\Android\sdk\platform-tools>adb -s 14502D1AF252D74 logcat > wear.log
```
If you see **unauthorized** description of your device, ensure that ADB debugging is enabled on your watch under Developer Options.

Enter the following cmd to generate a logcat log, where -s arg is your watch device if you have more than one device connected to your computer, otherwise, omit the -s arg. You can retrieve the device id using cmd adb devices: 

```
D:\Android\sdk\platform-tools>adb devices
D:\Android\sdk\platform-tools>adb -s 14502D1AF252D74 logcat > wear.log
```


### Help my NFC patched Sony SmartWatch 3 doesn't work with xDrip as a collector

The problem typically is the version of Google Play Services on the watch. You need at least 9.x installed.

On the watch check the version with `Settings` -> `About` -> `Versions`

If your google play services version is less than 9, then you should install version 9.

This is available from unofficial sources which you can find by googling, for example:

https://www.apkmirror.com/apk/google-inc/google-play-services-android-wear/google-play-services-android-wear-9-8-41-release/google-play-services-9-8-41-534-130237018-android-apk-download/

To install, connect your watch via usb and make sure adb debugging is enabled. You should be familiar with this process from how you originally installed the custom NFC firmware.

use the command:

`adb install -r downloaded-filename.apk`
