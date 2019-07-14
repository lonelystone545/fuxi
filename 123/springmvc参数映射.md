## springmvc参数映射

http请求，无论是get还是post，当url中带有？形式的参数时，只要变量名称和参数名称能够映射（相同或者requestParams参数），后端都能接收并解析。 

http请求中三个关键参数：

1. dataType:预期的服务器返回的数据类型，如果不一致则解析错误，如json 
2. type:"post" or "get"   请求方式
3. contentType:"application/json;charset=utf-8"  发送至服务器时的内容编码类型

springmvc在接收post请求时，@RequestBody注解用于接收content-type为application/json类型的数据，即前端需要通过JSON.stringify 格式化对象；如果不用@RequestBody注解，那么可以接收content-type类型为application/x-www-form-urlencoded类的数据，即键值对形式（form表单提交的数据默认是这种类型）

get请求将参数拼接至url后面，post请求既可以将参数拼接至url后面，也可以将参数放在body中进行传输。

当前端出传输多种数据类型到后端时，理论上来说，需要用多个RequestBody注解进行解析。但是，springmvc不支持多个RequestBody注解，因为RequestBody作用是获取http请求中body的数据，是流式的，只能处理一次。因此，处理方式：1. 在后端将多种数据类型封装为一个bean对象，然后对这个bean对象使用RequestBody注解；2. 使用@RequestBody Map<String,Object>接收参数。

前端发送http请求，不论是get还是post，后端既可以通过参数接收，也可以通过bean对象接收；对于get请求，不需要使用RequestBody注解，自动将请求参数映射为bean对象的属性；对于post请求且格式为json的数据，后端必须使用RequestBody注解才能实现参数和bean对象的属性进行映射。