<!-- TRANSLATED by md-translate -->
# HTTP 标头修饰符

[HTTPRoute资源](/api-types/httproute)可以修改HTTP请求的头和客户端的HTTP响应。 有两种[过滤器](/api-types/httproute#filters-optional)可以满足这些要求："RequestHeaderModifier "和 "ResponseHeaderModifier"。

本指南介绍了如何被引用这些功能。

请注意，这些功能是兼容的。传入请求的 HTTP 标头及其响应的标头都可以使用一个 [HTTPRoute 资源]（/api-types/httproute）来修改。

### HTTP 请求头修改器

HTTP 标头修改是在传入请求中添加、删除或修改 HTTP 标头的过程。

要配置 HTTP 标头修改，可定义一个带有一个或多个 HTTP 过滤器的 gateway 对象。 每个过滤器指定对传入请求进行的特定修改，如添加自定义标头或修改现有标头。

要在 HTTP 请求中添加头信息，请使用类型为 "RequestHeaderModifier "的过滤器，并加上 "add "操作以及头信息的名称和值：

```yaml
{% include 'standard/http-request-header-add.yaml' %}
```

要编辑现有标头，请使用 `set` 操作，并指定要修改的标头值和要设置的新标头值。

```yaml
filters:
    - type: RequestHeaderModifier
      requestHeaderModifier:
        set: 
          - name: my-header-name
            value: my-new-header-value
```

还可以通过使用 `remove` 关键字和标题名称列表来删除标题。

```yaml
filters:
    - type: RequestHeaderModifier
      requestHeaderModifier:
        remove: ["x-request-id"]
```

被引用后，HTTP 请求中的 `x-request-id` 头信息就会被移除。

### HTTP 响应头修改器

就像编辑请求标头有用一样，编辑响应标头也同样有用。 例如，它允许团队只为某个后端添加/删除 cookie，这有助于识别之前被重定向到该后端的某些用户。

另一个可能的用例是，当你的前端需要知道它是在与稳定版还是测试版的后端服务器通信时，才能呈现不同的用户界面或相应地调整响应解析。

修改 HTTP 头响应使用的语法与修改原始请求被引用的语法非常相似，只是过滤器（`ResponseHeaderModifier`）不同。

可以添加、编辑和删除标题，也可以添加多个标题，如下图所示：

```yaml
filters:
    - type: ResponseHeaderModifier
      responseHeaderModifier:
        add:
        - name: X-Header-Add-1
          value: header-add-1
        - name: X-Header-Add-2
          value: header-add-2
        - name: X-Header-Add-3
          value: header-add-3
```