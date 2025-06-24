# uni-app/uniappx 中调用鸿蒙原生扫码能力的实践

## 一、背景介绍

最近在开发一个鸿蒙应用时，遇到了扫码功能的需求。之前用过很多扫码方案，但都不太理想。直到发现了 hmos-scan 这个插件，终于解决了我们的痛点。下面分享一下使用心得。

### 1.1 为什么选择 hmos-scan？

说实话，之前踩过不少坑：

1. **传统扫码方案太坑了**：
   - WebView 扫码慢得要死，经常卡住
   - 引入第三方库后，应用体积直接翻倍
   - 不同手机表现不一样，有的能扫，有的扫不了
   - 稍微模糊一点的码就识别不出来，用户体验太差

2. **原生开发太痛苦**：
   - 写原生代码太费时间了
   - 每个平台都要写一遍，累死
   - 维护起来特别麻烦
   - 开发周期太长，老板等不及

3. **hmos-scan 真香**：
   - 用鸿蒙原生能力，扫码贼快
   - 识别率特别高，歪着扫都能识别
   - 几行代码就搞定了，太方便了
   - 性能好，不占内存
   - 还能从相册选图，太贴心了

### 1.2 实际使用案例

1. **电商比价**：
   ```typescript
   // 扫商品码比价
   async function scanProduct() {
     try {
       const barcode = await scanapiSync()
       // 调用比价接口
       const priceInfo = await comparePrice(barcode)
       showPriceResult(priceInfo)
     } catch (error) {
       showError('扫码失败，重试一下')
     }
   }
   ```

2. **快递扫描**：
   ```typescript
   // 扫快递单号
   async function scanExpress() {
     try {
       const trackingNumber = await scanapiSync()
       // 查物流信息
       const expressInfo = await queryExpress(trackingNumber)
       showExpressInfo(expressInfo)
     } catch (error) {
       showError('扫码失败，重试一下')
     }
   }
   ```

3. **会议签到**：
   ```typescript
   // 扫会议码签到
   async function scanMeeting() {
     try {
       const meetingCode = await scanapiSync()
       // 验证会议码
       const checkInResult = await verifyMeeting(meetingCode)
       showCheckInResult(checkInResult)
     } catch (error) {
       showError('签到失败，重试一下')
     }
   }
   ```

## 二、环境准备

1. **开发工具**：
   - HBuilderX 3.8.0 或以上版本
   - DevEco Studio（鸿蒙开发必备）

2. **项目要求**：
   - 用 uni-app x 框架
   - 选 Vue 3 就对了

## 三、插件使用

### 1. 插件安装

1. 去插件市场：[hmos-scan 插件](https://ext.dcloud.net.cn/plugin?id=23789)
2. 下载后导入 HBuilderX 就完事了

## 四、在项目中使用

### 1. 基础示例

```vue
<!-- pages/index/index.uvue -->
<template>
  <view class="content">
    <button @click="startScan">开始扫描</button>
    <text v-if="scanResult">扫描结果：{{scanResult}}</text>
  </view>
</template>

<script>
import { scanapiSync } from "@/uni_modules/hmos-scan/utssdk/app-harmony";

export default {
  data() {
    return {
      scanResult: ''
    }
  },
  methods: {
    async startScan() {
      try {
        const result = await scanapiSync()
        this.scanResult = result
        console.log('扫描结果：', result)
      } catch (error) {
        console.error('扫描失败：', error)
        this.scanResult = '扫描失败'
      }
    }
  }
}
</script>

<style>
.content {
  padding: 20px;
}
button {
  margin: 20px 0;
}
</style>
```

### 2. 高级示例（带历史记录）

```vue
<!-- pages/advanced/index.uvue -->
<template>
  <view class="container">
    <view class="scan-area">
      <button @click="startScan" :disabled="isScanning">
        {{isScanning ? '扫描中...' : '开始扫描'}}
      </button>
    </view>
    
    <view class="result-area" v-if="scanHistory.length > 0">
      <text class="title">扫描历史</text>
      <view v-for="(item, index) in scanHistory" :key="index" class="history-item">
        <text class="time">{{item.time}}</text>
        <text class="content">{{item.content}}</text>
      </view>
    </view>
  </view>
</template>

<script>
import { scanapiSync } from "@/uni_modules/hmos-scan/utssdk/app-harmony";

export default {
  data() {
    return {
      isScanning: false,
      scanHistory: []
    }
  },
  methods: {
    async startScan() {
      if (this.isScanning) return
      
      this.isScanning = true
      try {
        const result = await scanapiSync()
        this.scanHistory.unshift({
          time: new Date().toLocaleTimeString(),
          content: result
        })
      } catch (error) {
        console.error('扫描失败：', error)
      } finally {
        this.isScanning = false
      }
    }
  }
}
</script>

<style>
.container {
  padding: 20px;
}
.scan-area {
  margin-bottom: 20px;
}
.result-area {
  border-top: 1px solid #eee;
  padding-top: 20px;
}
.title {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 10px;
}
.history-item {
  padding: 10px;
  border-bottom: 1px solid #eee;
}
.time {
  font-size: 12px;
  color: #999;
}
.content {
  margin-top: 5px;
}
</style>
```

## 五、功能特点

1. **多模式支持**：
   - 二维码、条形码都能扫
   - 相册选图也支持

2. **错误处理**：
   - 各种异常都处理好了
   - 提示信息很友好
   - 日志记录很详细

3. **用户体验**：
   - 操作简单，一看就会
   - 有状态反馈，不会卡住
   - 异步处理，不阻塞界面

## 六、注意事项

1. **兼容性**：
   - 只支持鸿蒙系统
   - 确保设备有扫码功能

2. **性能优化**：
   - 注意内存使用
   - 及时释放资源
   - 别重复扫描

## 七、常见问题

1. **扫描失败**：
   - 看看设备支不支持
   - 查查日志找原因

2. **结果解析错误**：
   - 检查结果格式
   - 处理各种返回类型
   - 加好错误处理

## 八、总结

用了 hmos-scan 插件后，扫码功能开发变得特别简单。原生功能完整保留，开发体验又好，强烈推荐！

## 九、参考资料

1. [uni-app x 开发文档](https://uniapp.dcloud.net.cn/)
2. [鸿蒙开发文档](https://developer.huawei.com/consumer/cn/)
3. [UTS 插件开发指南](https://uniapp.dcloud.net.cn/plugin/uts-plugin.html)
4. [hmos-scan 插件下载地址](https://ext.dcloud.net.cn/plugin?id=23789) 