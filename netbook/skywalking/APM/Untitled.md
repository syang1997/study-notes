# 接口文档技术选型

## 一.swagger

### 使用方式

1. 首先项目中得添加swagger2的依赖
2. 创建swagger2 的配置类
3. **为接口添加swagger的注解**
4. 启动项目访问接口

 

 @ApiOperation(value = "获取一个用户", notes = "根据用户id获取用户信息")

 @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "String")

 @GetMapping("/{id}")

 public User getOne(@PathVariable String id){

   if(!users.containsKey(id)){

​     return null;

   }

   return users.get(id);

 }

 

 

## 二.apidoc

### 使用方式

1. 具有node环境，安装apidoc
2. 添加apidoc.json文件（配置文件）
3. 注释里加入apidoc的注解
4. 利用apidoc的命令来生成接口文档

 /**

 \* @apiGroup Product

 \* @api {GET} /product/{id}  查询一个产品

 \* @apiDescription 指定产品id , 删除产品的全部信息，包括关联的schema

 \* @apiParam {String} id 产品id(必填*)

 \* @apiSuccessExample SuccessExample

 \* HTTP/1.1 200

 \* {

 \* id: 'xxx',

 \* modelId: 'xxxxx',

 \* name: 'xxx',

 \* intro: 'xxxx'

 \* }

 \* @apiErrorExample ErrorExample

 \* HTTP/1.1 600

 \* 具体的异常信息

 */

 @GetMapping("/{id}")

 public Product getOneProduct(@PathVariable String id)

 {

   return productServ.findOne(id);

 }

## 三.smart-doc

### 使用方式

1. 导入依赖插件（maven，gradle）
2. 添加smart-doc.json文件（配置文件）
3. 编写注释
4. 调用插件生成文档

 @Getter

 @Setter

 public class User implements Serializable {

   /**

   \* 名称

   */

   private String name;

   /**

   \* 年纪

   */

   private Integer age;

 

   public User() {

   }

 }

## 四.对比

首先swagger不仅只能伴随应用生成，而且对代码具有强侵入。各个接口都具有swagger的注解。首先排除swagger。

在apidoc与smart之间进行对比，apidoc没有对注解和方法进行深入解析。需要在接口中使用较多的注解tag。smart对请求路径返回进行解析，并对泛型也进行处理。所以只需要在注释中没很多的注解tag。

apidoc，需要有node环境，而smart只需要在依赖中导入对应的插件依赖即可，maven，gradle都有支持。

smart返回的文档类型多种，有符合openapi协议的json，也有html，与md文档，还可以导入postman。还对swagger-ui进行了扩展支持。

而且由于smart可以具有gradle插件，所以可以单独运行，也可以配置到应用的各个阶段运行例如在build之前之后触发。

## 五.smart-doc详细介绍

依赖插件导入方式

 plugins {

   id 'java'

   id "com.github.shalousun.smart-doc" version "2.0.7-release"

 }

注释tag

