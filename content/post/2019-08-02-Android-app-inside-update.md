---
layout: post
title: Android应用内下载并安装apk
date: 2019-08-02 16:10:18
categories: [Android]
---

# FileProvider
Android 7 开始，不允许使用 file://URI 访问其他应用的私有目录文件，如调起系统安装器使用 file://URI来安装一个Apk。解决方法是：通过FileProvider提供的content://URI 访问。  
FileProvider是四大组件之一ContentProvider的子类。  
## 在AndroidManifest.xml中的<application>内配置  
    <provider
        android:name="android.support.v4.content.FileProvider"
        android:authorities="${applicationId}.fileprovider"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/filepaths" />
    </provider>

android:authorities 在Android系统必须具有唯一性，否则安装时报错。FileProvider.getUriForFile() 是通过 android:authorities 属性配置的值，来唯一确定由谁来响应这个 provider。  
## 在 res/xml 中新建 filepaths.xml，内容如下：  
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <paths>
            <external-path name="external" path="" />
            <files-path name="my_images"  path="images" />
            <cache-path name="cache" path="" />
        </paths>
    </resources>

paths的子元素可包括：  
```
<files-path>：内部存储空间应用私有目录下的 files/ 目录，等同于 Context.getFilesDir() 所获取的目录路径；
<cache-path>：内部存储空间应用私有目录下的 cache/ 目录，等同于 Context.getCacheDir() 所获取的目录路径；
<external-path>：外部存储空间根目录，等同于 Environment.getExternalStorageDirectory() 所获取的目录路径；
<external-files-path>：外部存储空间应用私有目录下的 files/ 目录，等同于 Context.getExternalFilesDir(null) 所获取的目录路径；
<external-cache-path>：外部存储空间应用私有目录下的 cache/ 目录，等同于 Context.getExternalCacheDir() 所获取的目录路径；
```
path 属性用于指定当前子元素所代表目录下需要共享的子目录名称。  
name 属性用于给 path 属性所指定的子目录名称取一个别名。  
例子：对于 /data/data/com.xxx.nodeservice/files/images/pic.png，因为它位于内部存储空间应用私有目录下的 files/ 目录，所以对应的是 `<files-path name="my_images"  path="images" />` 这一规则。  
    Uri uil = FileProvider.getUriForFile(this, BuildConfig.APPLICATION_ID + ".fileprovider",  new File('/data/data/com.xxx.nodeservice/files/images/pic.png')); 的返回值为
    content://com.xxx.nodeservice.fileprovider/my_images/pic.png  

## 生成Uri后，要授予访问权限
方式1：使用 Context.grantUriPermission(package, Uri, mode_flags) 方法向其他应用授权访问 URI 对象。三个参数分别表示授权访问 URI 对象的其他应用包名，授权访问的 Uri 对象，和授权类型。其中，授权类型为 Intent 类提供的读写类型常量：  
* FLAG_GRANT_READ_URI_PERMISSION
* FLAG_GRANT_WRITE_URI_PERMISSION

方式2：配合 Intent 使用。通过 setData() 方法向 intent 对象添加 Content URI。然后使用 setFlags() 或者 addFlags() 方法设置读写权限，可选常量值同上。  
拥有了授权权限的 content:// 的 Uri 之后，就可以通过 startActivity() 或者 setResult() 的方式，将 Uri 传递给其他的 App。  

# 下载
参见 [Android WebView 支持文件下载的几种方式 - 简书](https://www.jianshu.com/p/6e38e1ef203a)  
对于在模拟器中执行，要关闭允许漫游时下载，否则会出现下载任务不开始，状态为Queue的问题。  
    request.setAllowedOverRoaming(false);

# 未知来源安装权限
Android7 在设置打开“允许安装未知来源的应用”后，是针对整个系统所有应用的，都可安装第3方来源的应用。  
Android8 则针对每个应用分别设置，提高了安全性。不请求对应权限，不会弹出安装界面。  
## 解决方法：
1. 在AndroidManifest.xml 添加：  
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />  
2. haveInstallPermission = getPackageManager().canRequestPackageInstalls();   
    - 如haveInstallPermission 为 true，则说明应用有安装未知来源应用的权限，直接执行安装应用的操作即可。  
    - 如haveInstallPermission 为 false，则说明应用没有安装未知来源应用的权限，则无法安装应用。由于这个权限不是运行时权限，所以无法在代码中请求权限，需要用户跳转到设置界面中去打开权限。  
具体的操作是：  
    跳转到未知来源应用权限管理列表：  
    Intent intent = new Intent(Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES);  
    startActivityForResult(intent, 10086);  
    然后在onActivityResult中去接收结果：  
      if (resultCode == RESULT_OK && requestCode == 10086) {
        installProcess(); //再次执行安装流程，包含权限判等
      }

参见 [Android8.0未知来源应用安装权限最好的适配方案 - 知乎](https://zhuanlan.zhihu.com/p/32386135)  
手动撤销权限：  
    Settings –> Apps & notifications -> Advanced -> Special app access -> Install unknown apps