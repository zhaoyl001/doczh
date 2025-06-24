# 鸿蒙IPv4子网计算器开发笔记

## 基础知识

先来唠唠IP地址和子网这些基础知识,免得后面看不懂：

1. IP地址是啥
   - 就是给网络设备分配的地址
   - 比如你家电脑的IP可能是 192.168.1.100
   - 每个数字范围是0-255
   - 总共4个数字,用点分开

2. 子网掩码干啥用
   - 用来区分网络号和主机号
   - 比如 255.255.255.0
   - 1的部分是网络号
   - 0的部分是主机号

3. CIDR是啥
   - 就是子网掩码的简写
   - 比如 /24 就是 255.255.255.0
   - 数字表示1的个数
   - 范围是0-32

4. 网络地址咋算
   - IP地址和子网掩码做与运算
   - 比如 192.168.1.100/24
   - 网络地址就是 192.168.1.0

5. 广播地址干啥用
   - 用来给整个子网发消息
   - 就是网络地址最后一段改成255
   - 比如 192.168.1.255

## 前言

最近在搞鸿蒙开发者工具箱,想着加个子网计算器功能。这玩意主要是用来算IP地址的子网信息的,比如网络地址、广播地址、可用主机数这些。本来以为挺简单的,结果发现要处理各种边界情况,调试了好几次才搞定。

## 一、功能说明

### 1.1 主要功能
- 算网络地址
- 算广播地址
- 算可用主机数
- 算子网掩码
- 算通配符掩码
- 显示IP等级
- 支持CIDR格式

### 1.2 界面功能
- IP地址输入框
- 一键复制结果
- 显示详细计算
- 支持收藏
- 有使用说明

## 二、实现过程

### 2.1 计算原理

子网计算说白了就是位运算：

1. 网络地址计算
   - IP地址和子网掩码做与运算
   - 比如 192.168.1.1/24
   - 网络地址就是 192.168.1.0

2. 广播地址计算
   - 网络地址和通配符掩码做或运算
   - 比如 192.168.1.0/24
   - 广播地址就是 192.168.1.255

3. 可用主机数计算
   - 2^(32-前缀长度) - 2
   - 比如 /24 就是 2^8 - 2 = 254

4. 子网掩码计算
   - 根据前缀长度生成
   - 比如 /24 就是 255.255.255.0

5. IP等级判断
   - A类：1-126
   - B类：128-191
   - C类：192-223
   - D类：224-239
   - E类：240-255

### 2.2 代码实现

```typescript
@Entry
@Component
struct Ipv4subnetcalculator {
  @State ipAddress: string = '';           // 要算的IP
  @State networkMask: string = '';         // 网络掩码
  @State networkAddress: string = '';      // 网络地址
  @State subnetMask: string = '';          // 子网掩码
  @State subnetMaskBinary: string = '';    // 子网掩码二进制
  @State subnetMaskCIDR: string = '';      // 子网掩码CIDR
  @State wildcardMask: string = '';        // 通配符掩码
  @State totalHosts: number = 0;           // 可用主机数
  @State firstHost: string = '';           // 起始地址
  @State lastHost: string = '';            // 结束地址
  @State broadcastAddress: string = '';    // 广播地址
  @State ipClass: string = '';             // IP等级

  // 计算子网信息
  private calculateSubnet() {
    try {
      // 预处理输入
      const processedInput = this.preprocessIPAddress(this.ipAddress);
      
      // 解析输入
      let ip = processedInput;
      let cidr = '24'; // 默认子网掩码
      if (processedInput.includes('/')) {
        const parts = processedInput.split('/');
        ip = parts[0];
        cidr = parts[1];
      }

      // 验证IP地址格式
      if (!this.isValidIPAddress(ip)) {
        throw new Error('IP地址格式不对');
      }

      const prefix = parseInt(cidr);
      if (isNaN(prefix) || prefix < 0 || prefix > 32) {
        throw new Error('CIDR前缀不对(0-32)');
      }

      // 计算子网掩码
      const subnetMaskNum = ~((1 << (32 - prefix)) - 1);
      const subnetMaskParts = [
        (subnetMaskNum >>> 24) & 255,
        (subnetMaskNum >>> 16) & 255,
        (subnetMaskNum >>> 8) & 255,
        subnetMaskNum & 255
      ];
      this.subnetMask = subnetMaskParts.join('.');
      this.subnetMaskBinary = subnetMaskParts.map(n => 
        n.toString(2).padStart(8, '0')).join('.');
      this.subnetMaskCIDR = '/' + prefix;

      // 计算通配符掩码
      const wildcardParts = subnetMaskParts.map(n => 255 - n);
      this.wildcardMask = wildcardParts.join('.');

      // 计算网络地址
      const ipParts = ip.split('.').map(n => parseInt(n));
      const networkParts = ipParts.map((n, i) => n & subnetMaskParts[i]);
      this.networkAddress = networkParts.join('.');
      this.networkMask = this.networkAddress + this.subnetMaskCIDR;

      // 计算广播地址
      const broadcastParts = networkParts.map((n, i) => n | wildcardParts[i]);
      this.broadcastAddress = broadcastParts.join('.');

      // 计算可用主机范围
      const firstHostParts = [...networkParts];
      firstHostParts[3] += 1;
      this.firstHost = firstHostParts.join('.');

      const lastHostParts = [...broadcastParts];
      lastHostParts[3] -= 1;
      this.lastHost = lastHostParts.join('.');

      // 计算可用主机数
      this.totalHosts = Math.pow(2, 32 - prefix) - 2;

      // 判断 IP 等级
      const firstOctet = ipParts[0];
      if (firstOctet < 128) this.ipClass = 'A';
      else if (firstOctet < 192) this.ipClass = 'B';
      else if (firstOctet < 224) this.ipClass = 'C';
      else if (firstOctet < 240) this.ipClass = 'D';
      else this.ipClass = 'E';

    } catch (err) {
      console.error('计算失败:', err);
      throw err;
    }
  }
}
```

## 三、踩坑记录

### 3.1 遇到的问题

1. IP格式验证
   - 问题：各种格式的IP都要支持
   - 解决：写了个预处理函数

2. 边界情况
   - 问题：特殊IP地址处理
   - 解决：加了一堆判断

3. 性能问题
   - 问题：计算太频繁
   - 解决：加了防抖

4. 显示问题
   - 问题：结果太长显示不下
   - 解决：优化了布局

### 3.2 优化建议

1. 功能优化
   - 支持更多格式
   - 加个计算历史
   - 支持批量计算

2. 性能优化
   - 优化计算速度
   - 减少内存占用
   - 及时释放资源

3. 用户体验
   - 加个计算历史
   - 支持分享结果
   - 优化动画效果

4. 安全性
   - 验证输入格式
   - 限制计算频率
   - 保护用户隐私

## 四、总结

这个子网计算器基本功能都有了,可以：

- 计算子网信息
- 规划网络地址
- 排查网络问题
- 学习网络知识

## 五、参考资料

- [鸿蒙应用开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## 欢迎体验

这个子网计算器已经集成到鸿蒙开发者工具箱里了,欢迎下载体验！

[鸿蒙开发者工具箱](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> 作者：在人间耕耘
> 邮箱：1743914721@qq.com
> 版权声明：本文为原创文章 