**最小配置单元：**

 {

  "outPath": "D://md2" //指定文档的输出路径,相对路径时请用./开头，eg:./src/main/resources/static/doc

 }

 {

  "serverUrl": "http://127.0.0.1", //服务器地址,非必须。导出postman建议设置成http://{{server}}方便直接在postman直接设置环境变量

  "isStrict": false, //是否开启严格模式

  "allInOne": true,  //是否将文档合并到一个文件中，一般推荐为true

  "outPath": "D://md2", //指定文档的输出路径

  "coverOld": true,  //是否覆盖旧的文件，主要用于mardown文件覆盖

  "createDebugPage": true,//@since 2.0.0 smart-doc支持创建可以测试的html页面，仅在AllInOne模式中起作用。

  "packageFilters": "",//controller包过滤，多个包用英文逗号隔开

  "md5EncryptedHtmlName": false,//只有每个controller生成一个html文件是才使用

  "style":"xt256", //基于highlight.js的代码高设置,可选值很多可查看码云wiki，喜欢配色统一简洁的同学可以不设置

  "projectName": "smart-doc",//配置自己的项目名称

  "skipTransientField": true,//目前未实现

  "sortByTitle":false,//接口标题排序，默认为false,@since 1.8.7版本开始

  "showAuthor":true,//是否显示接口作者名称，默认是true,不想显示可关闭

  "requestFieldToUnderline":true,//自动将驼峰入参字段在文档中转为下划线格式,//@since 1.8.7版本开始

  "responseFieldToUnderline":true,//自动将驼峰入参字段在文档中转为下划线格式,//@since 1.8.7版本开始

  "inlineEnum":true,//设置为true会将枚举详情展示到参数表中，默认关闭，//@since 1.8.8版本开始

  "recursionLimit":7,//设置允许递归执行的次数用于避免一些对象解析卡主，默认是7，正常为3次以内，//@since 1.8.8版本开始

  "allInOneDocFileName":"index.html",//自定义设置输出文档名称, @since 1.9.0

  "requestExample":"true",//是否将请求示例展示在文档中，默认true，@since 1.9.0

  "responseExample":"true",//是否将响应示例展示在文档中，默认为true，@since 1.9.0

  "displayActualType":false,//配置true会在注释栏自动显示泛型的真实类型短类名，@since 1.9.6

  "ignoreRequestParams":[ //忽略请求参数对象，把不想生成文档的参数对象屏蔽掉，@since 1.9.2

   "org.springframework.ui.ModelMap"

  ],

  "dataDictionaries": [{ //配置数据字典，没有需求可以不设置

​    "title": "http状态码字典", //数据字典的名称

​    "enumClassName": "com.power.common.enums.HttpCodeEnum", //数据字典枚举类名称

​    "codeField": "code",//数据字典字典码对应的字段名称

​    "descField": "message"//数据字典对象的描述信息字典

  }],

  "errorCodeDictionaries": [{ //错误码列表，没有需求可以不设置

   "title": "title",

   "enumClassName": "com.power.common.enums.HttpCodeEnum", //错误码枚举类

   "codeField": "code",//错误码的code码字段名称

   "descField": "message"//错误码的描述信息对应的字段名

  }],

  "revisionLogs": [{ //文档变更记录

​    "version": "1.0", //文档版本号

​    "revisionTime": "2020-12-31 10:30", //文档修订时间

​    "status": "update", //变更操作状态，一般为：创建、更新等

​    "author": "author", //文档变更作者

​    "remarks": "desc" //变更描述

   }

  ],

  "customResponseFields": [{ //自定义添加字段和注释，api-doc后期遇到同名字段则直接给相应字段加注释，非必须

​    "name": "code",//覆盖响应码字段

​    "desc": "响应代码",//覆盖响应码的字段注释

​    "ownerClassName": "org.springframework.data.domain.Pageable", //指定你要添加注释的类名

​    "value": "00000"//设置响应码的值

  }],

  "requestHeaders": [{ //设置请求头，没有需求可以不设置

​    "name": "token",//请求头名称

​    "type": "string",//请求头类型

​    "desc": "desc",//请求头描述信息

​    "value":"token请求头的值",//不设置默认null

​    "required": false,//是否必须

​    "since": "-"//什么版本添加的改请求头

  }],

  "rpcApiDependencies":[{ // 项目开放的dubbo api接口模块依赖，配置后输出到文档方便使用者集成

​     "artifactId":"SpringBoot2-Dubbo-Api",

​     "groupId":"com.demo",

​     "version":"1.0.0"

  }],

  "rpcConsumerConfig": "src/main/resources/consumer-example.conf",//文档中添加dubbo consumer集成配置，用于方便集成方可以快速集成

  "apiObjectReplacements": [{ // 自smart-doc 1.8.5开始你可以使用自定义类覆盖其他类做文档渲染，非必须

​    "className": "org.springframework.data.domain.Pageable",

​    "replacementClassName": "com.power.doc.model.PageRequestDto" //自定义的PageRequestDto替换Pageable做文档渲染

  }],

  "apiConstants": [{//从1.8.9开始配置自己的常量类，smart-doc在解析到常量时自动替换为具体的值

​     "constantsClassName": "com.power.doc.constants.RequestParamConstant"

  }],

  "responseBodyAdvice":{ //自smart-doc 1.9.8起，ResponseBodyAdvice统一返回设置，可用ignoreResponseBodyAdvice tag来忽略

​     "className":"com.power.common.model.CommonResult" //通用响应体

  },

  "sourceCodePaths": [{ //设置代码路径, 插件已经能够自动下载发布的源码包，没必要配置

​    "path": "src/main/java",

​    "desc": "测试"

  }]

 }



## 六.文档的生成

选择smart-doc框架后，文档的生成推荐使用插件进行。而接口文档在展示上选择使用前端拉取的方式。那么文档就要伴随应用程序。

## 1.git仓库在生成tag时生成到jar包中

选择在gradle的build流程中进行文档的生成，并放置到资源包下。

 env = System.getProperty("smart.profile") ?: "dev"

 if (env.equals("dev")){

   //build之后触发smart的task

   build.finalizedBy(smartDocRestMarkdown)

   //然后将生成的文档传入托马斯进行展示

   build.finalizedBy(myCopy)

 }

这样在git仓库中不会包含文档文件，并且在发布版本时文档也能导入jar包中。

## 2.手动触发

在要提交到git时手动触发文档生成，并将新的文档更新到仓库中。仓库发布tag时将其打包生成。

## 七.文档的展示

文档生成到资源路径下后，暴露给前端静态资源映射即可。