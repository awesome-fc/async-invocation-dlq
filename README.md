
## 应用简介

本应用通过配置异步调用目标为 mns 队列，当函数执行失败时将异步消息推送到 mns 队列中来实现死信队列的功能，方便业务人员感知异步函数执行失败的消息并进行后续处理。

## 使用方式

1. 在控制台创建一个 mns 队列，如名称为  destination-demo；
2. 更新 [fun](https://help.aliyun.com/document_detail/140283.html) 工具版本 > 3.6.18，并配置好 fun 工具；
3. git clone 本项目。更改 template.yml 第 99/100 行 mns 队列为实际资源 ARN（请将 {please_replace_me} 替换为您队列的名称），并在项目目录中执行如下命令： `fun deploy --use-ros --stack-name xxxxx`
4. 您将看到创建了具有相应的异步配置的函数及 ram 资源。 

注意：操作 fun deploy 前请确认已经更新模板中的队列名称（第 3 步），避免因为消息队列不存在而造成发送目标失败。

## 实际测试
异步配置仅在异步调用时才会生效。您可以使用 sdk/cli 工具发送异步调用请求。
本示例我们使用 [fcli](https://help.aliyun.com/document_detail/52995.html) 发送异步触发请求，并观察 mns 队列：
`invk main -t Async -s '{"failure":true}'`
我们可以从 mns 队列接收到如下消息：
```
{"timestamp":xxxx,"requestContext":{"requestId":"xxxx","functionArn":"acs:fc:::services/Destination-destination.LATEST/functions/main","condition":"UnhandledInvocationError","approximateInvokeCount":1},"requestPayload":"{\"failure\":true}","responseContext":{"statusCode":200,"functionError":"UnhandledInvocationError"},"responsePayload":"{\n    \"errorMessage\": \"My unhandled exception\",\n    \"errorType\": \"MyError\",\n    \"stackTrace\": [\n        [\n            \"File \\\"/code/index.py\\\"\",\n            \"line 13\",\n            \"in handler\",\n            \"raise MyError('My unhandled exception')\"\n        ]\n    ]\n}"}
```

即当异步调用失败后，我们可以从 mns 接收到异步调用事件的详细错误信息。