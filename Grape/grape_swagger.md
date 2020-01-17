## Grape

更具体使用细节可以参考官方使用文档</br>

[https://github.com/ruby-grape/grape](https://github.com/ruby-grape/grape) 

### 是什么
Grape是Ruby的一个类似REST的API框架。它的设计是通过提供简单的DSL来方便地开发RESTful  APIs，在rack上运行或补充现有的Web应用框架（如Rails和sinatra）。它内置了对常见约定的支持，包括多种格式、子域/前缀限制、内容协商、版本控制等等。


### 怎么用
grape使用超级简单，只要继承 `Grape::API`下面官方给的例子

```ruby
module Twitter
  class API < Grape::API
    version 'v1', using: :header, vendor: 'twitter'
    format :json
    prefix :api

    helpers do
      def current_user
        @current_user ||= User.authorize!(env)
      end

      def authenticate!
        error!('401 Unauthorized', 401) unless current_user
      end
    end

    resource :statuses do
      desc 'Return a public timeline.'
      get :public_timeline do
        Status.limit(20)
      end

      desc 'Return a personal timeline.'
      get :home_timeline do
        authenticate!
        current_user.statuses.limit(20)
      end

      desc 'Return a status.'
      params do
        requires :id, type: Integer, desc: 'Status ID.'
      end
      route_param :id do
        get do
          Status.find(params[:id])
        end
      end

      desc 'Create a status.'
      params do
        requires :status, type: String, desc: 'Your status.'
      end
      post do
        authenticate!
        Status.create!({
          user: current_user,
          text: params[:status]
        })
      end

      desc 'Update a status.'
      params do
        requires :id, type: String, desc: 'Status ID.'
        requires :status, type: String, desc: 'Your status.'
      end
      put ':id' do
        authenticate!
        current_user.statuses.find(params[:id]).update({
          user: current_user,
          text: params[:status]
        })
      end

      desc 'Delete a status.'
      params do
        requires :id, type: String, desc: 'Status ID.'
      end
      delete ':id' do
        authenticate!
        current_user.statuses.find(params[:id]).destroy
      end
    end
  end
end
```


- `namespace` 命名空间，它也有其它的别名：`group`，`resource`，`resources` 和 `segment`，我们可以根据情况使用
- `desc` 描述一个 API
- `params` 描述 API 的参数，grape 可以帮忙检查类型，而且可以在里面加入验证让 grape 帮我们处理
- `route_param` 代表一个路径参数，一般是 id
- `get` `post` `put` `delete` 这些都是 RESTful API 的动作参数，代表 http request method.
- `version` API 的版本
- `format` 输出格式
- `prefix` 前缀