
## flutter 修改Gradle镜像
```
// 路径flutter\packages\flutter_tools\gradle\flutter.gradle
buildscript {
    repositories {
        google()
        jcenter()
        //添加内容
        //maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/repository/jcenter' }
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public' }
 
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.0'
    }
}
```
### 好用flutter 工具类
```

https://github.com/Sky24n/flustars
```
### flutter 裸眼3d参考
https://juejin.cn/post/6991409083765129229    


### 在线代码
https://dartpad.cn/dart

### 书
https://book.flutterchina.club/

### 包
https://pub.flutter-io.cn/
### icons
https://fonts.google.com/icons?selected=Material+Icons 
### 快速api说明 可视化
http://toly1994328.gitee.io/flutter_web/#/

#### githubapp 实战参考
https://www.jianshu.com/p/d1cd2c516980

### 老孟api
http://laomengit.com/flutter/widgets/AboutListTile.html

### 手机调试问题

 修改flutter默认安卓sdk位置
 https://www.cnblogs.com/joe235/p/10813075.html
 ```
  flutter config --android-sdk "E:\RicSoftWare\AndroidSDK"
 ```
### flutter 游戏引擎  flame
https://doc.flame-cn.com/docs/2.%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B/%E7%AB%8B%E5%8D%B3%E5%BC%80%E5%A7%8B/%E7%AB%8B%E5%8D%B3%E5%BC%80%E5%A7%8B.html  

### flutter 架构大全
https://juejin.cn/post/6844903890098339848

### 识别
https://www.jianshu.com/p/3aa810b35a5d

### 镜像
https://flutter.dev/community/china
#### 镜像大全
```
PUB_HOSTED_URL=https://dart-pub.mirrors.sjtug.sjtu.edu.cn
FLUTTER_STORAGE_BASE_URL=https://mirrors.sjtug.sjtu.edu.cn

清华大学 TUNA 协会的镜像站（与Flutter社区同步，推荐）
PUB_HOSTED_URL=https://mirrors.tuna.tsinghua.edu.cn/dart-pub
FLUTTER_STORAGE_BASE_URL=https://mirrors.tuna.tsinghua.edu.cn/flutter

CNNIC的镜像站 （与TUNA同步）
PUB_HOSTED_URL=http://mirrors.cnnic.cn/dart-pub
FLUTTER_STORAGE_BASE_URL=http://mirrors.cnnic.cn/flutter

腾讯开源的镜像站（与TUNA同步，定时更新）
PUB_HOSTED_URL=https://mirrors.cloud.tencent.com/dart-pub
FLUTTER_STORAGE_BASE_URL=https://mirrors.cloud.tencent.com/flutter


```


#### 切换chrome调试模式
```
flutter run -d chrome

```

#### vscode即时翻译插件
code-translate

