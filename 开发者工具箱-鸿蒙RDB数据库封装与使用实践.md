# 鸿蒙RDB数据库封装与使用实践

> 最近项目又要搞数据存储，鸿蒙的RDB用起来还挺啰嗦，干脆自己封装了个工具类，省得每次都写一堆重复代码。这里随手记下，万一以后自己忘了还能翻出来看看。

## 一、SQL基础知识

### 1.1 什么是SQL
SQL(Structured Query Language)是用来操作关系型数据库的标准语言。其实最常用的就那几句，真遇到复杂的，百度/ChatGPT一搜一大把。

### 1.2 常用SQL语句
1. 创建表
```sql
CREATE TABLE user_info (
  id INTEGER PRIMARY KEY AUTOINCREMENT,  -- 主键,自增
  name TEXT,                             -- 文本类型
  age INTEGER,                           -- 整数类型
  score REAL,                            -- 浮点类型
  is_active INTEGER DEFAULT 0            -- 默认值
);
```

2. 插入数据
```sql
INSERT INTO user_info (name, age) VALUES ('张三', 18);
```

3. 查询数据
```sql
-- 查询所有字段
SELECT * FROM user_info;

-- 条件查询
SELECT name, age FROM user_info WHERE age > 18;

-- 排序
SELECT * FROM user_info ORDER BY age DESC;

-- 限制数量
SELECT * FROM user_info LIMIT 10;
```

4. 更新数据
```sql
UPDATE user_info SET age = 20 WHERE name = '张三';
```

5. 删除数据
```sql
DELETE FROM user_info WHERE age < 18;
```

## 二、开发背景

其实一开始我没打算封装，结果每次用都得写一堆初始化、建表、try-catch，烦得很。后来干脆写了个RDBUtils，省心多了。

## 三、实现步骤

### 3.1 创建RDB工具类

```typescript
import relationalStore from '@ohos.data.relationalStore';
import common from '@ohos.app.ability.common';

// 数据库配置
const STORE_CONFIG: relationalStore.StoreConfig = {
  name: 'user.db',
  securityLevel: relationalStore.SecurityLevel.S1
};

// 数据库版本号
const DB_VERSION = 2;

// 创建表的SQL语句
const SQL_CREATE_TABLE = `
  CREATE TABLE IF NOT EXISTS user_info (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    avatarUri TEXT,
    nickName TEXT,
    unionID TEXT,
    openID TEXT,
    authorizationCode TEXT,
    createTime INTEGER,
    memo TEXT,
    isSynced INTEGER DEFAULT 0
  )
`;

export class RDBUtils {
  private static instance: RDBUtils | null = null;
  private rdbStore: relationalStore.RdbStore | null = null;
  private context: common.Context;

  // 单例模式
  public static getInstance(context: common.Context): RDBUtils {
    if (!RDBUtils.instance) {
      RDBUtils.instance = new RDBUtils(context);
    }
    return RDBUtils.instance;
  }

  // 初始化数据库
  async initRDB(): Promise<void> {
    if (this.rdbStore) {
      return;
    }

    try {
      this.rdbStore = await relationalStore.getRdbStore(this.context, STORE_CONFIG);
      await this.rdbStore.executeSql(SQL_CREATE_TABLE);
      await this.checkAndUpdateVersion();
    } catch (err) {
      console.error(`初始化数据库失败: ${err}`);
      throw err;
    }
  }

  // 数据库升级
  private async checkAndUpdateVersion(): Promise<void> {
    // 检查版本并执行升级
    const currentVersion = await this.getCurrentVersion();
    if (currentVersion < DB_VERSION) {
      await this.upgradeDatabase(currentVersion);
    }
  }
}
```

### 3.2 实现CRUD操作

