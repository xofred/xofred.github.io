---
title:  "解决 webmock 报错 Unregistered request: POST"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: ruby gem testing
---

一句话总结：使用 gem 之前请看说明书😂

---

## 问题

以前项目用 `fakeweb` 来模拟外部 API 调用，但 `fakeweb` 很久没维护了，所以这次找了个更多 star 而且有维护的 gem——`webmock`

但是，一开波就报错了

```shell
Failure/Error: expect { client.perform_post(path + 'something_else', 'xml_body') }.not_to raise_error

       expected no Exception, got #<WebMock::NetConnectNotAllowedError: Real HTTP connections are disabled. Unregistered request: POST ..._api").
         with(
           body: "xml_body")
```

## 解决

原因正如 [这个 issue](https://github.com/bblimke/webmock/issues/584) 说的，`stub_request` 和发起请求的地址对不上。。。

```ruby
stub_request(:post, 'A 地址').with(body: '请求 body').to_return(
	body: "返回 body",
    status: 400,
)
response = Net::HTTP.post('一定也要是 A 地址，差一点都不行')
expect(response.code).to eq '400'
expect(response.body).to eq "返回 body"
```

## Refernece

[WebMock::NetConnectNotAllowedError: Real HTTP connections are disabled. Unregistered request: POST with `NET::GTTP` #584](https://github.com/bblimke/webmock/issues/584)

[webmock](https://github.com/bblimke/webmock)
