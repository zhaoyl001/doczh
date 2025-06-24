# 鸿蒙DNS查询工具开发笔记

## 前言

最近在搞鸿蒙开发者工具箱,想着加个DNS查询功能。这玩意主要是用来查域名解析的,比如A记录、AAAA记录、CNAME记录这些。本来以为挺麻烦的,结果发现鸿蒙对DNS查询支持得还不错,直接用系统API就能搞定。调了几次,基本功能就出来了。

## 一、功能说明

### 1.1 主要功能
- 查A记录(IPv4地址)
- 查AAAA记录(IPv6地址)
- 查CNAME记录(别名)
- 查MX记录(邮件服务器)
- 查TXT记录(文本)
- 支持批量查询
- 显示详细记录

### 1.2 界面功能
- 域名输入框
- 记录类型选择
- 查询结果显示
- 支持收藏
- 有使用说明

## 二、实现过程

### 2.1 查询原理

DNS查询说白了就是向DNS服务器发请求：

1. A记录查询
   - 查域名的IPv4地址
   - 返回一个或多个IP
   - 比如 baidu.com 返回 39.156.66.10

2. AAAA记录查询
   - 查域名的IPv6地址
   - 返回IPv6格式的地址
   - 比如 ipv6.google.com

3. CNAME记录查询
   - 查域名的别名
   - 返回另一个域名
   - 比如 www.baidu.com 指向 baidu.com

4. MX记录查询
   - 查邮件服务器
   - 返回邮件服务器域名
   - 比如 gmail.com 的邮件服务器

5. TXT记录查询
   - 查文本记录
   - 返回任意文本信息
   - 比如域名验证信息

### 2.2 代码实现

```typescript
@Entry
@Component
struct DnsQuery {
  @State domain: string = '';           // 要查的域名
  @State isQuerying: boolean = false;   // 是否在查
  @State queryType: DnsRecordType = DnsRecordType.A;  // 记录类型
  @State queryResults: string[] = [];   // 查询结果
  @State isError: boolean = false;      // 是否出错

  // 开始查询
  private async startQuery(): Promise<void> {
    if (!this.domain) {
      DnsQuery.toast('请输入要查的域名');
      return;
    }

    try {
      this.isQuerying = true;
      this.queryResults = [];
      this.isError = false;

      // 执行DNS查询
      const results = await NetworkService.dnsLookup(this.domain, this.queryType);
      this.queryResults = Array.isArray(results) ? results : [results];

      if (this.queryResults.length === 0) {
        this.isError = true;
        DnsQuery.toast('没找到DNS记录');
      }
    } catch (error) {
      this.isError = true;
      this.queryResults = [`查询失败: ${error?.message || '未知错误'}`];
      DnsQuery.toast('DNS查询失败');
    } finally {
      this.isQuerying = false;
    }
  }

  // 格式化查询结果
  private formatQueryResult(result: string): string {
    if (result.startsWith('未找到') || result.startsWith('查询失败')) {
      return result;
    }

    switch (this.queryType) {
      case DnsRecordType.A:
        return `📍 IP地址: ${result}`;
      case DnsRecordType.AAAA:
        return `🌐 IPv6地址: ${result}`;
      case DnsRecordType.CNAME:
        return `🔄 别名指向: ${result}`;
      case DnsRecordType.MX:
        return `📧 ${result}`;
      case DnsRecordType.TXT:
        return `📝 文本记录: ${result}`;
      default:
        return result;
    }
  }
}
```

## 三、踩坑记录

### 3.1 遇到的问题

1. 查询超时
   - 问题：有些域名查得太慢了
   - 解决：设置合理的超时时间

2. 结果解析
   - 问题：不同记录类型格式不一样
   - 解决：每种类型单独处理

3. 错误处理
   - 问题：查询失败提示不够友好
   - 解决：优化错误提示

4. 界面刷新
   - 问题：结果更新不及时
   - 解决：优化状态管理

### 3.2 优化建议

1. 查询优化
   - 支持更多记录类型
   - 加个查询历史
   - 支持批量查询

2. 性能优化
   - 优化查询速度
   - 减少内存占用
   - 及时释放资源

3. 用户体验
   - 加个查询历史
   - 支持分享结果
   - 优化动画效果

4. 安全性
   - 验证域名格式
   - 限制查询频率
   - 保护用户隐私

## 四、总结

这个DNS查询工具基本功能都有了,可以：

- 查询域名解析
- 诊断DNS问题
- 验证域名配置
- 排查网络问题

## 五、参考资料

- [鸿蒙应用开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## 欢迎体验

这个DNS查询工具已经集成到鸿蒙开发者工具箱里了,欢迎下载体验！

[鸿蒙开发者工具箱](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> 作者：在人间耕耘
> 邮箱：1743914721@qq.com
> 版权声明：本文为博主原创文章，转载请附上原文出处链接及本声明。 