```typescript
export class RDBUtils {
  // ... 前面的代码 ...

  // 插入数据
  async insertUser(user: UserInfo): Promise<number> {
    if (!this.rdbStore) {
      throw new Error('数据库未初始化');
    }

    const valueBucket: relationalStore.ValuesBucket = {
      'avatarUri': user.avatarUri || '',
      'nickName': user.nickName || '',
      'unionID': user.unionID || '',
      'openID': user.openID || '',
      'authorizationCode': user.authorizationCode || '',
      'createTime': user.createTime || Date.now(),
      'memo': user.memo || '',
      'isSynced': user.isSynced ? 1 : 0
    };

    try {
      return await this.rdbStore.insert('user_info', valueBucket);
    } catch (err) {
      console.error(`插入数据失败: ${err}`);
      throw err;
    }
  }

  // 更新数据
  async updateUser(user: UserInfo): Promise<number> {
    if (!this.rdbStore || !user.id) {
      throw new Error('参数错误');
    }

    const valueBucket: relationalStore.ValuesBucket = {
      'avatarUri': user.avatarUri || '',
      'nickName': user.nickName || '',
      // ... 其他字段
    };

    const predicates = new relationalStore.RdbPredicates('user_info');
    predicates.equalTo('id', user.id);

    try {
      return await this.rdbStore.update(valueBucket, predicates);
    } catch (err) {
      console.error(`更新数据失败: ${err}`);
      throw err;
    }
  }

  // 查询数据
  async queryUserById(id: number): Promise<UserInfo | null> {
    if (!this.rdbStore) {
      throw new Error('数据库未初始化');
    }

    const predicates = new relationalStore.RdbPredicates('user_info');
    predicates.equalTo('id', id);

    try {
      const resultSet = await this.rdbStore.query(predicates, ['*']);
      if (resultSet.goToNextRow()) {
        const user = this.parseUserFromResultSet(resultSet);
        resultSet.close();
        return user;
      }
      resultSet.close();
      return null;
    } catch (err) {
      console.error(`查询数据失败: ${err}`);
      throw err;
    }
  }
}
```

## 四、踩坑记录

- 单例不做，内存飙升，手机直接卡死，血的教训。
- 查询结果集忘记close，内存泄漏，调了半天才发现。
- 批量插入不加事务，慢得想砸电脑。
- 插入数据时没处理空值，数据库直接报错。
- 更新数据时没检查ID，更新失败，debug半天。

## 五、使用示例

比如我有个用户表，插入、查、改、删都靠这套，写起来就几行代码，舒服。

```typescript
// 初始化数据库
const rdbUtils = RDBUtils.getInstance(context);
await rdbUtils.initRDB();

// 插入数据
const user: UserInfo = {
  avatarUri: 'https://example.com/avatar.jpg',
  nickName: '张三',
  unionID: '123456',
  openID: '789012',
  authorizationCode: 'abc123'
};
const id = await rdbUtils.insertUser(user);

// 查询数据
const user = await rdbUtils.queryUserById(id);
console.info(`查询到用户: ${user?.nickName}`);

// 更新数据
if (user) {
  user.nickName = '李四';
  await rdbUtils.updateUser(user);
}

// 删除数据
await rdbUtils.deleteUser(id);
```

## 六、注意事项

- 千万别忘了异常处理，出错了不提示，用户一脸懵。
- 结果集不用就close，不然你会怀疑人生。
- 数据类型别写错，调试起来很费劲。
- 合理用索引，批量操作记得加事务。
- 敏感数据要加密，别让用户数据裸奔。

## 七、总结

反正这玩意我自己用着还挺顺手，大家有更好的写法欢迎留言交流，别让我一个人踩坑。

## 八、参考资源

- [鸿蒙应用开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)
- [SQLite教程](https://www.runoob.com/sqlite/sqlite-tutorial.html)

## 欢迎体验

这个RDB工具类已经集成到鸿蒙开发者工具箱里了,欢迎下载体验！

[鸿蒙开发者工具箱](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---

> 作者：在人间耕耘
> 邮箱：1743914721@qq.com
> 版权声明：本文为博主原创文章，转载请附上原文出处链接及本声明。 