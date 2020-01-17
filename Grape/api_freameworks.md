### 调研目标
API框架希望能达到的效果主要是几点:

+ 参数合法性检查
+ 代码和api文档同时维护
+ 支持api测试



### 工具探索
+ ~~markdown 轻量易用，但是与需求不符~~
+ ~~[graphql](https://graphql.org/) 一种用于 API 的查询语言，目测即将流行的API工具，与我们的当前需求不符~~

+ [Swagger](https://swagger.io/)
Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。
    - [grape-swagger](https://github.com/ruby-grape/grape-swagger) Grape based. 开箱即用，grape项目首推使用。
    - [rswag](https://github.com/rswag/rswag) Rails based. `star: 576` 过写spec测试来生成swagger.json 使用swagger-ui来渲染swagger页面。
    - [apipie-rails](https://github.com/Apipie/apipie-rails) Rails based. `star: 2.1k` 本身比较轻量，可以结合swagger使用
    - ***[swagger-blocks](https://github.com/fotinakis/swagger-blocks)*** `star: 543` 可以把ruby代码块转成Swagger JSON，然后使用`swagger-ui`来渲染swagger页面。 理论上支持常见的ruby web框架包括Sinatra 可能是目前找到的最贴近我们需求的工具，但是也有一些问题
    - [api_schema](https://github.com/qonto/api_schema) `star: 18` 基于swagger-blocks 进行封装的用于生成Swagger JSON 工具
    - [sinatra-swagger](https://github.com/jphastings/sinatra-swagger) `star: 10` Provides helper functions for accessing Swagger documentation from within a Sinatra webapp. 5年内没有维护过的项目

### swagger-blocks 使用问题

+ 结合`Sinatra`使用的文档非常少
+ 需要学习相关DSL语法&定制相关的Model 增加了开发学习成本和代码量
+ `github star:` 543, 工具的成熟度有待验证
