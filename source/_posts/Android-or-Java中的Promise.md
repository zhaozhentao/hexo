---
title: Android or Java中的Promise
date: 2021-09-14 10:55:36
tags:
---

#### 写过一段时间的前端，感觉前端里面出现的一些写法，不论是在代码的阅读方面还是在开发人员的编码方面，确实能带来不少的便利。

#### 比如`async` `await` `promise`这一套，先看看javascript里面的用法。

```javascript
    async submit() {
      console.log('start')

      const res = await this.sleep()

      console.log(res)

      // 业务代码1 
      await this.sleep()
  
      // 业务代码2
      await this.sleep()
  
      // 业务代码3 
    },

    sleep() {
      return new Promise(resolve => {
        setTimeout(() => resolve('finish'), 1000)
      })
    }
```

#### 在 `async` 修饰的submit方法里面，`await` sleep方法，将会在调用sleep方法1s后才继续往下运行，这样可以使得submit方法里面的代码保持从下往上顺序，不需要多一层丑陋的嵌套。

#### 对比一下不使用 `async` `await` 的代码...，回调地狱，谜之嵌套

```javascript
submit() {
  console.log('start')

  setTimeout(() => {
    // 1s后执行业务代码1
    
    setTimeout(() => {
      // 2s后执行业务代码2
      
      setTimeout(() => {
        // 3s后执行业务代码3
      }, 1000)
    }, 1000)
  }, 1000)
}
```

#### 对比明显。

# 思考

#### 写Java代码很长的一段时间里面，尤其是Android客户端，涉及大量的用户交互操作(异步)，不得不写出了很多嵌套的代码。常见的比如下面的代码...

``` java
new AlertDialog.Builder(context)
    .setNegativeButton("取消", (dialogInterface, i) -> {
        // 点击取消后的业务代码
    })
    .setPositiveButton("确定", (dialogInterface, i) -> {
        // 点击确定后，提交表单
        Http.post('xxx url', res -> {
            // 网络响应后，更新UI
            
            // 假如需要继续请求网络，套娃...
            Http.post('xxx url', res -> {
                
            })
        })
    })
    .show();
```

#### 今天发现了一个可以实现js里面类似写法的一个库 [`parseq`](https://mvnrepository.com/artifact/com.linkedin.parseq/parseq)，借助这个库可以将很多的异步操作Promise化，使得被诟病已久的Java代码也可以变得清晰一点。下面是将弹窗操作Promise化。

```java
public class PromiseDialog {

    public static boolean confirm(Activity activity, String message) {
        SettablePromise<Boolean> promise = Promises.settable();

        activity.runOnUiThread(() -> new AlertDialog.Builder(activity)
            .setTitle("提示")
            .setMessage(message)
            .setNegativeButton("取消", (d, i) -> promise.done(false))
            .setPositiveButton("确定", (d, i1) -> promise.done(true))
            .show());

        try {
            promise.await();
            return promise.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        return false;
    }
}

// 下面是调用示例
boolean result = PromiseDialog.confirm(
    someActivity,
    "确定要删除吗？"
);

// result true or false
```

#### 经过包装后，异步的交互代码变成了同步代码，其他异步的操作也可以进行类似改造🎉🎉🎉
