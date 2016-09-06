---
layout: post
title:  "Firebase初探：身份认证"
date:   2016-08-26 00:32
categories: Android
tags: Android Firebase
---


* content
{:toc}

用户使用app的第一步，就是身份认证，包括注册，登陆还有登出功能。Firebase的Auth模块就是对应身份认证这一点的。Firebase的登陆方式除了基本的邮箱+密码方式，还有第三方账号的登陆方式，包括Google，Facebook，twitter和Gihub，除此之外，Firebase也允许开发者自定义登陆方式
在启用某种登录方式时，需要在Firebase控制台的Auth模块中，在”登录方法“标签页下进行对应的启用





## 邮箱+密码

首先先来看最基本的邮件+密码方式。
为了使用身份认证功能，需要为项目添加对firebase-auth库的依赖：

```
dependencies {
	// ...
	compile 'com.google.firebase:firebase-auth:9.0.0'
}
```

FirebaseAuth是Firebase身份认证的入口，就是由它来完成身份认证的各种功能
FirebaseAuth使用了单例模式，应用的FirebaseAuth对象通过FirebaseAuth.getInstance()获得（说是单例但其实不是很准确，因为getInstance()方法的说明是”返回默认的FirebaseApp实例所对应的FirebaseAuth实例“，该方法也有一个接收FirebaseApp参数的重载版本，但是应用通常不需要和FirebaseApp交互，所使用的FirebaseApp实例就是默认的版本，所以狭义地来说是”单例的“）

### 注册
FirebaseAuth的注册方法是：

```java
public Task<AuthResult> createUserWithEmailAndPassword (String email, String password)
```

其中这两个参数分别是邮件和密码。方法将会立即返回一个Task对象，Task类是用于处理Firebase异步操作的，必须使用一个TResult作为类型参数，表示此次操作的结果。对于身份，认证这里最终会返回一个AuthResult对象，表示此次认证操作的结果
为了监听认证操作的结果，需要为这个Task对象添加一个OnCompleteListener：

```java
public Task<TResult> addOnCompleteListener (OnCompleteListener<TResult> listener)
```

在对OnCompleteListener的实现中，需要实现onComplete方法，
我们可以根据Task对象的isSuccessful()方法来判断此次操作是否成功，然后进行相应的逻辑
所以，总体的代码是：

```java
mAthu.createUserWithEmailAndPassword(
             email.getText().toString(),
             password.getText().toString()
         )
         .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
             @Override
             public void onComplete(@NonNull Task<AuthResult> task) {
                 if (!task.isSuccessful()) {
                     // ...
                 } else {
                     // ...
                 }
             }
         });
```

当注册成功后，用户会自动登录，即FirebaseAuth会相应地发生改变，为了监听FirebaseAuth状态的变化，可以给其添加一个AuthStateListener。需要注意的是，因为FirebaseAuth是单例的，而且可能会在多个多个活动中使用到FirebasAuth（比如登入，登出，注册等等），所以需要在活动start和stop的时候分别进行add和remove监听器，以防发生冲突：

```java
private FirebaseAuth.AuthStateListener mAuthListener;

@Override
protected void onCreate(Bundle savedInstanceState) {

    mAuthListener = new FirebaseAuth.AuthStateListener() {
        @Override
        public void onAuthStateChanged(@NonNull FirebaseAuth firebaseAuth) {
            FirebaseUser user = firebaseAuth.getCurrentUser();
            if (user != null) {
                // 用户已登入或发生改变
            } else {
                // 用户已注销
            }
        }
    };
}

@Override
public void onStart() {
    super.onStart();
    mAuth.addAuthStateListener(mAuthListener);
}

@Override
public void onStop() {
    super.onStop();
    if (mAuthListener != null) {
        mAuth.removeAuthStateListener(mAuthListener);
    }
}
```

通过getCurrentUser()得到对应的FirebaseUser对象，它提供了当前用户的各种用户信息，例如：
|方法|作用|
|:---------|:------------|
|String getUid()|返回用户的uid|
|String getDisplayName()|返回用户名|
|String getEmail()|返回用户email|
|String getPhotoUrl()|返回用户头像|
|boolean isAnonymous()|是否匿名|
|Task<Void> updatePassword(String)|更新密码|
|Task<Void> updateEmail(String)|更新email|



