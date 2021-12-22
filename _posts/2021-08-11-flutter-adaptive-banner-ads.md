---
title: How to add adaptive banner ads in your flutter app using google mobile ads?
tags: [Flutter, Adaptive Banner Ads, Google Admob]
style: border
color: primary
description: Learn how to add & customize adaptive banner ads in a flutter application using google mobile ads.
permalink: blog/how-to-use-adaptive-banner-ads-in-your-flutter-app-using-google-mobile-ads
date: 2021-08-11
---

Author: [Deepanshu](https://www.github.com/ChaudharyDeepanshu)

Hi! This is a short post about how to implement adaptive banner ads into a flutter application using Google mobile ads. Once you've followed all the steps given below in this post, you'll be able to add amazing banner ads to your app like this:

<img src="{{site.baseurl}}/assets/images/posts/flutter adaptive banner ads/Adaptive banner ads.png" alt="Adaptive banner ads" width="YYY"/>

Step 1: Add [google_mobile_ads](https://pub.dev/packages/google_mobile_ads) and [provider](https://pub.dev/packages/provider) to your pubspec.yaml.

Step 2: Change your minSdkVersion to 19 in app's `android/app/build.gradle` as shown in the image below:

<img src="{{site.baseurl}}/assets/images/posts/flutter adaptive banner ads/App build gradle snip.png" alt="Adaptive banner ads" width="YYY"/>

Step 3: Update AndroidManifest.xml. Add your "AdMob App ID" to the app's `android/app/src/main/AndroidManifest.xml` .Add `<meta-data>` tag like this:

```xml
<manifest>
    <application>
        <meta-data
            android:name="com.google.android.gms.ads.APPLICATION_ID"
            android:value="ca-app-pub-3940256099942544~3347511713"/>
    </application>
</manifest>
```

**Note**: If you don't have an "Admob App Id" for now then you can use this id:

`ca-app-pub-3940256099942544~3347511713`

It is provided for testing. So, I will be using this id in the example code below but make sure to replace it with yours in your application. 
Quick tip: Create an account on Admob for "Admob App Id" and then for more info go [here](https://support.google.com/admob/answer/7356431?hl=en).

Step 4: Create a dart file in your project named "ad_state.dart" with the following code in it:

```dart
import 'package:google_mobile_ads/google_mobile_ads.dart';
import 'dart:io';

class AdState {
  Future<InitializationStatus> initialization;
  AdState(this.initialization);

  String? get bannerAdUnitId => Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/6300978111' //test AdUnitId
      //? null // use this stop ads
      : 'ca-app-pub-3940256099942544/2934735716';

  AdManagerBannerAdListener get adListener => _adListener;

  AdManagerBannerAdListener _adListener = AdManagerBannerAdListener(
    // Called when an ad is successfully received.
    onAdLoaded: (Ad ad) => print('Ad loaded.'),
    // Called when an ad request failed.
    onAdFailedToLoad: (Ad ad, LoadAdError error) {
      // Dispose the ad here to free resources.
      ad.dispose();
      print('Ad failed to load: $error');
    },
    // Called when an ad opens an overlay that covers the screen.
    onAdOpened: (Ad ad) => print('Ad opened.'),
    // Called when an ad removes an overlay that covers the screen.
    onAdClosed: (Ad ad) => print('Ad closed.'),
    // Called when an impression occurs on the ad.
    onAdImpression: (Ad ad) => print('Ad impression.'),
    onAppEvent: (ad, name, data) =>
        print('App event : ${ad.adUnitId}, $name, $data.'),
  );
}
```

Step 5: Change your "main" function in "main.dart" to like this:

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final initFuture = MobileAds.instance.initialize();
  final adState = AdState(initFuture);
  runApp(Provider.value(
    value: adState,
    builder: (context, child) => MyApp(),
  ));
}
```

Step 6: Create a dart file in your project named “anchored_adaptive_banner_adSize.dart” with the following code in it:

```dart
import 'package:google_mobile_ads/google_mobile_ads.dart';
import 'package:flutter/material.dart';

Future<AnchoredAdaptiveBannerAdSize?> anchoredAdaptiveBannerAdSize(
    BuildContext context) async {
  return await AdSize.getAnchoredAdaptiveBannerAdSize(
    MediaQuery.of(context).orientation == Orientation.portrait
        ? Orientation.portrait
        : Orientation.landscape,
    MediaQuery.of(context).size.width.toInt(),
  );
}
```

Step 7: Create a dart file in your project named "banner_ad.dart" with the following code in it:

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';
import 'package:provider/provider.dart';
import 'ad_state.dart';
import 'anchored_adaptive_banner_adSize.dart';

class BannerAD extends StatefulWidget {
  const BannerAD({Key? key}) : super(key: key);

  @override
  _BannerADState createState() => _BannerADState();
}

class _BannerADState extends State<BannerAD> {
  BannerAd? banner;

  AnchoredAdaptiveBannerAdSize? size;

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    final adState = Provider.of<AdState>(context);
    adState.initialization.then((value) async {
      size = await anchoredAdaptiveBannerAdSize(context);
      setState(() {
        if (adState.bannerAdUnitId != null) {
          banner = BannerAd(
            listener: adState.adListener,
            adUnitId: adState.bannerAdUnitId!,
            request: AdRequest(),
            size: size!,
          )..load();
        }
      });
    });
  }

  @override
  Widget build(BuildContext context) {
    return banner ==
            null //banner is only null for a very less time //don't think that banner will be null if ads fails loads
        ? SizedBox()
        : Container(
          color: Colors.grey,
          width: size!.width.toDouble(),
          height: size!.height.toDouble(),
          child: AdWidget(
            ad: banner!,
          ),
        );
  }
}
```

Step 8: Now use "BannerAD()" as a widget where ever you want to. For example:

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';
import 'package:provider/provider.dart';
import 'ads/ad_state.dart';
import 'ads/banner_ad.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final initFuture = MobileAds.instance.initialize();
  final adState = AdState(initFuture);
  runApp(Provider.value(
    value: adState,
    builder: (context, child) => MyApp(),
  ));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Adaptive Banner Ad Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Adaptive Banner Ad'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key? key, required this.title}) : super(key: key);

  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.end,
          children: <Widget>[
            BannerAD(),
          ],
        ),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}
