# 数据埋点

### 什么是数据埋点？

数据埋点针对客户端或者网页应用，对其中某些或全部流程记录用户操作的日志信息，用来分析当前应用的使用情况，建立用户画像，做好数据埋点可以为产品的发展优化提供数据支持。

可以记录的参量，访问数（Visits），访客数（Visitor），用户停留时长（Time On Site），页面浏览数（Page Views）和跳出率（Bounce Rate）等。

### 鲁大师客户端是如何实现数据打点的呢？

创建 image 对象来提交进行异步操作的监控和处理

为什么要创建一个 image 对象呢，**创建图像对象并使用其加载事件是**一**种通用的异步操作处理方式**，用于在操作完成或失败时通知代码，而不会阻塞页面或应用程序的其他部分。在这种情况下，图像对象被用来表示异步操作的状态，而不是实际加载图像的操作。

```tsx
/**
 * 打点操作
 * @param action
 * @param type
 * @param extra 额外属性
 */
export function sendStat(
  action: string,
  type: string = "monitor_panel",
  platform: string = "url2",
  extra: Dict = {}
) {
  const statParam = store.getState().global.statParam;
  if (isEmpty(statParam)) return Promise.resolve();
  const { pid, mid, mid2, appver, modver } = statParam;
  const query = {
    type,
    action,
    pid,
    mid,
    mid2,
    appver,
    modver,
    timestamp: Date.now(),
    ...extra,
  };
  return new Promise((resolve) => {
    let img = new Image();
    img.onload = img.onerror = () => {
      // @ts-ignore
      img = null;
      resolve("");
    };
    img.src = `http://s.ludashi.com/${platform}?${stringify(query, {
      skipNulls: true,
    })}`;
  });
}
```

### 书写一个完整的打点操作

1. 异步获取 statParam, baseData。（获取时机：主窗口被创建的时候，MainWebWndCreated）

const [statParam, baseData] = await _Promise_.all([CefGetStatisticParam(), CefGetBaseData()]);

<aside>
💡 在electron中怎么去描述主窗口建立的，这个主窗口的概念是什么，怎么去理解

</aside>

1. 将获取到的数据库存入数据仓库

为什么要获取 ProductDir （当前项目的安装路径）

读取本地配置并保存在数据仓库

# 广播

<aside>
💡 在前端的哪些领域用到了广播这一技术的

</aside>

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5a71c159-2e84-4634-b35b-e45e5286c0af/152c45a8-ef01-44b3-b9e6-14b51ef74d63/Untitled.png)

[收获](https://www.notion.so/9db3311fb4514360bae6b76fa8b68f41?pvs=21)

### 为什么是 JSON？

json 是一种用于在**数据交换**中存储和表示结构化数据的轻量级文本格式，它具有跨语言跨平台的特性，所以适合数据交换，它支持嵌套所以可以表示复杂的关系和层次结构

# 窗口大小及位置计算

主窗口设置居中

step1. 获取电脑工作区

step2: 计算居中

step3: 设置窗口展现层级

```tsx
// 设置位置居中
// 先获取电脑工作区
const { x, y, width, height } = await CefGetDesktopWorkArea();
// 计算居中
const maxX = max([0, Number(width) - DEFAULT_WIDTH]);
const maxY = max([0, Number(height) - DEFAULT_HEIGHT]);
const centerX = (maxX || 0) / 2;
const centerY = (maxY || 0) / 2;
CefSetWndRect({
  width: DEFAULT_WIDTH,
  height: DEFAULT_HEIGHT,
  x: Number(x) + centerX,
  y: Number(y) + centerY,
});
CefShowWnd("3");
setTimeout(() => {
  CefShowWnd("1");
}, 500);
```
