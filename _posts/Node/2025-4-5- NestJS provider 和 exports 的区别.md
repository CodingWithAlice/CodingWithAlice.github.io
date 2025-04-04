---
layout:     post
title:    NestJS provider 和 exports 的区别
subtitle:  
date:       2025-4-5
author:     
header-img: 
catalog: true
tags:
    - < node >
typora-root-url: ..
---



# NestJS provider 和 exports 的区别

背景：在 AModule 中引入 BModule 后，无法直接使用 BService

解决方案：在 BModule 中通过 `exports` 导出

```js
// b.module.ts
@Module({
  	providers: [BService], // 错误以为这个是导出
  	exports: [BService], // 🌹--> 其实这个才是导出
    ...
})
export class BModule {}
// a.module.ts
@Module({
    imports: [BModule], // 这里引入 BModule
    ...
})
export class AModule {}
// a.service.ts
@Injectable()
export class AService {
    constructor(
    @InjectModel(A)
     private aModel: typeof A,
     private readonly bService: BService, // 声明 BService
    ) {}
	async findAll(){
        const Bres = await this.bService.findAll(); // 以为可以使用
    }
}
```



没有区分清楚两个指令的功能：

- providers 声明就是一种 **注册服务**，让它可以在当前模块中使用，不是导出功能

|              | providers: [BService]            | exports: [BService]             |
| ------------ | -------------------------------- | ------------------------------- |
| 作用         | 让 NestJS 知道如何创建 BService  | 允许其他模块使用 BService       |
| **可见性**   | 仅当前模块 (`BModule`) 内部可用  | 其他导入 `BModule` 的模块可用   |
| **是否必需** | 是 (必须先在 `providers` 中声明) | 可选 (只有需要共享时才需要导出) |





