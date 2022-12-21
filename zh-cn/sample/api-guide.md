# 接口开发-最佳实践指南

接口的设计从请求到响应的整个流程中， 主要遵循以下几个原则：
- 基于Restful风格设计
- 入参出参维持相同的风格
- 响应码有一定的规则可循

在Restful风格的接口设计中，我们可能会遇到包括但不限于以下几类问题：
1. 路由命名时名词用单数还是复数形式
2. 多个资源获取路由如何定义参数
3. 多个资源创建或更新的入参格式
4. 路由命名中中划线和下划线如何选用

> [Restful同时获取多个资源的解决方法](https://api.stackexchange.com/docs/answers-by-ids)
> 这里给出的解决方案是多个资源时 id用 `:` 隔开传入

接口设计的同时，需要考虑以下因素：
- 并发过高时荷载问题
- 重复请求确保幂等且不会带来额外压力
- 参数异常不会触发程序异常、 空参数参数、多余空格带来的参数异常处理
- 恶意请求的发现和拦截
- 接口版本的控制
- 

> 一个成规模的应用， 一般的api路由数量可能在一百多甚至多达几百的数量， 如果路由存储没有规则、或者规则混乱，则定位的时候难度就会相当之大

## 路由的设计和存储

路由的存储 **应该** 按照业务系统的模块进行存储，同个模块的路由文件中， **应该** 按照迭代再做区分分组，注释说明
以下是简明的例子：
```php
Router::addGroup('/api', function () {
    Router::addGroup('/smart-template/', function () {
        // v1.0 & v1.1
        Router::post('template', [\App\Controller\SmartTemplate\TemplateController::class, 'create']);
        Router::get('template/{template_id}', [\App\Controller\SmartTemplate\TemplateController::class, 'info']);
        Router::get('template', [\App\Controller\SmartTemplate\TemplateController::class, 'query']);
        Router::put('template/{template_id}', [\App\Controller\SmartTemplate\TemplateController::class, 'save']);
        Router::delete('template/{template_id}', [\App\Controller\SmartTemplate\TemplateController::class, 'delete']);
        Router::get('image-config', [\App\Controller\SmartTemplate\TemplateController::class, 'queryImageConfig']);

        // v1.2
        Router::post('image-crop', [\App\Controller\SmartTemplate\ImageController::class, 'imageCrop']);
    });
}, [
    'middleware' => [
        GaoDingGatewayAuth::class,
    ]
]);
```
> 路由命名中采用中划线，参数命名采用下划线
> 


用户识别、权限控制、空参数、多余空格、并发限制、恶意请求的发现和拦截、参数格式的转换，都 **应该** 通过中间件来处理

## 接口的请求和响应

接口请求 **必须** 严格按照既往约定 `小写下划线` 风格， 响应也是如此，一个响应的标准格式 **必须** 如下：

```json
{
    "code": 10000,
    "message": "success",
    "data": [
        {},
        {}
    ],
    "page":1,
    "page_size":20,
    "total":100
}
```
> 分页时返回 `page`、 `page_size`、 `total`
> 
> Header中响应每个请求的TraceId 以便异常时追溯

资源批量获取时，路由传参按照以下形式传入：
```
v2/api/merchant/1/profile/2:3:5:12 #冒号（或逗号？）隔开多个id
```

请求参数到控制器层时，经过数据校验的，放行到业务逻辑层继续处理，处理后的结果响应`Response` 对象 一个简单的例子如下：
```php
    public function create(RequestInterface $request)
    {
        $rules = [
            'title' => ['required', 'string', 'max:30', Rule::unique('smart_template')->where(function ($query) {
                $query->where('org_id', BaseService::getOrgId());
            })],
            'type' => 'required|string|in:clothes,shoes',
            'platform_id' => 'required|integer|gt:0',
            'modules' => 'required|array',
            'modules.*.image_type' => 'required|string',
            'modules.*.select_type' => 'required|integer|in:1,2,3',
            'modules.*.fill_type' => 'required|integer|in:0,1,2',
            'modules.*.content' => 'required|array',
            'modules.*.content.type_tag' => 'required_if:modules.*.fill_type,2|numeric',
            'modules.*.content.tag' => 'array',
            'modules.*.content.tag.*' => 'integer',
            'modules.*.content.file_tag' => 'array',
            'modules.*.content.file_tag.*' => 'integer',
        ];
        $messages = [];
        $validator = $this->validator->make($request->all(), $rules, $messages);
        if ($validator->fails()) {
            throw new ValidationException(AppCode::VALIDATION_FAILED, $validator->errors()->first());
        }
        return $this->response->success($this->service->create($validator->getData()));
    }
```

## 常用中间件