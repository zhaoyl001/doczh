# 鸿蒙HTTP请求测试器开发笔记

## 前言

最近在搞鸿蒙工具箱,心血来潮想加个HTTP请求测试功能。这玩意主要是用来测试接口,让那些乱七八糟的接口调用变得简单直观。一开始觉得不就是个简单的请求嘛,结果一上手才发现,各种请求方法、参数、请求头、请求体,搞得我头都大了。

写这个工具的时候踩了不少坑,比如请求封装、错误处理、参数处理等等。不过功夫不负有心人,最后都搞定了,现在用起来还挺顺手的。

## 一、功能说明

### 1.1 主要功能
- 支持GET、POST、PUT、DELETE请求方法
- 支持URL自动补全和验证
- 支持查询参数（Query Parameters）
- 支持自定义请求头（Headers）
- 支持多种请求体格式（Form-data/JSON）
- JSON响应自动格式化
- 响应内容支持复制
- 收藏功能

### 1.2 界面功能
- URL输入框
- 请求方法选择
- 参数配置区域
- 请求体编辑器
- 响应展示区域
- 发送按钮

## 二、实现过程

### 2.1 开发小故事

记得刚开始写这个功能的时候,我直接用了系统自带的http模块,结果各种问题接踵而至。比如有个前端开发的小伙伴发了个格式错误的请求过来,程序直接崩溃了。用户一看就炸了："这什么鬼？我要的是友好的错误提示！"我心想："不就是个错误提示嘛,简单！"结果一上手才发现,得加try-catch,还得给出具体的错误信息,真是让人头大。

还有一次,有个后端开发的小伙伴说请求头不够灵活,只能固定几个。我心想："不就是个请求头嘛,简单！"结果一上手才发现,得处理各种请求头,还得考虑性能问题。最后加了个自定义请求头功能,让用户可以自己添加,这才算完事。

最搞笑的是处理大请求体的问题。一开始没考虑性能,结果有个用户发了个超大的JSON过来,程序直接卡死了。我心想："不就是个请求体嘛,简单！"结果一上手才发现,得考虑内存占用,还得做异步处理。最后改了好几遍,加了个加载状态,这才好多了。

还有个小插曲,有个用户说复制功能不好用,我一开始还觉得挺委屈："这不是挺简单的吗？"结果自己试了一下,发现确实有点问题,比如复制的时候没有提示,而且有时候会复制失败。后来改了好几版,加了个复制成功的提示,这才好多了。

最让我头疼的是URL自动补全的问题。有个用户说输入URL太麻烦了,能不能自动补全。我心想："不就是个补全嘛,简单！"结果一上手才发现,得处理各种URL格式,还得考虑性能问题。最后改了好几版,加了个URL自动补全功能,这才好多了。

还有个有趣的事,有个用户说响应内容不够直观,能不能加个格式化。我心想："不就是个格式化嘛,简单！"结果一上手才发现,得处理各种响应格式,还得考虑性能问题。最后改了好几版,加了个响应格式化功能,这才好多了。

### 2.2 请求封装

1. 导入模块
```typescript
import http from '@ohos.net.http'
import {
  ParamItem,
  SelectOption,
  HttpHeader as CustomHttpHeader,
  HttpOptions
} from '../../common/model/http'
import { HttpUtils } from '../../utils/http'
import router from '@ohos.router'
import promptAction from '@ohos.promptAction'
import clipboard from '@ohos.pasteboard'
import { StorageUtil } from '../../utils/storage'
```

2. 请求方法封装
```typescript
enum HttpMethod {
  GET = 'GET',
  POST = 'POST',
  PUT = 'PUT',
  DELETE = 'DELETE'
}
```

3. 请求选项封装
```typescript
class HttpOptions {
  method: string
  header: Record<string, string>
  extraData?: string

  constructor(method: string, header: Record<string, string>) {
    this.method = method
    this.header = header
  }
}
```

