# cordova-plugin-inapppurchase 📱💰

[![Build Status](https://travis-ci.org/AlexDisler/cordova-plugin-inapppurchase.svg?branch=master)](https://travis-ci.org/AlexDisler/cordova-plugin-inapppurchase)
[![Coverage Status](https://coveralls.io/repos/github/AlexDisler/cordova-plugin-inapppurchase/badge.svg?branch=master&a=1)](https://coveralls.io/github/AlexDisler/cordova-plugin-inapppurchase?branch=master)

A lightweight Cordova plugin for in app purchases on iOS/Android. See [demo app](https://github.com/AlexDisler/cordova-inapppurchases-app) and [blog post](https://alexdisler.com/2016/02/29/in-app-purchases-ionic-cordova/).

## Features

- Simple, promise-based API
- Support for consumable/non-consumable products and paid/free subscriptions
- Support for restoring purchases
- Uses well tested native libraries internally - [RMStore](https://github.com/robotmedia/RMStore) for iOS and an adjusted  [com.google.payments](https://github.com/MobileChromeApps/cordova-plugin-google-payments/tree/master/src/android) for Android

## Install

    $ cordova plugin add cordova-plugin-inapppurchase

## Configuration

### iOS

No configuration is necessary.

### Android

You must create a ```manifest.json``` in your project's ```www``` folder with your Android Billing Key:

    { "play_store_key": "<Base64-encoded public key from the Google Play Store>" }

You can get this key from the Google Play Store (under "Services & APIs") after uploading your app.

## Setting up and testing purchases

- [Configuring Cordova in app purchases on iOS and Android](https://alexdisler.com/2016/04/30/configuring-in-app-purchases-cordova-ionic-ios-android/)
- [Testing in app purchases](https://alexdisler.com/2016/04/04/testing-cordova-in-app-purchases-on-ios-android/)
- [Receipt validation (with nodejs)](https://alexdisler.com/2016/03/20/validating-cordova-in-app-purchases-on-ios-and-android-using-nodejs/)
- [Tips for signing and running Cordova apps on Android to test in app purchases locally](https://alexdisler.com/2016/04/01/tips-for-signing-installing-cordova-apps-on-android/)

## API

All functions return a Promise.

### Get Products

#### inAppPurchase.getProducts(productIds)

- ___productIds___ - an array of product ids

Retrieves a list of full product data from Apple/Google. **This function must be called before making purchases.**

If successful, the promise resolves to an array of objects. Each object has the following attributes:

- ```productId``` - SKU / product bundle id (such as 'com.yourapp.prod1')
- ```title``` - short localized title
- ```description``` - long localized description
- ```price``` - localized price

___Example:___

```js
inAppPurchase
  .getProducts(['prod1', 'prod2', ...])
  .then(function (products) {
    console.log(products);
    /*
       [{ productId: 'prod1', 'title': '...', description: '...', price: '...' }, ...]
    */
  })
  .catch(function (err) {
    console.log(err);
  });
```

### Buy

#### inAppPurchase.buy(productId)

- ___productId___ - a string of the productId

If successful, the promise resolves to an object with the following attributes that you will need for the receipt validation:

- ```transactionId``` - The transaction/order id
- ```receipt``` - On ***iOS*** it will be the base64 string of the receipt, on ***Android*** it will be a string of a json with all the transaction details required for validation such as ```{"orderId":"...","packageName:"...","productId":"...","purchaseTime":"...", "purchaseState":"...","purchaseToken":"..."}```
- ```signature``` - On Android it can be used to [consume](https://github.com/AlexDisler/cordova-plugin-inapppurchase#inapppurchaseconsumeproducttype-receipt-signature) a purchase. On iOS it will be an empty string.
- ```productType``` - On Android it can be used to [consume](https://github.com/AlexDisler/cordova-plugin-inapppurchase#inapppurchaseconsumeproducttype-receipt-signature) a purchase. On iOS it will be an empty string.

***Receipt validation:*** - To [validate your receipt](https://alexdisler.com/2016/03/20/validating-cordova-in-app-purchases-on-ios-and-android-using-nodejs/), you will need the ```receipt``` and ```signature``` on Android and the ```receipt``` and ```transactionId``` on iOS.

___Example:___

```js
inAppPurchase
  .buy('prod1')
  .then(function (data) {
    console.log(data);
    /*
      {
        transactionId: ...
        receipt: ...
        signature: ...
      }
    */
  })
  .catch(function (err) {
    console.log(err);
  });
```

### Subscribe

#### inAppPurchase.subscribe(productId)

- ___productId___ - a string of the productId

This function behaves the same as [buy()](https://github.com/AlexDisler/cordova-plugin-inapppurchase#buy) but with subscriptions.

### Consume

#### inAppPurchase.consume(productType, receipt, signature)

- ___productType___ - string
- ___receipt___ - string (containing a json)
- ___signature___ - string

All 3 parameters are returned by the [buy()](https://github.com/AlexDisler/cordova-plugin-inapppurchase#buy) or [restorePurchases()](https://github.com/AlexDisler/cordova-plugin-inapppurchase#inapppurchaserestorepurchases) functions.

Call this function after purchasing a "consumable" product to mark it as consumed.

___NOTE: This function is only relevant to Android purchases.___

On ***Android***, you must consume products that you want to let the user purchase multiple times. If you will not consume the product after a purchase, the next time you will attempt to purchase it you will get the error message:
```Unable to buy item / Item already owned```.

On ***iOS*** there is no need to "consume" a product. However, in order to make your code cross platform, it is recommended to call it for iOS consumable purchases as well.

___Example:___

```js
// fist buy the product...
inAppPurchase
  .buy('consumable_prod1')
  .then(function (data) {
    // ...then mark it as consumed:
    return inAppPurchase.consume(data.productType, data.receipt, data.signature);
  })
  .then(function () {
    console.log('product was successfully consumed!');
  })
  .catch(function (err) {
    console.log(err);
  });
```

### Restore Purchases

#### inAppPurchase.restorePurchases()

If successful, the promise resolves to an array of objects with the following attributes:

- ```productId```
- ```state``` - the state of the product. On ***Android*** the statuses are: ```0 - ACTIVE, 1 - CANCELLED,  2 - REFUNDED```
- ```transactionId```
- ```date``` - timestamp of the purchase
- ```productType``` - On Android it can be used to [consume](https://github.com/AlexDisler/cordova-plugin-inapppurchase#inapppurchaseconsumeproducttype-receipt-signature) a purchase. On iOS it will be an empty string.
- ```receipt``` - On Android it can be used to [consume](https://github.com/AlexDisler/cordova-plugin-inapppurchase#inapppurchaseconsumeproducttype-receipt-signature) a purchase. On iOS it will be an empty string.
- ```signature``` - On Android it can be used to [consume](https://github.com/AlexDisler/cordova-plugin-inapppurchase#inapppurchaseconsumeproducttype-receipt-signature) a purchase. On iOS it will be an empty string.

___Example:___

```js
inAppPurchase
  .restorePurchases()
  .then(function (data) {
    console.log(data);
    /*
      [{
        transactionId: ...
        productId: ...
        state: ...
        date: ...
      }]
    */
  })
  .catch(function (err) {
    console.log(err);
  });
```

See [Differences Between Product Types](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Chapters/Products.html)

### Get Receipt

#### inAppPurchase.getReceipt()

On ***iOS***, you can get the receipt at any moment by calling the getReceipt() function. Note that on iOS the receipt can contain multiple transactions. If successful, the promise returned by this function will resolve to a string with the receipt.

On ***Android*** this function will always return an empty string since it's not needed for Android purchases.

___Example:___

```js
inAppPurchase
  .getReceipt()
  .then(function (receipt) {
    console.log(receipt);
  })
  .catch(function (err) {
    console.log(err);
  });
```

## Known issues
### My getProducts returns an empty array
To solve it, please check the following steps:

- Make sure that your ```manifest.json``` contains the same billing keys as the [Google Play Developer Console](https://play.google.com/apps/publish) *(Your Android Billing Key under the "Services & APIs)*.
- Make sure that you have added the following billing permissions in ```AndroidManifest.xml```:
```xml
<uses-permission android:name="android.permission.BILLING" />
<uses-permission android:name="com.android.vending.BILLING" />
```
- Check that your ```productIds``` is correctly filled, here is an example:
```js
var productIds = ['credit_10', 'credit_20', ...];
```

## Developing

### Build:

    $ npm install
    $ gulp watch

### Run tests:

    $ npm test

Or, if you would like to watch and re-run tests:

    $ npm run watch

Coverage report:

    $ nyc npm test

## Thanks / Credits

- [Adar Porat](https://github.com/aporat)
- [Robot Media](https://github.com/robotmedia)
- [MobileChromeApps](https://github.com/MobileChromeApps)

## More

- [cordova-icon](https://github.com/AlexDisler/cordova-icon) - automatic icon resizing for cordova
- [ng-special-offer](https://github.com/AlexDisler/ng-special-offer) - prompt users to rate your cordova app in the app store
- [ionic-lock-screen](https://github.com/AlexDisler/ionic-lock-screen) - passcode lock screen for ionic (with touch id support for iOS)
- [ionic-zoom-view](https://github.com/AlexDisler/ionic-zoom-view) - an easy way to add a zoom view to images using an ionic modal
- [ng-persist](https://github.com/AlexDisler/ng-persist) - store data on mobile devices (using cordova) that persists even if the user reinstalls the app

## License

The MIT License

Copyright (c) 2016, Alex Disler

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
