---
layout:     post
title:      vue element UI 组件相关问题
subtitle:  
date:       2019-08-27
author:     
header-img: 
catalog: true
tags:
    - < 工作遇到的问题记录 >
typora-root-url: ..
---

##  vue element UI 组件相关问题

### 1.`el-upload` 导入excel

`element UI`组件，进行upload上传excel导入功能的时候，`on-success`/`before-upload`等钩子函数，要写在实例对象的`methods`中，才能够正常调用，否则会报`undefined in render` 的错。

### 2.`el-tree` 树形控件

`ElementUI Tree` 树形控件` https://blog.csdn.net/qq_42255106/article/details/80753096 ` 写的很细

有关element UI 中树形组件 `el-tree` 实现各个节点多选功能

```html
<!-- 在html中写的标签结构 -->
<el-tree
    :data="groupTreeData"
    show-checkbox
    ref="DeviceGroupTree"
    node-key="id"
    check-strictly
    @check="checkGroupNode">
</el-tree>
```

其中每一项的配置解析如下：

```javascript
/*	:data树形结构的数据
	show-checkbox 显示复选框
	ref的绑定 可以实现this.$refs.DeviceGroupTree拿到此控件
	node-key 给节点的编号【树形数据结构中有id字段】
	check-stricty 父、子节点之间没有关联【不写这个，选了父节点会默认选择全部的子节点】
	@check 复选框选择、取消选择时触发的事件 */
```

在JS中写的代码：

```javascript
// 在script标签中写方法
checkGroupNode: function (a, b) {
    if (b.checkedKeys.length > 0) {
     this.$refs.DeviceGroupTree.setCheckedKeys([a.id]);
    }
}
```

### 3.`el-input` 文字提醒

在做输入文字提醒的时候，可以使用三级列表形式提醒，使用的是`element`中的`el-input` 和`el-tree` ，如果想要直接以数据形式提醒的话，可以使用element当中的`el-autocomplete`。

### 4. element UI版本更新

在使用`element`文档内提供的组件的时候，如果使用之后发现有些功能无效，注意检查`element UI`的版本，可能是更新的属性。

更新升级element UI的版本方法：

```javascript
// 接下来的代码都在终端中输入
 npm view element-ui versions // 查看可升级到的element UI的版本
 npm update element-ui //然后更新到最新的版本，如果要指定版本，就接@版本号
 npm run serve // 启动项目，这里启动用serve还是dev自行确认下
```

### 5. `el-cascader` 三级级联组件

想要获取点击的内容时，可用` v-model="value"`来绑定，同时绑定事件`@change="handleChange"` 这样就可以在`methods`里面定义`handleChange`这个方法，同时传入value参数，进行数据处理。代码如下：

```javascript
// value 参数是标签中绑定的value值，作为参数传入
// 这里是为了实现多级级联多选时候，选中的节点，只取最后一项内容保存在school中
handleChange(value) {      
    for (let item of value) {       
        this.three.push(item[item.length - 1]);             
    }  
}
```

① 在级联中，options用于绑定数据，记得在标签里面**写的时候要加冒号**：；

②`:props`绑定后，可以传入对象，重置value（id） label（name） 和children（arrayData）的变量名

```javascript
data:{
	defaultprops:{
        value:"id",
        label:"name",
        children:"arrayData"
    }
}
```

#### el-cascader中获取选中的内容

在级联中，我要获得级联显示的内容，但是直接通过`v-model`、`:value`来绑定值，得到的是一个`[295,343,323]`，对应着dictionaryId值，但是我**想要的是label里面的值**，在官方文档里面没有提供获取label值的直接方法，所以要自己写。 

首先，级联在html中是：

```html
<!-- 三级级联 -->                  
<el-cascader                    
             ref="Address"                    
             :options="clubLists"                    
             :props="props"                    
             v-model="addressValue"                    
             @change="handleareaChange"                    
             style="width:350px"                    
             clearable                  
             ></el-cascader> 
```

其中，`clubLists`是通过接口获得到的级联的显示的数据，

```javascript
//级联学校列表    
async getClubLists() {      
    var res = await getArea();      
    if (res.code === "success") {        
        this.clubLists = res.data;      
    }    
},
```

`props`是级联数据中自定义的数据列表

```javascript
props: {        
    value: "dictionaryId",        
    label: "value",        
    children: "childDomain"      
};

```

