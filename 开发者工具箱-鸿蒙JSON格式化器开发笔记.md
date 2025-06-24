# 鸿蒙JSON格式化器开发笔记

## 前言

最近在搞鸿蒙工具箱,心血来潮想加个JSON格式化功能。这玩意主要是用来格式化JSON文本,让那些乱七八糟的JSON变得整整齐齐。一开始觉得不就是个简单的格式化嘛,结果一上手才发现,各种缩进、换行、错误处理,搞得我头都大了。

写这个工具的时候踩了不少坑,比如JSON解析、错误处理、缩进设置等等。不过功夫不负有心人,最后都搞定了,现在用起来还挺顺手的。

## 一、功能说明

### 1.1 主要功能
- 支持JSON文本格式化
- 支持自定义缩进空格数
- 支持一键复制结果
- 支持错误提示
- 实时预览格式化结果
- 收藏功能

### 1.2 界面功能
- JSON输入框
- 缩进设置
- 格式化按钮
- 结果显示
- 复制按钮

## 二、实现过程

### 2.1 开发小故事

记得刚开始写这个功能的时候,我直接用了系统自带的JSON.parse,结果各种问题接踵而至。比如有个用户发了个格式错误的JSON过来,程序直接崩溃了。用户一看就炸了："这什么鬼？我要的是友好的错误提示！"我心想："不就是个错误提示嘛,简单！"结果一上手才发现,得加try-catch,还得给出具体的错误信息,真是让人头大。

还有一次,有个前端开发的小伙伴说缩进不够灵活,只能固定2个空格。我心想："不就是个缩进嘛,简单！"结果一上手才发现,得处理各种缩进大小,还得考虑性能问题。最后加了个计数器,让用户可以自己调整缩进大小,这才算完事。

最搞笑的是处理大JSON的问题。一开始没考虑性能,结果有个用户发了个超大的JSON过来,程序直接卡死了。我心想："不就是个格式化嘛,简单！"结果一上手才发现,得考虑内存占用,还得做异步处理。最后改了好几遍,加了个加载状态,这才好多了。

还有个小插曲,有个用户说复制功能不好用,我一开始还觉得挺委屈："这不是挺简单的吗？"结果自己试了一下,发现确实有点问题,比如复制的时候没有提示,而且有时候会复制失败。后来改了好几版,加了个复制成功的提示,这才好多了。

### 2.2 临时解决方案

1. JSON解析问题
   - 临时方案：直接用JSON.parse,简单粗暴
   - 问题：格式错误就崩溃,用户体验差
   - 最终方案：加try-catch,给出友好提示

2. 缩进设置问题
   - 临时方案：固定2个空格,省事
   - 问题：用户说不够灵活
   - 最终方案：加计数器,让用户自己调

3. 大JSON处理问题
   - 临时方案：同步处理,简单
   - 问题：大JSON会卡死
   - 最终方案：异步处理,加加载状态

4. 复制功能问题
   - 临时方案：直接复制,简单
   - 问题：没有提示,体验差
   - 最终方案：加复制成功提示

### 2.3 调试案例

1. JSON格式化
```typescript
private async formatJson() {
  if (!this.inputJson) {
    await promptAction.showToast({ message: '请输入JSON文本' })
    return
  }

  try {
    this.isProcessing = true
    const parsed: object = JSON.parse(this.inputJson)
    this.formattedJson = JSON.stringify(parsed, null, this.indentSize)
    await promptAction.showToast({ message: '格式化成功' })
  } catch (err) {
    console.error('JSON格式化失败:', err)
    await promptAction.showToast({ message: 'JSON格式错误' })
  } finally {
    this.isProcessing = false
  }
}
```

这个格式化函数我真是改了又改。最开始用了个简单的JSON.parse,结果发现处理不了错误。后来加了个try-catch,但是提示不够友好。最后改成了现在的算法,虽然代码看起来有点乱,但是能处理各种情况。

2. 复制功能
```typescript
private async copyToClipboard(): Promise<void> {
  try {
    const pasteData: pasteboard.PasteData = pasteboard.createData(
      pasteboard.MIMETYPE_TEXT_PLAIN,
      this.formattedJson
    )
    const systemPasteboard = pasteboard.getSystemPasteboard()
    await systemPasteboard.setData(pasteData)
    await promptAction.showToast({ message: '已复制到剪贴板' })
  } catch (err) {
    console.error('复制失败:', err)
    await promptAction.showToast({ message: '复制失败' })
  }
}
```

这个复制函数也改了好几次。最开始用了个简单的复制,结果发现没有提示。后来加了个提示,但是处理不了错误。最后改成了现在的算法,虽然代码看起来有点乱,但是能处理各种情况。

## 三、踩坑记录

### 3.1 遇到的问题

1. JSON解析问题
   - 问题：格式错误就崩溃
   - 解决：加try-catch
   - 建议：给出友好提示

2. 缩进设置问题
   - 问题：不够灵活
   - 解决：加计数器
   - 建议：让用户自己调

3. 大JSON处理问题
   - 问题：会卡死
   - 解决：异步处理
   - 建议：加加载状态

4. 复制功能问题
   - 问题：没有提示
   - 解决：加提示
   - 建议：处理错误

### 3.2 优化建议

1. 功能优化
   - 支持更多格式
   - 加个历史记录
   - 支持批量格式化
   - 格式分类、导入导出、管理、分享、备份啥的都能加

2. 性能优化
   - 优化解析速度
   - 减少内存占用
   - 及时释放资源
   - 多线程、算法优化、结果缓存、异步处理都可以试试

3. 用户体验
   - 加个使用说明
   - 支持快捷键
   - 动画效果、主题、分享、收藏、导入、备份啥的都能加

## 四、总结

这个JSON格式化工具,基本功能都齐了：

- 支持JSON格式化
- 实时预览结果
- 一键复制结果
- 收藏常用设置

有些边角问题其实还没完全搞定,不过大部分场景都能用。后面有空再慢慢优化吧。

## 五、参考资料

- [鸿蒙应用开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## 欢迎体验

这个JSON格式化工具已经集成到鸿蒙开发者工具箱里了,欢迎下载体验！

[鸿蒙开发者工具箱](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> 作者：在人间耕耘
> 邮箱：1743914721@qq.com
> 版权声明：本文为原创文章

---

如果你也遇到类似问题,欢迎留言交流,搞不定咱们一起头疼！ 