# 鸿蒙MD5工具开发笔记 - crypto-js三方库使用实践

## 基础知识

MD5这玩意其实挺简单的,就是用来算哈希值的。我刚开始接触的时候也是一脸懵逼,后来用多了就明白了。

1. 是啥
   - 就是个哈希算法,把任意长度的数据变成固定长度的字符串
   - 输出32位字符串,看起来像这样: 5d41402abc4b2a76b9719d911017c592
   - 算完就回不去了,想从MD5值反推出原文是不可能的
   - 主要用来校验,比如下载文件后算一下MD5,看看文件有没有损坏

2. 干啥用
   - 校验文件完整性,下完文件算一下MD5,对比一下就知道文件有没有损坏
   - 存密码,现在很多网站都用MD5存密码,不过现在不太安全了
   - 签名,可以用来生成数字签名
   - 去重,可以用来判断文件是否重复

3. 特点
   - 输出长度固定,都是32位
   - 算得快,比SHA256快多了
   - 不容易撞,虽然理论上可能碰撞,但实际很难
   - 算完就回不去了,这是它的优点也是缺点

4. 啥时候用
   - 下文件校验,特别是大文件
   - 存密码,不过现在建议用更安全的算法
   - 签名,生成数字签名
   - 去重,判断文件是否重复

## 前言

最近在搞鸿蒙工具箱,想着加个MD5工具。这玩意主要是用来算文本或文件的MD5值,方便校验文件完整性。本来以为挺简单的,结果发现要处理各种边界情况,调试了好几次才搞定。

写这个工具的时候遇到不少坑,比如中文编码问题,大文件内存问题,性能问题等等。不过最后都解决了,现在用起来还挺顺手的。

## 一、功能说明

### 1.1 主要功能
- 算MD5,支持文本和文件
- 大小写切换,方便复制
- 复制结果,一键复制
- 收藏功能,方便下次用

### 1.2 界面功能
- 输入框,可以输入文本
- 大小写开关,切换大小写
- 计算按钮,开始计算
- 结果显示,显示结果
- 复制按钮,复制结果

## 二、实现过程

### 2.1 三方库使用

#### 2.1.1 安装crypto-js

首先得安装crypto-js这个库,我用的是ohpm安装的。具体步骤：

1. 打开终端,进入项目目录
```bash
cd harmonyos-developer-toolbox
```

2. 安装crypto-js
```bash
ohpm install @ohos/crypto-js
```

3. 等待安装完成,会看到类似这样的输出：
```
[ohpm] Installing @ohos/crypto-js...
[ohpm] Successfully installed @ohos/crypto-js
```

4. 检查oh-package.json5,会看到新增了依赖：
```json5
{
  "dependencies": {
    "@ohos/crypto-js": "^1.0.0"
  }
}
```

5. 如果安装失败,可以试试：
```bash
ohpm cache clean
ohpm install @ohos/crypto-js --force
```

6. 安装完成后,需要重新构建项目：
```bash
ohpm build
```

7. 如果遇到版本冲突,可以在oh-package.json5中指定版本：
```json5
{
  "dependencies": {
    "@ohos/crypto-js": "1.0.0"  // 指定具体版本
  }
}
```

#### 2.1.2 使用crypto-js

安装好了就可以用了,主要用到了这些：

1. 引入
```typescript
import CryptoJS from '@ohos/crypto-js'
```

2. 主要API
```typescript
// 算MD5
CryptoJS.MD5(text)

// 转字符串
hash.toString()

// 转大写
result.toUpperCase()
```

3. 注意点
   - 空输入要处理,不然会报错
   - 异常要处理,比如编码问题
   - 编码要处理,特别是中文
   - 性能要处理,大文件要分块

### 2.2 代码实现

```typescript
@Entry
@Component
struct Md5Tool {
  @State private inputText: string = '';    // 输入文本
  @State private md5Result: string = '';    // MD5结果
  @State private isUpperCase: boolean = false;  // 是否大写
  @State private isProcessing: boolean = false;  // 是否正在算

  // 算MD5
  private async calculateMd5(): Promise<void> {
    if (!this.inputText) {
      await promptAction.showToast({ message: '请输入要计算的文本' });
      return;
    }

    try {
      this.isProcessing = true;
      // 用 CryptoJS 算 MD5
      const hash = CryptoJS.MD5(this.inputText);
      let result = hash.toString();

      if (this.isUpperCase) {
        result = result.toUpperCase();
      }

      this.md5Result = result;
      await promptAction.showToast({ message: '算好了' });
    } catch (err) {
      console.error('算失败了:', err);
      await promptAction.showToast({ message: '算失败了' });
    } finally {
      this.isProcessing = false;
    }
  }
}
```

## 三、踩坑记录

### 3.1 遇到的问题

1. 性能问题
   - 问题：大文本算得慢,特别是中文
   - 解决：加了进度提示,让用户知道在算
   - 建议：分块算,别一次性算完

2. 编码问题
   - 问题：中文乱码,算出来的结果不对
   - 解决：统一用UTF-8,保证编码一致
   - 建议：算前先转码,避免编码问题

3. 内存问题
   - 问题：大文件内存爆了,直接OOM
   - 解决：流式处理,边读边算
   - 建议：分批处理,别一次性读太多

4. 错误处理
   - 问题：算失败没提示,用户不知道咋回事
   - 解决：加了错误提示,告诉用户失败原因
   - 建议：加个日志,方便排查问题

### 3.2 优化建议

1. 功能优化
   - 支持文件计算,现在只能算文本
   - 加个历史记录,方便查看历史
   - 支持批量计算,一次算多个
   - 加个校验功能,对比两个MD5
   - 支持拖拽,直接拖文件进来
   - 加个进度条,显示计算进度
   - 支持导出,导出计算结果
   - 加个对比,对比两个结果

2. 性能优化
   - 优化计算速度,现在有点慢
   - 减少内存占用,大文件会爆
   - 及时释放资源,避免内存泄漏
   - 用多线程,提高计算速度
   - 优化算法,减少计算量
   - 缓存结果,避免重复计算
   - 压缩数据,减少内存占用
   - 异步处理,避免卡顿

3. 用户体验
   - 加个使用说明,告诉用户咋用
   - 支持快捷键,提高效率
   - 优化动画效果,看起来更流畅
   - 加个主题,支持暗黑模式
   - 支持分享,分享计算结果
   - 加个收藏,收藏常用结果
   - 支持导入,导入历史记录
   - 加个备份,备份设置

## 四、总结

这个MD5工具基本功能都有了,可以：

- 算文本MD5,支持中文
- 校验文件,看看文件有没有损坏
- 生成签名,生成数字签名
- 学习加密,了解加密原理

## 五、参考资料

- [鸿蒙应用开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)
- [CryptoJS文档](https://github.com/brix/crypto-js)

## 欢迎体验

这个MD5工具已经集成到鸿蒙开发者工具箱里了,欢迎下载体验！

[鸿蒙开发者工具箱](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> 作者：在人间耕耘
> 邮箱：1743914721@qq.com
> 版权声明：本文为原创文章 