### 登录和注销
FirebaseAuth的邮箱登录和注销方法分别是

```java
Task<AuthResult> signInWithEmailAndPassword(String, String)
void SignOut()
```

登录和注册实现的区别，就是按钮点击事件中把对应的方法替换即可，其他的监听代码部分，除了处理逻辑外大致相同


----------
## Google登录
Google Play Services SDK中也提供了使用Google账号的验证方式
为了使用这个功能，除了必须的firebase-auth库，还需要为项目添加依赖：

```
dependencies {
	compile 'com.google.android.gms:play-services-auth:9.0.0'
}
```

### 配置Google Sign In方案
#### GoogleSignInOptions
GoogleSignInOptions将为后面的登录设定一些选项，它使用建造者模式来产生一个对象。通常会使用默认的DEFAULT_SIGN_IN，然后再为其添加一些额外选项。注意，如果要使用后端服务器或者Google Sign-In进行认证，必须为其提供服务器的OAuth2.0客户端ID，对于Google Sign-In，在将google-services.json添加到项目中并sync完成后，XML中将会自动地加入一系列地配置信息，其中就包括默认的服务器客户端ID：

```
GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
                .requestIdToken(getString(R.string.default_web_client_id))
                .requestEmail()
                .build();
```

#### GoogleApiClient
GoogleSignInOptions是用来提供登录选项的，那谁来负责登录呢？就是GoogelApiClient了，关于GoogleApiClient的生成比较复杂，根据官方的建议，一般的写法是：

```java
mGoogleApiClient = new GoogleApiClient.Builder(this)
        .enableAutoManage(this /* FragmentActivity */, this /* OnConnectionFailedListener */)
        .addApi(Auth.GOOGLE_SIGN_IN_API, gso)
        .build();
```

需要注意的是，enableAutoManage方法是用一个FragmentActivity来自动管理GoogleApiClient，一般是当前所在的活动，该活动必须继承于FragmentActivity或AppCompatActivity，否则就必须手动管理GoogleApiClient的连接周期。另一个参数则是用来监听连接失败时间的监听器
在addApi方法中，传入之前创建的GoogleSignInOptions对象

#### SignInButton
Google推荐使用专门的Button来作登录按钮：
![](https://developers.google.com/identity/images/btn_google_signin_light_normal_web.png)

### 

这个控件就是SignInButton。它提供了setSize()，setColorScheme()和setScopes()来为外观进行定制
|set方法|预设常量|
|:-----|:-------|
|setSize(int)|SIZE_ICON_ONLY<br>SIZE_STANDARD <br>SIZE_WIDE|
|setColorScheme(int)|COLOR_AUTO<br>COLOR_DARK<br>COLOR_LIGHT|

需要使用到的三个部件介绍完了，接下来就是Google Sign-In的登录流程：

 1. SignInButton点击事件
 
	
	```java
	Intent signInIntent = Auth.GoogleSignInApi.getSignInIntent(mGoogleApiClient);
	startActivityForResult(signInIntent, RC_SIGN_IN);
	```

	Google账号登录需要开启一个专门的活动来进行账号的选择

 2. 设置回调逻辑
 
	```java
	@Override
	public void onActivityResult(int requestCode, int resultCode, Intent data) {
	    super.onActivityResult(requestCode, resultCode, data);
	
	    // Result returned from launching the Intent from GoogleSignInApi.getSignInIntent(...);
	    if (requestCode == RC_SIGN_IN) {
	        GoogleSignInResult result = Auth.GoogleSignInApi.getSignInResultFromIntent(data);
	        if (result.isSuccess()) {
		        // ...
	        } else {
		        // ...
	        }
	    }
	}
	```

Google Sign-In的介绍就大概是这样了
至于第三方账号的支持，如Facebook，Twitter和Github，流程基本都是根据对应SDK获得对应的令牌对象，然后根据令牌创建AuthCredential，最后FirebaseAuth调用signInWithCredential()方法进行登录即可


----------
由于服务器自定义认证的方式涉及到服务器的配置，所以介绍的篇幅会比较长，这里就不作介绍，等到以后有机会再写吧
Firebase的身份认证模块就介绍到这了，后面将会持续对其他模块进行学习