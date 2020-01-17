## grape 结合 sinatra
首先引入[`grape`](https://github.com/ruby-grape/grape) 目前最新稳定版本为 1.2.5

### 安装
第一步先本地安装
``` ruby
gem install grape -v 1.2.5
```

### 引入
在项目里引入 `grape`
```ruby
#zddiv3/server/lib/zcoreddi/util.rb

require 'grape'
```

### 定义路由
以 `grid_service` 服务为目标在其中使用 `grape` 路由
```ruby
module DDI
    class API < Grape::API
        # 格式化输出格式为json
        format :json

        desc 'get log level'
        get "log-level" do
            { level: DDI::Log.get_level }
        end
    end

    class GridService < Sinatra::Base
        include DDI::HostNetworkHelper
        include DDI::GRID::ServiceManager
        include DDI::WebServiceHelper
        include DDI::ErrMessageHelper
        #...
        #...
        #...
        get "/log-level" do
            {"level" => DDI::Log.get_level}.to_json
        end
        #...
        #...
        #...
    end

    if $0 == __FILE__
    port = ARGV[0] || 20120
    ssl_options = {
        :private_key_file => DDI::KEY_PATH,
        :cert_chain_file => DDI::CRT_PATH,
        :verify_peer => false,
    }
    rack_handler_config = {
        :server => 'thin',
        :Port => port
    }
    # DDI::API, DDI::GridService 注意先后顺序
    Rack::Handler::Thin.run(Rack::Cascade.new([DDI::API, DDI::GridService]), rack_handler_config) do |server|
        server.ssl = true
        server.ssl_options = ssl_options
    end
end

```

### 启动服务
在终端里单独启动 `grid_service` 服务并指定服务端口为`8080`
```bash
ruby /root/work/zddiv3/bin/grid_service/grid_service.rb 8080

```

### 验证路由

使用`curl`模拟请求来验证路由是否成功
``` bash
curl -X GET https://127.0.0.1:8080/log-level -H Content-Type:application/json -k -u admin:admin
{"level":"info"}
```
可以看到执行成功并返回数据

### 小结
从执行结果上看`grape`路由：`log-level` 覆盖了`sinatra`里定义的路由：`log-level`</br> 
这是由于我们启动服务的时候按照我们要求的顺序优先使用`DDI::API`</br>
```ruby
Rack::Cascade.new([DDI::API, DDI::GridService])
```
所以遇到匹配的路由就不会继续往下找`DDI::GridService`里面的路由</br>
至此初步达到了引入`grape`的目的

