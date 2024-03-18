# Overview
The​ ​ eSewa​ ​ Mobile​ ​ SDK​ ​ enable​ ​ to​ ​ easily​ ​ accept​ ​ eSewa​ ​ payments. The​ ​ SDK​ ​ supports​ ​ only​ ​ one​ ​ use​ ​ case​ ​ for​ ​ making​ ​ payment​ –​ Single​ ​Payment​ ​(i.e.​ ​ One​ ​ payment​ ​ per​ ​ one​ ​ user​ ​ log​ ​in).

IMPORTANT: Please make sure your flutter project is migrated to dart sound null safety feature.

# Getting Started

## Android Configuration
### Step-1

- Go to your project's root folder

- Go to android folder > app > src > main > AndroidManifest.xml

inside tag :
```
android:theme="@style/Theme.AppCompat.Light.NoActionBar"
<uses-permission android:name="android.permission.INTERNET"/>
```
```
<application
        android:label="your_app"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar"
        android:icon="@mipmap/ic_launcher">
```
### Step-2

Android Gradle plugin & Kotlin version Update

Since, the new android sdk lib comes packaged with AGP 7.0+, you are required to update your flutter project's gradle version to 7+

- Go to your project's root folder

- Go to android folder > build.gradle

- Add the following:
```
  dependencies {
      classpath 'com.android.tools.build:gradle:7.0.2'
      classpath 'org.jetbrains.kotlin:kotlin-gradle-plugin:1.5.31'
  }
```
- Go to gradle folder > gradle-wrapper.properties

Add the following:
```
  distributionUrl=https\://services.gradle.org/distributions/gradle-7.0.2-all.zip
```
- Sync your gradle

- Done

# How to use SDK for Payment?
## eSewa Config
```
Environment environment  
String      clientId  
String      secretId
```

## NOTE*
Use Environment.live for live credentials & then client Id and secret Key provided by Esewa

Credentials For Test Env
```
CLIENT_ID = JB0BBQ4aD0UqIThFJwAKBgAXEUkEGQUBBAwdOgABHD4DChwUAB0R  

SECRET_KEY = BhwIWQQADhIYSxILExMcAgFXFhcOBwAKBgAXEQ==
```
eSewa Payment
```
String productId : send unique product Id for the product you are paying for

String productName: name of the product

String amount: amount you are paying

String call-back url (optional) : eSewa sends a copy of proof of payment to this URL after successful payment in live environment. Callback URL is a POST request API(Refer to the developer documentation for it's details)

String ebpNo(optional) : This field is required in case of government payments.
```

## eSewa Payment Success Result
```
String productId
String productName
String totalAmount
String environment
String code
String merchantName
String message
String date
String status
String refId
```
## Transaction Verification

```dart

  try {
      EsewaFlutterSdk.initPayment(
        esewaConfig: EsewaConfig(
          environment: Environment.test,
          clientId: CLIENT_ID,
          secretId: SECRET_KEY,
        ),
        esewaPayment: EsewaPayment(
          productId: "1d71jd81",
          productName: "Product One",
          productPrice: "20",
        ),
          onPaymentSuccess: (EsewaPaymentSuccessResult data) {
          debugPrint(":::SUCCESS::: => $data");
          verifyTransactionStatus(data);
        },
        onPaymentFailure: (data) {
          debugPrint(":::FAILURE::: => $data");
        },
        onPaymentCancellation: (data) {
          debugPrint(":::CANCELLATION::: => $data");
        },
      );
    } on Exception catch (e) {
      debugPrint("EXCEPTION : ${e.toString()}");
    }

void verifyTransactionStatus(EsewaPaymentSuccessResult result) async {
    var response = await callVerificationApi(result);
    if (response.statusCode == 200) {
      var map = {'data': response.data};
      final sucResponse = EsewaPaymentSuccessResponse.fromJson(map);
      debugPrint("Response Code => ${sucResponse.data}");
      if (sucResponse.data[0].transactionDetails.status == 'COMPLETE') {
       //TODO Handle Txn Verification Success
        return;
      }
      //TODO Handle Txn Verification Failure
    } else {
      //TODO Handle Txn Verification Failure
    }
  }
```
NOTE : <ebpNo> in EsewaPayment is an optional field; include it if you are making government payment. Your payment without <ebpNo> will be recognized as a normal payment.

## VIA CALLBACK URL
Esewa sends a proof of payment in the callback-URL(if provided)after successful payment in live environment.

## TRANSACTION VERIFICATION API (Recommended Method)
In case of mobile devices, We suggest to use the TXN verification API with txnRefId to verify ur transaction status and check for “status” key in “transactionDetails” object from the response body. “status” => “COMPLETE” means the transaction verification is successful.
```
https://esewa.com.np/mobile/transaction?txnRefId={refId}
```
REQUEST TYPE : GET                                  Headers
```
merchantId : *********************************************

merchantSecret *******************************************

Content-Type : application/json
```
# RESPONSE
```json
[
    {
        "productId": "1999",
        "productName": "Android SDK Payment",
        "totalAmount": "25.0",
        "code": "00",
        "message": {
            "technicalSuccessMessage": "Your transaction has been completed.",
            "successMessage": "Your transaction has been completed."
        },
        "transactionDetails": {
            "date": "Mon Dec 26 12:58:14 NPT 2022",
            "referenceId": "0004VZR",
            "status": "COMPLETE"
        },
        "merchantName": "Android SDK Payment"
    }
]
```
# Payment Failure/Cancellation
In case of payment failure and cancellation, onPaymentFailure and onPaymentCancellation callbacks are triggered which return the error message
For more information regarding various cases of payment failures and cancellation.

# Visit 
```
https://developer.esewa.com.np/#/android?id=error-cases-and-handling
```

