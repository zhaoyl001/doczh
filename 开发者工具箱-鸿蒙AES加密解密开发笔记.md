# 鸿蒙AES加密解密开发笔记

## 基础知识

先来说说加密解密是啥玩意：

1. AES是啥
   - 高级加密标准
   - 对称加密算法
   - 速度快安全性好
   - 现在最常用的加密算法

2. 加密方式
   - ECB模式：最简单,但不够安全
   - CBC模式：需要IV,更安全
   - CFB模式：流加密,适合实时
   - OFB模式：流加密,可并行
   - CTR模式：流加密,可并行

3. 密钥长度
   - 128位：够用
   - 192位：更安全
   - 256位：最安全
   - 越长越安全,但越慢

4. 加密过程
   - 输入明文和密钥
   - 生成密文
   - 可以解密还原
   - 密钥要保管好

5. 使用场景
   - 密码存储
   - 文件加密
   - 通信加密
   - 数据保护

6. 安全建议
   - 密钥要足够长
   - 定期更换密钥
   - 不要用简单密码
   - 保护好密钥
   - 加密前先压缩
   - 加个随机盐值
   - 用安全的模式
   - 验证数据完整性

7. 常见问题
   - 密钥忘了咋办
   - 密文损坏咋办
   - 加密太慢咋办
   - 内存不够咋办
   - 中文乱码咋办
   - 跨平台咋办
   - 性能问题咋办
   - 安全问题咋办

## 前言

最近在搞鸿蒙开发者工具箱,想着加个AES加密解密功能。这玩意主要是用来保护数据的,比如加密一些敏感信息。本来以为挺简单的,结果发现要处理各种边界情况,调试了好几次才搞定。

## 一、功能说明

### 1.1 主要功能
- 文本加密
- 文本解密
- 密钥管理
- 结果复制
- 支持收藏

### 1.2 界面功能
- 密钥输入框
- 文本输入区
- 加密解密按钮
- 结果显示区
- 一键复制

## 二、实现过程

### 2.1 加密原理

AES加密主要是这样：

1. 加密过程
   - 输入明文和密钥
   - 生成密文
   - 可以Base64编码
   - 方便传输存储

2. 解密过程
   - 输入密文和密钥
   - 还原明文
   - 验证完整性
   - 处理错误

3. 密钥处理
   - 验证密钥长度
   - 处理特殊字符
   - 保存密钥
   - 安全传输

### 2.2 代码实现

```typescript
@Entry
@Component
struct AesEncryption {
  @State private plaintext: string = '';    // 明文
  @State private ciphertext: string = '';   // 密文
  @State private keyText: string = '';      // 密钥
  @State private isEncrypting: boolean = false;  // 是否正在加密
  @State private currentMode: 'encrypt' | 'decrypt' = 'encrypt';  // 当前模式

  // 加密方法
  private async encrypt() {
    if (!this.inputText || !this.keyText) {
      await promptAction.showToast({ message: '请输入明文和密钥' });
      return;
    }

    try {
      this.isEncrypting = true;
      this.currentMode = 'encrypt';
      // 调用加密库
      const encrypted = CryptoJS.AES.encrypt(this.inputText, this.keyText);
      this.ciphertext = encrypted.toString();
      this.plaintext = '';
      await promptAction.showToast({ message: '加密成功' });
    } catch (err) {
      console.error('加密失败:', err);
      await promptAction.showToast({ message: '加密失败' });
    } finally {
      this.isEncrypting = false;
    }
  }

  // 解密方法
  private async decrypt() {
    if (!this.inputText || !this.keyText) {
      await promptAction.showToast({ message: '请输入密文和密钥' });
      return;
    }

    try {
      this.isEncrypting = true;
      // 调用解密库
      const decrypted = CryptoJS.AES.decrypt(this.inputText, this.keyText);
      const decryptedText = decrypted.toString(CryptoJS.enc.Utf8);

      if (!decryptedText) {
        throw new Error('解密结果为空');
      }

      this.plaintext = decryptedText;
      this.ciphertext = '';
      this.currentMode = 'decrypt';
      await promptAction.showToast({ message: '解密成功' });
    } catch (err) {
      console.error('解密失败:', err);
      await promptAction.showToast({ message: '解密失败，请检查密钥是否正确' });
    } finally {
      this.isEncrypting = false;
    }
  }
}
```

## 三、踩坑记录

### 3.1 遇到的问题

1. 密钥验证
   - 问题：密钥格式不对
   - 解决：加了个验证函数
   - 建议：用正则表达式验证

2. 编码问题
   - 问题：中文乱码
   - 解决：统一用UTF-8
   - 建议：加密前先转码

3. 性能问题
   - 问题：大文本加密慢
   - 解决：加了进度提示
   - 建议：分块加密

4. 错误处理
   - 问题：解密失败没提示
   - 解决：加了错误提示
   - 建议：加个日志

5. 内存问题
   - 问题：大文件内存溢出
   - 解决：流式处理
   - 建议：分批处理

6. 安全问题
   - 问题：密钥泄露
   - 解决：加个盐值
   - 建议：定期更换

### 3.2 优化建议

1. 功能优化
   - 支持更多模式
   - 加个密钥生成
   - 支持文件加密
   - 加个密钥管理
   - 支持批量处理
   - 加个历史记录
   - 支持导入导出
   - 加个备份恢复

2. 性能优化
   - 优化加密速度
   - 减少内存占用
   - 及时释放资源
   - 使用多线程
   - 优化算法
   - 缓存结果
   - 压缩数据
   - 异步处理

3. 用户体验
   - 加个加密历史
   - 支持分享结果
   - 优化动画效果
   - 加个使用说明
   - 支持快捷键
   - 加个进度条
   - 支持拖拽
   - 加个主题

4. 安全性
   - 验证输入格式
   - 限制加密频率
   - 保护用户隐私
   - 加个密码强度
   - 支持双因素
   - 加个日志
   - 支持加密
   - 加个备份

## 四、总结

这个AES加密解密工具基本功能都有了,可以：

- 加密敏感信息
- 保护重要数据
- 安全传输数据
- 学习加密知识

## 五、参考资料

- [鸿蒙应用开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## 欢迎体验

这个AES加密解密工具已经集成到鸿蒙开发者工具箱里了,欢迎下载体验！

[鸿蒙开发者工具箱](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> 作者：在人间耕耘
> 邮箱：1743914721@qq.com
> 版权声明：本文为原创文章 