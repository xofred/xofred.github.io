---
title:  "学习用 go 发起 http post 并读取服务器返回"
---

*tl; dr：之前用爬虫库 colly 做了个爬虫。现在想图片去重，发现 deep ai 有现成 API*

```go
// 调用 deep ai 图片相似度计算 API 并返回计算结果
var downloaded_urls []string

func check_images_similarity(downloaded_urls []string, url string) bool {
  ready_to_go := true

  for _, url_to_compare := range downloaded_urls {
    // 请求部分
    deep_ai_endpoint := "https://api.deepai.org/api/image-similarity"
    deep_ai_api_key := "注册 deep ai 后就有 API key 了"

    type Params struct {
      Image1      string
      Image2      string
    }
    params := Params{Image1: url, Image2: url_to_compare}
    b := new(bytes.Buffer)
    json.NewEncoder(b).Encode(params)

    // To set custom headers, use NewRequest and Client.Do.
    req, err := http.NewRequest("POST", deep_ai_endpoint, b)
    req.Header.Set("Api-Key", deep_ai_api_key)
    req.Header.Set("Content-Type", "application/json")

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
      panic(err)
    }
    defer resp.Body.Close()

    // 返回部分
    type Result struct {
      Distance int
    }
    result := Result{Distance: 21}
    json.NewDecoder(resp.Body).Decode(&result)

    if result.Distance < 20 {
      fmt.Println(result.Distance)
      if result.Distance == 0 {
        fmt.Println("We got two identical images!")
      } else {
        fmt.Println("We got two similar images!")
      }
      fmt.Println(url)
      fmt.Println(url_to_compare)

      ready_to_go = false
      break
    }
  }

  return ready_to_go
}

```

### Reference

[http - The Go Programming Language](https://golang.org/pkg/net/http/#Client.Post)

[Image Similarity](https://deepai.org/api-docs/#image-similarity)
