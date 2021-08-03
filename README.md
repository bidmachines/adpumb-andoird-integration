# ADPUMP android integration guide
## Prerequisite ##
You need to register with us before starting the integration. You can drop a mail to asil@rynvpn.com
## Steps ##
1) Add your admob publisher id to the android-manifest.xml of your app: Adpump doesn't use your admob adunits, however adpump uses the underlying admob APIs for which publisher id is mandatory. Even those admob accounts which are having ad limitation will work fine with Adpump.

2) Add library dependency: Adpump is currently not hosted in maven central, hence you need to add the repository details to your gradle script to get the Adpump dependency resolved. 

Please add the following to you build.gradle file of your app
```gradle
repositories {
    maven {
        url 'https://maven.adpumb.com/nexus/content/repositories/adpump'
    }
}
dependencies {
    implementation 'com.adpump:bidmachine:0.25'
    *********************

```
3) Register Adpump library: You should register Adpump at the entry point of your application. Adpump takes some time to load the advertisements, it's better to register as soon as possible.
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Adpump.register(this,BuildConfig.DEBUG); // here this points to the current activity, generally the main activity. 
        setContentView(R.layout.activity_main);
        .........................
        }
```
4) Create placement: Adpump is designed on the concept of placement rather than adunit. A placement is a predefined action sequence which ends up in showing an Ad. Consider the example of a calculator, where a user presses the addition (+) button and an ad is shown. Here we can consider the addition button click as a placement.
```java
private void onAdditionButtonClick() {
   InterstitialPlacement addition = new InterstitialPlacementBuilder()
            .name("addition") //Name of the placement is very important. Revenue dashboard will track the placement based on the name given. 
            .showLoaderTillAdIsReady(true)
            .loaderTimeOutInSeconds(5)
            .frequencyCapInSeconds(1)
            .build();            
   DisplayManager.getInstance().showAd(addition);
}            
```
In this example if the user presses the addition button before the ad is loaded/received from the server, then a loader will be shown. If the ad is not ready within 5 seconds then the loader will be removed. However if the ad is already loaded or it got loaded while the loader is shown, then ad loader will be hidden and ad will be shown to user.
For a particular placement you need to create only one placement object, which can be used to show multiple ads.
For example:

```java
private static InterstitialPlacement addition = new InterstitialPlacementBuilder()
            .name("addition").build();
            
public void onResume(){
  DisplayManager.getInstance().showAd(addition);
}
```
5) Callbacks: You can register callbacks to the placement
```java
InterstitialPlacement placement = new InterstitialPlacementBuilder()
                .name("division")
                .frequencyCapInSeconds(0)
                .showLoaderTillAdIsReady(true)
                .loaderTimeOutInSeconds(10000)
                .onAdCompletion(new AdCompletion() {
                    @Override
                    public void onAdCompletion(boolean isSuccess, PlacementDisplayStatus status) {
                        if(isSuccess){
                            Toast.makeText(MainActivity.this, "Thank you for watch the ad", Toast.LENGTH_LONG).show();
                        }else{
                            Toast.makeText(MainActivity.this, "Why you didnt watch the ad?", Toast.LENGTH_LONG).show();
                        }
                    }
                }).build();
        DisplayManager.getInstance().showAd(placement);
```
6) Customising loader animation: You can customize the loader using the loader settings for each placement
```java
        LoaderSettings loaderSettings = new LoaderSettings();
        loaderSettings.setLogoResID(R.drawable.arithmatic_button);
        loaderSettings.setMessageStyle(R.color.colorAccent, R.color.colorPrimary);
        //there are more options in loader settings which you can try
        InterstitialPlacement buttonPlacement = new InterstitialPlacementBuilder()
                .name("multiplication")  
                .loaderUISetting(loaderSettings)
                .showLoaderTillAdIsReady(true) //this will show loader anima
                .build();
        DisplayManager.getInstance().showAd(buttonPlacement);
```


