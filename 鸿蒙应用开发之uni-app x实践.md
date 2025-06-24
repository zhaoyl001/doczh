# 鸿蒙应用开发之uni-app x实践

## 前言

最近在开发鸿蒙应用时，发现uni-app x从4.61版本开始支持纯血鸿蒙（Harmony next），可以直接编译成ArkTS原生应用。这里记录一下开发过程中的一些经验和踩过的坑。

## 一、环境搭建

### 1.1 开发工具
- HBuilderX 4.61+（必须）
- DevEco Studio 5.0.7.210+（必须）
- 鸿蒙手机 API版本 14+（必须）

### 1.2 踩坑记录
1. DevEco Studio安装
   - 下载特别大，10G+
   - 安装时注意磁盘空间
   - 建议用SSD安装，机械硬盘太慢

2. 证书问题
   - 调试证书要自己申请
   - 真机调试必须签名
   - 证书有效期要注意

## 二、开发过程

### 2.1 项目创建
1. HBuilderX新建项目
2. 选鸿蒙平台
3. 配置manifest.json
4. 配置harmony-config

### 2.2 开发中遇到的问题

1. 编译问题
   - 每次改代码都要重新build
   - 不能热更新，很烦
   - 断点调试倒是可以用

2. 性能问题
   - 内存泄漏要特别注意
   - ArkTS引擎还在优化中
   - 复杂动画要谨慎使用

3. 界面问题
   - 横屏不支持
   - rpx不能用
   - 字体加载要手动更新

### 2.3 代码示例

```typescript
// 一个简单的页面
@Entry
@Component
struct Index {
  @State message: string = 'Hello HarmonyOS'

  build() {
    Column() {
      Text(this.message)
        .fontSize(50)
        .fontWeight(FontWeight.Bold)
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

### 2.4 模块配置

manifest.json配置示例：
```json
{
  "app-harmony": {
    "distribute": {
      "modules": {
        "uni-location": {
          "system": {} // 定位模块
        },
        "uni-map": {
          "tencent": {} // 地图模块
        },
        "uni-oauth": {
          "huawei": {} // 华为登录
        }
        // 其他模块...
      }
    }
  }
}
```

### 2.5 权限配置

harmony-config/permissions.json:
```json
{
  "permissions": [
    "ohos.permission.INTERNET",
    "ohos.permission.LOCATION",
    "ohos.permission.READ_MEDIA",
    "ohos.permission.WRITE_MEDIA"
  ]
}
```

## 三、调试和发布

### 3.1 调试方法
1. HBuilderX调试
   - 日志查看
   - 断点调试
   - 性能分析

2. DevEco Studio调试
   - 内存分析
   - 性能分析
   - 深度调试

### 3.2 发布流程
1. 准备证书
2. 配置信息
3. 打包签名
4. 上传市场

## 四、踩坑记录

### 4.1 开发坑
1. Windows路径问题
   - 路径太长会报错
   - 项目路径要短
   - uni_modules目录名要短

2. 权限问题
   - 权限要手动配
   - 申请流程复杂
   - 容易漏配

3. 组件问题
   - rich-text有问题
   - animateTo动画有问题
   - 部分组件不兼容

### 4.2 性能坑
1. 内存问题
   - 要及时释放
   - 避免泄漏
   - 注意循环引用

2. 渲染问题
   - 组件嵌套要控制
   - 避免过度渲染
   - 注意性能开销

## 五、经验总结

### 5.1 开发建议
1. 用TypeScript
2. 按规范开发
3. 做好错误处理
4. 注意性能优化
5. 模块要按需引入

### 5.2 项目结构
```
project/
├── src/
│   ├── pages/
│   ├── components/
│   ├── utils/
│   └── static/
├── harmony-config/
└── manifest.json
```

## 六、总结

uni-app x开发鸿蒙应用，虽然现在还有很多限制，但基本功能都能实现。开发过程中要注意性能优化，避免内存泄漏。随着鸿蒙系统的完善，开发体验应该会越来越好。

## 参考资料
- [uni-app x鸿蒙开发指南](https://doc.dcloud.net.cn/uni-app-x/app-harmony/)
- [鸿蒙开发文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide) 