```

Result:

<img src="{{site.baseurl}}/assets/images/posts/flutter adaptive banner ads/Adaptive banner ads.png" alt="Adaptive banner ads" width="YYY"/>

**Want something more?** 

So, as everything is working fine and you have successfully implemented adaptive ads but if you still want some more personalization to ads such as a special message when the user turns off the internet or a special message when ads get load then follow the below optional steps:

(Optional) Step 9: Add [connectivity_plus](https://pub.dev/packages/connectivity_plus)  to your pubspec.yaml.

(Optional) Step 10: Replace the "banner_ad.dart" file code which I provided above with the code below:

```dart
import 'dart:async';
import 'package:connectivity_plus/connectivity_plus.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';
import 'package:provider/provider.dart';
import 'ad_state.dart';
import 'anchored_adaptive_banner_adSize.dart';

class BannerAD extends StatefulWidget {
  const BannerAD({Key? key}) : super(key: key);

  @override
  _BannerADState createState() => _BannerADState();
}

class _BannerADState extends State<BannerAD>{
  ConnectivityResult _connectionStatus = ConnectivityResult.none;
  final Connectivity _connectivity = Connectivity();
  late StreamSubscription<ConnectivityResult> _connectivitySubscription;

  BannerAd? banner;

  @override
  void dispose() {
    _connectivitySubscription.cancel();
    super.dispose();
  }

  AnchoredAdaptiveBannerAdSize? size;

  @override
  void initState() {
    super.initState();
    initConnectivity();
    _connectivitySubscription =
        _connectivity.onConnectivityChanged.listen(_updateConnectionStatus);
  }

  // Platform messages are asynchronous, so we initialize in an async method.
  Future<void> initConnectivity() async {
    late ConnectivityResult result;
    // Platform messages may fail, so we use a try/catch PlatformException.
    try {
      result = await _connectivity.checkConnectivity();
    } on PlatformException catch (e) {
      print(e.toString());
      return;
    }

    // If the widget was removed from the tree while the asynchronous platform
    // message was in flight, we want to discard the reply rather than calling
    // setState to update our non-existent appearance.
    if (!mounted) {
      return Future.value(null);
    }

    return _updateConnectionStatus(result);
  }

  void _updateConnectionStatus(ConnectivityResult result) {
    setState(() {
      _connectionStatus = result;
      if (banner != null) {
        banner!.load();
      }
    });
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    final adState = Provider.of<AdState>(context);
    adState.initialization.then((value) async {
      size = await anchoredAdaptiveBannerAdSize(context);
      setState(() {
        if (adState.bannerAdUnitId != null) {
          banner = BannerAd(
            listener: adState.adListener,
            adUnitId: adState.bannerAdUnitId!,
            request: AdRequest(),
            size: size!,
          )..load();
        }
      });
    });
  }

  @override
  Widget build(BuildContext context) {
    print('Connection Status: ${_connectionStatus.toString()}');
    return banner == null
        ? SizedBox()
        : _connectionStatus == ConnectivityResult.none
            ? Container(
              height: AdSize.banner.height.toDouble() + 10,
              width: size!.width.toDouble(),
              color: Colors.grey,
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Expanded(
                        child: Text(
                          'To support the app please connect to internet.',
                          style: TextStyle(
                              fontSize: 20, fontWeight: FontWeight.bold),
                          textAlign: TextAlign.center,
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            )
            : Container(
              color: Colors.grey,
              width: size!.width.toDouble(),
              height: size!.height.toDouble(),
              child: Stack(
                children: [
                  Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Row(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          Expanded(
                            child: Text(
                              'Ad loading...\nThanks for your support',
                              style: TextStyle(
                                  fontSize: 20,
                                  fontWeight: FontWeight.bold),
                              textAlign: TextAlign.center,
                            ),
                          ),
                        ],
                      ),
                    ],
                  ),
                  AdWidget(
                    ad: banner!,
                  ),
                ],
              ),
            );
  }
}
```

Your adaptive ads are all set now. To play with the demo app, clone this [repository](https://github.com/chaudharydeepanshu/flutter_adaptive_banner_demo).

Thanks for reading this article.