4. 请求发送封装
```typescript
async sendRequest(): Promise<void> {
  if (this.loading) return
  this.loading = true

  let httpRequest: http.HttpRequest | null = null

  try {
    // 1. 构建请求URL
    let url = this.url.trim()
    if (!url.startsWith('http://') && !url.startsWith('https://')) {
      url = 'https://' + url
    }

    // 2. 添加查询参数
    const enabledParams = this.queryParams.filter(p => p.enabled && p.key.trim())
    if (enabledParams.length > 0) {
      const queryString = enabledParams
        .map(p => `${encodeURIComponent(p.key.trim())}=${encodeURIComponent(p.value.trim())}`)
        .join('&')
      url += url.includes('?') ? '&' : '?'
      url += queryString
    }

    // 3. 构建请求头
    const headers: Record<string, string> = {}
    this.headers
      .filter(header => header.enabled && header.key.trim())
      .forEach(header => {
        headers[header.key.trim()] = header.value.trim()
      })

    // 4. 创建请求选项
    const options = new HttpOptions(this.method, headers)

    // 5. 处理请求体
    if (this.method !== 'GET') {
      if (this.bodyType === 'form-data') {
        const formData: Record<string, string> = {}
        this.bodyParams
          .filter(param => param.enabled && param.key.trim())
          .forEach(param => {
            formData[param.key.trim()] = param.value.trim()
          })
        options.extraData = JSON.stringify(formData)
      } else {
        try {
          JSON.parse(this.jsonBody) // 验证 JSON 格式
          options.extraData = this.jsonBody
        } catch {
          this.response = '请求体JSON格式错误'
          this.loading = false
          return
        }
      }
    }

    // 6. 发送请求
    console.info('Request URL:', url)
    console.info('Request Options:', JSON.stringify(options))

    httpRequest = http.createHttp()
    const result = await httpRequest.request(url, options)

    // 7. 处理响应
    if (result.responseCode === 200) {
      this.response = this.formatResponse(result.result as ResponseType)
    } else {
      this.response = `请求失败: ${result.responseCode}\n${JSON.stringify(result.result, null, 2)}`
    }
  } catch (error) {
    console.error('Request error:', error)
    this.response = `请求错误: ${error instanceof Error ? error.message : String(error)}`
  } finally {
    if (httpRequest) {
      httpRequest.destroy()
    }
    this.loading = false
  }
}
```

### 2.3 临时解决方案

1. 请求方法问题
   - 临时方案：直接用http模块,简单粗暴
   - 问题：格式错误就崩溃,用户体验差
   - 最终方案：加try-catch,给出友好提示

2. 请求头问题
   - 临时方案：固定几个请求头,省事
   - 问题：用户说不够灵活
   - 最终方案：加自定义请求头功能

3. 大请求体问题
   - 临时方案：同步处理,简单
   - 问题：大请求体会卡死
   - 最终方案：异步处理,加加载状态

4. 复制功能问题
   - 临时方案：直接复制,简单
   - 问题：没有提示,体验差
   - 最终方案：加复制成功提示

5. URL自动补全问题
   - 临时方案：手动输入,简单
   - 问题：用户体验差
   - 最终方案：加自动补全功能

6. 响应格式化问题
   - 临时方案：直接显示,简单
   - 问题：不够直观
   - 最终方案：加格式化功能

## 三、踩坑记录

### 3.1 遇到的问题

1. 请求方法问题
   - 问题：格式错误就崩溃
   - 解决：加try-catch
   - 建议：给出友好提示

2. 请求头问题
   - 问题：不够灵活
   - 解决：加自定义请求头
   - 建议：让用户自己添加

3. 大请求体问题
   - 问题：会卡死
   - 解决：异步处理
   - 建议：加加载状态

4. 复制功能问题
   - 问题：没有提示
   - 解决：加提示
   - 建议：处理错误

5. URL自动补全问题
   - 问题：用户体验差
   - 解决：加自动补全
   - 建议：优化补全算法

6. 响应格式化问题
   - 问题：不够直观
   - 解决：加格式化
   - 建议：支持更多格式

### 3.2 优化建议

1. 功能优化
   - 支持更多请求方法
   - 加个历史记录
   - 支持批量请求
   - 请求分类、导入导出、管理、分享、备份啥的都能加

2. 性能优化
   - 优化请求速度
   - 减少内存占用
   - 及时释放资源
   - 多线程、算法优化、结果缓存、异步处理都可以试试

3. 用户体验
   - 加个使用说明
   - 支持快捷键
   - 动画效果、主题、分享、收藏、导入、备份啥的都能加

## 四、总结

这个HTTP请求测试工具,基本功能都齐了：

- 支持多种请求方法
- 支持多种参数格式
- 实时预览结果
- 一键复制结果
- 收藏常用设置

有些边角问题其实还没完全搞定,不过大部分场景都能用。后面有空再慢慢优化吧。

## 五、参考资料

- [鸿蒙应用开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)

## 欢迎体验

这个HTTP请求测试工具已经集成到鸿蒙开发者工具箱里了,欢迎下载体验！

[鸿蒙开发者工具箱](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> 作者：在人间耕耘
> 邮箱：1743914721@qq.com
> 版权声明：本文为原创文章

---

如果你也遇到类似问题,欢迎留言交流