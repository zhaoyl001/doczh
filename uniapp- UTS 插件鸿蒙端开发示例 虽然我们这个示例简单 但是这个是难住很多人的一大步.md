
# UTS 插件鸿蒙端开发示例
以上示例已开源 
项目地址 请参考 [示例代码](https://gitee.com/zhaoyl1/hongmeng-uts-call-instance)。

---

> **前言**
>
> 虽然这个 UTS 插件鸿蒙端的示例看起来很简单，但说实话，这一步其实难住了不少开发者。很多人第一次做 UTS 插件，尤其是要跑通鸿蒙端，都会在这里卡壳。希望这份文档能帮你少走弯路，顺利迈过这道坎。

---

## 基础知识补充

### 什么是 UTS 插件？

UTS 插件其实就是 uni-app x 扩展 API 的标准插件形式。你可以把它理解成"写一份 TypeScript 风格的代码，编译后在不同平台都能用"。

说个实话，刚接触 uni-app x 的时候，很多人一看到"插件"两个字就头大，觉得一定很复杂。其实 UTS 插件的本质，就是把你想要的原生能力用 TypeScript 包一层，剩下的交给编译器搞定。

### UTS 与 ArkTS 的关系

UTS 和 ArkTS 都是基于 TypeScript 的扩展，但有些细节不同。**特别注意：鸿蒙端开发时，所有对象字面量都必须定义类型，不能用 any 类型，否则会直接编译报错。**

比如 ArkTS 不允许无类型的对象字面量，UTS 会自动帮你加上类型，但你自己写代码时一定要养成良好习惯：

```typescript
// 错误写法（鸿蒙端会报错）
const obj = { a: 1 };

// 正确写法
interface Obj { a: number }
const obj: Obj = { a: 1 };
// 或
const obj = { a: 1 } as Obj;
```

你只需要记住：UTS 写的代码，最终会被编译成 ArkTS（.ets）文件，然后就能愉快地调用鸿蒙的原生 API 了。

### 配置鸿蒙依赖

鸿蒙的依赖管理工具叫 ohpm，和 npm 很像。三方 SDK 用 .har 文件（有点像 Android 的 .aar）。

配置依赖时，记得在 `utssdk/app-harmony/config.json` 里写清楚：

```json
{
  "dependencies": {
    "@cashier_alipay/cashiersdk": "15.8.26",
    "local-deps": "./libs/local-deps.har"
  }
}
```

注意：config.json 不能有注释，本地依赖路径是相对的。

### 资源文件与权限配置

- 插件资源（图片、字体等）放在 `utssdk/app-harmony/resources`。
- 权限、模块信息等写在 `utssdk/app-harmony/module.json5`。

比如你要用定位权限，可以这样写：

```json5
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.LOCATION",
        "usedScene": { "when": "inuse" },
        "reason": "$string:permission_location_reason"
      }
    ]
  }
}
```

### context 获取

很多鸿蒙原生 API 需要 context。大多数场景下直接用 `getContext()` 就行：

```typescript
import settings from '@ohos.settings';
const context: Context = getContext();
settings.getValue(context, settings.display.SCREEN_BRIGHTNESS_STATUS, (err, value) => {
  if (err) {
    console.error(`Failed to get the setting. ${err.message}`);
    return;
  }
  console.log(`SCREEN_BRIGHTNESS_STATUS: ${JSON.stringify(value)}`)
});
```

有一次小王同学写插件，死活拿不到 context，结果发现是忘了在页面生命周期里调用，调试了半天才恍然大悟。遇到问题别慌，先查查官方文档和社区经验，很多"坑"其实大家都踩过。

---

> 更多细节和常见问题，建议随时查阅官方文档：[UTS for HarmonyOS](https://doc.dcloud.net.cn/uni-app-x/plugin/uts-for-harmony.html)

---

# 步骤详解

> **友情提示：**
> 
> 虽然下面的步骤看起来很基础，但每一步都很关键。尤其是接口定义和鸿蒙端实现，很多人就是在这里卡住的。别嫌简单，能跑通才是王道。
> 
> **再次强调：鸿蒙端开发时，所有对象字面量都必须定义类型，不能用 any 类型！**

### 第一步：定义插件接口（interface.uts）

**目的：**
- 明确插件对外暴露的 API 规范，方便多端实现和 IDE 智能提示。
- 这是 UTS 插件开发的基础，所有端的实现都要遵循这里定义的接口。

**操作：**
- 在 `uni_modules/你的插件名/utssdk/` 下新建或编辑 `interface.uts` 文件。
- 定义你要暴露的类型、方法签名。例如：

```typescript
// uni_modules/tt-ost/utssdk/interface.uts
export type MyApiSync1 = (paramA: string) => string;
```

---

### 第二步：鸿蒙端实现接口（app-harmony/index.uts）

**目的：**
- 按照接口定义，实现鸿蒙端的具体逻辑。
- 这是很多开发者卡壳的地方，需注意导入接口类型、调用鸿蒙 API、正确导出方法。
- **注意：所有对象字面量都要定义类型，不能用 any！**

**操作：**
- 在 `uni_modules/你的插件名/utssdk/app-harmony/` 下新建或编辑 `index.uts` 文件。
- 按照接口定义，实现方法。例如：

```typescript
// uni_modules/tt-ost/utssdk/app-harmony/index.uts
import { MyApiSync1 } from '../interface.uts';
import { promptAction } from '@kit.ArkUI';

interface ShowToastOptions {
  message: string;
}

export const myApiSync1: MyApiSync1 = function (paramA: string): string {
  let ddd: ShowToastOptions = { message: paramA };
  promptAction.showToast(ddd);
  return paramA;
}
```

- 这里以 Toast 弹窗为例，实际可根据业务需求调用鸿蒙原生能力。

---

### 第三步：在页面中调用插件方法

**目的：**
- 验证插件功能是否生效。
- 体验 UTS 跨端调用的便捷性。

**操作：**
- 在页面脚本中引入并调用插件方法。例如：

```vue
<script>
import { myApiSync1 } from '@/uni_modules/tt-ost';
export default {
  methods: {
    showToast() {
      const msg = 'Hello Harmony!';
      const result = myApiSync1(msg);
      console.log(result); // 输出: Hello Harmony!
    }
  }
}
</script>
```

---

## 说明
- 该插件支持多端，鸿蒙端实现了 `myApiSync1`，会调用 ArkUI 的 `promptAction.showToast`。
- 其他端（如 Android/iOS）可根据需要实现对应方法。
- 适合演示 UTS 跨端插件的基本用法。

---

如需更多信息，请参考 [uni-app x 官方 UTS 插件开发文档](https://doc.dcloud.net.cn/uni-app-x/plugin/uts-for-harmony.html)。 