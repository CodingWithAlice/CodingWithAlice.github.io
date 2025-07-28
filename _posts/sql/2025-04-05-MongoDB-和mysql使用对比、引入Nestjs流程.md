---
layout:     post
title:     MongoDB-和mysql使用对比、引入Nestjs流程
subtitle:  
date:       2025-04-05
author:     
header-img: 
catalog: true
tags:
    - < database >
typora-root-url: ..
---



# MongoDB-和mysql使用对比、引入Nestjs流程

总结：

- 知识点的补充：

    - mongodb 中没有表的概念，对应 mysql 的表 - 是“集合”
        - 没有下划线和驼峰的转换，所以建议集合**定义时直接使用驼峰**
        - mysql 中的 findOrCreate = mongodb 中的 findOneAndUpdate + upsert 组合
        - mysql 中的 plain、raw 配置 = mongodb 中的 `.select(‘-_id’).lean()`
    - 适用的 半结构化、非结构化数据格式，对比 mysql 核心是 schema 的约束释放
    - mongodb 甚至不需要手动创建数据库 - 向一个不存在的数据库里插入数据时，会自动创建该数据库

- 引入工程的流程

    - Step1： `app.module.ts`  Nestjs 框架中，和 Sequelize 一样声明模块 

        ```js
        MongooseModule.forRootAsync({ /*使用 useFactory 配置 mongodb 的 uri*/ })
        ```

    - Step2：`answer.schema.ts`  声明对应集合的 schema，使用装饰器 @schema 

        ```js
        @Schema({ timestamps: true, collection:'answer_table' })
        ```

        - 注意：集合和 schema 的绑定是隐式完成的，如果不声明 collection 会自动将模型名称转换成 **小写复数** 形式作为集合名(注意驼峰不会转换为下划线 UserName -> usernames)

    - Step3： `answer.module.ts` 使用 forFeature 注册对应模型，name-用于依赖注入时识别模型，schema-用于内部隐式绑定集合名称

        ```js
        @Module({
            imports: [MongooseModule.forFeature([
                { schema: AnswerSchema, name: Answer.name }
            ])]
        })
        ```

    - Step4： `answer.service.ts` 在服务层注入 model，使用 @InjectModel 获取操作句柄

        ```js
        @Injectable()
        export class AnswerService{
            constructor( 
            	@InjectModel(Answer.name)
                 private answerModel: Model<Answer>
            ){}
        }
        ```

        



背景： LTN 学习法中，使用 mongodb 管理题目的答案 和 做题记录

记录创建、访问、建表过程中学习到内容



- MySQL 中的 表的概念，对应的是 MongoDB 的“集合”的概念
- 本地需安装 mongodb 服务 `brew install mongodb-community@6.0`



#### 一、数据库选型

- 错误记录 + 题目答案 

##### 选择 mongodb 的考量

- 题目答案 - 非结构化数据，选择 mongodb，避免关系型数据库的 schema 约束
- 每次做题的元数据（日期、时长、错误信息统计等）- 半结构化，固定字段 + 动态错误信息
- 写入频率高，MongoDB 的写入性能更优
- 分析需求时，聚合错误统计，MongoDB 的聚合管道比 SQL 的 `GROUP BY` 更灵活

```js
// question_answers 集合
{
  _id: ObjectId("..."),
  question_id: 123,  // 关联 MySQL 的题目ID
  content: {
    text: "这是解析...",
    code: "console.log(...)", 
    format: "mixed"  // enum: ['text', 'code', 'mixed']
  }
}
// answer_records 集合
{
  _id: ObjectId("..."),
  question_id: 123,       // 关联题目
  date: ISODate("2023-10-01"),
  duration_sec: 120,
  is_correct: false,
  errors: ["类型错误", "未处理边界条件"]  // 可动态扩展
}
```



#### 二、 NoSQLBooster for mongodb 学习

之前都是读写，第一次创建、设计结构，记录相关基础知识点

- admin 数据库：MongoDB 的系统管理数据库，具备最高权限 - 诸如创建用户、分配角色、关闭数据库服务等管理操作
- config 数据库：主要用于存储分片集群的配置信息（如果 MongoDB 部署的不是分片集群，则无啥用）
- local 数据库：存储的是本地服务器的相关数据，这些数据不会被复制到其他副本集成员上（如操作日志、临时数据等）
- test 数据库：默认创建的测试数据库，一般用于测试和开发阶段（在生产环境中，通常不会使用这个数据库存储正式的数据）
- users 数据库：并非 MongoDB 的系统默认数据库

