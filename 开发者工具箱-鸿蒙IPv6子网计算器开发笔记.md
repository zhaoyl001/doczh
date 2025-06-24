# 鸿蒙IPv6子网计算器开发笔记

## 基础知识

先来唠唠IPv6是啥玩意：

1. IPv6地址长啥样
   - 8组16位的十六进制数
   - 用冒号分开
   - 比如 2001:0db8:0000:0000:0000:0000:0000:0001
   - 可以简写成 2001:db8::1

2. IPv6地址类型
   - 全局单播地址：公网用的
   - 链路本地地址：局域网用的
   - 唯一本地地址：私网用的
   - 多播地址：一对多通信
   - 环回地址：本地测试用

3. 地址压缩规则
   - 每组前面的0可以省略
   - 连续的0可以用::代替
   - 但::只能用一次
   - 比如 2001:0db8:0000:0000:0000:0000:0000:0001
   - 可以写成 2001:db8::1

4. 子网前缀
   - 用/后面的数字表示
   - 范围是0-128
   - 比如 /64 表示前64位是网络号
   - 后64位是主机号

5. IPv6地址结构
   - 前48位：全球路由前缀
   - 中间16位：子网ID
   - 后64位：接口ID
   - 比如 2001:db8:1234:5678::1
   - 2001:db8 是前缀
   - 1234 是子网
   - 5678::1 是接口

6. 特殊地址
   - ::/128：未指定地址
   - ::1/128：环回地址
   - fe80::/10：链路本地
   - ff00::/8：多播地址
   - 2000::/3：全局单播

7. 地址分配
   - 2000::/3：公网地址
   - fc00::/7：私网地址
   - fe80::/10：本地链路
   - ff00::/8：多播地址

8. 子网划分
   - /64：标准子网
   - /48：大型网络
   - /56：中型网络
   - /64：小型网络
   - /128：单机地址

9. 地址配置
   - 手动配置
   - DHCPv6
   - SLAAC(无状态)
   - 双栈模式

10. 常见问题
    - 地址太长不好记
    - 配置比较复杂
    - 需要双栈支持
    - 工具支持不够

## 前言

最近在搞鸿蒙开发者工具箱,想着加个IPv6子网计算器。这玩意比IPv4复杂多了,光是地址格式就够喝一壶的。不过好在最后搞定了,现在可以算IPv6的子网信息了。

## 一、功能说明

### 1.1 主要功能
- 算网络地址
- 算可用主机数
- 算首尾可用地址
- 支持地址压缩
- 支持地址展开
- 支持多种地址类型

### 1.2 界面功能
- IPv6地址输入框
- 一键复制结果
- 显示详细计算
- 支持收藏
- 有使用说明

## 二、实现过程

### 2.1 计算原理

IPv6子网计算主要是位运算：

1. 网络地址计算
   - IP地址和子网掩码做与运算
   - 比如 2001:db8::1/64
   - 网络地址就是 2001:db8::

2. 可用主机数计算
   - 2^(128-前缀长度) - 2
   - 比如 /64 就是 2^64 - 2
   - 这个数太大了,用字符串表示

3. 首尾地址计算
   - 首地址：网络地址+1
   - 尾地址：广播地址-1
   - 比如 2001:db8::1 到 2001:db8::ffff

### 2.2 代码实现

```typescript
// IPv6工具类
class IPv6UtilsClass {
  // 展开IPv6地址
  expandIPv6(ip: string): string {
    // 处理双冒号
    if (ip.includes('::')) {
      const parts = ip.split('::');
      const left = parts[0].split(':').filter(p => p);
      const right = parts[1].split(':').filter(p => p);
      const missing = 8 - left.length - right.length;
      const expanded = left.concat(Array(missing).fill('0000'), right);
      return expanded.map(p => p.padStart(4, '0')).join(':');
    }
    return ip.split(':').map(p => p.padStart(4, '0')).join(':');
  }

  // 压缩IPv6地址
  compressIPv6(ip: string): string {
    const parts = ip.split(':').map(p => p.replace(/^0+/, '') || '0');
    // 找最长的连续0
    let maxZeroStart = -1;
    let maxZeroLength = 0;
    let currentZeroStart = -1;
    let currentZeroLength = 0;

    for (let i = 0; i < parts.length; i++) {
      if (parts[i] === '0') {
        if (currentZeroStart === -1) {
          currentZeroStart = i;
        }
        currentZeroLength++;
      } else {
        if (currentZeroLength > maxZeroLength) {
          maxZeroLength = currentZeroLength;
          maxZeroStart = currentZeroStart;
        }
        currentZeroStart = -1;
        currentZeroLength = 0;
      }
    }

    if (maxZeroLength > 1) {
      const before = parts.slice(0, maxZeroStart);
      const after = parts.slice(maxZeroStart + maxZeroLength);
      return [...before, '', ...after].join(':');
    }

    return parts.join(':');
  }

  // 计算网络地址
  calculateNetworkAddress(ip: string, prefix: number): string {
    const binary = this.ipv6ToBinary(ip);
    const networkBinary = binary.map((bit, index) => index < prefix ? bit : 0);
    return this.binaryToIPv6(networkBinary);
  }

  // 计算可用主机数
  calculateHostCount(prefix: number): string {
    const hostBits = 128 - prefix;
    if (hostBits > 64) {
      return `2^${hostBits}`;
    }
    const count = BigInt(2) ** BigInt(hostBits) - BigInt(2);
    return count.toString();
  }
}
```

## 三、踩坑记录

### 3.1 遇到的问题

1. 地址格式验证
   - 问题：IPv6地址格式太复杂
   - 解决：写了个超长的正则表达式

2. 大数计算
   - 问题：2^64 这种数太大了
   - 解决：用 BigInt 和字符串

3. 地址压缩
   - 问题：压缩规则太复杂
   - 解决：写了个专门的压缩函数

4. 性能问题
   - 问题：计算太慢
   - 解决：优化了算法

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

这个IPv6子网计算器基本功能都有了,可以：

- 计算子网信息
- 规划网络地址
- 排查网络问题
- 学习网络知识

## 五、参考资料

- [鸿蒙应用开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## 欢迎体验

这个IPv6子网计算器已经集成到鸿蒙开发者工具箱里了,欢迎下载体验！

[鸿蒙开发者工具箱](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> 作者：在人间耕耘
> 邮箱：1743914721@qq.com
> 版权声明：本文为原创文章 