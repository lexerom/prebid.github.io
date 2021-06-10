---

layout: page_v2
title: Prebid Mobile Rendering GAM Line Item Setup
description: Prebid Mobile Rendering Modules GAM line item setup
sidebarType: 2

---

# Google Ad Manager Integration

The integration of Prebid Rendering Module with Google Ad Manager (GAM) assumes that publisher has an account on GAM and has already integrated the Google Mobile Ads SDK (GMA SDK) into the app project. 


If you do not have GAM SDK in the app yet, refer the the [Google Integration Documentation](https://developers.google.com/ad-manager/mobile-ads-sdk/android/quick-start).

Prebid Rendering Module was tested with **GAM SDK 7.61.0**. 


## GAM Integration Overview

![Rendering with GAM as the Primary Ad Server](/assets/images/prebid-mobile/modules/rendering/Prebid-In-App-Bidding-Overview-GAM.png)

**Steps 1-2** Prebid Rendering Module makes a bid request. Prebid server runs an auction and returns the winning bid.

**Step 3** Prebid Rendering Module via GAM Event Handler sets up the targeting keywords into the GAM's ad unit.

**Step 4** GMA SDK makes an ad request. GAM returns the winner of the waterfall.

**Step 5** Basing on the ad response Prebid GAM Event Handler decided who won on the GAM - the Prebid bid or another ad source on GAM.

**Step 6** The winner is displayed in the App with the respective rendering engine.
  
Prebid Rendering Module supports these ad formats:

- Display Banner
- Display Interstitial
- Native
- Native Styles
- Video Interstitial 
- Rewarded Video
- Outstream Video

They can be integrated using these API categories.

- [**Banner API**](#banner-api) - for *Display Banner* and *Outstream Video*
- [**Interstitial API**](#interstitial-api) - for *Display* and *Video* Interstitials
- [**Rewarded API**](#rewarded-api) - for *Rewarded Video*
- [**Native API**](#native-styles) - for *Native Ads*


## Init Prebid Rendering Module

To start running bid requests you have to provide an **Account Id** for your organization on Prebid server to the SDK:

```
PrebidRenderingSettings.setBidServerHost(HOST)
PrebidRenderingSettings.setAccountId(YOUR_ACCOUNT_ID)
```

The best place to do it is the `onCreate()` method of your Application class.

> **NOTE:** The account ID is an identifier of the **Stored Request**.

### Event Handlers

GAM Event Handlers is a set of classes that wrap the GAM Ad Units and manage them respectively to the In-App Bidding flow. These classes are provided in the form of library that could be added to the app via Gradle:

Root build.gradle

```
allprojects {
    repositories {
      ...
      mavenCentral()
      ...
    }
}
```

App module build.gradle:

```
implementation('org.prebid:prebid-mobile-sdk-gamEventHandlers:x.x.x')
```


## Banner API

To integrate the banner ad you need to implement three easy steps:


``` kotlin
// 1. Create banner custom event handler for GAM ad server.
val eventHandler = GamBannerEventHandler(requireContext(), GAM_AD_UNIT, GAM_AD_SIZE)

// 2. Create a bannerView instance and provide GAM event handler
bannerView = BannerView(requireContext(), configId, eventHandler)
// (Optional) set an event listener
bannerView?.setBannerListener(this)

// Add bannerView to your viewContainer
viewContainer?.addView(bannerView)

// 3. Execute ad loading
bannerView?.loadAd()
```

#### Step 1: Create Event Handler

GAM's event handlers are special containers that wrap GAM Ad Views and help to manage collaboration between GAM and Prebid views.

**Important:** you should create and use a unique event handler for each ad view.

To create the event handler you should provide a GAM Ad Unit Id and the list of available sizes for this ad unit.

#### Step 2: Create Ad View

**BannerView** - is a view that will display the particular ad. It should be added to the UI. To create it you should provide:

- **configId** - an ID of Stored Impression on the Prebid server
- **eventHandler** - the instance of the banner event handler

Also, you should add the instance of `BannerView` to the UI.

And assign the listeners for processing ad events.

#### Step 3: Load the Ad

Simply call the `loadAd()` method to start [In-App Bidding](../android-in-app-bidding-getting-started.md) flow. The In-App Bidding SDK starts the  bidding process right away.

### Outstream Video

For **Outstream Video** you also need to specify video placement type of the expected ad:

``` kotlin
bannerView.videoPlacementType = PlacementType.IN_BANNER // or any other available type
```

## Interstitial API

To integrate interstitial ad you need to implement four easy steps:


``` kotlin
// 1. Create interstitial custom event handler for GAM ad server.
val eventHandler = GamInterstitialEventHandler(requireContext(), gamAdUnit)

// 2. Create interstitialAdUnit instance and provide GAM event handler
interstitialAdUnit = InterstitialAdUnit(requireContext(), configId, minSizePercentage, eventHandler)
// (Optional) set an event listener
interstitialAdUnit?.setInterstitialAdUnitListener(this)

// 3. Execute ad load
interstitialAdUnit?.loadAd()

//....

// 4. After ad is loaded you can execute `show` to trigger ad display
interstitialAdUnit?.show()

```

The way of displaying **Video Interstitial Ad** is almost the same with two differences:

- Need to customize the ad unit format
- No need to set up `minSizePercentage`

``` kotlin
// 1. Create interstitial custom event handler for GAM ad server.
val eventHandler = GamInterstitialEventHandler(requireContext(), gamAdUnit)

// 2. Create interstitialAdUnit instance and provide GAM event handler
interstitialAdUnit = InterstitialAdUnit(requireContext(), configId, AdUnitFormat.VIDEO, eventHandler)

// (Optional) set an event listener
interstitialAdUnit?.setInterstitialAdUnitListener(this)

// 3. Execute ad load
interstitialAdUnit?.loadAd()

//....

// 4. After ad is loaded you can execute `show` to trigger ad display
interstitialAdUnit?.show()

```


#### Step 1: Create Event Handler

GAM's event handlers are special containers that wrap the GAM Ad Views and help to manage collaboration between GAM and Prebid views.

**Important:** you should create and use a unique event handler for each ad view.

To create an event handler you should provide a GAM Ad Unit.

#### Step 2: Create Interstitial Ad Unit

**InterstitialAdUnit** - is an object that will load and display the particular ad. To create it you should provide:

- **configId** - an ID of Stored Impression on the Prebid server
- **minSizePercentage** - specifies the minimum width and height percent an ad may occupy of a device’s real estate.
- **eventHandler** - the instance of the interstitial event handler

Also, you can assign the listeners for processing ad events.

> **NOTE:** minSizePercentage - plays an important role in a bidding process for display ads. If provided space is not enough demand partners won't respond with the bids.


#### Step 3: Load the Ad

Simply call the `loadAd()` method to start [In-App Bidding](../android-in-app-bidding-getting-started.md) flow. The ad unit will load an ad and will wait for explicit instructions to display the Interstitial Ad.


#### Step 4: Show the Ad when it is ready


The most convenient way to determine if the interstitial ad is ready for displaying is to listen to the particular listener method:

``` kotlin
override fun onAdLoaded(interstitialAdUnit: InterstitialAdUnit) {
//Ad is ready for display
}
```

## Rewarded API

To display an Rewarded Ad need to implement four easy steps:


``` kotlin
// 1. Create rewarded custom event handler for GAM ad server.
val eventHandler = GamRewardedEventHandler(requireActivity(), gamAdUnitId)

// 2. Create rewardedAdUnit instance and provide GAM event handler
rewardedAdUnit = RewardedAdUnit(requireContext(), configId, eventHandler)

// (Optional) set an event listener
rewardedAdUnit?.setRewardedAdUnitListener(this)

// 3. Execute ad load
rewardedAdUnit?.loadAd()

//...

// 4. After ad is loaded you can execute `show` to trigger ad display
rewardedAdUnit?.show()
```

The way of displaying the **Rewarded Ad** is totally the same as for the Interstitial Ad. You can customize a kind of ad:


To be notified when user earns a reward - implement `RewardedAdUnitListener` interface:

``` kotlin
 fun onUserEarnedReward(rewardedAdUnit: RewardedAdUnit)
```

The actual reward object is stored in the `RewardedAdUnit`:

``` kotlin
val reward = rewardedAdUnit.getUserReward()
```

#### Step 1: Create Event Handler

GAM's event handlers are special containers that wrap the GAM Ad Views and help to manage collaboration between GAM and Prebid views.

**Important:** you should create and use a unique event handler for each ad view.

To create an event handler you should provide a GAM Ad Unit.


#### Step 2: Create Rewarded Ad Unit

**RewardedAdUnit** - is an object that will load and display the particular ad. To create it you should provide

- **configId** - an ID of Stored Impression on the Prebid server
- **eventHandler** - the instance of rewarded event handler

Also, you can assign the listener for processing ad events.


#### Step 3: Load the Ad

Simply call the `loadAd()` method to start [In-App Bidding](../android-in-app-bidding-getting-started.md) flow. The ad unit will load an ad and will wait for explicit instructions to display the Rewarded Ad.


#### Step 4: Show the Ad when it is ready


The most convenient way to determine if the ad is ready for displaying is to listen for particular listener method:

``` kotlin
override fun onAdLoaded(rewardedAdUnit: RewardedAdUnit) {
//Ad is ready for display
}
```


## Native Ads

The general integration scenario requires these steps from publishers:

1. Prepare the ad layout.
2. Create Native Ad Unit and appropriate GAM ad loader.
3. Configure the Native Ad unit using [NativeAdConfiguration](../native/android-native-ad-configuration.md).
    * Provide the list of **[Native Assets](../android-in-app-bidding-native-guidelines-info.md#components)** representing the ad's structure.
    * Tune other general properties of the ad.
4. Make a bid request.
5. Prepare publisherAdRequest using `GamUtils.prepare`
6. After receiving response from GAM  - check if prebid has won and find native ad using `GamUtils`
7. Bind the winner data from the native ad response with the layout.

``` kotlin
val builder = AdManagerAdRequest.Builder()
val publisherAdRequest = builder.build()
nativeAdUnit?.fetchDemand { result ->
    val fetchDemandResult = result.fetchDemandResult
    if (fetchDemandResult != FetchDemandResult.SUCCESS) {
        loadGam(publisherAdRequest)
        return@fetchDemand
    }
    
    GamUtils.prepare(publisherAdRequest, result)
    loadGam(publisherAdRequest)
}
```
**NOTE:** `loadGam` method is creating GAM adLoader and executing `loadAd(publisherAdRequest)`.


Example of handling NativeAd response (the same applies to Custom):

``` kotlin
private fun handleNativeAd(nativeAd: NativeAd) {
    if (GamUtils.didPrebidWin(nativeAd)) {
        GamUtils.findNativeAd(nativeAd) {
            inflateViewContentWithPrebid(it)
        }
    }
    else {
        inflateViewContentWithNative(nativeAd)
    }
}
```

## Native Styles 

[See Google Ad Manager Integration page](android-in-app-bidding-gam-info.md) for more info about GAM order setup and GAM Event Handler integration.

To integrate a banner ad you need to implement three easy steps:

``` kotlin
// 1. Create banner custom event handler for GAM ad server.
val eventHandler = GamBannerEventHandler(requireContext(), GAM_AD_UNIT, GAM_AD_SIZE)

// 2. Create a bannerView instance and provide GAM event handler
bannerView = BannerView(requireContext(), configId, eventHandler)
// (Optional) set an event listener
bannerView?.setBannerListener(this)

// 3. Provide NativeAdConfiguration
val nativeAdConfiguration = createNativeAdConfiguration()
bannerView?.setNativeAdConfiguration(nativeAdConfiguration)

// Add bannerView to your viewContainer
viewContainer?.addView(bannerView)

// 4. Execute ad loading
bannerView?.loadAd()
```

#### Step 1: Create Event Handler

GAM's event handlers are special containers that wrap GAM Ad Views and help to manage collaboration between GAM and Prebid views.

**Important:** you should create and use a unique event handler for each ad view.

To create the event handler you should provide a GAM Ad Unit Id and the list of available sizes for this ad unit.
**Note:** There is a helper function `convertGamAdSize` in GamBannerEventHandler to help you convert GAM AdSize into Prebid AdSize.


#### Step 2: Create Ad View

**BannerView** - is a view that will display the particular ad. It should be added to the UI. To create it you should provide:

- **configId** - an ID of Stored Impression on the Prebid server
- **eventHandler** - the instance of the banner event handler

Also, you should add the instance of `BannerView` to the UI.

And assign the [listeners](../android-in-app-bidding-listeners.md) for processing ad events.

#### Step 3: Create and provide NativeAdConfiguration

NativeAdConfiguration creation example:

``` kotlin
private fun createNativeAdConfiguration(): NativeAdConfiguration {
    val nativeAdConfiguration = NativeAdConfiguration()
    nativeAdConfiguration.contextType = NativeAdConfiguration.ContextType.SOCIAL_CENTRIC
    nativeAdConfiguration.placementType = NativeAdConfiguration.PlacementType.CONTENT_FEED
    nativeAdConfiguration.contextSubType = NativeAdConfiguration.ContextSubType.GENERAL_SOCIAL

    val methods = ArrayList<NativeEventTracker.EventTrackingMethod>()
    methods.add(NativeEventTracker.EventTrackingMethod.IMAGE)
    methods.add(NativeEventTracker.EventTrackingMethod.JS)
    val eventTracker = NativeEventTracker(NativeEventTracker.EventType.IMPRESSION, methods)
    nativeAdConfiguration.addTracker(eventTracker)

    val assetTitle = NativeAssetTitle()
    assetTitle.len = 90
    assetTitle.isRequired = true
    nativeAdConfiguration.addAsset(assetTitle)

    val assetIcon = NativeAssetImage()
    assetIcon.type = NativeAssetImage.ImageType.ICON
    assetIcon.wMin = 20
    assetIcon.hMin = 20
    assetIcon.isRequired = true
    nativeAdConfiguration.addAsset(assetIcon)

    val assetImage = NativeAssetImage()
    assetImage.hMin = 20
    assetImage.wMin = 200
    assetImage.isRequired = true
    nativeAdConfiguration.addAsset(assetImage)

    val assetData = NativeAssetData()
    assetData.len = 90
    assetData.type = NativeAssetData.DataType.SPONSORED
    assetData.isRequired = true
    nativeAdConfiguration.addAsset(assetData)

    val assetBody = NativeAssetData()
    assetBody.isRequired = true
    assetBody.type = NativeAssetData.DataType.DESC
    nativeAdConfiguration.addAsset(assetBody)

    val assetCta = NativeAssetData()
    assetCta.isRequired = true
    assetCta.type = NativeAssetData.DataType.CTA_TEXT
    nativeAdConfiguration.addAsset(assetCta)
    
    nativeAdConfiguration.nativeStylesCreative = nativeStylesCreative

    return nativeAdConfiguration
}
```

See more NativeAdConfiguration options [here](../native/android-native-ad-configuration.md).

#### Step 4: Load the Ad

Simply call the `loadAd()` method to start [In-App Bidding](../android-in-app-bidding-getting-started.md) flow.