##### 1、创建数据库

- 向一个不存在的数据库里插入数据时，MongoDB 会自动创建该数据库

```js
use coding_db
db.createCollection("question_answers")
db.createCollection("answer_records")
```

##### 2、引入 MongoDB 

- step1：只需要在 AppModule 里面引入 MongooseModule.forRoot

    - 由于线上使用 config.env 所以改为 forRootAsync 配置 uri

    ```js
    @Modules({
        MongooseModule.forRoot(process.env.MONGO_URI),
    })
    ```

- **集合(Collection)的绑定是通过 Schema 定义隐式完成的**

    - 在 `@Schema()` 装饰器中不显式指定集合名称时，Mongoose 会 **自动将模型名称转换为小写复数形式** 作为集合名
        - 自动复数化：`User` → `users`
        - 驼峰转下划线：`UserProfile` → `userprofiles`（非 `user_profiles`）
    - `collection: 'question_answers'` 强制指定集合名

```js
@Schema({ timestamps: true, collection: 'question_answers' })
export class Answer extends Document {
  @Prop({ required: true })
  answer_text: string;
}
export const AnswerSchema = SchemaFactory.createForClass(Answer);
```

- step2：注册模型时的关键连接 forFeature ：name - 用于依赖注入时识别模型；schema - 内部隐式绑定了集合名称

    ```js
    // answer.module.ts 
    @Module({
        imports: [
            MongooseModule.forFeature([{ schema: AnswerSchema, name: Answer.name }]),
        ]
    })
    ```

- Step3：服务层注入 - 通过 @InjectModel(XXX.name) 获取操作句柄

    ```js
    // answer.service.ts
    @Injectable()
    export class AnswersService {
      constructor(
        @InjectModel(Answer.name)
        private answerModel: Model<Answer>,
      ) {}
    }
    ```

    

#### 三、数据库查询 和 Sequelize 差异

- findByIdAndUpdate、findById ：专门用于通过文档的 _id 字段进行更新的方法
- findOneAndUpdate 使用普通查询条件，查找并更新

按功能分类记忆

- **查询**：`find` 查询多条文档, `findOne` 查询单条文档, `findById`
- **更新**：`updateOne`, `updateMany`, `findOneAndUpdate` 查找并更新
- **删除**：`deleteOne`, `deleteMany`
- **聚合**：`aggregate` 复杂聚合查询
- **配置项**：`new`, `upsert` 不存在时创建, `lean` 返回纯JS对象(非Document，提升性能), `select` 字段投影(选择返回的字段), `sort`排序

```js
 .findOne({ topicId: +topicId })
 .sort({ submitTime: -1 }) // 按时间倒排
 .select('submitTime durationSec -_id') // 只返回submitTime字段，排除_id
 .lean()

 .findOneAndUpdate(
        { topicId: dto.topicId, submitTime: dto.submitTime },
        {
          new: true, // 返回更新后的文档
          upsert: true, // 如果不存在则创建
          setDefaultsOnInsert: true, // 如果创建，应用 schema 默认值
          rawResult: true, // 返回完整的 mongodb 操作结果
        },
      )
```





### 四、服务器安装 mongodb 

```js
// 拉取官方镜像
docker pull mongo:latest
// 创建数据持久化目录
mkdir -p /data/mongodb
// 运行容器（推荐方式
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -v /data/mongodb:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=yourstrongpassword \
  --restart always \
  mongo
  
// 验证是否正在运行 
docker ps -a | grep mongodb
```

上述 MONGO_INITDB_ROOT_PASSWORD 配置了密码，我修改了，配置在 config.env 中

```js
// 进入 mongodb 的 shell 容器
docker exec -it mongodb mongosh
// 切换 admin 数据库并创建新用户
use admin
db.createUser({
  user: "alice",
  pwd: "",
  roles: ["root"]
})
```

配置远程连接（需要账号和密码）

```shell
// 进入容器检查配置
docker exec -it mongodb mongosh -u admin -p yourstrongpassword
// 执行
use admin
db.runCommand({getCmdLineOpts: 1})  // 查看 "net.bindIp" 是否为 0.0.0.0 - '*' 通用
```

在工程中 修改（Nestjs 框架）app.module.ts 中的 MongooseModule.forRootAsync 配置

- 之前在做 deepseek 账号密码的时候已经通过 dotenv 将项目外部的 config.env 引入环境变量
- 添加信息即可 MONGO_URI='mongodb://alice:xxx@121.43.164.209:27017/admin?authSource=admin'
