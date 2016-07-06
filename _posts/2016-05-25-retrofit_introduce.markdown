---
layout:     post
title:      "Retrofit介绍"
subtitle:   " \"Retrofit介绍\""
date:       2016-04-25 12:00:00
author:     "X-ray"
header-img: "img/post-bg-2015.jpg"
tags:
    - Android 
---

## 前言

Retrofit是Square开发的一个用于网络请求的开源库，内部封装了okhttp,并且和RxAndroid完美的兼容，使得Android的开发效率增加不少的同时也使代码变得清晰易读。



## Gradle 依赖

retrofit可以很方便的使用Maven和Gradle依赖，在1.x时retrofit默认是没有引入okhttp作为http client,需要手动的依赖。但是2.0版本已经将okhttp作为retrofit的默认http client,引入retrofit2只需要在gradle中配置



```

compile 'com.squareup.retrofit2:retrofit:2.0.0'
```

如果你不想使用retrofit2中自带的okhttp，你也可以导入你自己的okhttp,为了避免导入冲突可以按下面的依赖：



```

compile ('com.squareup.retrofit2:retrofit:2.0.0') {  

  exclude module: 'okhttp'
}
compile 'com.squareup.okhttp3:okhttp:3.2.0'  

```



retrofit2默认没有导入gson,需要gson作为转换器：



```

compile 'com.squareup.retrofit2:converter-gson:2.0.0' 

```



与RxAndroid使用需要依赖：



```

compile 'com.squareup.retrofit2:adapter-rxjava:2.0.0'  
compile 'io.reactivex:rxandroid:1.0.1'

﻿

```



## 请求参数注解说明



#### @Query @QueryMap



Http Get请求参数：



```

@GET("group/users")
Call<List<User>> groupList(@Query("id") int groupId);
```



等同于



```

@GET("group/users?id=groupId")
```



多个请求参数



```


@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);


```







#### @Field



用于Post方式传递参数,需要在请求接口方法上添加@FormUrlEncoded,表示以表单的方式传递参数



```

@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);


```



#### @Path



用于URL占位符



```

@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);


```



#### @Header



添加http header



```

@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)


```



#### @Body

请求体，对象会被自动转化成Json格式



```


@POST("users/new")
Call<User> createUser(@Body User user);


```



## 拦截器Interceptor



retrofit2默认的集成了okhttp, okhttp可以设置拦截器，比如个请求添加统一的Header



```

httpClient.addNetworkInterceptor(new Interceptor() {
    @Override
    public Response intercept(Chain chain) throws IOException {

        Request request = chain.request();
        Request.Builder requestBuilder = request.newBuilder().addHeader("Accept", "application/json");
        Request request1 = requestBuilder.build();

        return chain.proceed(request);
    }
});
```



#### Post多个参数提交



如果有很多默认的参数需要每次添加时：



```

@FormUrlEncoded
@POST("/feedback")
Call<ResponseBody> sendFeedbackSimple(  
    @Field("osName") String osName,
    @Field("osVersion") int osVersion,
    @Field("device") String device,
    @Field("message") String message,
    @Field("userIsATalker") Boolean userIsATalker);



```



```

private void sendFeedbackFormSimple(@NonNull String message) {  
    // create the service to make the call, see first Retrofit blog post
    FeedbackService taskService = ServiceGenerator.create(FeedbackService.class);

    // create flag if message is especially long
    boolean userIsATalker = (message.length() > 200);

    Call<ResponseBody> call = taskService.sendFeedbackSimple(
            "Android",
            android.os.Build.VERSION.SDK_INT,
            Build.MODEL,
            message,
            userIsATalker
    );

    call.enqueue(new Callback<ResponseBody>() {
        ...
    });
}


```

可以写成下面的形式:



```

@POST("/feedback")
Call<ResponseBody> sendFeedbackConstant(@Body UserFeedback feedbackObject);  


public class UserFeedback {

    private String osName = "Android";
    private int osVersion = android.os.Build.VERSION.SDK_INT;
    private String device = Build.MODEL;
    private String message;
    private boolean userIsATalker;

    public UserFeedback(String message) {
        this.message = message;
        this.userIsATalker = (message.length() > 200);
    }

    // getters & setters
    // ...
}


```



```



private void sendFeedbackFormAdvanced(@NonNull String message) {  
    FeedbackService taskService = ServiceGenerator.create(FeedbackService.class);

    Call<ResponseBody> call = taskService.sendFeedbackConstant(new UserFeedback(message));

    call.enqueue(new Callback<ResponseBody>() {
        ...
    });
}


```



## Retrofit2 + RxAndroid 封装和使用

首先创建一个工厂类根据不通的api interface 创建其实现。这个解释起来比较抽象，可以看下面的代码：



