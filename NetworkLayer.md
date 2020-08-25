
# Network Layer Documentation

  * [**1. Core Components:**](#1-core-components)
    * [A. TransportManager:](#a-transportmanager)
    * [B: TransportItem](#b-transportitem)
  * [**2. List of Available Request Response Actions**](#2-list-of-available-request-response-actions)
    * [Few examples](#few-examples)
  * [**3. How to prepare a sample request?**](#3-how-to-prepare-a-sample-request)
    * [RAW JSONs in Request/Response](#raw-jsons-in-requestresponse)
  * [**4. List of API Calls**](#4-list-of-api-calls)
    * [API Request Header Key `mobkey`](#api-request-header-key-mobkey)
    * [4.1 Onboarding:](#41-onboarding)
      * [4.1.1 Validation of onboarding link:](#411-validation-of-onboarding-link)
      * [4.1.2 Link to policy:](#412-link-to-policy)
      * [4.1.3 SMS Verification:](#413-sms-verification)
      * [4.1.4 Confirm Policy](#414-confirm-policy)
      * [4.1.5 Update Firebase FCM Token:](#415-update-firebase-fcm-token)
    * [4.2. Trip Data Upload](#42-trip-data-upload)
    * [4.3. Push Notification Actions:](#43-push-notification-actions)
      * [4.3.1 Trip Report Pull](#431-trip-report-pull)
      * [4.3.1 Weekly/DrivePoints Resport Pull](#432-weeklydrivepoints-report-pull)
    * [4.4. UI Layer API Calls](#44-ui-layer-api-calls)
      * [4.4.1 GET DRIVER LIST FOR CURRENT POLICY](#441-get-driver-list-for-current-policy)
  
  
## 1. Core Components:

### A. TransportManager:
[`TransportManager`](https://github.com/ua108/android_lib/blob/master/shared_lib/src/main/java/com/urbananalytica/android/shared_lib/transport/TransportManager.java) is a component which interacts with the `Server Side API` calls using various methods`. 

**We can prepare and execute multiple API actions using a single API call, the details payload(request body) are encapsulated inside `TransportItem`s array from this incoming `TransportManager`**. 

**TransportManager.java**

```
public class TransportManager {

    
  // 1. Common Key-Value Pairs to be used for all actions(Client Driver Token, 
  //    any Policy Token etc) 
    @SerializedName("KV")
    public HashMap<String, String> KV;

  // 2. An array list of API payloads as `<TransportItem>` for seperate actions
    @SerializedName("I")
    public ArrayList<TransportItem> I;

// Constructor
    public TransportManager()
    {
        this.I = new ArrayList<>();
        this.KV = new HashMap<>();
    }
  
  // Payload to fetch trip report after successful trip upload
   public static void postJobToPullReport(String token, Boolean delay)
    {
      ......
    }

 // Payload to fetch trip report after some delay on a successful trip upload
    public void postTripPullTransportJobWithDelay()
    {
        .......
    }
    
 // Payload to upload a trip
    public void postTransportJob()
    {
       ......
    }

// Payload to upload a trip with a fixed id
    public void postTransportJobFixedJobId(Integer jobId)
    {
       ......
    }
}
```
#### 1. `KV: <String, String>`
This is the common Key-Value map which needs to be sent for authentification/validation for all actions to be requested at server side. This key-value map may contain `client-driver-token`, `policy-token` etc.

#### 2. `I = <TransportItem>`
This array contains jobs OR actions types along with payload or request data. To have an inner look, let try to see the internal skeleton of [`TransportItem `](https://github.com/ua108/android_lib/blob/master/shared_lib/src/main/java/com/urbananalytica/android/shared_lib/transport/TransportItem.java).

### B: TransportItem

[`TransportItem.java`](https://github.com/ua108/android_lib/blob/master/shared_lib/src/main/java/com/urbananalytica/android/shared_lib/transport/TransportItem.java)

```
public class TransportItem: Codable {

   // Action (class name, etc)
   @SerializedName("A")
   public String A; 
   
   // (1) JSON -> gzip -> b64 - supporting 2
   @SerializedName("Z")
   public String Z; 
   
   // JSON -> b64
   @SerializedName("B")
   public String B; 

   public TransportItem(String action, String json, Boolean zipIt) {
        this.A = action;
        this.Z = "";
        this.B = "";

        if (zipIt) {
         // 1. Convert `json` string -> Data
	       // 2. Compress the data 
            this.Z = Base64.encodeToString(TransportItem.compress(json), Base64.NO_WRAP); //base64 encoded string of the compressed data
        } else {
        
          // 1. Base64 encode of incoming `json` and set 
            this.B = Base64.encodeToString(json.getBytes(), Base64.NO_WRAP);
        }
    }
    
    // Constructor
    public TransportItem() {
        this.A = "";
          this.Z = "";
        this.B = "";
    }

  // Decodes and prepares the string value of `JSON` response/request
  public String getJson() {

    // 1. If `B` is NOT EMPTY return base64 decoded string value of `B`
    // 2. Else if `Z` is NOT EMPT
    // 2.1 Decode base64 value of `Z`.
    // 2.2 Unzip the value
    // 2.3 convert unzipped data to string and return
    // 3. Else return empty string "" instead
    
  }
}
```
## 2. List of Available Request Response Actions

[Android Documentation](https://github.com/ua108/android_docs/blob/master/Api/ReqResponseParams.docx)

The following is the list of request/response actions we are using from Android Carpeesh APP.

[TransportItemActions.java](https://github.com/ua108/android_lib/blob/android_dev/shared_lib/src/main/java/com/urbananalytica/android/shared_lib/jobs/TransportItemActions.java)

```
public class TransportItemActions {
   public static String None_minus_1 = "-1";
    // General 0-10
    public static String CreateClientDriverToken_0 = "0" ;// request

    // Testing 11-20
    public static String NonNetworkTests_20 = "20 ";// ?
    public static String GetHello_11 = "11"; // request
    public static String Goodbye_12 = "12"; // response
    public static String DebugList_14 = "14"; // request

    // Other 21-100
    public static String SetDataPointCollection_21 = "21"; // request. note - don't piggy back any other actions. keep this action separate.
    public static String TripSubmitResponse_22= "22"; // response
    public static String SetTripMeta_23 = "23"; // request
    public static String GetEngineOutputClient_24= "24";
    public static String ClientOutput_25 = "25"; // response
    public static String UpdateClientDriverMeta_28 = "28"; // request
    public static String ClientDriverTokenResponse_39= "39"; // response
    public static String SetTripDeleted_40 = "40"; // request
    public static String UpdateFirebaseFCMToken_56 = "56"; // ?
    public static String PushClientDriverCriticalError_58= "58"; // request
    public static String PushClientDriverCriticalErrorNoSNS_59 = "59"; // request
    public static String ClientOutputSilent_42 = "42"; // no notification on the client side.
    public static String GetPointZ_43 = "43";

    public static String TripTokensBatchedResult_62 = "62";
    public static String RequestNextClientOutputBatch_63 = "63";

    public static String SendServerFirebaseFCMToken_70 = "70"; // ?
    public static String FirebaseUpdatedAck_72= "72"; // response
    public static String IncrementClientPullAckCount_81= "81"; // request
    public static String GetCarpeeshPolicyData_82 = "82";
    public static String CarpeeshPolicyData_83= "83";
    public static String SetCarpeeshOnboarded_84 = "84";
    public static String CarpeeshOnboardedAck_85 = "85";
    public static String GetCarpeeshListedDriversData_86 = "86";
    public static String CarpeeshListedDriversData_87 = "87";
    public static String InsertCarpeeshTripData_88 = "88";
    public static String GetCarpeeshWeeklyReport_89 = "89";
    public static String FCMDeliveryAck_90 = "90";
    public static String CarpeeshWeeklyReport_91 = "91";

    // Error 101-201 (handled errors)
    public static String ErrorNoTripToken_101 = "101"; // reponse
    //public static String PointzNull                       = 103 // response - NOT IN USE
    //public static String DatabaseError                    = 104 // response - NOT IN USE
    public static String EngineError_105 = "105"; // response
    public static String ThisActionWillSelfDestruct_107 = "107"; // request. this will generate a server side error - useful for testing client handling.
    public static String CarpeeshPolicyDataNotFound_108= "108";
    public static String SetCarpeeshOnboardedError_109 = "109";
    public static String CarpeeshListedDriversDataNotFound_110 = "110";
    public static String CarpeeshWeeklyReportError_111 = "111";
}
```
### Few examples
1. `SetDataPointCollection/REQUEST ACTION CLIENT -> SERVER`: used to upload trip data `request` action
2. `SetCarpeeshOnboarded/REQUEST ACTION CLIENT -> SERVER`: Acknowledge server that onboading is completed.
3. `TripSubmitResponse/RESPONSE SERVER -> CLIENT`: Servers acknowledges of receiving one trip data.
4. `ClientOutput/RESPONSE SERVER -> CLIENT`: server sends the trip report to mobile client

## 3. How to prepare a sample request?
In this section let's dicuss how to prepare one API request. We are going to discussion the API data exchange for pulling down the weekly report for a policy. This API call is initiated mobile client receives one PUSH NOTIFICATION with action `CarpeeshWeeklyReport`. The following code snippet demonstrates how to prepare the following API call to perform two actions in a single request.

```
// 1. Extract auth info from incoming push notification payload
 String delToken = data.get("DELIVERY_TOKEN");
 String polToken = data.get("POLICY_TOKEN");
 String clientDriverToken = data.get("CLIENT_DRIVER_TOKEN");
    
// 2.1 FCM Delivery Transport Item Preparation
 TransportItem ti2 = new TransportItem("FCMDeliveryAck", "", false);
 
// 2.2 Get CarpeeshWeeklyReport Transport Item Preparation
  TransportItem ti3 = new TransportItem("GetCarpeeshWeeklyReport", "", false);

// 3. ZIP the above two actions inside single `TransportManager` item    
  TransportManager tm = new TransportManager();
  tm.I.add(ti2);
  tm.I.add(ti3);
  tm.KV.put("DeliveryToken", delToken);
  tm.KV.put("CarpeeshPolicyToken", polToken);
  tm.KV.put("ClientDriverToken", clientDriverToken);
                          
// 4. Call API
tm.postTransportJob();

The RAW request body JSON may look like this
                                  
```

### RAW JSONs in Request/Response
##### REQUEST
```
{
	"I": [{
		"A": "FCMDeliveryAck",
		"B": "",
		"Z": ""
	}, {
		"A": "GetCarpeeshWeeklyReport",
		"B": "",
		"Z": ""
	}],
	"KV": {
		"DeliveryToken": "DELIVERY_TOKEN",
		"CarpeeshPolicyToken": "CARPEESH_POLICY_TOKEN",
		"ClientDriverToken": "CLIENT_DRIVER_TOKEN"
	}
}
```
##### RESPONSE
```
{
	"I": [{
		"A": 90,
		"B": "",
		"E": "",
		"K": "",
		"Z": ""
	
	},
	{
		"A": 91,
		"B": "BASE_64_ENCODED_JSON_RESPONSE==",
		"E": "",
		"K": "",
		"Z": ""
	}],
	"KV": {
		"CarpeeshClientDriverToken": "CLIENT_DRIVER_TOKEN"
	}
}

// SAMPLE DECODED VALUE FOR "B" WITH ACTION 91
{
	"Driving": {
		"DistanceMeters": 0,
		"FeedbackMsg": "Keep up the good driving. Extremely harsh aceleration detected; these events increase your crash risk.",
		"FeedbackMsg2": null,
		"HAScore": 0,
		"HBScore": 15,
		"PolicyToken": "POLICY_TOKEN",
		"SPScore": 100,
		"TODScore": 100,
		"TotalScore": 61
	},
	"GenerationUTC": "2020-07-27T12:43:07.7517561Z"
}
```


## 4. List of API Calls

### API Request Header Key `mobkey`
For it's value refer to `KeysToken.plist` document in `1Password`.

![KeysTokensPlist.png](https://github.com/ua108/ios_shared_pod/blob/develop/Documentation/DocUnderRapidCircle/screenshots/KeysTokensPlist.png)



### 4.1. Onboarding:
#### 4.1.1 Validation of onboarding link:
Typical onboarding link tapped by an user: [https://app.carpeesh.com/BfoBiEZciFrkw2MQA](https://app.carpeesh.com/BfoBiEZciFrkw2MQA)

The details of incoming dynamic link retrieved using [Firebase Dynamic Link API](https://firebase.google.com/docs/reference/android/com/google/firebase/dynamiclinks/FirebaseDynamicLinks#public-abstract-taskpendingdynamiclinkdata-getdynamiclink-uri-dynamiclinkuri`)

We receive one full url from last step. 
```
app.carpeesh.com://google/link/?deep_link_id=https://link.carpeesh.com?d=eyJUT0tFTiI6IkZ6S1JwTXRKU3F....1PQklMRSI6Iis2MTQyMjkzNzE0NCJ9&match_type=unique
```
We extract query param value for `deep_link_id ` and then check value for another field `d` inside deep link.

We get the following `JSON` after base64 decoding.

```
{
	"TOKEN": "VDXJ-....-Z5RK",
	"MOBILE": "+614....7932"
}
```
An onboarding link(`https://app.carpeesh.com/BfoBiEZ*****kw2MQA`) is validated if we find `TOKEN` inside this `JSON`. This token will be used while liking this policy.


#### 4.1.2 Link to policy:

[Android Documentation](https://github.com/ua108/android_docs/blob/master/Api/LinkPolicyRequest.docx)

![sidemeu_driver_list.PNG](https://github.com/ua108/android_docs/blob/master/Screenshots/Onboarding1.JPG)
[<img src="https://github.com/ua108/android_docs/blob/master/Screenshots/onboarding_!.jpg" width = "266" height="466"/>](https://github.com/ua108/android_docs/blob/master/Screenshots/onboarding_!.jpg)

```
END POINT
---------
URL: https://uapi.drivedq.com/mob/post
METHOD: POST

HEADERS:
-------
"mobkey":"yKJB5.....ptJma6H"
"CFBundleShortVersionString":"93" <--- Build number
"Content-Type":"application/json"

REQUEST BODY:
------------
{
	"I": [{
		"A": 82,
		"Z": "",
		"B": ""
	}],
	"KV": {
		"InstallToken": "VDXJ-....-Z5RK"
	}
}
SUCCESS RESPONSE BODY:
----------------------
{
	"I": [{
		"A": 83,
		"B": "eyJJSFFEcml2ZXJHVUlE...LVo1UksifQ==",
		"E": "",
		"K": "",
		"Z": ""
	}],
	"KV": {
		"CarpeeshClientDriverToken": "JTqDR........drYscN"
	}
}
Base 64 Decoded Value for Field `B`:
------------------------------------
{
	"IHQDriverGUID": "TU93V***********lpsMjFBZz09",
	"ClientDriverToken": "JTqDRD*******rYscN",
	"FirstName": "First",
	"LastName": "Last",
	"Mobile": "+614********",
	"IHQPolicyGUID": "SFR6Y3dZ*********bkxZMDRYUT09",
	"PolicyToken": "Y27at*********T9nirWE",
	"PolicyNumber": "CAR200*****",
	"CarRego": "******",
	"MakeModel": "",
	"ClaimsURL": "https://carpeesh-uat.insuredhq.com/tiles/update_client_details.php?OTAzWUU0ZHRDcl***URacDg9",
	"PolicyURL": "https://carpeesh-uat.insuredhq.com/tiles/?OTAzWUU0******0Q3Y1B4RT0=",
	"IsPolicyHolder": "True",
	"InstallToken": "VDXJ-....-Z5RK"
}

```
#### 4.1.3 SMS Verification:
[<img src="https://github.com/ua108/android_docs/blob/master/Screenshots/phone_verification.jpg" width = "266" height="466"/>](https://github.com/ua108/android_docs/blob/master/Screenshots/phone_verification.jpg)
##### 4.1.3.1 GET OTP
We use [Firebase Authentication](https://firebase.google.com/docs/auth/android/phone-auth#send-a-verification-code-to-the-users-phone) to send OTP you phone.

##### 4.1.3.2 VERIFY OTP
The given OTP is verified using [Firebase Auth API](https://firebase.google.com/docs/auth/android/phone-auth#sign-in-the-user).

#### 4.1.4 Confirm Policy:
[Android Documentation](https://github.com/ua108/android_docs/blob/master/Api/ConfirmPolicyDetails.docx)

[<img src="https://github.com/ua108/android_docs/blob/master/Screenshots/confirm_details.jpg" width = "266" height="466"/>](https://github.com/ua108/android_docs/blob/master/Screenshots/confirm_details.jpg)

```
END POINT
---------
URL: https://uapi.drivedq.com/mob/post
METHOD: POST

HEADERS:
-------
"mobkey":"yKJB5.....ptJma6H"
"CFBundleShortVersionString":"79" <--- Build number
"Content-Type":"application/json"

REQUEST BODY:
------------
{
	"I": [{
		"A": SetCarpeeshOnboarded,
		"Z": "",
		"B": ""
	}],
	"KV": {
		  "Platform":"android_au",
      "FirebaseUID":"i7yzfE3Gy*****lLaSRgIY2",
      "ClientDriverToken":"JVZ3UCu*****GKCrV"
	}
}
SUCCESS RESPONSE BODY:
----------------------
{
	"I": [{
		"A": 85,
		"B": "",
		"E": "",
		"K": "",
		"Z": ""
	}],
	"KV": {}
}
```
#### 4.1.5 Update Firebase FCM Token:

##### EVENTS
1. On policy confirmation success.
2. Trip ended and before uploading Trip Data.
3. Once device registers to FCM and receives that token.

```
END POINT
---------
URL: https://uapi.drivedq.com/mob/post
METHOD: POST

HEADERS:
-------
"mobkey":"yKJB5.....ptJma6H"
"CFBundleShortVersionString":"93" <--- Build number
"Content-Type":"application/json"

REQUEST BODY:
------------
 {
 	"I": [{
 		"A": 56,
 		"Z": "",
 		"B": "WwogICJKV******VGkzZmRyWXNjTiIKXQ=="
 	}],
 	"KV": {
 		"FirebaseFCMToken": "edhHrRMM6_o:APA91bEKOH0_*******-dSF3i2rTptWMkrg"
 	}
 }
---------------------------------------------------------------------------------  
NOTE: `B` in above JSON is base64 encoded array of client tokens of drivers 
who are currently onboaded on this device.
--------------------------------------------------------------------------------- 
[
  "JTqDR*****i3fdrYscN"
]
--------------------------------------------------------------------------------- 

SUCCESS RESPONSE BODY:
---------------------
{
	"I": [{
		"A": 85,
		"B": "",
		"E": "",
		"K": "",
		"Z": ""
	}],
	"KV": {}
}
```

### 4.2. Trip Data Upload

Data taken from [Android Documentation](https://github.com/ua108/android_docs/blob/master/Api/UploadTrip.docx)

##### EVENTS:
1. On trip end.
2. Background Refresh(searches for an old `TripDelivery` which is not uploaded yet)
3. App becomes active(searches for an old `TripDelivery` which is not uploaded yet)


```
END POINT
---------
URL: https://uapi.drivedq.com/mob/post
METHOD: POST

HEADERS:
-------
"mobkey":"yKJB5.....ptJma6H"
"CFBundleShortVersionString":"93" <--- Build number
"Content-Type":"application/json"

REQUEST BODY:
------------
{
	"I": [{
		"A": "SetDataPointCollection",
		"B": "",
		"Z": "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\u003d\u003d"
	}],
	"KV": {
		"TripSeconds": "141",
		"TripClientGUID": "e680c602-****-****-****-*****789",
		"StartReason": "ab",
		"ClientDriverToken": "JV********CrV",
		"TotalPoints": "151",
		"StopReason": "il"
	}
}
------------------------------------------------------------------------------
NOTE: `Z` in above JSON is base64 encoded and then zipped trip data point
collection details. The following JSON is the decoded value for that `Z`
------------------------------------------------------------------------------
{
	"Accuracys": [],
	"DPAMeta": {
		"AutoStarted": false,
		"ELogs": ["16-07 :: 15.15.30.688 [TripRecordingService] gone into foreground", "16-07 :: 15.15.30.725 [UAAppClass] requestActivityRecognitionUpdates onSuccess", "16-07 :: 15.15.30.771 [ActivityDetectionService] (STILL 100%)  ( |  POINTS:-150 TP:-150 TH:165", "16-07 :: 15.15.44.628 [TripRecordingService] GPS_EVENT_STOPPED", "16-07 :: 15.15.46.636 [TripRecordingService] first real location detected", "16-07 :: 15.15.46.938 [ReverseGeocodeJobService] onStartJob", "16-07 :: 15.15.47.498 [ReverseGeocodeJobService] r-geo-coded start | Queensberry Street, Carlton", "16-07 :: 15.16.09.129 [ActivityDetectionService] (STILL 100%)  (m,m,m,m,m) |  POINTS:-150 TP:-150 TH:165", "16-07 :: 15.16.17.803 [ActivityDetectionService] (STILL 100%)  (m,m,m,m,m) |  POINTS:-150 TP:-450 TH:165", "16-07 :: 15.19.04.826 [TripRecordingService] slow gps updates and little distance covered \u0026 old or ads total points \u003c\u003d 0. meters \u0026 timeDiff: | 91.71716,60333", "16-07 :: 15.19.13.080 [TripRecordingService] onDestroy StopReason: IDLE_THEN_LOST_GPS_SIGNAL", "16-07 :: 15.19.13.149 [TripRecordingService] BAT_START:100% BAT_STOP:100% | LIFETIME:3 min, 42 sec", "16-07 :: 15.19.13.188 [UAAppClass] stopBeaconRangeScanning unbind", "16-07 :: 15.19.13.249 [TripRecordingService] trip OK so far, but trimmed end | SIZE:151 INDEX1:150 INDEX2:148 INDEX3:151", "16-07 :: 15.19.15.282 [TripRecordingService] au bounded", "16-07 :: 15.19.15.311 [TripRecordingService] trip ok, will queue for delivery", "16-07 :: 15.19.20.636 [TripRecordingService] ClientDriverToken: JVZ3UCubJby6GdUGKCrV"],
		"KV": {
			"BuildVersion": "1",
			"LastLonLat": "",
			"StartReason": "ab",
			"StopReason": "il"
		},
		"LastLonLat": "",
		"RecNotes": [],
		"SLogs": ["16-07 :: 15.15.10.332 [UAAppClass] app started", "16-07 :: 15.15.10.891 [MyApplication] MyApplication called", "16-07 :: 15.15.13.887 [OptimizeTripDetectionService] startService", "16-07 :: 15.15.22.448 [ActivityTransitionBroadcastReceiver] Transition update set up", "16-07 :: 15.15.28.447 [BluetoothTransitionBroadcastReceiver] onReceive", "16-07 :: 15.15.28.526 [BluetoothTransitionBroadcastReceiver] ACTION_ACL_CONNECTED", "16-07 :: 15.15.30.368 [TripRecordingService] onCreate", "16-07 :: 15.15.30.631 [TripRecordingService] onStartCommand action: AUTO_START_VIA_BT"],
		"TLogs": ["16-07 :: 15.15.44.628 [TripRecordingService] GPS_EVENT_STOPPED", "IS OPTIMIZED IN FOREGROUND"]
	},
	"Lats": [],
	"Longs": [],
	"Utcs": [],
	"Velos": [],
	"Weather": [],
	"ZAccuracys": [14, 0, 0, 0, 0, 0, -1, 0, 0, 0, 0, 0, -3, 0, 0, 0, -4, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 4, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, -2, 0, 0, 0, 0, 0, 4, 0, 0, 0, 0, 0, -2, 0, 0, 0, -1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, -3, 0, 1, 0, 0, 0, 1],
	"ZLats": [-37804240, -127, -141, -140, -141, -162, -162, -162, -162, -184, -184, -188, -189, -101, -28, -27, -27, -26, -27, -21, -23, -24, -23, -24, -24, -23, -25, -27, -26, -25, -24, -22, -21, -19, -20, -21, -20, -24, -24, -26, -25, -26, -25, -22, -21, -19, -19, -17, -18, -17, -18, -17, -17, -18, -18, -18, -18, -20, -18, -18, -18, -21, -21, -19, -20, -19, -19, -20, -22, -25, -22, -20, -19, -20, -20, -21, -22, -23, -23, -20, -25, -25, -25, -22, -22, -21, -21, -23, -22, -26, -26, -23, -23, -25, -24, -22, -22, -24, -24, -26, -26, -28, -29, -29, -30, -30, -30, -31, -33, -30, -29, -31, -32, -30, -33, -31, -31, -31, -27, -28, -27, -30, -29, -31, -31, -27, -27, -28, -28, -23, -23, -23, -23, -25, -25, -26, -26, -26, -27, -30, -31, -32, -30, -32, -32, -33, -33, -35, -35, -13, 23],
	"ZLongs": [144960353, -4, -26, -26, -26, -31, -30, -30, -31, -34, -34, -35, -35, 129, 253, 240, 240, 241, 240, 215, 214, 215, 214, 225, 224, 219, 218, 218, 218, 211, 210, 183, 182, 177, 177, 186, 186, 212, 213, 229, 229, 225, 226, 195, 196, 168, 169, 156, 157, 156, 157, 156, 157, 162, 163, 163, 163, 156, 157, 174, 174, 198, 198, 189, 189, 180, 181, 196, 196, 195, 196, 196, 197, 196, 196, 216, 215, 231, 231, 230, 229, 229, 229, 201, 200, 190, 190, 201, 201, 231, 231, 203, 203, 225, 225, 200, 201, 225, 225, 252, 253, 274, 274, 286, 286, 290, 290, 308, 308, 286, 286, 303, 303, 286, 286, 275, 274, 268, 268, 271, 271, 287, 288, 301, 302, 273, 273, 272, 273, 252, 252, 233, 232, 254, 255, 266, 267, 252, 253, 279, 280, 290, 291, 306, 306, 319, 319, 345, 345, 346, 3],
	"ZUtcs": [1594892746711, 117, 64, 15, 22, 19, 13, 24, 8, 642, 854, 1015, 1994, 32, 959, 1013, 977, 1007, 4558, 10, 9, 9, 419, 1040, 974, 1011, 1023, 954, 1017, 993, 987, 1006, 1008, 1021, 976, 1020, 977, 1011, 1000, 990, 1054, 956, 1003, 1006, 1005, 990, 1001, 1003, 998, 1007, 984, 1062, 942, 1019, 979, 1016, 993, 1082, 928, 981, 1037, 956, 1044, 978, 994, 998, 1002, 1008, 1013, 984, 1009, 1012, 988, 1001, 985, 1009, 1001, 991, 1014, 1055, 966, 987, 1003, 986, 1009, 986, 1017, 995, 995, 1009, 1028, 953, 1022, 1020, 979, 976, 1008, 1016, 985, 1006, 988, 1012, 1010, 1019, 994, 1008, 959, 1018, 1019, 998, 981, 1003, 999, 1007, 983, 1012, 995, 1020, 972, 998, 1002, 990, 1050, 961, 1032, 973, 1010, 1005, 999, 974, 1011, 1026, 1003, 987, 980, 1011, 1003, 1015, 989, 1000, 1008, 989, 1012, 993, 993, 1021, 998, 1016, 992, 1005, 971],
	"ZVelos": [0, 158, 0, 0, 0, 24, 0, 0, 0, 25, 0, 5, 0, 12, 0, -11, 0, 0, 0, -23, 0, 0, 0, 9, 0, -5, 0, 0, 0, -7, 0, -25, 0, -5, 0, 8, 0, 24, 0, 14, 0, -3, 0, -26, 0, -25, 0, -10, 0, 0, 0, 0, 0, 5, 0, 1, 0, -6, 0, 15, 0, 22, 0, -8, 0, -8, 0, 14, 0, 0, 0, 0, 0, 0, 0, 17, 0, 14, 0, -2, 0, 0, 0, -25, 0, -10, 0, 10, 0, 27, 0, -25, 0, 19, 0, -21, 0, 21, 0, 25, 0, 19, 0, 11, 0, 3, 0, 16, 0, -20, 0, 16, 0, -15, 0, -11, 0, -5, 0, 2, 0, 15, 0, 12, 0, -25, 0, 0, 0, -19, 0, -17, 0, 19, 0, 11, 0, -12, 0, 24, 0, 10, 0, 13, 0, 12, 0, 23, 0, 0, 0]
}
--------------------------------------------------------------------------------- 

SUCCESS RESPONSE BODY:
---------------------
{
	"I": [{
		"A": 22,
		"B": "eyJPdGhlckluZm8iOl*****0ZWQiOnRydWV9",
		"E": "",
		"K": "",
		"Z": ""
	}],
	"KV": {}
}

------------------------------------------------------------------------------------------------
NOTE: `B` in above JSON is base64 encoded JSON Response. The decoded JSON value is the following
------------------------------------------------------------------------------------------------
{
	"OtherInfo": [],
	"TripToken": "Etzf********CAx8",
	"ClientDriverToken": "JVZ3********UGKCrV",
	"Validated": true
}

```


### 4.3. Push Notification Actions:
#### 4.3.1 Trip Report Pull

[Android Documentation](https://github.com/ua108/android_docs/blob/master/Api/PullTriprequest.docx)

##### EVENT: 
Once uploaded trip data is processed in server side, mobile client receives a PUSH NOTIFICATION with action `PullTrip`.
Then mobile client extracts `TRIP_TOKEN` info from Push Notification Payload and initiates the following API call.

```
END POINT
---------
URL: https://uapi.drivedq.com/mob/post
METHOD: POST

HEADERS:
-------
"mobkey":"yKJB5.....ptJma6H"
"CFBundleShortVersionString":"93" <--- Build number
"Content-Type":"application/json"

REQUEST BODY:
------------
{
	"I": [{
		"A": 24,
		"Z": "",
		"B": ""
	}],
	"KV": {
		"TripToken": "uUZ4x*****Cr2jtw"
	}
}

SUCCESS RESPONSE BODY:
---------------------
{
	"I": [{
		"A": 42,
		"B": "",
		"E": "",
		"K": "",
		"Z": "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"
	}],
	"KV": {
		"TripToken": "uUZ4x********r2jtw"
	}
}
Decoded Value for Field `B`:
------------------------------------
{
	"ClientReport": {
		"IntProps": {
			"TripTimeEfficiencyPercent": 78,
			"IdleTimeTotalSeconds": 0,
			"IdleTimePercent": 0,
			"DurationSecs": 126,
			"DistanceMeters": 1000,
			"PostedSpeedLimitCoveragePercent": 100,
			"IntersectionCount": 3,
			"MergeCount": 0,
			"RoundaboutCount": 5,
			"EstimatedPercentOfTripAnalysed": 100,
			"NightPercent": 0,
			"UrbanPercent": 100,
			"IsKMH": 1,
			"ScoreModCode": 0
		},
		"RoadTypes": ["100% residential"],
		"StringProps": {
			"WayCollectionB64": "H4sIAAAAAAAEAEWQyWrDMBCGXyXoPIVotPtaFxrSlkBCeyg+GCKCQJGNpRZC8Lt3lKVBywzf/P9oObM25MKab6mWIDigkyAQkAswS01EdMBWeb15ZU2Zfjyw7ej9/i0cQ8lkI9d1PnG45+TYlimkw2YaRhKd2fqz7pw17KuP4xD9ggTeFwYMCU4+h71PJfSRiCBCQVJ4OcSQypAeckX0vQ9p0U7h1xPQBJ6HSLp8V80z0DmnVVsviBYRjauPk1orjfw/qwyt4dop4EJJq40yIKyQXFh3Sxx2l24f/dHXfmSHuiQo0NfS7jReSvRtj9HNfxJOBNtbAQAA",
			"StartLocation": "Walpole Street Kew",
			"StartOffset": "+10:00",
			"StartDateTime": "2020-08-15 15:05:17Z",
			"EndLocation": "Collins Street Kew",
			"EndOffset": "+10:00",
			"EndDateTime": "2020-08-15 15:07:23Z",
			"Score": "4.2",
			"SpeedingScore": "5.0",
			"AccelScore": "0.9",
			"DecelScore": "5.0",
			"TODScore": "5.0",
			"ScoreConfidence": "100%",
			"StopReason": "lp",
			"StartReason": "bm",
			"IssueSummaryV1": null,
			"IssueSummaryLine1": "Hard acceleration was detected very often",
			"IssueSummaryLine2": "",
			"WeatherConditions": ""
		},
		"Suburbs": ["100% Kew"]
	},
	"Glg": {
		"GeoLayers": [{
			"CProps": {
				"Dist": [999.53]
			},
			"DProps": {},
			"GLines": ["zrveFcpetZqI_AQH]SsNcBYBDCKR[tEYtEENGFIn@KJMCa@j@a@~@IXI\\O`BO|AGpAm@I"],
			"HProps": {},
			"Name": "TripHeader",
			"TProps": {}
		}, {
			"CProps": {
				"EstDist": [989.7],
				"EstDur": [91.2]
			},
			"DProps": {
				"RouteSummary": [10]
			},
			"GLines": ["zrveFyoetZ{H}@]?EDEd@q@nL_NwA_@CeABEFO@a@^INe@hAMf@QxAY`Ek@K"],
			"Name": "AlternativeRoute",
			"TProps": {}
		}, {
			"CProps": {
				"TimeEfficiencyRatio": [100, 83, 99, 83, 100, 51, 100, 64],
				"AvgSegmentSpeed": [12.1, 8.7, 8.3, 8.4, 8.8, 7.4, 9.5, 7.6]
			},
			"DProps": {},
			"GLines": ["zrveFcpetZQCWCUCQCUCSCYCYEOCWC", "blveFmqetZQCSCUCMAQCKAQAOHA?CA", "rgveF{qetZYQMCMAMCOCMAKAOCOAKA", "xbveFgsetZKCKAOAIAKCOAOCKASCIAMAQCKAQCSCQAOCSCQCOAOCOCOAGAC?EAYBBA@AKR", "dvueF}tetZ?HAHC\\CXEp@CZCVAN", "juueF{netZARABA`@ARALARAB?JANCTAJAPAJ?J?FADAFCF??", "htueFagetZGFIn@KJMCQVORGNCFIPKT", "xpueFuaetZIXI\\EZATEVAVCVCVALABANAJALAT?HAJ?NAFC@A?ICIAGAAAE?CA"],
			"Name": "TripTimeEfficiency",
			"TProps": {}
		}, {
			"CProps": {
				"PostedSpeed": [50, 50]
			},
			"DProps": {},
			"GLines": ["zrveFcpetZqI_AQH]SMCeN_BYB", "dvueF}tetZ[tEYtEENGFIn@KJMCa@j@a@~@IXI\\O`BO|AGpAm@I"],
			"Name": "SpeedDataAvailability",
			"TProps": {}
		}, {
			"CProps": {
				"TurnEntryCSV": ["710,600,600,610,450"],
				"TurnCSV": ["300,230"],
				"TurnExitCSV": ["310,330,370,310,320"],
				"ScoreDescs": ["141"],
				"ScoreType": [8]
			},
			"DProps": {
				"ManeuverGroupType": [23],
				"ManeuverType": [26],
				"ManeuverModifier": [27],
				"StartLocation": [28],
				"EndLocation": [29]
			},
			"GLines": ["`oueFyxdtZCl@i@G"],
			"Name": "ManeuverWindow",
			"TProps": {}
		}],
		"Lu": {
			"KV": {
				"10": "Princess Street, Main Drive",
				"23": "TurnRight",
				"26": "Turn",
				"27": "Right",
				"28": "Main Drive",
				"29": ""
			}
		}
	},
	"TripToken": "uUZ4xz*********r2jtw"
}
```



#### 4.3.2 Weekly/DrivePoints Report Pull

Data taken from [Android Documentation](https://github.com/ua108/android_docs/blob/master/Api/PullWeeklyReport.docx)

##### EVENT: 
Mobile client receives a PUSH NOTIFICATION with action `CarpeeshWeeklyReport`. Then mobile client extracts 
`DELIVERY_TOKEN`, `POLICY_TOKEN` & `CLIENT_DRIVER_TOKEN` info from Push Notification payload and initiates 
the following API call.

![WeeklyReport.png](https://github.com/ua108/android_docs/blob/master/Screenshots/drive_points.jpg)

```
END POINT
---------
URL: https://uapi.drivedq.com/mob/post
METHOD: POST

HEADERS:
-------
"mobkey":"yKJB5.....ptJma6H"
"CFBundleShortVersionString":"79" <--- Build number
"Content-Type":"application/json"

REQUEST BODY:
------------
{
	"I": [{
		"A": "FCMDeliveryAck",
		"B": "",
		"Z": ""
	}, {
		"A": "GetCarpeeshWeeklyReport",
		"B": "",
		"Z": ""
	}],
	"KV": {
		"DeliveryToken": "AZE*****W5BhB",
		"CarpeeshPolicyToken": "onY9d*****TMAme",
		"ClientDriverToken": "JVZ3******GKCrV"
	}
}

SUCCESS RESPONSE BODY:
-----------------------
{
	"I": [{
		"A": 91,
		"B": "eyJEcml2aW5nIjp7IkR*****2WiJ9",
		"E": "",
		"K": "",
		"Z": ""
	}],
	"KV": {
		"CarpeeshClientDriverToken": "xopmFh*******65WZEW"
	}
}
Decoded Value for Field `B`(base64 decode):
------------------------------------------
{
	"Driving": {
		"DistanceMeters": 0,
		"FeedbackMsg": "There are no barriers to you becoming a great driver. Extremely harsh aceleration detected; these events increase your crash risk.",
		"FeedbackMsg2": null,
		"HAScore": 13,
		"HBScore": 40,
		"PolicyToken": "Jdc854*****MBR",
		"SPScore": 100,
		"TODScore": 77,
		"TotalScore": 68
	},
	"GenerationUTC": "2020-07-28T01:30:38.3029106Z"
}
```


### 4.4. UI Layer API Calls

#### 4.4.1 GET DRIVER LIST FOR CURRENT POLICY

[Android Documentation](https://github.com/ua108/android_docs/blob/master/Api/GetPolicyHolderDetails.docx)

##### EVENTS:
1. Opening Left Side Menu(every 10th time to refresh unonboarded driver count)
2. User opens listed driver screen


![sidemeu_driver_list.PNG](https://github.com/ua108/android_docs/blob/master/Screenshots/side_bar.jpg)
![list_of_drivers.PNG](https://github.com/ua108/android_docs/blob/master/Screenshots/driver_details.jpg)

```
END POINT
---------
URL: https://uapi.drivedq.com/mob/post
METHOD: POST

HEADERS:
-------
"mobkey":"yKJB5.....ptJma6H"
"CFBundleShortVersionString":"93" <--- Build number
"Content-Type":"application/json"

REQUEST BODY:
------------
{
	"I": [{
		"A": 86,
		"Z": "",
		"B": ""
	}],
	"KV": {
		"ClientDriverToken": "JTqDRD7q******YscN",
		"CarpeeshPolicyToken": "Y27at4******nirWE"
	}
}

SUCCESS RESPONSE BODY:
---------------------
{
	"I": [{
		"A": 87,
		"B": "W3siRmlyZWJhc2VEeW5hbWljTGluayI6I****VGkzZmRyWXNjTiJ9XQ==",
		"E": "",
		"K": "",
		"Z": ""
	}],
	"KV": {
		"CarpeeshClientDriverToken": "JTqDRD7q******YscN"
	}
}
Base 64 Decoded Value for Field `B`:
------------------------------------
[
  {
     "FirebaseDynamicLink":"https://app.carpeesh.com/okvApskk3C7wzS916",
     "FirstName":"Jutta",
     "LastName":"Chatto",
     "MobileNumber":"+61422397144",
     "Onboarded":"False",
     "IsPolicyHolder":"False",
     "ClientDriverToken":"TLmLQ*****V3oGLa"
  },
  {
     "FirebaseDynamicLink":"https://app.carpeesh.com/mkBFwkfFwM2zm1x16",
     "FirstName":"Ben",
     "LastName":"30 March SQS - 2",
     "MobileNumber":"+61422397144",
     "Onboarded":"True",
     "IsPolicyHolder":"False",
     "ClientDriverToken":"P2KUUj6*****3ZSUHL"
  },
  {
     "FirebaseDynamicLink":"https://app.carpeesh.com/zoeojFzhHaPmGAqq5",
     "FirstName":"Harry",
     "LastName":"Potter",
     "MobileNumber":"+61433477932",
     "Onboarded":"True",
     "IsPolicyHolder":"True",
     "ClientDriverToken":"JVZ3UCu*****GKCrV"
  }
]


```
