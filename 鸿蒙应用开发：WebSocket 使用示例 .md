# 鸿蒙应用开发：WebSocket 使用示例

## 前言
最近在开发鸿蒙应用时，遇到了需要实现实时通信的需求。经过一番研究，发现鸿蒙 5.0 提供了更完善的 WebSocket 支持，于是就写了个简单的 demo 来测试。下面分享一下我的实现过程，希望能帮到有同样需求的同学。

## 开发环境
- DevEco Studio 4.0
- HarmonyOS SDK API 14（鸿蒙 5.0）
- 测试设备：华为 Mate 60 Pro

## 实现过程

### 1. 先导入需要的包
```typescript
import { webSocket } from '@kit.NetworkKit';
import { BusinessError } from '@kit.BasicServicesKit';
```

### 2. 写个简单的组件
这里我写了一个简单的组件，主要用来测试 WebSocket 的 ping 功能。鸿蒙 5.0 的组件系统有了很大改进，写起来更顺手了。

```typescript
@Entry
@Component
struct WebSocketExample {
  @State pingResult: string = '';
  private ws: webSocket.WebSocket | undefined = undefined;
  private pingStartTime: number = 0;
  private pingCount: number = 0;
  private totalPingTime: number = 0;

  // 组件创建时初始化
  aboutToAppear() {
    this.initWebSocket();
  }

  // 组件销毁时记得关闭连接
  aboutToDisappear() {
    this.closeWebSocket();
  }
}
```

### 3. 初始化 WebSocket
鸿蒙 5.0 的 WebSocket API 做了不少优化，错误处理更完善了。这部分代码看起来有点长，但其实就是几个事件监听器：

```typescript
private initWebSocket() {
  // 创建实例
  this.ws = webSocket.createWebSocket();
  
  // 连接成功时的处理
  this.ws.on('open', (err: BusinessError, value: Object) => {
    if (err) {
      console.error('连接失败:', JSON.stringify(err));
      this.pingResult = '连接失败';
      return;
    }
    console.log('连接成功啦！');
  });

  // 收到消息时的处理
  this.ws.on('message', (error: BusinessError, value: string | ArrayBuffer) => {
    if (error) {
      console.error('消息错误:', JSON.stringify(error));
      return;
    }
    
    // 如果是 pong 消息，计算一下延迟
    if (value === 'pong') {
      const endTime = Date.now();
      const duration = endTime - this.pingStartTime;
      this.pingCount++;
      this.totalPingTime += duration;
      const avgPing = this.totalPingTime / this.pingCount;
      this.pingResult = `延迟: ${duration}ms (平均: ${avgPing.toFixed(2)}ms)`;
    }
  });

  // 错误处理
  this.ws.on('error', (err: BusinessError) => {
    console.error('出错了:', JSON.stringify(err));
    this.pingResult = '连接错误';
  });

  // 连接关闭时的处理
  this.ws.on('close', (err: BusinessError, value: webSocket.CloseResult) => {
    console.log(`连接关闭了: ${value.code} - ${value.reason}`);
  });

  // 开始连接服务器
  this.ws.connect('ws://your-server-address', {
    header: {
      'Content-Type': 'application/json'
    },
    // API 14 新增的配置项
    protocols: ['my-protocol'],
    timeout: 10000, // 10秒超时
    proxy: {
      host: '192.168.0.150',
      port: 8888,
      exclusionList: []
    }
  }, (err: BusinessError, value: boolean) => {
    if (err) {
      console.error('连接失败:', JSON.stringify(err));
      this.pingResult = '连接失败';
    }
  });
}
```

### 4. 实现 Ping 功能
鸿蒙 5.0 的 WebSocket 发送消息更稳定了，这个功能很简单：

```typescript
private sendPing() {
  if (!this.ws) {
    this.pingResult = '还没连接呢';
    return;
  }

  this.pingStartTime = Date.now();
  this.ws.send('ping', (err: BusinessError, value: boolean) => {
    if (err) {
      console.error('发送失败:', JSON.stringify(err));
      this.pingResult = '发送失败';
    }
  });
}
```

### 5. 关闭连接
API 14 的关闭连接更可靠了：

```typescript
private closeWebSocket() {
  if (this.ws) {
    this.ws.close((err: BusinessError) => {
      if (err) {
        console.error('关闭失败:', JSON.stringify(err));
      }
    });
  }
}
```

### 6. 写个简单的界面
鸿蒙 5.0 的 UI 系统有了很大改进，写起来更顺手了：

```typescript
build() {
  Column() {
    Button('测试延迟')
      .onClick(() => this.sendPing())
    Text(this.pingResult)
      .fontSize(16)
      .margin({ top: 10 })
  }
  .width('100%')
  .height('100%')
  .justifyContent(FlexAlign.Center)
}
```

## 踩过的坑
1. 刚开始没注意在组件销毁时关闭连接，导致内存泄漏
2. 错误处理没做好，导致应用崩溃
3. 连接超时时间设置太短，经常连接失败
4. 忘记处理重连逻辑，网络不稳定时体验不好
5. API 14 新增的配置项没用好，导致连接不稳定

## 总结
这个 demo 虽然简单，但基本覆盖了 WebSocket 的主要功能。鸿蒙 5.0 的 WebSocket API 做了很多改进，使用起来更稳定了。实际项目中可能还需要添加：
- 自动重连
- 心跳检测
- 消息队列
- 断线重传
等功能，具体看项目需求吧。

## 参考资料
- [鸿蒙 5.0 WebSocket API 文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)
- [WebSocket 协议规范](https://tools.ietf.org/html/rfc6455)

## 后记
写这篇文章的时候，发现鸿蒙 5.0 的文档更新了不少，API 也变得更易用了。特别是 WebSocket 模块，增加了不少实用的功能。希望这篇文章能帮到正在学习鸿蒙开发的同学。如果有问题，欢迎在评论区讨论。 