```

/**
 * Created by cfp on 16-4-18.
 */
public class RetrofitFactory {


    public static final String GITHUB_HTTP_URL = "https://api.github.com";

    public static final String OTHER_HTTP_URL = "https://yoursite";


    //默认超时时间
    private final static int DEFAULT_TIMEOUT = 5;

    private static OkHttpClient.Builder httpClient = new OkHttpClient.Builder();

    private static Retrofit.Builder githubBuilder = new Retrofit.Builder()
            .baseUrl(GITHUB_HTTP_URL)
            .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
            .addConverterFactory(GsonConverterFactory.create());


    private static Retrofit.Builder otherBuilder = new Retrofit.Builder()
            .baseUrl(OTHER_HTTP_URL)
            .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
            .addConverterFactory(GsonConverterFactory.create());

    /**
     * 为了防止一个类里放置所有的api,可以根据程序的功能分成不通的模块，把他们存在map中
     */
    private static Map<Class<?>, Object> githubApiMap = new HashMap<Class<?>, Object>();
    private static Map<Class<?>, Object> otherApiMap = new HashMap<Class<?>, Object>();


    /**
     * 默认的server为github
     *
     * @param instanceClass
     * @param <S>
     * @return
     */
    public synchronized static <S> S getInstance(Class<S> instanceClass) {

        return getInstance(GITHUB_HTTP_URL, instanceClass);
    }


    public RetrofitFactory() {
    }

    public synchronized static <S> S getInstance(String server, Class<S> instanceClass) {

        final Retrofit retrofit;

        switch (server) {

            case GITHUB_HTTP_URL:

                if (githubApiMap.containsKey(instanceClass)) {

                    return (S) githubApiMap.get(instanceClass);
                } else {
                    //添加拦截器，可以给请求添加header
                    httpClient.addNetworkInterceptor(new Interceptor() {
                        @Override
                        public Response intercept(Chain chain) throws IOException {

                            Request origin = chain.request();
                            Request.Builder requestBuilder = origin.newBuilder()
                                    .addHeader("devicetype", "ANDROID");
                            Request request = requestBuilder.build();

                            return chain.proceed(request);
                        }
                    });
                    retrofit = githubBuilder.client(httpClient.build()).build();
                    S instance = retrofit.create(instanceClass);
                    githubApiMap.put(instanceClass, instance);
                    return instance;


                }
            case OTHER_HTTP_URL:

                if (otherApiMap.containsKey(instanceClass)) {

                    return (S) githubApiMap.get(instanceClass);
                } else {

                    retrofit = githubBuilder.client(httpClient.build()).build();
                    S instance = retrofit.create(instanceClass);
                    otherApiMap.put(instanceClass, instance);
                    return instance;
                }

            default:
                return null;

        }

    }
}


```



另外就是网络请求的抽象回调类，继承自Subscriber



```

/**
 * 网络请求的回调抽象类
 * Created by cfp on 16-5-24.
 */
public abstract class AbsCallBackSubscriber<T> extends Subscriber<T> {
    @Override
    public void onCompleted() {

        //解除注册
        if (this.isUnsubscribed()){

            this.unsubscribe();
        }
        onFinish();
    }

    @Override
    public void onError(Throwable e) {
        // TODO: 16-5-25 一些错误处理
    }

    @Override
    public void onNext(T t) {
        onSuccess(t);
    }

    public abstract void onFinish();
    public abstract void onSuccess(T t);

}


```



最后是真正的网络请求部分，如果有需求某次请求的错误信息需要单独处理，可以复写

AbsCallBackSubscriber中的onError方法。


```

RetrofitFactory.getInstance(GithubApi.class).getFollowings(username)
        .subscribeOn(Schedulers.io())
        .unsubscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new AbsCallBackSubscriber<List<UserInfo>>() {
            @Override
            public void onFinish() {

            }

            @Override
            public void onSuccess(List<UserInfo> userInfos) {

            }
        });


```

最后是接口的interface



```

/**
 * Created by cfp on 16-4-18.
 */
public interface GithubApi {

    @GET("/users/{username}")
    Observable<UserInfo> getUserInfo(@Path("username") String username);
    @GET("/users/{username}/following")
    Observable<List<UserInfo>> getFollowings(@Path("username") String username);
}
```



## 总结

retrofit2 + reAndroid简单的流程用法差不多介绍了，也做了个简单的封装，至于更高级的用法，还需要继续的研究，也希望大家多多交流，指出不足共同进步^v^。





## 引用

1. http://www.loongwind.com/archives/242.html

2. http://xiuweikang.github.io/2016/03/17/Retrofit%E5%88%86%E4%BA%AB/

3. https://futurestud.io/blog/retrofit-2-upgrade-guide-from-1-9



















































