# v2.0 升级指南（Beta 版本）

v2.0 属于 beta 版本，仍可能存在破坏性更新。

v2.0 新增功能：

1. 云端录制：您无需通过屏幕录制等方式进行录制。当您开启了云端录制功能后，我们将自动在云端进行录制。
1. 回放功能：提供回放API
1. 白板页面管理功能

v2.0 注意内容：

1. v1.0与v2.0房间不互通；但是 SDK 所使用的 Token 不需要更新。
2. v2.0移除了部分 API，您需要通过下面的文档，使用新 API 实现。

*我们不会关闭 v1.0 的服务，但我们依然推荐你迁移到 v2.0。*

## 新增概念——场景

首先，为了增强白板的页面管理功能，我们引入一些新概念：场景以及场景目录。
如果您不能理解场景这个概念，我们建议您参考资源管理器以及文件的概念，进行参考。

场景目前主要包括：场景名，PPT（背景图），PPT宽，PPT高 这几个内容。
还有一个与场景有关，不是由场景本身持有的内容：场景路径。场景路径由场景目录+场景名，后者由场景本身持有。

场景目录，则是文件的所在目录。（SDK 中的场景目录，格式参考的是 Unix 系统下的文件格式。`\dir1\dir`）

# 修改的 API

2.0的 API 修改主要在场景这一块。为了支持更复杂的页面管理需求，我们抛弃了过去，白板是一串页面数组的形式。转而使用资源管理器的方式进行管理。

## PPT API

### 获取 ppt API

我们仍然提供获取 ppt API，但是我们不再推荐使用此 API。因为即使您获取到了 ppt 地址，也无法通过ppt地址所在的index 索引进行页面管理。所以，我们更推荐使用以下的方法获取当前页面的内容：

```Java
//返回当前场景目录下，所有的场景，ppt属性可能为空。
public void getScenes(final Promise<Scene[]> promise)

//获取的 WhiteSceneState 中，有当前场景目录，该场景目录下所有的场景列表，当前场景在场景列表中的索引。
public void getSceneState(final Promise<SceneState> promise)
```

目前，您需要自行管理场景目录。如果您没有多个场景列表（多维数组）的需求。我建议您使用固定的场景目录（例如"\"）。

### 插入 PPT API

旧方法：

```Java
public void pushPptPages(PptPage[] pages)
```

新方法：

```Java
/**
 插入，或许新建多个页面

 @param dir scence 页面组名称，相当于目录。
 @param scenes WhiteScence 实例；在生成 WhiteScence 时，可以同时配置 ppt
 @param index 选择在页面组，插入的位置。index 即为新 scence 的 index 位置。如果想要放在最末尾，可以传入 NSUIntegerMax。
 */
public void putScenes(String dir, Scene[] scenes, int index)
```

## 页面管理 API

### 删除页面 API

旧方法:

```Java
public void removePage(int index)
```

新方法：

```Java
/**
 当有
 /ppt/page0
 /ppt/page1
 传入 "/ppt/page0" 时，则只删除对应页面。
 传入 "/ppt" 时，会将两个页面一起移除。

 @param dirOrPath 页面具体路径，或者为页面组路径
 */
public void removeScenes(String dirOrPath)
```

现在删除，不再接受index 索引，对应的，接受的是场景的路径，或者是目录。

### 插入页面 API

```Java
/**
 插入，或许新建多个页面

 @param dir scence 页面组名称，相当于目录。
 @param scenes WhiteScence 实例；在生成 WhiteScence 时，可以同时配置 ppt
 @param index 选择在页面组，插入的位置。index 即为新 scence 的 index 位置。如果想要放在最末尾，可以传入 NSUIntegerMax。
 */
public void putScenes(String dir, Scene[] scenes, int index)
```

现在插入页面 API，增加了插入时，自定义内容（ppt）的接口。所以插入页面 API 和插入 PPT API，现在已经合并成了同一个 API。

*我们现在提供新API支持移动，重命名白板页面*

## 图片替换 API

由于图片替换 API，同时对互动房间与回放生效，所以我们将该设置迁移到了 WhiteSdkConfig 中，在初始化 WhiteSDK 时，就需要设置好。

```Java
WhiteSdkConfiguration sdkConfig = new WhiteSdkConfiguration(DeviceType.touch, 10, 0.1);
//必须在sdk 初始化时，就设置替换
sdkConfig.setHasUrlInterrupterAPI(true);

//初始一个实现了 UrlInterrupter interface 的类，作为 WhiteSDK 初始化参数即可。
UrlInterrupterObject interrupter = new UrlInterrupterObject()
WhiteSdk whiteSdk = new WhiteSdk(whiteBroadView PlayActivity.this, interrupter);
```