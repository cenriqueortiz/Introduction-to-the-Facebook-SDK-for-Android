# Introduction

In this article I cover an overview of the Facebook SDK for Android, how to integrate it into your app, and how to use the Login, Graph and Share APIs which are commonly used in mobile apps. The article begins with by covering how set up the application with Facebook, permissions, application development modes, and test users. The article continues with a sample app and code.

>## Prerequisites
>To follow along with this article, you need the following skills and tools:
>* Basic knowledge of Java™ technology
>* Basic knowledge of Android
>* Java Development Kit (version 10.0.1, as of this writing)
>* Android Studio (version 3.2.1, as of this writing)

# Registering and setting up your application with Facebook

Before you begin, you need to register your app with Facebook. The process of registering an application with Facebook and setting up the application is as follows.

Figure 1 - Typical Steps for New Apps
![setup-flow.jpg](resources/4B080A0605C71F7B4F224712439AB360.jpg =968x99)

During application registration, the application is added, the app ID created, and the app settings properly defined and confifgured, including updating the Android string resource file with app id information, and setting the roles and test users for app testing.

## Add New App

To register your application, go to [Facebook's Developer Website](https://developers.facebook.com) (https://developers.facebook.com). Click on **"Add New App"**.

Figure 2 - Registering the App with Facebook
![fbdebsite_newapp.png](resources/12D48BFEA3E3F5AFD84367D3A0CF5FBC.png =670x552)

Next, create the new App's ID.

## Create App ID

In the "Create a New App ID" dialog enter a **Display Name** and **Contact Email** for your application.

Figure 3 - Create a New App ID
![CreateNewAppID.png](resources/05D7F59EE113AA03F2C438C317CFAC60.png =716x327)

Pressing the **Create App ID** button will generate the app ID and take you to the application's Dashboard.

## The App Dashboard

On the top left of the Dashboard you will find a drop-down to navigate across your different apps, and next to it you will see the assigned App ID. The **application ID** and associated secret key (clientSecret) are used in your application when making the different Facebook API calls.

> A Facebook app ID is a unique number that is assigned to your app, and used to identify your app when contacting Facebook.

You can view and manage the application development status, view the app's analytics, manage the app's basic and advanced settings, manage roles and test users, manage the app review process, and manage the Facebook products used by the app.

Figure 4 - App Dashboard
![fb-app-panel.png](resources/83234C61D7AEACEEF83FBC5E213007F7.png =2087x870)

You can press _Skip_, and go directly to the Basic and Advanced Settings. While at the Basic Settings, press **Add Platform** to add your Android App add your related website, as illustrated next.

Figure 5 - Add Platform
![add-platform.png](resources/0E04232975604435721BF36BEAA6D84D.png =955x41)

Press "**+ Add Platform**" and enter the requested information.

![add-platform2.png](resources/9F1CA47ED209C771574CEBAFF60AE029.png =959x695)

In the Basic Settings screen enter the Google Play Package name, the Default Activity Class Name, and the key hashes for the app, as well as other settings such as enabling Single Sign-on and Deeplinking.

## Keyhashes and Single Sign On

Keyhashes are needed to authenticate the exchange of information between your app and Facebook. Without a keyhash, the Facebook integration may not work properly once you put your app in the store.

You can enable Single Sign on (SSO) for your app. SSO is a way to login and authorize the user by delegating authentication to the official Facebook native app, if it has been installed in the handset. If the user is already authenticated with the Facebook native app, they won't be prompted for username or password on your app. If SSO is enabled but the Facebook native app is not installed, or if SSO is not enabled, authentication will default to the OAuth-based Webview Facebook screen interfaces and flows. Note that in this article I only cover the Webview screens, but it is recommended to enable SSO to make it easier for the user to login, if the Facebook native app is installed.

To enable SSO, you need to specify the app's keyhash (debug or release as appropriate) and turn on SSO in the app settings, as illustrated above Figure 5 above.

> Side note: The Android platform requires that all APKs be digitally signed with a debug or a release certificate as appropriate before they are installed on a device or updated (for more informations see [Sign your app](https://developer.android.com/studio/publish/app-signing) (Android Developer Website). 

To create the app's keyhash use the `keytool` command line as follows (note: the example belows are for MacOS).

**Debug mode**
```java
 keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
```
**Release mode**
```java
keytool -exportcert -alias YOUR_RELEASE_KEY_ALIAS -keystore YOUR_RELEASE_KEY_PATH | openssl sha1 -binary | openssl base64
```

Next, enter the generated keyhash value in the "**Key Hashes**" field.

# App Modes and Test Users

As in any software that you write, there is a development and testing phase, and there is going live. In Facebook, when you register your application, it will start in _Development Mode_ by default. Development mode keeps the app only visible to you (is hidden from the App Center), and the app is _automatically_ approved for all login permissions and features requested so that you can test your application. Once you are ready to test your application and integrations with Facebook, you can use Test Users for your end-to-end testing. Test Users are not real Facebook users and are a great way to test your app end-to-end.

You can create Test User directly within the App Dashboard in the Roles Tab. Click on the Add button next to Test Users.

Figure 6 - Test Users
![fb test users.png](resources/1DF6B316D07135241108C1ECABAD7869.png =1064x709)

You can add more test users and edit test user's information, permissions, set expiration times and other, as needed.

> See [Accessing the Test User Management Tool](https://developers.facebook.com/docs/apps/test-users#managetool) for more information.

Once you are ready to release the app, just toggle the app Status on the top of the app Dashboard (see Figure 4). If your app is using permissions or features that require app review, you must submit your app for App Review, before making your app public.

For more information, see Facebook page on [Managing Development Cycles](https://developers.facebook.com/docs/apps/managing-development-cycle/).

# Understanding Access Tokens and Permissions

Facebook uses _access tokens_ to validate and authenticate the client and control the access to the Facebook Platform services and resources. 

Facebook defines the following types of access tokens (1) User Access, (2) App Access, (3) Page Access, and (4) Client Tokens. In this article I only cover the _User Access Token_. Facebook APIs to read, modify or write user data require an _user access token_ to be specified. 

You can read more about the different kinds of access tokens by visiting the [Facebook's Access Tokens page](https://developers.facebook.com/docs/facebook-login/access-tokens/). 

In the SDK, Access Tokens is represented by `AccessToken`, a class that represents an immutable access token for using Facebook APIs, and that includes associated metadata such as expiration date and permissions. You can test if there is an active token by calling `AccessToken.isCurrentAccessTokenActive()`.

A typical login sequence is illustrated next. In this sequence, a user decides to login and grants permissions. Next, the app/SDK will pass the access request and permissions to Facebook. Once a user is authenticated, and the permissions are granted, the app receives a **User Access Token** that gives temporary access to Facebook APIs and user data. Default expiration period for _data access_ is 90 days.

Figure 7 - Login Sequence
![Login Sequence.png](resources/5E2ECF1CF8D70F102B7AAFD2E9E1D800.png =590x493)

Requesting and granting permissions gives the application access to user data or to operations such as sharing or updating basic information, or the friend's list. You can see the current access tokens associated with your account by visiting the [Access Token page](https://developers.facebook.com/tools/accesstoken).

Applications should be mindful of the types and amount of permissions the app requests from the user. A minimalistic approach to permissions reduces user hesitation when installing the app, helping with app adoption. Requesting certain permissions will require app review before you can make your app public. Permissions that do not require app review are (1) *Basic Profile*: the user's Facebook id, firstname, lastname, middlename, picture, short name, and (2) the user's *email address*. All other permissions may result in app review by Facebook before the app can go puiblic. For more information see Facebook's [app review and approval proces](https://developers.facebook.com/docs/facebook-login/review).

> Side note: While in development mode, the app is private, and is automatically approved for all login permissions and features requested so that you can test your application. Also note that in 2018, Facebook revamped the user permissions, adding new requirements on security. These new changes may or not break older apps. Refer to [Breaking Changes](https://developers.facebook.com/docs/graph-api/changelog/breaking-changes/) to learn more about potential breaking changes.

For information about all the permissions associated with the Facebook Login, see [Permissions Reference - Facebook Login](https://developers.facebook.com/docs/facebook-login/permissions).

# Overview of the Facebook SDK for Android

The Facebook SDK for Android is a Java programming language on top of Facebook platform REST APIs. The SDK is open source, and it is hosted at GitHub's [facebook / facebook-android-sdk](http://https://github.com/facebook/facebook-android-sdk) repository. Note that due to the evolving nature of the Facebook API and the open-source SDK, expect future changes that might require application updates.

The Facebook SDK for Android consists of a number of related SDKs that all together provide the collection of APIs to build social apps across Android, iOS, React Native, and others platforms. The Facebook SDK for Android consists of 7 related SDKs as illustrated next.

Figure 8 - Components of the Facebook SDK for Android
![android-sdk-diagram.jpg](resources/4570732402B59FFA92BA89C1087E4D17.jpg =469x428)

As a developer, you can include the complete set of SDKs or just individual SDKs as needed. Only including the SDKs that your app needs is a simple way of managing the overall app size. The following table describes each SDK in more detail including a link to the related official documentation.

Table 1 - Components of the Facebook SDK for Android
|Component|Description| Documentation |
|---|-----------|-----|---------------|
|**Core APIs**|Core APIs that including Graph, App Events. Analytic APIs.| [Graph API](https://developers.facebook.com/docs/graph-api), [Facebook Analytics](https://developers.facebook.com/docs/analytics/implementation), [App Events](http://https://developers.facebook.com/docs/app-events/getting-started-app-events-android) |
|**Login**|API for to login using Facebook, and requesting app permissions.| [The Facebook Login SDK](https://developers.facebook.com/docs/facebook-login) |
|**Share**|API to share or send a messages.| [The Facebook Sharing SDK](https://developers.facebook.com/docs/sharing) |
|**Places**|API to search for places, place discovery, location sharing, and geo-tagging. | [The Facebook Places SDK](https://developers.facebook.com/docs/places) |
|**Messenger**|API API to share both links and media from your apps to Messenger, and enable bot chat extentions.| [The Facbeook Messenger SDK](https://developers.facebook.com/docs/messenger-expressions) |
|**App Links**|API to deep link to content within the mobile app.| [The Facebook App Links SDK](https://developers.facebook.com/docs/applinks) |

Visit Facebook's [Facebook for Android SDK documentation](https://developers.facebook.com/docs/android) for the complete, official documentation.

In addition to the above set of SDKs, Facebook also provides other SDKs such as the **Account Kit** SDK for login using a phone number or email address without needing a password, and an **Audience Network** SDK for audience-centric, targeted and rich adversiting.

In this article I only cover the following SDKs and functionality: (1) Login API, (2) the Graph API, and (3) Sharing API.

# Integrating the Android SDK

To integrate the Android SDK, you can download the [open source SDK from GitHub](https://github.com/facebook/facebook-android-sdk), or you can download the SDKs Android Archive Library (aar) libraries from the [Facebook Android SDK direct downloads page](https://developers.facebook.com/docs/android/downloads/), but the _prefered way_ is to use Gradle's dependency management using _Maven Central_ or _JCenter_ repository. Add the repositiory to the Project's gradle file, and the Dependencies to the app's gradle file.

**(1) Add to the Project's gradle file either _maven central_ or _jcentral_ as the SDK repository**

Listing 1 - Add the repositiory (Project Gradle File)
```javascript
repositories {
  // You can use mavenCentral() or jcenter()
  mavenCentral() 
}
```

**(2) Next, add the appropriate dependencies into the app gradle file at `/app/build.gradle`. Recall that you can include only the components that your app needs, or you can include the whole SDK.**

Listing 2 - Add the whole Android SDK dependencies (App Gradle File)

```javascript
dependencies { 
  // Facebook Android SDK (everything)
  compile 'com.facebook.android:facebook-android-sdk:4.+'
}
```

Or, include the individual SDKs, as follows.

Listing 3 - Add the individual dependencies (App Gradle File)
```javascript
dependencies { 
  // Facebook SDK Core only (Analytics)
  compile 'com.facebook.android:facebook-core:4.+'
  // Facebook Login only
  compile 'com.facebook.android:facebook-login:4.+'
  // Facebook Share only
  compile 'com.facebook.android:facebook-share:4.+'
  // Facebook Places only
  compile 'com.facebook.android:facebook-places:4.+'
  // Facbeook Messenger only
  compile 'com.facebook.android:facebook-messenger:4.+'
  // Facebook App Links only
  compile 'com.facebook.android:facebook-applinks:4.+'
}
```

Selecting individual SDKs will give you more control and flexibility over the mobile app size.

# The Sample App

Let's now look at a sample app. In this article we will explore an Android app with the views and layouts and functionallity as illustrated below. 

Figure 9 - The Sample App
![Screen Shot 2019-02-09 at 7.42.46 PM.png](resources/568BD65A9EFA3A739AC247A5F1D7609D.png =420x743)

In this sample app I show two different ways to login (using Facebook's `LoginButton` and using the Login API), how to get the user's basic profile information, how to get the user's friends list, how to reauthorize permissions, and two ways to share (using Facebook's `ShareDialog`, and using Facebook's `ShareButton`).

The application consists of 3 classes:
* `MainActivity` - the main activity for the application.
* `Friend` - a class that represents a friend.
* `FriendArrayAdapter` - an array adapter for the friend list.

The application has the following resource files:
* `main.xml` - defines a `LinearLayout` for the main screen.
* `rowlayout.xml` - defines a `LinearLayout` for the the friend's list.
* `string.xml` - defines the various string resources.

# The Android Manifest

All Android apps have an associated Android Manifist. The manifest for this sample app is next. 

A couple of important items:

1. Set the permission to access the `android.permission.INTERNET`. (line 7)
2. Define the `meta-data`, to set the Facebook application ID. Recall that you can get the APP ID from the Dashboard. Set the app ID in the `strings.xml` file, together with other application strings such as the button strings. (line 13)
3. Define the `FacebookContentProvider` that will be used to share content, if sharing binary attachments such as photos. (line 26)

Listing 4 - Android Manifest
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	package="com.cenriqueortiz.usingandroidsdk"
	android:versionCode="1"
	android:versionName="1.0">

	<uses-permission android:name="android.permission.INTERNET"/>

	<application
		android:icon="@drawable/icon"
	    android:label="@string/app_name">

		<meta-data
			android:name="com.facebook.sdk.ApplicationId"
			android:value="@string/facebook_app_id"/>

		<activity android:name=".MainActivity"
		    android:label="@string/app_name"
            android:configChanges="keyboard|keyboardHidden|screenLayout|screenSize|orientation">
			<intent-filter>
  			<action android:name="android.intent.action.MAIN" />
  			<category android:name="android.intent.category.LAUNCHER" />
			</intent-filter>
		</activity>
		
    <provider android:authorities="com.facebook.app.FacebookContentProvider{@string/facebook_app_id}"
      android:name="com.facebook.FacebookContentProvider"
      android:exported="true"/>

    </application>

</manifest>
```
# The `string.xml` file

The following code snippet shows the `string.xml` file for this sample app. Note the `facebook_app_id` (line 3) that you can find on the app's Facebook Dasbhoard.

Listing 5 - The string.xml file
```xml
<resources>
    <string name="app_name">Using FB Android SDK</string>
    <string name="facebook_app_id">[HERE INSERT THE APP ID]</string>
    <string name="loginactivity_name">Login Activity</string>
    <string name="login">Login</string>
    <string name="logout">Logout</string>
    <string name="share">Share</string>
    <string name="friends">Get Friends</string>
    <string name="reauthorize">Reauthorize</string>
    <string name="noprofileinfo">No profile Info.</string>
    <string name="me">Me Request</string>
    <string name="blankspace"></string>
</resources>
```

# The Screen Layout Resource File

The following code snippet shows the `main.xml` layout resource file for the app's main screen. In this file we have a `LinearLayout`, with a number of `Buttons`, a `TextView`, and a `ListView`. Also, note the two Facebook button views.

1. `com.facebook.login.widget.LoginButton` which uses the Facebook SDK `LoginButton`. (line 10)
2. `com.facebook.share.widget.ShareButton` which is the Facebook SDK `ShareButton`. (line 57)

Listing 6 - The main.xml layout resource file
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/white">

        <com.facebook.login.widget.LoginButton
            android:id="@+id/login_fbbutton"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="center_horizontal"
            android:layout_margin="5dp"/>

        <Button
            android:id="@+id/login_button"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            android:layout_gravity="center_horizontal"
            android:text="@string/login"
            />

        <Button
            android:id="@+id/me_button"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            android:layout_gravity="center_horizontal"
            android:text="@string/me"
            />

        <Button
            android:id="@+id/friends_button"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            android:layout_gravity="center_horizontal"
            android:text="@string/friends"
            />

        <Button
            android:id="@+id/reauthorize_button"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            android:layout_gravity="center_horizontal"
            android:text="@string/reauthorize"
            />

        <Button
            android:id="@+id/share_button"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            android:layout_gravity="center_horizontal"
            android:text="@string/share"
            />

        <com.facebook.share.widget.ShareButton
            android:id="@+id/fb_share_button"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:text="FB ShareButton"
            android:layout_margin="5dp"/>

        <TableRow
            android:layout_width="match_parent"
            android:layout_height="1dp"
            android:background="#444"
            android:layout_margin="5dp"/>

            <TextView
                android:id="@+id/textView_name"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_margin="5dp"
                android:text="@string/noprofileinfo"/>

         <TableRow
            android:layout_width="match_parent"
            android:layout_height="1dp"
            android:background="#444"
            android:layout_margin="5dp"/>

        <ListView
            android:id="@+id/listview"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textFilterEnabled="true"
            android:layout_margin="5dp"/>

        <TextView
            android:id="@+id/textView2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="5dp"/>

</LinearLayout>
```

The following sections will cover the application code in detail.

# The MainActivity

Before we start, let's go over the beginning of `MainActivity.java`, the app's main class. Important to note is the defined `FacebookCallback` that is invoked by `LoginManager` when login requests complete; this callback is used by some of the API calls covered below. The `MainActivity` class also defines all of the activity's life-cycle methods as expected, and defines `onActivityResult()` that is called when Facebook UI login activities complete; note the call to `CallbackManager.onActivityResult()` that is necessary to ensure the SDK callback manager properly handles login state changes.

Listing 7 - The MainActivity
```java
/**
 * MainActivity.
 */
public class MainActivity extends Activity {

    private static final String EMAIL = "email";
    private static final String USER_POSTS = "user_posts";
    private static final String BIRTHDAY = "user_birthday";
    private static final String FRIENDS = "user_friends";
    private static final String AUTH_TYPE = ""; //"rerequest";
    private static final List<String> mPermissions = Arrays.asList(EMAIL, USER_POSTS, BIRTHDAY, FRIENDS);
    private final ArrayList<Friend> mFriends = new ArrayList<>();
    private FriendsArrayAdapter mFriendsArrayAdapter;
    private LoginManager mLoginManager;
    private CallbackManager mCallbackManager;
    private static final String TAG = MainActivity.class.getSimpleName();

    ///////////////////////////////////////////////////////////////////////////////////
    // LoginManager FacebookCallback, used by LoginManager (and Facebook LoginButton)
    ///////////////////////////////////////////////////////////////////////////////////
    private FacebookCallback<LoginResult> mFacebookCallback = new FacebookCallback<LoginResult>() {
        @Override
        public void onSuccess(LoginResult loginResult) {
            Log.d(TAG, "***** mFbLoginButton.onSuccess().granted: " +
                    loginResult.getRecentlyGrantedPermissions().toString() +
                    " denied: " + loginResult.getRecentlyDeniedPermissions().toString());
        }

        @Override
        public void onCancel() {
            Log.d(TAG, "***** mFbLoginButton.OnError().onCancel");
        }

        @Override
        public void onError(FacebookException e) {
            // Handle exception
            Log.d(TAG, "***** mFbLoginButton.OnError().FacebookException: " + e);
        }
    };

    /**
     * Activity onActivityResult.
     * Calls the Facebook's onActivityResult Callback.
     */
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        Log.d(TAG,"***** onActivityResult: " + resultCode);
        mCallbackManager.onActivityResult(requestCode, resultCode, data);
        resetScreenViews();
    }

    /**
     * Helper method to reset the different views in the main screen.
     * Update Login Button text accordingly, and clear text fields.
     */
    private void resetScreenViews() {
        final Button loginButton;
        loginButton = findViewById(R.id.login_button);
        if (loginButton != null) {
            if (AccessToken.isCurrentAccessTokenActive()) {
                loginButton.setText(R.string.logout);
            } else {
                loginButton.setText(R.string.login);
                mFriends.clear();
                TextView tv = findViewById(R.id.textView_name);
                tv.setText(R.string.blankspace);
            }
        }
    }

    /**
     * Activity onCreate.
     * Initializes the Activity, and the Facebook Login Widget Button.
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d( "***** MainActivity", "onCreate");
        setContentView(R.layout.main);

        // Get instances for CallbackManager and LoginManager, and register the LoginManager callback
        mCallbackManager = CallbackManager.Factory.create();
        mLoginManager = LoginManager.getInstance();
        LoginManager.getInstance().registerCallback(mCallbackManager, mFacebookCallback);
        :
        :
```

The rest of the articles focuses on how to use the Login API, the Graph API, and the Sharing API.

# Using the Login API

Before accessing any of Facebook resources, the user must login. The Facebook Login API provides two main functions: 

1. User authentication using Facebook credentials, and,
2. Requesting permisions and granting permissions for user data access or for operations such as sharing, getting the user's profile, getting the user's list of friends, or accesing pages. 

Once a user has been authenticated, and has granted permissions, the app will receive a _User Access Token_ that is passed to subsequent API calls. Recall that if SSO has been enabled and the Facebook native app is installed on the handset, authentication will be depegated to the Facebook native app, otherwise, the app will default OAuth Webview-based authentication and permission flow.

## Using Facebook's `LoginButton`

In this section I cover how to initiate the login flow using Facebook SDK `com.facebook.login.widget.LoginButton`, a native button control provided by the SDK that keeps login state and performs the login/logout for the app. For this button to work, it requires the app ID to be specified in the AndroidManifest.xml.

Let's look at the login button's definition in the `main.xml` layout resource file.

Listing 8 - The LoginButton (com.facebook.login.widget.LoginButton)
```xml
<com.facebook.login.widget.LoginButton
    android:id="@+id/login_fbbutton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center_horizontal"/>
```

The above will display a Facebook login or logout native button that uses Facebook's branding colors.

The code to setup the `LoginButton` is next. First, we get the reference to the `LoginButton`itself, set the permissions, teh auth type, and register the `FacebookCallback`.

Listing 9 - Using the LoginButton
```java
:
:
/////////////////////////////////////////////////////////////////////////////////////////
// This snippet of code shows how to use Facebook's Login Button.
//  After getting the reference to LoginButton, set its callback.
// Register the mFbLoginButton callback (which sets the LoginManager callback)
/////////////////////////////////////////////////////////////////////////////////////////
LoginButton mFbLoginButton;
mFbLoginButton = findViewById(R.id.login_fbbutton);
mFbLoginButton.setReadPermissions(mPermissions);
mFbLoginButton.setAuthType(AUTH_TYPE);
mFbLoginButton.registerCallback(mCallbackManager, mFacebookCallback);
:
:
```

The `FacebookCallback` (defined earlier in this article) is called by the `LoginManager` when the login completes successfully, or is canceled or if there is an error.

Using the `LoginButton` is very simple, as it hides most of the logic that you would have to do if using the `LoginManager` directly, as shown next. 

## Login using Facebook's `Login Manager`

You can use the `LoginManager` directly to login the user. The LoginManager manages login and permissions for Facebook. 

As in the above example, let's look at the button's definion in the `main.xml` layout resource file.

Listing 10 - The Login Button
```xml
<Button
  android:id="@+id/login_button"
  android:layout_width="match_parent"
  android:layout_height="40dp"
  android:layout_gravity="center_horizontal"
  android:text="@string/login"
  />
```

The above defines a basic `ButtonView` that you can style as needed. Next, is the code to setup the `ButtonView`. We get a reference to the `ButtonView` itself, set the button's text based on the current access token state, and set the button click listener. 

Listing 11 - Using the Login Manager
```java
:
:
/////////////////////////////////////////////////////////////////////////////////////////
// This code snippet shows how to use LoginManager (versus LoginButton)
/////////////////////////////////////////////////////////////////////////////////////////
final Button mLoginButton;
mLoginButton = findViewById(R.id.login_button);
if (mLoginButton != null) {
    // Update Login Button text accordingly
    if (AccessToken.isCurrentAccessTokenActive()) {
        mLoginButton.setText(R.string.logout);
    } else {
        mLoginButton.setText(R.string.login);
    }
    // Set Login Button on click callback
    mLoginButton.setOnClickListener(
        new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Log.d(TAG, "***** mLoginButton.onClick");

                if (AccessToken.isCurrentAccessTokenActive()) {
                    Log.d(TAG, "***** mLoginButton.onClick -> Logout");
                    LoginManager.getInstance().logOut();
                    mLoginButton.setText(R.string.login); // toggle button text to read "login"
                    resetScreenViews();
                } else {
                    Log.d(TAG, "***** mLoginButton.onClick -> Login");
                    LoginManager.getInstance().logInWithReadPermissions(MainActivity.this, mPermissions);
                    mLoginButton.setText(R.string.logout); // toggle button text to read "logout"
                }
            }

        });
}
:
:
```

When the button is clicked, the button's click listener is called. The click listener in turn calls the `LoginManager` to login with permissions (`logInWithReadPermissions()`) or `logout()` as appropriate, based on the state of current access token (`AccessToken.isCurrentAccessTokenActive()`).

Check out Facebook's [Login for Android - Quickstart](https://developers.facebook.com/docs/facebook-login/android) that walks you throught the different steps for setting up and using the Login API, and learn more about Facebook [Login Security](https://developers.facebook.com/docs/facebook-login/security/).

# The Social Graph and the Graph API

Facebook uses a [graph](https://en.wikipedia.org/wiki/Graph_(discrete_mathematics)) to represent user data, in what they called the _Facebook social graph_. To access the social graph, Facebook defined the [Graph API](https://developers.facebook.com/docs/graph-api/using-graph-api/). The following diagram illustrates the social graph.

Figure 10 - The Social Graph
![fb_social_graph.jpg](resources/E285FF58B63A8CCFBF57642B61D98D23.jpg =1113x508)

In Facebook's social there are:

* **Nodes** or individual _objects_, such as a User, a Photo, a Page, or a Comment. Graph API objects are assigned a unique ID and are easily addressable using a URL scheme that can be further qualified to address a specific object/connection. The general structure of an object URL is as follows: "`https://graph.facebook.com/ObjectID/ConnectionType`" where _ObjectID_ is the object's unique ID and _ConnectionType_ is one of the connection types supported by the object. 
* **Edges** or _connections or relationships_ between a collection of objects and a single object. For example, a page supports the following connections: feed/wall, photos, notes, posts, members, and so on. 
* **Fields** that are the actual _data_ or attributes about an object, such as a User's name or gender, or a Page's name.

With the Graph API, you can retrieve an object, delete an object, and publish objects. You can search, update objects, filter results, and even dynamically discover the connections/relationships of an object. You can also retrieve the fields (details) for a given object or node.

In this article I cover two examples of using the Graph API: retrieving the user profile for the currently logged in user (/me), and retrieving the user's friends list. 

## Using the Graph API to make a "Me Request"
This section covers how to make a "Me Request". 

Similarly to the login button, let's first define a `ButtonView` for the "Me Request" button.

Listing 12 - The Me Request `ButtonView`
```xml
<Button
  android:id="@+id/me_button"
  android:layout_width="match_parent"
  android:layout_height="40dp"
  android:layout_gravity="center_horizontal"
  android:text="@string/me"
  />
```

Let's also define a TextView to display the results of the "me request".

Listing 13 - The Me Request `TextView`
```xml
<TextView
  android:id="@+id/textView_name"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  android:layout_margin="5dp"
  android:text="@string/noprofileinfo"/>
```

When the "Me Request" button is clicked, the click listener invokes the asynchronous method `GraphRequest.newMeRequest()` to make the actual "me request" to Facebook, passing the current access token, and defining a `GraphRequest.GraphJSONObjectCallback()` that is called when the `newMeRequest()` completes. The callback parses the JSON response and populates the `TextView` with the response.

Listing 14 - Using the Graph API to get the user's profile
```java
:
:
/////////////////////////////////////////////1////////////////////////////////////////////
// The following code snippet shows how to make a Facebook "Me Request".
//  Get reference to the "Me Request" button, set its "on click handler", which makes
//      the Graph API
/////////////////////////////////////////////////////////////////////////////////////////
Button mMeButton;
mMeButton = findViewById(R.id.me_button);
mMeButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Log.d(TAG,"***** mMeButton.onClick");

        if (AccessToken.isCurrentAccessTokenActive()) {

            Log.d(TAG,
                    "***** mMeButton.onClick, isCurrentAccessTokenActive: " +
                            AccessToken.isCurrentAccessTokenActive() + " / " +
                            AccessToken.getCurrentAccessToken().getPermissions());

            // Me Facebook Request
            GraphRequest meRequest = GraphRequest.newMeRequest(
                    AccessToken.getCurrentAccessToken(),
                    new GraphRequest.GraphJSONObjectCallback() {
                        public void onCompleted(JSONObject json, GraphResponse response) {
                            Log.d(TAG,"***** meRequestCallback: " + response.getJSONObject());

                            TextView tv;
                            String s = "";
                            tv = findViewById(R.id.textView_name);
                            try {
                                s = " ID: " + response.getJSONObject().getString("id") + "\n";
                            } catch (JSONException e) {
                                e.printStackTrace();
                            }

                            try {
                                s += " Name: " + response.getJSONObject().getString("name") + "\n";
                            } catch (JSONException e) {
                                e.printStackTrace();
                            }

                            try {
                                s += " Email: " + response.getJSONObject().getString("email") + "\n";
                            } catch (JSONException e) {
                                e.printStackTrace();
                            }
                            try {
                                s += " Birthday: " + response.getJSONObject().getString("birthday") + "\n";
                            } catch (JSONException e) {
                                e.printStackTrace();
                            }
                            s += " Permissions: " + AccessToken.getCurrentAccessToken().getPermissions().toString();
                            tv.setText(s);
                        }

                    });
            Bundle parameters = new Bundle();
            parameters.putString("fields", "id, name, email, birthday");
            meRequest.setParameters(parameters);
            meRequest.executeAsync();
        }
    }
});
:
:
```

In the above "Me Request" example, the app requests specific object fields "id, name, email, birthday" by passing a `Bundle` as a parameter, as highlighted next.

Listing 15 - Request specific object fields for the "Me Request" API Call
```java
Bundle parameters = new Bundle();
parameters.putString("fields", "id, name, email, birthday");
meRequest.setParameters(parameters);
meRequest.executeAsync();
```

The following snippet shows the "Me request" JSON response.

Listing 16 - Example of "Me request" JSON response
```json
{"id":"106615540438964","name":"Annod Leywor","email":"annod_puryjks_leywor@tfbnw.net","birthday":"01\/19\/2000"}
```

## Using the Graph API to make a "Friends request"

This section covers how to make a "Friends Request". 

Let's first look at the related user interface Views, as defined in `main.xml` resource file. The `ButtonView` is used to initiate the request, and the `ListView` is used to display the list of friends received from Facebook.

Listing 17 - `ButtonView` and `ListView` for the Friends request.
```xml
<Button
    android:id="@+id/friends_button"
    android:layout_width="match_parent"
    android:layout_height="40dp"
    android:layout_gravity="center_horizontal"
    android:text="@string/friends"
    />
    
<ListView
  android:id="@+id/listview"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:textFilterEnabled="true"
  android:layout_margin="5dp"/>
```

Next, let's look at the code behind the friends request. Similar to the "Me request", we first get a reference to the `ButtonView`. A click listener for the button is defined, which when invoked, it calls the asynchronous method `GraphRequest.newMyFriendsRequest()` passing as arguments the `AccessToken.getCurrentAccessToken()` and a `GraphRequest.GraphJSONArrayCallback()` that is called when the request completes. On completion, the callback parses the JSON response and populates the `ListView`.

Listing 18 - The Friends Request
```java
/////////////////////////////////////////////////////////////////////////////////////////
// Using the Friends Request
/////////////////////////////////////////////////////////////////////////////////////////
Button mFriendsButton;
mFriendsButton = findViewById(R.id.friends_button);
mFriendsButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Log.d(TAG,"***** mFriendsButton.onClick");

        if (AccessToken.isCurrentAccessTokenActive()) {
            Log.d(TAG,"***** mFriendsButton.onClick, isCurrentAccessTokenActive: " + AccessToken.isCurrentAccessTokenActive() + " / " +
                    AccessToken.getCurrentAccessToken().getPermissions());
        }

        // My Friends Request.
        GraphRequest friendsRequest = GraphRequest.newMyFriendsRequest(
                AccessToken.getCurrentAccessToken(),
                new GraphRequest.GraphJSONArrayCallback() {
                    public void onCompleted(JSONArray json, GraphResponse response) {
                        if (json == null) {
                            Log.d(TAG,"***** mFriendsButton.onCompleted, json is NULL");
                            return;
                        } else {
                            Log.d(TAG,"***** mFriendsButton.onCompleted, json: " + json);
                        }
                        mFriends.clear();
                        try {
                            for (int i=0; i<json.length(); i++) {
                                JSONObject o = json.getJSONObject(i);
                                String n = o.getString("name");
                                String id = o.getString("id");
                                Friend f = new Friend();
                                f.setId(id);
                                f.setName(n);
                                mFriends.add(f);
                            }
                        } catch (JSONException e) {
                            e.printStackTrace();
                        }
                        mFriendsArrayAdapter.notifyDataSetChanged();
                    }
                });

        Bundle parameters = new Bundle();
        parameters.putString("fields", "id, name, email, birthday");
        friendsRequest.setParameters(parameters);
        friendsRequest.executeAsync();

    }
});

//////////////////////////////////////////////////////////////////////////
// Setup the ListView Adapter that is loaded when selecting "get friends"
//////////////////////////////////////////////////////////////////////////
ListView mListView;
mListView = findViewById(R.id.listview);
mFriendsArrayAdapter = new FriendsArrayAdapter(MainActivity.this, R.layout.rowlayout, mFriends);
mListView.setAdapter(mFriendsArrayAdapter);
```

The `GraphRequest.newMyFriendsRequest()` API call returns a JSON file. An example of the "Me request" JSON response is next.

Listing 19 - Example of "Friends request" JSON response
```json
[{"id":"108073063622369","name":"Mark Alcbeijbfhjgj Fergiesen"},{"id":"103290550773109","name":"Margaret Alcbgfdicahfi Wisemanberg"},{"id":"105320080571114","name":"Tommy Pearson"},{"id":"103683167403118","name":"Betty Crocker"}]
```

In support of the friends list `ListView`, we have `FriendsArrayAdapter`, an `ArrayAdapter` to maintain the `ListView` updated using the received `Friend` information. The following code snippet shows the `FriendsArrayAdapter`.

Listing 20 - The `FriendsArrayAdapter`

```java
/**
 * ListView Friends ArrayAdapter
 */
public class FriendsArrayAdapter extends ArrayAdapter<Friend> {
    private final Activity mContext;
    private final ArrayList<Friend> mFriends;
    private int mResourceId;

    /**
     * Constructor
     * @param context the application content
     * @param resourceId the ID of the resource/view
     * @param friends the bound ArrayList
     */
    FriendsArrayAdapter(
            Activity context, 
            int resourceId, 
            ArrayList<Friend> friends) {
        super(context, resourceId, friends);
        this.mContext = context;
        this.mFriends = friends;
        this.mResourceId = resourceId;
    }

    /**
     * Updates the view
     * @param position the ArrayList position to update
     * @param convertView the view to update/inflate if needed
     * @param parent the groups parent view
     */
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolderItem viewHolder;
        if (convertView == null) {
            LayoutInflater inflater = (mContext).getLayoutInflater();
            convertView = inflater.inflate(mResourceId, parent, false);
            // Set up the ViewHolder
            viewHolder = new ViewHolderItem();
            viewHolder.textViewItem = convertView.findViewById(R.id.textViewItem);
            // Store the holder with the view.
            convertView.setTag(viewHolder);
        } else {
            // Use the viewHolder
            viewHolder = (ViewHolderItem) convertView.getTag();
        }

        // object item based on the position
        Friend f = mFriends.get(position);
        if (f != null) {
            // Get the TextView from the ViewHolder and then set the text (item name) and tag (item ID) values
            String txt = f.getId() + " | " + f.getName();
            viewHolder.textViewItem.setText(txt);
            viewHolder.textViewItem.setTag(f.getId());
        }
        return convertView;
    }

    // ViewHolder to cache views
    static class ViewHolderItem {
        TextView textViewItem;
    }
}
```

A `ViewHolder` design pattern is used to improve app UI performance by minimizing the number of lookups needed to find `View` elements. To learn more about this pattern, see [Making ListView Scrolling Smooth](https://developer.android.com/training/improving-layouts/smooth-scrolling#java) (Google developers website).

Last, is the `Friend` class that holds the information for each friend in the JSON response by Facebook.

Listing 21 - The `Friend` class
```java
/**
 * Represents a Friend
 */
public class Friend {
	private String id;
	private String name;
	private byte[] picture;
	private Bitmap pictureBitmap;

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public byte[] getPicture() {
		return picture;
	}

	public void setPicture(byte[] picture) {
		this.picture = picture;
	}

	public Bitmap getPictureBitmap() {
		return pictureBitmap;
	}

	public void setPictureBitmap(Bitmap pictureBitmap) {
		this.pictureBitmap = pictureBitmap;
	}
}
```

# Reauthorizing Data Access

When a user access token expires, typically after 90 days after the user was last active, the app won't have access to the user' data. To regain access, your app must re-authorize permissions. To reauthorize, call the `LoginManager.reauthorizeDataAccess()` method, which presents to the user the Facebook permissions screen for the user to grant or decline permissions. The following code snippet show how to use the `reauthorizeDataAccess()` method.

Listing 22 - Reauthorize Request
```java
/////////////////////////////////////////////////////////////////////////////////////////
// Reauthorize Request
/////////////////////////////////////////////////////////////////////////////////////////
Button mReauthorizeButton;
mReauthorizeButton = findViewById(R.id.reauthorize_button);
mReauthorizeButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        mLoginManager.reauthorizeDataAccess(MainActivity.this);
    }
});
```

When the reauthorization completes, the method `onActivityResult()` is called which in turn calls `mCallbackManager.onActivityResult()`.

# The Share API

In this section I will cover two approaches to sharing data, the first one is using Facebook's `ShareButton`, and the second one is using the `ShareDialog` interface directly. 

Faceboook Share APIs allow you to share different types of content. Starting with SDK version 4.0, the folowing _content models_ are supported:

Table 2 - Sharing API: Supported Content Types

| Content Type | Model |Notes|
|------------- | ------|-----|
|Links | ShareLinkContent| Share a Link. |
|Photos | SharePhotoContent| Share photos. Must have the native Facebook for Android app installed.|
|Videos | ShareVideoContent| Share videos.|
|Multimedia | ShareMediaContent|Share a combination of photos and videos. Must have the native Facebook for Android app installed. Up to 6 items per request.|

In this example, I only cover using `ShareLinkContent` content model.

If your app is going to share binary attachments such as images, you need to define a `FacebookContentProvider`in the `AndroidManifest.xml`, as shown next.

Listing X - Defining `FacebookContentProvider` in the AndroidManifest.xml
```xml
<provider android:authorities="com.facebook.app.FacebookContentProvider{@string/facebook_app_id}"
    android:name="com.facebook.FacebookContentProvider"
    android:exported="true"/>
```

Note how the `authorities` attribute concatenates `@string/facebook_app_id` from the `string.xml` file.

## Using the Facebook `ShareButton` Interface

The Facebook SDK provides `com.facebook.share.widget.ShareButton`, a native button that follows Facebook oficial branding look and feel, keeps the login state and provides basic login/logout flow functionality.

First, let's cover the `ShareButton` view in the `main.xml` resource file.

Listing 23 - The `ButtonView` for Using the Facebook `ShareButton`
```xml
<com.facebook.share.widget.ShareButton
            android:id="@+id/fb_share_button"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:text="FB ShareButton"
            android:layout_margin="5dp"/>
```

Next, the following code snippet shows how to create a `ShareLinkContent` to share a basic hyperlink when the `ShareButton` is clicked and its click listerner is called.

Listing 24 - Using the Facebook `ShareButton`
```java
/////////////////////////////////////////////////////////////////////////////////////////
// The following code snippet uses the Facebook Share Button.
//  Get reference to share button, set click listener callback.
/////////////////////////////////////////////////////////////////////////////////////////
final ShareButton fbShareButton;
fbShareButton = findViewById(R.id.fb_share_button);
fbShareButton.setEnabled(true);
fbShareButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Log.d(TAG,"***** fbShareButton.onClick");
        // For this example, sharing URL via ShareLinkContent.
        final String url = "http://developers.facebook.com/android";
        ShareLinkContent linkContent = new ShareLinkContent.Builder()
                .setContentUrl(Uri.parse(url))
                .build();
        fbShareButton.setShareContent(linkContent);
    }
});
```

Using the Facebook `ShareButton` is quite simple: define the ShareButton in the layout resource file, set a click listener, create the content model as appropriate, and call `ShareButton.setShareContent()` method.

## Using Share Dialog Interface

Another approach to sharing data is to use the `ShareDialog` directly. 

First, let's cover the `ButtonView` to trigger the sharing.

Listing 25 - The `ButtonView` for Using the Share Dialog
```xml
<Button
  android:id="@+id/share_button"
  android:layout_width="match_parent"
  android:layout_height="40dp"
  android:layout_gravity="center_horizontal"
  android:text="@string/share"
  />
```

Next, is setting the click listener for the ButtonView, setting the FacebookCallback that is called when sharing completes successfully, is canceled or an error is encountered. When the button is clicked, the appropriate content model is created, in this case a `ShareLinkContent`, which is passed to the method `ShareDialog.show()` if the share dialog supports such content model.

Listing 26 - Using the Facebook Share Dialog
```java
/////////////////////////////////////////////////////////////////////////////////////////
// The following code snippet uses the Facebook Share Dialog.
//  Get reference to share button, set click listener callback.
/////////////////////////////////////////////////////////////////////////////////////////
Button mShareButton;
mShareButton = findViewById(R.id.share_button);
mShareButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        mCallbackManager = CallbackManager.Factory.create();
        ShareDialog shareDialog = new ShareDialog(MainActivity.this);
        shareDialog.registerCallback(mCallbackManager, new FacebookCallback<Sharer.Result>() {
            @Override
            public void onSuccess(Sharer.Result result) {
                Log.d(TAG,"***** mShareButton.onClick.shareDialog.registerCallback->onSuccess(): " + result.toString());
            }

            @Override
            public void onCancel() {
                Log.d(TAG,"***** mShareButton.onClick.shareDialog.registerCallback->onCancel()");
            }

            @Override
            public void onError(FacebookException error) {
                Log.d(TAG,"***** mShareButton.onClick.shareDialog.registerCallback->onError(): " + error.toString());
            }
        });

        // For this example, sharing URL via ShareLinkContent.
        final String url = "http://developers.facebook.com/android";
        if (ShareDialog.canShow(ShareLinkContent.class)) {
            Log.d(TAG, "shareOnWall.ShareDialog.canShow");
            ShareLinkContent linkContent = new ShareLinkContent.Builder()
                    .setContentUrl(Uri.parse(url))
                    .build();
            shareDialog.show(linkContent, ShareDialog.Mode.AUTOMATIC);
        }

    }
});
```

The above code snippet is similar to using the ShareButton, but uses a bit more code for the callback and to test if the `ShareDialog` can show the requested content model.

For more information on how to share content with Facebook, read [Sharing on Android](https://developers.facebook.com/docs/sharing/android) (Facebook developer site).

# Summary
In this article I covered a lot of information. The article starte with information on how to register your application with Facebook, touched on app modes and the use of Test Users, and we covered access tokens and permissions. The article then covered the Facebook SDK for Android and how to integrate the Android SDK into your app and build process. We then covered a sample app that showed how to use the Login API, the Graph API, and the Sharing API.

This article just covered a fraction of what the SDK offers. The Facebook API and the related Android SDK offers a large number of APIs and functionality. I encourage you to read [Getting Started Android SDK](https://developers.facebook.com/docs/android/getting-started), and [Using the Graph API](https://developers.facebook.com/docs/graph-api/using-graph-api/), as well as reading about [Authentication Versus Data Access](https://developers.facebook.com/docs/facebook-login/auth-vs-data) and about [Access Tokens](https://developers.facebook.com/docs/facebook-login/access-tokens/#apptokens).

# About the Author
C. Enrique Ortiz is a longtime software technologist and tech author. He is author or co-author two books on mobile development, dozens of popular tech articles and presentations, and as time permits, Enrique likes to blog about tech at [http://weblog.cenriqueortiz.com](http://weblog.cenriqueortiz.com). You can follow him on [Twitter (@eortiz)](https://twitter.com/eortiz) and [LinkedIn (cenriqueortiz)](https://www.linkedin.com/in/cenriqueortiz/).