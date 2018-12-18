之前真机调试,一直以为只要在一个内网中(192.168.x.x)就可以了,把连接的ip从```localhost```改为电脑本机的ip就可以了,但是最近新换工作,公司内网并不是家用路由器那样的ip网段(以我现在的为例,为:172.26.x.x),再用之前的方法就连不上,经过一段时间的摸索,大概的解决方案如下

## 1,首先请确保电脑的ip和手机的ip在同一个路由器下
请确保电脑和手机的ip前三段是一致的(如172.26.1.1和172.26.1.2是可以的,172.26.1.1和172.26.2.1是不可以的,请确保第三段保持一致)

## 2,和以前的方法一样,修改AppDelegate中的url

现在使用rn初始化的项目代码如下
```
  NSURL *jsCodeLocation;
  jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index.ios" fallbackResource:nil];
  RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                      moduleName:@"xxx"
                                               initialProperties:nil
                                                   launchOptions:launchOptions];
```
首先断点来获取```jsCodeLocation```的值,不出意外应该是这样的
```
http://localhost:8081/index.ios.bundle?platform=ios&dev=true&minify=false
```
我们真机调试的时候就需要把这里的```localhost```改为本机的ip
如:
```
http://172.26.9.15:8081/index.ios.bundle?platform=ios&dev=true&minify=false
```

## 3,如果你的项目中使用了realm了,请注意这时候,开启debug开关是开启不了的
如果你的项目使用了realm,并且在开启debug的时候报这个错
```
Failed to load 'http://<DEVICEIP>:8083/create_session'
```
还需要再做一步
找到```Libraries->RealmReact.xcodeproj->RealmReact```中的```RealmReact.mm```文件
搜索```Access-Control-Allow-Origin```,这个key对应的值应该为```localhost```,同样需要修改为你的电脑本机ip   

至此,大功告成.
[参考连接](https://github.com/realm/realm-js/issues/1787#issuecomment-397924172)
```
The change is pretty straightforward. Open the RealmReact.mm in XCode under
Libraries->RealmReact.xcodeproj->RealmReact
search for "Access-Control-Allow-Origin", which is the name of an additional header parameter to be set to server response, and set the value of this header to "*" like setValue:@"*"

Save the change (CMD+S) and restart/rerun on device (CMD+R)

Hope that helps
```


# 最后,标题一定要大,调试之后,记得改回```localhost```,还有最好不要提交git(如果只有你一个人开发,请随意)
另外如果想多人写作的时候,想要更方便的修改这个ip(毕竟每个人的ip都是不一样的,提交到git别人也不能用),可以参考这个([点我](https://www.jianshu.com/p/ba6532a3d077))