`addressValue`是通过v-model绑定的选择的内容**（选中的dictionaryId值）**；
`handleareaChange`是**更改选择后会触发的方法**；

```javascript
handleareaChange() {      
    this.vals = this.getCascaderObj(this.addressValue, this.clubLists);    
},
```

 这里调用了一个获取内容的方法`getCascaderObj`，选择变化之后，vals里面的值也会变化，将绑定的选中的id值和数据源传入该方法；

```javascript
// 获取级联中的内容    
getCascaderObj(val, opt) {      
    return val.map(function(value, index, array) {        
        for (var itm of opt) {          
            if (itm.dictionaryId == value) {            
                opt = itm.childDomain;            
                return itm;          
            }  }        
        return null;    });   },  
```

其中，传入的参数：`value`代表的是**当前选中的id值**，`index`表示的是**当前id值在选中的数组中的序号**，`array`代表了选中的三项的id数组，在opt遍历的itm表示**数据源中的数据**，如下图。

![1566655598049](/../img/assets_2019/1566655598049.png)

通过`getCascaderObj`方法得到的vals数组是一个包含了源数组中id值的部分，所以要单独获得每一个选择的内容的话，要通过数组：` this.vals[0].value；this.vals[1].value；this.vals[2].value`

![1566655586000](/../img/assets_2019/1566655586000.png)

### 6. `el-table` 表格

在`el-table`中，`scope.row`表示的是**一行的数据**，如果我想要输出一个数组中的每一个对象的值，如下，可以直接使用`{{scope.row.title}}`插槽即可。

```json
"items":[
    {
        "id":"2000001",
        "school":"浙江大学",
        "title":"前端开发",
        "contnt":"HTML+CSS+JavaScript",
        "detail":"计算机基础",
        "adress":"VScode",
        "likes":0,
        "views":45,
        "start":null,
        "schoolsNames":[
        	{
        		"id":"3000001",
        		"name":"华南理工",
        		"sort":2,
        		"likes":2,
        		"views":4,
        		"child":0,
        		"friends":9
    		},{},{}
        ]
    },{}
]
```

若对象中有数组，数组中以对象形式罗列，如上`person`数组，那么若要循环输出数组中的键值对，那么就要在这一行建立`template`，设置`slot-scope=“scope”`，在template里写个div,span都可以，用来承载`v-for`语句。

```html
<el-table-column prop="schoolsNames" label="学校"  show-overflow-tooltip > 
    <template scope="scope">  
        <span v-for="schoolsName in scope.row.schoolsNames" :key="schoolsName.id"> 								<span>{{ schoolsName.name }}</span >  
		</span>  
	</template> 
</el-table-column>
```

#### el-table批量删除方法

```javascript
	delLists() {
            var ids = this.$refs.multipleTable.selection;
            var checks = [];
            if (ids.length === 0) {
                this.$message.warning("请选择");
                return false;
            } else {
                for (let i in ids) {
                    checks.push(ids[i].id);
                }
                var checkIds = checks.join("%");
            }
            this.$confirm("此操作将永久删除项数据, 是否继续?", "提示", {
                    confirmButtonText: "确定",
                    cancelButtonText: "取消",
                    type: "warning"
                })
                .then(() => {
                /*getDel是一个接收checkIds参数来删除的接口*/
                    getDel(checkIds).then(res => {
                        if (res.code === "success") {
                            this.$message({
                                type: "success",
                                activity: "删除成功!"
                            });
                            /*删除之后，重新获取table展示的数据，对当前页面进行刷新*/
                            this.getList();
                        }
                    });
                })
                .catch(() => {
                    this.$message({
                        type: "info",
                        activity: "已取消删除"
                    });
                });
        }
```

#### el-table 的格式化属性

用户从input输入的数据在回传到model之前也可以先处理；

```html
<el-table-column 
                 label="状态" 
                 align="center" 
                 :formatter="switchCode" 
                 show-overflow-tooltip></el-table-column>
```

 在表格中 有一个`formatter`用来格式化内容 `Function(row, column, cellValue, index) `
**使用流程：**

1.先在列中定义方法，给它一个方法名，它会自动获取参数；

2.然后再methods里面定义该方法，并传入默认的参数，这里传两个参数，row参数获取了本行的数组

```javascript
// 订单状态    
			switchCode(row, column) {
                    let Status = "";
                    switch (row.orderStatus) {
                        case 1:
                            Status = "已付款";
                            break;
                        case 2:
                            Status = "已发货";
                            break;
                        case 3:
                            Status = "已签收";
                            break;
                        case 4:
                            Status = "取消交易";
                            break;
                    }
                    return Status;
                }
```

