Branch io Adobe AIR Native Extension for iOS & Android.

Installation
============

Download the raw files
----------------------
Download the latest version with source code from this Git repository [https://github.com/BranchMetrics/Branch-AIR-ANE-SDK/archive/master.zip](https://github.com/BranchMetrics/Branch-AIR-ANE-SDK/archive/master.zip) you can also clone it.

Import the ANE
--------------
Import the `Branch.ane` into your project. Depending your IDE you might need to import the `Branch.swc` as well.  
Inside your `*-app.xml` be sure to add this line `<extensionID>io.branch.nativeExtensions.Branch</extensionID>`

Register you app
----------------
You can sign up for your own app id at [https://dashboard.branch.io](https://dashboard.branch.io)

Special settings on iOS
-----------------------
Inside the `*-app.xml` you must add **your Branch App Key** (refer to the [dashboard](https://dashboard.branch.io) to get it).
```xml
<iPhone><InfoAdditions><![CDATA[
	<!-- other stuff -->
	<key>branch_key</key>
	<string>key_live_dcixJiAqOixZkdkLxgiTLkeovycqdUPp</string>
]]></InfoAdditions></iPhone>
```
For a full example of the `*-app.xml` please refer to the [demo](https://github.com/BranchMetrics/Branch-AIR-ANE-SDK/blob/master/bin/Branch-AIR-ANE-SDK-app.xml).

Special settings on Android
---------------------------
Inside the `*-app.xml` you must add **your Branch App Key** (refer to the [dashboard](https://dashboard.branch.io) to get it). And also don't forget to set the Branch activity:
```xml
<android><manifestAdditions><![CDATA[
	<!-- other stuff -->
	<application>
		<meta-data android:name="io.branch.sdk.BranchKey" android:value="key_live_dcixJiAqOixZkdkLxgiTLkeovycqdUPp" />
		<activity android:name="io.branch.nativeExtensions.branch.BranchActivity" android:launchMode="singleTask" android:theme="@android:style/Theme.Translucent.NoTitleBar">
		</activity
	</application>
]]></manifestAdditions></android>
```
For a full example of the `*-app.xml` please refer to the [demo](https://github.com/BranchMetrics/Branch-AIR-ANE-SDK/blob/master/bin/Branch-AIR-ANE-SDK-app.xml).

Configuration (for tracking)
----------------------------
Ideally, you want to use our links any time you have an external link pointing to your app (share, invite, referral, etc) because:

1. Our dashboard can tell you where your installs are coming from
1. Our links are the highest possible converting channel to new downloads and users
1. You can pass that shared data across install to give new users a custom welcome or show them the content they expect to see

Our linking infrastructure will support anything you want to build. If it doesn't, we'll fix it so that it does: just reach out to alex@branch.io with requests.

Register a URI scheme direct deep linking (optional but recommended)
--------------------------------------------------------------------
In your project's `*-app.xml` file, you can register your app to respond to direct deep links (yourapp:// in a mobile browser) by adding a URI scheme. Also, make sure to change **yourApp** to a unique string that represents your app name.  
On iOS:
```xml
<key>CFBundleURLTypes</key>
<array>
	<dict>
		<key>CFBundleURLSchemes</key>
		<array>
			<string>yourApp</string>
		</array>
	</dict>
</array>
```
On Android:
```xml
<activity android:name="io.branch.nativeExtensions.branch.BranchActivity" android:launchMode="singleTask" android:theme="@android:style/Theme.Translucent.NoTitleBar">
	<intent-filter>
		<data android:scheme="yourApp android:host="open" />
		<action android:name="android.intent.action.VIEW" />
		<category android:name="android.intent.category.DEFAULT" />
		<category android:name="android.intent.category.BROWSABLE" />
	</intent-filter>
</activity>
```
For a full example of the `*-app.xml` please refer to the [demo](https://github.com/BranchMetrics/Branch-AIR-ANE-SDK/blob/master/bin/Branch-AIR-ANE-SDK-app.xml).

Initialize SDK And Register Deep Link Routing Function
------------------------------------------------------
*For a full example refer to the demo [as3 file](https://github.com/BranchMetrics/Branch-AIR-ANE-SDK/blob/master/test/src/BranchTest.as).*

Inside your `Main.as` make the following import:
```as3
import io.branch.nativeExtensions.branch.Branch;
import io.branch.nativeExtensions.branch.BranchEvent;
```

Then create a Branch instance:  
`var branch:Branch = new Branch();`  
Note that Branch is a **Singleton**, it can only have one instance which can also be used thanks to a static:  
`Branch.getInstance();`

Then you can register two events before initializing the SDK:  
```as3
branch.addEventListener(BranchEvent.INIT_FAILED, initFailed);
branch.addEventListener(BranchEvent.INIT_SUCCESSED, initSuccessed);

private function initFailed(bEvt:BranchEvent):void {
	trace("BranchEvent.INIT_FAILED", bEvt.informations);
}

private function initSuccessed(bEvt:BranchEvent):void {
	trace("BranchEvent.INIT_SUCCESSED", bEvt.informations);
	
	// params are the deep linked params associated with the link that the user clicked before showing up
	// params will be empty if no data found
	var referringParams:Object = JSON.parse(bEvt.informations);
	
	//trace(referringParams.user);
}
```

Once is done, initialize the SDK: `branch.init();`  
Be sure to have the `INIT_SUCCESSED` event called, otherwise read the `bEvt.informations` from the `INIT_FAILED` event.

Persistent identities
---------------------

Often, you might have your own user IDs, or want referral and event data to persist across platforms or uninstall/reinstall. It's helpful if you know your users access your service from different devices. This where we introduce the concept of an 'identity'.

To identify a user, just call:`branch.setIdentity("your user id");`

Logout
------
If you provide a logout function in your app, be sure to clear the user when the logout completes. This will ensure that all the stored parameters get cleared and all events are properly attributed to the right identity.

**Warning** this call will clear the referral credits and attribution on the device.  
`branch.logout();`

Generate Tracked
================

Shortened links
---------------
There are a bunch of options for creating these links. You can tag them for analytics in the dashboard, or you can even pass data to the new installs or opens that come from the link click. How awesome is that? You need to pass a callback for when your link is prepared (which should return very quickly, ~ 50 ms to process).

For more details on how to create links, see the [Branch link creation guide](https://github.com/BranchMetrics/Branch-Integration-Guides/blob/master/url-creation-guide.md).

```as3
//be sure to add the event listeners:
branch.addEventListener(BranchEvent.GET_SHORT_URL_FAILED, getShortUrlFailed);
branch.addEventListener(BranchEvent.GET_SHORT_URL_SUCCESSED, getShortUrlSuccessed);

private function getShortUrlSuccessed(bEvt:BranchEvent):void {
	trace("BranchEvent.GET_SHORT_URL_SUCCESSED", "my short url is: " + bEvt.informations);
}

private function getShortUrlFailed(bEvt:BranchEvent):void {
	trace("BranchEvent.GET_SHORT_URL_FAILED", bEvt.informations);
}

var dataToInclude:Object = {
	user:"Joe",
	profile_pic:"https://avatars3.githubusercontent.com/u/7772941?v=3&s=200",
	description:"Joe likes long walks on the beach...",
	
	// customize the display of the Branch link
	"$og_title":"Joe's My App Referral",
	"$og_image_url":"https://branch.io/img/logo_white.png",
	"$og_description":"Join Joe in My App - it's awesome"
};

var tags:Array = ["version1", "trial6"];

branch.getShortUrl(tags, "text_message", BranchConst.FEATURE_TAG_SHARE, "level_3", JSON.stringify(dataToInclude));
```

There are other methods which exclude tag and data if you don't want to pass those.

**Note**
You can customize the Facebook OG tags of each URL if you want to dynamically share content by using the following _optional keys in the data dictionary_:

| Key | Value
| --- | ---
| "$og_title" | The title you'd like to appear for the link in social media
| "$og_description" | The description you'd like to appear for the link in social media
| "$og_image_url" | The URL for the image you'd like to appear for the link in social media
| "$og_video" | The URL for the video 
| "$og_url" | The URL you'd like to appear
| "$og_app_id" | Your OG app ID. Optional and rarely used.

Also, you do custom redirection by inserting the following _optional keys in the dictionary_:

| Key | Value
| --- | ---
| "$desktop_url" | Where to send the user on a desktop or laptop. By default it is the Branch-hosted text-me service
| "$android_url" | The replacement URL for the Play Store to send the user if they don't have the app. _Only necessary if you want a mobile web splash_
| "$ios_url" | The replacement URL for the App Store to send the user if they don't have the app. _Only necessary if you want a mobile web splash_
| "$ipad_url" | Same as above but for iPad Store
| "$fire_url" | Same as above but for Amazon Fire Store
| "$blackberry_url" | Same as above but for Blackberry Store
| "$windows_phone_url" | Same as above but for Windows Store

You have the ability to control the direct deep linking of each link by inserting the following _optional keys in the dictionary_:

| Key | Value
| --- | ---
| "$deeplink_path" | The value of the deep link path that you'd like us to append to your URI. For example, you could specify "$deeplink_path": "radio/station/456" and we'll open the app with the URI "yourapp://radio/station/456?link_click_id=branch-identifier". This is primarily for supporting legacy deep linking infrastructure. 
| "$always_deeplink" | true or false. (default is not to deep link first) This key can be specified to have our linking service force try to open the app, even if we're not sure the user has the app installed. If the app is not installed, we fall back to the respective app store or $platform_url key. By default, we only open the app if we've seen a user initiate a session in your app from a Branch link (has been cookied and deep linked by Branch).