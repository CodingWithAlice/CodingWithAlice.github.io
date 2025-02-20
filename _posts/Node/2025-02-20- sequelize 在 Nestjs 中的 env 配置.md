---
layout:     post
title:     sequelize 在 Nestjs 中的 env 配置
subtitle:  
date:       2025-02-20
author:     
header-img: 
catalog: true
tags:
    - < node >
typora-root-url: ..
---



# sequelize 在 Nestjs 中的 env 配置

总结：

- 使用 cross-env 在指令中传入 环境变量NODE_ENV
- 使用 @nestjs/config 在 app.module.ts 中 获取到配置，查询配置 进行不同环境数据库的连接



1、安装依赖，传入、获取配置

```js
npm i @nestjs/config
⭐️ // envFilePath 读取不同配置的文件 + configService 获取配置
npm i cross-env
⭐️ // process.env.NODE_ENV
```

2、传入配置 package.json

```js
"script": {
    "start:dev": "cross-env NODE_ENV=development nest start --watch",
    "start:prod": "cross-env NODE_ENV=production node dist/main"
}
```

3、获取配置 在 app.module.ts 中

```js
import { ConfigModule, ConfigService } from '@nestjs/config';
@Module({
    imports: [
        ConfigModule.forRoot({
            envFilePath: `.env.${process.env.NODE_ENV || 'development'}`,
            isGlobal: true, // 使 ConfigModule 在整个应用中可用
        }),
        SequelizeModule.forRootAsync({ // 使用异步方式配置 Sequelize
            imports: [ConfigModule],
            useFactory: (configService: ConfigService) => ({
                dialect: 'mysql',
                host: configService.get<string>('DB_HOST'),
                port: 3306,
                username: 'root',
                password: configService.get<string>('DB_PASSWORD'),
                database: 'Daily',
                models: [Ltn, Routine, Time],
            }),
            inject: [ConfigService],
        }),
        LtnsModule,
        RoutinesModule,
        TimesModule,
    ],
})
export class AppModule {}
```

