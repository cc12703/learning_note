

# Retrofit学习


## 功能

* 封装HTTP库的接口
* 将HTTP请求抽象成一个接口，使用注解来描述
* 将java对象转换成请求数据，响应数据转换成java对象
  
  
## 原理

* 使用java动态代理将接口调用，转换成具体的请求
 

## 用法

### 创建接口
```java
public interface GitHub {
    @GET("/repos/{owner}/{repo}/contributors")
    Call<List<Contributor>> contributors(
        @Path("owner") String owner,
        @Path("repo") String repo);
}
```

### 创建Retrofit对象
```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .addConverterFactory(GsonConverterFactory.create())
    .build();
```

### 创建请求对象
```java
GitHub github = retrofit.create(GitHub.class);
Call<List<Contributor>> call = github.contributors("square", "retrofit");
```

### 发送请求
```java
call.enqueue(new Callback<List<Contributor>>() {
    @Override
    public void onResponse(Response<List<Contributor>> response) {
        for (Contributor contributor : response.body()) {
            System.out.println(contributor.login + " (" + contributor.contributions + ")");
        }
    }
    @Override
    public void onFailure(Throwable t) {
    }
});
```

## 接口说明

### Callback
* retrofit请求数据返回的接口
* 包括两个方法：
   - onResponse  请求成功的回调
   - onFailure      请求失败的回调

### Converter
* 将HTTP返回的数据解析成java对象

### Call
* 发送一个HTTP请求，默认失效为OkHttpCall

### CallAdapter
* 用于将Call对象转换成另一个对象