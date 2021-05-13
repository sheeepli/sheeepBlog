---
title: 关于 umi-request 再封装的那件事
date: 2021-05-13 10:26:19
tags: ['ts', '基础']
---

每个人的封装都有自己的风格在里边，所以有自己的风格没有问题。

这是一个关于 umi-request 通用请求的的封装，已实现的部分功能：

* 默认注入 `newHeaders.Authorization` 减少调用时的 token 输入；
* 根据 `withToken` 参数不校验 token；
* 隐藏报错也可以使用 `showError=false` 实现；（尽量不用吧，性能问题）

``` ts
import { request as umiRequest } from 'umi';
import { RequestOptionsInit, RequestResponse } from 'umi-request';

interface extendedRequestOptions {
  skipErrorHandler?: boolean;
  withToken?: boolean;
  showError?: boolean;
}

interface RequestOptions extends RequestOptionsInit, extendedRequestOptions {}

interface RequestOptionsWithResponse extends RequestOptionsInit, extendedRequestOptions {
  getResponse: true;
}

interface RequestOptionsWithoutResponse extends RequestOptionsInit, extendedRequestOptions {
  getResponse: false;
}

// 根据实际项目的基本返回外层定义
export interface ResponseResult<T> {
  data: T;
  code: string;
  desc: string;
}

interface RequestMethod<R = false> {
  <T = any>(url: string, options: RequestOptionsWithResponse): Promise<RequestResponse<ResponseResult<T>>>;
  <T = any>(url: string, options: RequestOptionsWithoutResponse): Promise<ResponseResult<T>>;
  <T = any>(url: string, options?: RequestOptions): R extends true
    ? Promise<RequestResponse<ResponseResult<T>>>
    : Promise<ResponseResult<T>>;
}

// 这里的 options 其实是 umi-request 本来的 options 外加上自定义的一些属性。
// 可以把自定义的一些属性单独拿出来作为第三个属性，这样会不会比较好看一些
const request: RequestMethod = async (url: any, options: any = {}) => {
  const { withToken = true, showError = true, headers, ...restOptions } = options as RequestOptions;
  let newHeaders: Record<string, string> = {};
  if (withToken) {
    const token = getToken(); // 获取本地保存的 token
    newHeaders['Authorization'] = `Bearer ${token}`;
  }

  const newOptions: RequestOptions = {
    ...restOptions,
    headers: { ...headers, ...newHeaders },
  };

  return umiRequest(url, newOptions).catch(err => {
    showError && console.error(err); // 至于这里的报错展示可以自定义展示
  });
};

export default request;
```

使用方法：

```ts
const getTitle = (params) => {
  return request('/api/bababa', {
    method: 'get',
    // ...
  })
}

// 实际上会返回一个promise，该promise返回一个对象{code, data, success}，但实际上还是要根据实际业务情况来
const {data} = getTitle(params)
```