**注意**：过程中，我将需要返回的Status变量，一开始定义在data中，导致无限循环报错，将要返回的变量局部定义在该函数中即可避免；



### 7.`el-menu ` 左侧导航

 1.要实现路由跳转，**先要在el-menu标签上添加router属性**，然后只要在每个**el-menu-item标签内的index属性**设置一下`url`即可实现点击el-menu-item实现路由跳转。

2.导航当前项设置：在el-menu标签中绑定 ` :default-active="$route.path"`,注意是绑定属性，不要忘了加`“:”`,当`$route.path`等于el-menu-item标签中的index属性值时则该item为当前项。

```html
        <el-menu 
                 router 
                 :default-active="$route.path" 
                 @open="handleOpen"
                 @close="handleClose" >
            <el-submenu index="1">
                <template slot="title">
                    <i class="el-icon-location"></i>
                    <span>信息</span>
                </template>
                <el-menu-item-group>
                    <el-menu-item index="/user/account">账号</el-menu-item>
                    <el-menu-item index="/user/password">密码</el-menu-item>
            </el-submenu>
            <el-submenu index="2">
                <template slot="title">
                    <i class="el-icon-location"></i>
                    <span>信息</span>
                </template>
                <el-menu-item-group>
                    <el-menu-item index="/company/manager">管理</el-menu-item>
                    <el-menu-item index="/company/edit">添加</el-menu-item>
                </el-menu-item-group>
            </el-submenu>
        </el-menu>
```

### 8. `el-form` 发布框必填/验证/重置

#### 必填/验证

①流程：在对发布框进行必填操作时，**先在 el-form 标签中 ，绑定rules**；然后在各个el-form-item标签中，**写prop属性，来匹配rules**；最后**在data中定义rules对应的验证**；

在rules中定义一个可以**验证 输入内容是否是数字的输入框**：

 ②其中一定要注意`v-model.number`绑定值的时候，绑定了number

```html
<el-form-item            
              label="积分"            
              prop="productPrice"            
              :step="1"            
              step-strictly            
              class="product_price">            
    <el-input 
              placeholder="请输入积分" 
              type="productPrice" 
              v-model.number="activityForm.productPrice" ><!--注意这里的v-model-->        
        <template slot="append">积分</template>            
    </el-input>          
</el-form-item>
```

```javascript
//  rules里面填写
productPrice: [          
    { type: 'number', message: "必须为数字值", trigger: "blur" },          
    { required: true, message: "请输入积分", trigger: "blur" }        
],  
```

③在验证的时候，**一定要把prop绑定的类别放置在父级**。如果跨开一级，就会一直失败

```html
<el-form-item label="类别" >              
    <el-select  
               prop="type" 
               v-model="Type" >                
        <el-option                  
                   v-for="item in List"                  
                   v-model="item.Name"                  
                   :key="item.Id">
        </el-option>              
    </el-select>            
</el-form-item>
```

#### `el-form`表单提交及表单数据重置

```html
  <el-form :model="dataForm" :inline="true" :rules="rules" label-position="left">
        <el-form-item label="名称" prop="name">
            <el-input v-model="dataForm.name"></el-input>
        </el-form-item>
        <el-form-item label="价格" prop="price">
            <el-input v-model="dataForm.price"></el-input>
        </el-form-item>
        <el-form-item label="数量" prop="num">
            <el-input v-model="dataForm.num"></el-input>
        </el-form-item>
        <el-form-item label="日期" prop="date">
            <el-input v-model="dataForm.date"></el-input>
        </el-form-item>
        <el-button @click="Submit('dataForm')">提 交</el-button>
        <el-button @click="giveSubmit('dataForm')">取 消</el-button>
   </el-form>
```

表单提交的时候，如果点击取消，就需要把内部的数据重置，**流程**：

1.先在`el-form`标签中，用`：model=“dataForm” `将所有的填写的内容保存在里面，

2.然后在取消按钮上面绑定相应的取消方法 ：`<el-button @click="giveSubmit('dataForm')">取 消</el-button> ` ，同时将`dataForm`这个存储值的对象传入，

3.最后在方法中定义： 利用`resetFields();`重置这个里面的数据

```javascript
giveSubmit(formName) {      
    this.productDialog = false;   
    this.$refs[formName].resetFields();    
}, 
```



​     