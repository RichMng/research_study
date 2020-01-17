## committee
[committee](https://github.com/interagent/committee)是Rack服务验证的中间件。它可以使用[JSON Schema](https://json-schema.org/)、[OpenAPI 2](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md)、[OpenAPI 3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md)来验证请求和响应参数, 并可以将参数转换为特定的Ruby对象（例如, 将datetime字符串转换为datetime类）。

### 版本说明
当前最新版本为`v3.3.0`

| committee 版本 | OpenAPI 版本 | ruby 版本 |
| :-: | :-: | :-: |
| 2.x | 2 | 2.0 ~ 2.6 |
| 3.x | 3 | 2.3 ~ 2.6 |

在我们的项目中使用的`ruby`版本为`2.2.x`, 所以有两种使用方案:

+ 升级项目`ruby`版本
    - 相关依赖都需要升级
    - 风险比较大
+ 不升级`ruby`使用`committee2.x`版本
    - 只能使用OpenAPI 2

针对两种方案各有优缺点, 升级`ruby`到高版本(>= 2.3)可以使用最新版本的`committee3.3.0`。 但是牵扯的范围比较大, 比如一些依赖包需要随着一起升级, 这可能会引入一些未知的风险, 所以综合考虑最终选择使用`committee2.5.1`(OpenAPI 2)

### 安装 & 解决 rack 依赖问题

首先安装`gem`指定版本为`2.5.1`

```shell
gem install committee -v 2.5.1

Fetching: rack-2.1.1.gem (100%)
Successfully installed rack-2.1.1
Successfully installed committee-2.5.1
Parsing documentation for rack-2.1.1
Installing ri documentation for rack-2.1.1
Parsing documentation for committee-2.5.1
Done installing documentation for rack, committee after 4 seconds
2 gems installed
```

安装成功后启动项目, 发现项目无法正常启动, 查看日志可以看到如下错误

```
/usr/local/lib/ruby/2.2.0/rubygems/specification.rb:2112:in `raise_if_conflicts': Unable to activate sinatra-1.4.2, because rack-2.1.1 conflicts with rack (>= 1.5.2, ~> 1.5) (Gem::ConflictError)
: Unable to activate sinatra-1.4.2, because rack-2.1.1 conflicts with rack (>= 1.5.2, ~> 1.5) (Gem::ConflictError)
    from /usr/local/lib/ruby/2.2.0/rubygems/specification.rb:1280:in `activate'
    from /usr/local/lib/ruby/2.2.0/rubygems.rb:198:in `rescue in try_activate'
    from /usr/local/lib/ruby/2.2.0/rubygems.rb:195:in `try_activate'
    from /usr/local/lib/ruby/2.2.0/rubygems.rb:198:in `rescue in try_activate'
    from /usr/local/lib/ruby/2.2.0/rubygems.rb:195:in `try_activate'
    from /usr/local/lib/ruby/2.2.0/rubygems/core_ext/kernel_require.rb:126:in `rescue in require'
    from /usr/local/lib/ruby/2.2.0/rubygems/core_ext/kernel_require.rb:39:in `require'
    from /usr/local/lib/ruby/gems/2.2.0/gems/zcoreddi-0.0.1/server/lib/zcoreddi/util.rb:4:in `<top (required)>'
    from /usr/local/lib/ruby/2.2.0/rubygems/core_ext/kernel_require.rb:39:in `require'
    from /usr/local/lib/ruby/gems/2.2.0/gems/zcoreddi-0.0.1/server/lib/zcoreddi/util.rb:4:in `<top (required)>'
    from /usr/local/lib/ruby/2.2.0/rubygems/core_ext/kernel_require.rb:128:in `require'
    from /usr/local/lib/ruby/2.2.0/rubygems/core_ext/kernel_require.rb:128:in `rescue in require'
    from /usr/local/lib/ruby/2.2.0/rubygems/core_ext/kernel_require.rb:39:in `require'
    from /usr/local/include/.pVYE/.GsVRk/.YZKCf/.le/.YmpK83/.aHx/system_service/system_service.rb:1:in `<top (required)>'
    from /usr/local/lib/ruby/2.2.0/rubygems/core_ext/kernel_require.rb:128:in `rescue in require'
    from /usr/local/lib/ruby/2.2.0/rubygems/core_ext/kernel_require.rb:39:in `require'
    from /usr/local/include/.pVYE/.GsVRk/.YZKCf/.le/.YmpK83/.aHx/system_service/system_service.rb:1:in `<top (required)>'
    from /usr/bin/zdnsloader:6:in `load'
    from /usr/bin/zdnsloader:6:in `block in run_service'
    from /usr/bin/zdnsloader:5:in `fork'
    from /usr/bin/zdnsloader:5:in `run_service'
    from /usr/bin/zdnsloader:6:in `block in run_service'
    from /usr/bin/zdnsloader:5:in `fork'
    from /usr/bin/zdnsloader:5:in `run_service'
    from /usr/bin/zdnsloader:34:in `<main>'
```
错误信息指出rack版本不正确, 再回头看看我们安装时候执行的信息, 其中有一句

```shell
Fetching: rack-2.1.1.gem (100%)
Successfully installed rack-2.1.1
```
原来是安装`committee`过程中也安装了`rack`的最新版本`rack-2.1.1`, 而我们项目中原来为`rack-1.5.2`(2013-02-07发布的)</br>
查看`committee-2.5.1`的`committee/committee.gemspec`有如下代码

```ruby
  # Rack 2.0+ requires Ruby >= 2.2.2 which is problematic for the test suite on
  # older Ruby versions. Check Ruby the version here and put a maximum
  # constraint on Rack if necessary.
  if RUBY_VERSION >= '2.2.2'
    s.add_dependency "rack", ">= 2.0.6"
  else
    s.add_dependency "rack", ">= 1.6.11", "< 2.0"
  end
```
注释的大概意思低版本的ruby对测试套件支持的不太好, 对于我们的场景目前不用关注测试的问题, 所以我们选择把代码clone到本地然后去掉这个限制（注释掉这几行代码）, 然后手工打包安装。

```shell
git clone https://github.com/interagent/committee.git

cd committee

git checkout v2.5.1

# 注释掉代码后执行

gem build committee/committee.gemspec

gem install committee -l
```

### 文档编写
可以使用在线[`Swagger Editor`](https://editor.swagger.io/)对文档进行编写, 针对我们的项目需要使用[OpenAPI 2](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md)的规范</br>
中文可以参考[Swagger2.0结构说明](https://www.jianshu.com/p/c22bf8e173a1) 配合[openapi-map](http://openapi-map.apihandyman.io/)一起使用效果更佳</br>
由于工具的问题, 项目中目前只能使用`JSON`格式的文档, 所以记得写完`yaml`格式的文档后使用`Swagger Editor`转换成`JSON`格式

### 项目中使用

`committee`使用非常简单, 下面以`zddiv3/bin/grid_service/grid_service.rb`为例

在文件中引入`committee`
```ruby
# coding: utf-8
require 'zcoreddi/util'
require 'zcoreddi/grid'
require 'zcoreddi/web'
# 引入committee
+ require 'committee'

module DDI
    class GridService < Sinatra::Base
+       # /root/work/zddiv3/docs/test.json是我测试使用OpenAPI 2文档的位置 
+       SCHEMA = Committee::Drivers.load_from_data(JSON.parse(File.read('/root/work/zddiv3/docs/test.json')))
+       # committee解析OpenAPI 2文档中描述的接口说明, 并根据其描述对参数进行校验
+       use Committee::Middleware::RequestValidation, schema: SCHEMA
        include DDI::HostNetworkHelper
        include DDI::GRID::ServiceManager
        include DDI::WebServiceHelper
        include DDI::ErrMessageHelper
        DDI.env_init("Grid")
        #...
    end
```
通过引入`Committee::Middleware::RequestValidation`重启服务后, 再访问接口前都会先走`committee`的参数校验逻辑, 如果请求的路由没有在接口文档中定义`committee`则不对其进行校验。

### 验证过程中踩到的坑

以上一气呵成看似行云流水般的操作的背后都是心酸和血泪还有无数次对人生的怀疑..., 主要的坑如下

+ 使用yaml类型文件, 参数校验不起作用, 需用json类型, 理论上讲两种类型的文件读取解析后都是Hash类型的对象, 不应该会出现差异
+ OpenAPI描述文档中的basePath参数如果不需要此参数则不写, 不能写成'/'
+ 不支持`format`的 `int32`、`int64`类型
+ 挂载中间件时, 按照文档操作一直没有成功, 后来发现需要手动指定驱动类型, 官方文档却没有明确说明, 最后阅读源码才找到使用方法

### 小结

以上就是使用`committee`的基本方式, 更多的还需要后面使用中继续探索。本次引入这个工具的目标就是达到***文档先行***的基本思想, 让各个端根据契约进行功能开发, 促进协作。

其整体的大致流程为：</br>
确定需求 -> 确定接口 -> 使用[`swagger editor`](https://editor.swagger.io/)描述接口文档 -> 文档放到项目中(前端可看到) -> 完成接口功能并测试 
