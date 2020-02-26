---
description: >-
  Ctx结构表示保存HTTP请求和响应的上下文。它具有用于请求查询字符串，参数，正文，HTTP标头等的方法。
---

# 🧠 上下文

## Accepts

检查指定的**扩展名**或**内容类型**是否可接受。

{% hint style="info" %}
基于请求的[Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) HTTP标头。
{% endhint %}

**签名**

```go
c.Accepts(types ...string) string
c.AcceptsCharsets(charsets ...string) string
c.AcceptsEncodings(encodings ...string) string
c.AcceptsLanguages(langs ...string) string
```

**示例**

```go
// Accept: text/*, application/json

app.Get("/", func(c *fiber.Ctx) {
  c.Accepts("html")             // "html"
  c.Accepts("text/html")        // "text/html"
  c.Accepts("json", "text")     // "json" "text"
  c.Accepts("application/json") // "application/json"
  c.Accepts("image/png")        // ""
  c.Accepts("png")              // ""
})
```

Fiber为其他支持的头提供了类似的功能。

```go
// Accept-Charset: utf-8, iso-8859-1;q=0.2
// Accept-Encoding: gzip, compress;q=0.2
// Accept-Language: en;q=0.8, nl, ru

app.Get("/", func(c *fiber.Ctx) {
  c.AcceptsCharsets("utf-16", "iso-8859-1") 
  // "iso-8859-1"
  
  c.AcceptsEncodings("compress", "br") 
  // "compress"
  
  c.AcceptsLanguages("pt", "nl", "ru") 
  // "nl" "ru"
})
```

## Append

将指定的**value**附加到HTTP响应头字段。

{% hint style="warning" %}
如果头尚**未设置**，则会创建具有指定值的头。
{% endhint %}

**签名**

```go
c.Append(field, values ...string)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Append("Link", "http://google.com", "http://localhost")
  // => Link: http://localhost, http://google.com

  c.Append("Link", "Test")
  // => Link: http://localhost, http://google.com, Test
})
```

## Attachment

将HTTP响应[Content-Disposition](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition)标头字段设置为`附件`。

**签名**

```go
c.Attachment(file ...string)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Attachment()
  // => Content-Disposition: attachment

  c.Attachment("./upload/images/logo.png")
  // => Content-Disposition: attachment; filename="logo.png"
  // => Content-Type: image/png
})
```

## BaseURL

返回基本URL \(**protocol** + **host** \)作为`字符串`。

**签名**

```go
c.BaseURL() string
```

**示例**

```go
// GET https://example.com/page#chapter-1

app.Get("/", func(c *fiber.Ctx) {
  c.BaseURL() // https://example.com
})
```

## Body

包含在**POST**请求中提交的原始主体。

**签名**

```go
c.Body(key ...string) string
```

**示例**

```go
// curl -X POST http://localhost:8080 -d user=john

app.Post("/", func(c *fiber.Ctx) {
  // Get raw body from POST request:
  c.Body() // user=john

  // Get body value by specific key:
  c.Body("user") // "john"
})
```

## BodyParser

将请求主体绑定到结构。 `BodyParser`支持基于`Content-Type`头对以下内容类型进行解码:

* `application/json`
* `application/xml`
* `application/x-www-form-urlencoded`
* `multipart/form-data`

**签名**

```go
c.BodyParser(v interface{})
```

**示例**

```go
// curl -X POST -H "Content-Type: application/json" \ 
// --data '{"name":"john","pass":"doe"}'  localhost:3000

// curl -X POST -H "Content-Type: application/xml" \ 
// --data '<Login><name>john</name><pass>doe</pass><Login>'  localhost:3000

// curl -X POST -H "Content-Type: application/x-www-form-urlencoded" \
// --data 'name=john&pass=doe'  localhost:3000

// curl -v -F name=john -F pass=doe http://localhost:3000

type Person struct {
    Name string `json:"name" xml:"name" form:"name"`
    Pass string `json:"pass" xml:"pass" form:"pass"`
}

app.Post("/", func(c *fiber.Ctx) {
    var person Person

    if err := c.BodyParser(&person); err != nil {
        // Handle error
    }

    // Do something with person.Name or person.Pass
})
```

## ClearCookie

通过设置过去的过期日期，清除**所有**客户端cookie \(或单个由**名称**指定的单个cookie\)。

**签名**

```go
c.ClearCookie()
c.ClearCookie(key string)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  // Clears all cookies:
  c.ClearCookie()

  // Expire specific cookie by name:
  c.ClearCookie("user")

  // Expire multiple cookies by names:
  c.ClearCookie("token", "session", "track_id", "version")
})
```

## Cookie

通过**name**和**value**设置cookie

**签名**

```go
c.Cookie(name, value string, options ...*Cookie{})
```

**Cookie 结构**

{% hint style="warning" %}
如果设置了**MaxAge**，则将不使用**Expire**选项
{% endhint %}

```go
&fiber.Cookie{
  Expire   int64  // Unix timestamp
  MaxAge   int    // Seconds
  Domain   string
  Path     string
  HTTPOnly bool
  Secure   bool
  SameSite string
}
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Cookie("name", "john")
  // => Cookie: name=john;

  c.Cookie("name", "john", &fiber.Cookie{
    MaxAge:   60,
    Domain:   "example.com",
    Path:     "/",
    HTTPOnly: true,
    Secure:   true,
    SameSite: "lax",
  })
  // => name=john; max-age=60; domain=example.com; path=/;
  //    HttpOnly; secure; SameSite=Lax

})
```

## Cookies

获取 cookie.

**签名**

```go
c.Cookies(key ...string) string
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  // Get raw cookie header:
  c.Cookies() // name=john;

  // Get cookie by key:
  c.Cookies("name") // "john"
})
```

## Download

从路径中将文件作为`附件`传输。
Transfers the file from path as an `attachment`.

通常，浏览器会提示用户下载。 默认情况下，[Content-Disposition](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition)头`filename =`参数是文件路径\(通常会出现在浏览器对话框中\)。

使用**filename**参数覆盖此默认值。

**签名**

```go
c.Download(path, filename ...string)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Download("./files/report-12345.pdf")
  // => Download report-12345.pdf

  c.Download("./files/report-12345.pdf", "report.pdf")
  // => Download report.pdf
})
```

## Fasthttp

可以**访问**并使用所有**Fasthttp**方法和属性。

**签名**

{% hint style="info" %}
请阅读[Fasthttp文档](https://pkg.go.dev/github.com/valyala/fasthttp?tab=doc)以获取更多信息。
{% endhint %}

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Fasthttp.Request.Header.Method()
  // => []byte("GET")

  c.Fasthttp.Response.Write([]byte("Hello, World!"))
  // => "Hello, World!"
})
```

## Error

这包含由panic发或通过[`Next(err)`](https://github.com/gofiber/docs/tree/8d965e1e05fb67f965934586c78335ef29f52128/context/README.md#error)方法传递的错误信息。

**签名**

```go
c.Error() error
```

**示例**

```go
func main() {
  app := fiber.New()
  app.Post("/api/register", func (c *fiber.Ctx) {
    if err := c.JSON(&User); err != nil {
      c.Next(err)
    }
  })
  app.Get("/api/user", func (c *fiber.Ctx) {
    if err := c.JSON(&User); err != nil {
      c.Next(err)
    }
  })
  app.Put("/api/update", func (c *fiber.Ctx) {
    if err := c.JSON(&User); err != nil {
      c.Next(err)
    }
  })
  app.Use("/api", func(c *fiber.Ctx) {
    c.Set("Content-Type", "application/json")
    c.Status(500).Send(c.Error())
  })
  app.Listen(1337)
}
```

## Format

在[Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)HTTP头上执行内容协商。 它使用[Accepts](context.md#accepts)选择适当的格式。

{% hint style="info" %}
如果未指定标头或格式不正确，则使用**text/plain**。
{% endhint %}

**签名**

```go
c.Format(body interface{})
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  // Accept: text/plain
  c.Format("Hello, World!")
  // => Hello, World!

  // Accept: text/html
  c.Format("Hello, World!")
  // => <p>Hello, World!</p

  // Accept: application/json
  c.Format("Hello, World!")
  // => "Hello, World!"
})
```

## FormFile

可以按名称检索MultipartForm文件，然后返回给定键的**first**文件。

**签名**

```go
c.FormFile(name string) (*multipart.FileHeader, error)
```

**示例**

```go
app.Post("/", func(c *fiber.Ctx) {
  // Get first file from form field "document":
  file, err := c.FormFile("document")

  // Check for errors:
  if err == nil {
    // Save file to root directory:
    c.SaveFile(file, fmt.Sprintf("./%s", file.Filename))
  }
})
```

## FormValue

可以按名称检索MultipartForm值，并返回给定键的**first**值。

**签名**

```go
c.FormValue(name string) string
```

**示例**

```go
app.Post("/", func(c *fiber.Ctx) {
  // Get first value from form field "name":
  c.FormValue("name")
  // => "john" or "" if not exist
})
```

## Fresh

[https://expressjs.com/en/4x/api.html\#req.fresh](https://expressjs.com/en/4x/api.html#req.fresh)

{% hint style="info" %}
尚未执行，欢迎提出请求！
{% endhint %}

## Get

返回由字段指定的HTTP请求标头。

{% hint style="success" %}
匹配**不区分大小写**。
{% endhint %}

**Signature**

```go
c.Get(field string) string
```

**Example**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Get("Content-Type") // "text/plain"
  c.Get("CoNtEnT-TypE") // "text/plain"
  c.Get("something")    // ""
})
```

## Hostname

包含从[Host](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Host)HTTP头派生的主机名。

**签名**

```go
c.Hostname() string
```

**示例**

```go
// GET http://google.com/search

app.Get("/", func(c *fiber.Ctx) {
  c.Hostname() // "google.com"
})
```

## IP

返回请求的远程IP地址。

**签名**

```go
c.IP() string
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.IP() // "127.0.0.1"
})
```

## IPs

返回[X-Forwarded-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)请求标头中指定的IP地址数组。

**签名**

```go
c.IPs() []string
```

**示例**

```go
// X-Forwarded-For: proxy1, 127.0.0.1, proxy3

app.Get("/", func(c *fiber.Ctx) {
  c.IPs() // ["proxy1", "127.0.0.1", "proxy3"]
})
```

## Is

如果传入请求的[Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type)HTTP头字段匹配，则返回匹配的**内容类型**由type参数指定的[MIME类型](https://developer.mozilla.org/ru/docs/Web/HTTP/Basics_of_HTTP/MIME_types)。

{% hint style="info" %}
如果请求的主体为**no**，则返回**false**。
{% endhint %}

**签名**

```go
c.Is(t string) bool
```

**示例**

```go
// Content-Type: text/html; charset=utf-8

app.Get("/", func(c *fiber.Ctx) {
  c.Is("html")  // true
  c.Is(".html") // true
  c.Is("json")  // false
})
```

## JSON

使用[Jsoniter](https://github.com/json-iterator/go)将任何**interface**或**string**转换为JSON。

{% hint style="info" %}
JSON还将内容标头设置为**application/json**。
{% endhint %}

**签名**

```go
c.JSON(v interface{}) error
```

**示例**

```go
type SomeStruct struct {
  Name string
  Age  uint8
}

app.Get("/json", func(c *fiber.Ctx) {
  // Create data struct:
  data := SomeStruct{
    Name: "Grame",
    Age:  20,
  }

  c.JSON(data)
  // => Content-Type: application/json
  // => "{"Name": "Grame", "Age": 20}"

  c.JSON(fiber.Map{
    "name": "Grame",
    "age": 20,
  })
  // => Content-Type: application/json
  // => "{"name": "Grame", "age": 20}"
})
```

## JSONP

发送带有JSONP支持的JSON响应。 除了选择加入JSONP回调支持外，此方法与[JSON](context.md#json)相同。 默认情况下，回调名称只是回调。

通过在方法中传递**name string**来覆盖它。

**签名**

```go
c.JSONP(v interface{}, callback ...string) error
```

**示例**

```go
type SomeStruct struct {
  name string
  age  uint8
}

app.Get("/", func(c *fiber.Ctx) {
  // Create data struct:
  data := SomeStruct{
    name: "Grame",
    age:  20,
  }

  c.JSONP(data)
  // => callback({"name": "Grame", "age": 20})

  c.JSONP(data, "customFunc")
  // => customFunc({"name": "Grame", "age": 20})
})
```

## Links

在属性后面加入链接，以填充响应的[Link](https://developer.mozilla.org/ru/docs/Web/HTTP/Headers/Link)HTTP头字段。

**签名**

```go
c.Links(link ...string)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Link(
    "http://api.example.com/users?page=2", "next",
    "http://api.example.com/users?page=5", "last",
  )
  // Link: <http://api.example.com/users?page=2>; rel="next",
  //       <http://api.example.com/users?page=5>; rel="last"
})
```

## Locals

存储范围为请求的字符串变量的方法，因此该方法仅可用于与请求匹配的路由。

{% hint style="success" %}
如果要将某些**特定**的数据传递给下一个中间件，这很有用。
{% endhint %}

**签名**

```go
c.Locals(key string, value ...interface{}) interface{}
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Locals("user", "admin")
  c.Next()
})

app.Get("/admin", func(c *fiber.Ctx) {
  if c.Locals("user") == "admin" {
    c.Status(200).Send("Welcome, admin!")
  } else {
    c.SendStatus(403)
    // => 403 Forbidden
  }
})
```

{% hint style="info" %}
您可以将任何类型放入**Locals**中，但是在使用变量时，请不要忘记将其转换回来。
{% endhint %}

```go
type SomeStruct struct {
  Message string `json:"message"`
}

app.Get("/", func(c *fiber.Ctx) {
  c.Locals("user", SomeStruct{"Hello, World!"})
  // => user: {"message":"Hello, World!"}

  c.Next()
})

app.Get("/", func(c *fiber.Ctx) {
  if val, ok := c.Locals("user").(SomeStruct); ok {
    fmt.Println(val.Message)
    // => "Hello, World!"
  }
})
```

## Location

将响应[Location](https://developer.mozilla.org/ru/docs/Web/HTTP/Headers/Location)HTTP头设置为指定的路径参数。

**签名**

```go
c.Location(path string)
```

**示例**

```go
app.Post("/", func(c *fiber.Ctx) {
  c.Location("http://example.com")
  c.Location("/foo/bar")
})
```

## Method

包含与请求的HTTP方法相对应的字符串: `GET`，`POST`，`PUT`等。

**签名**

```go
c.Method() string
```

**示例**

```go
app.Post("/", func(c *fiber.Ctx) {
  c.Method() // "POST"
})
```

## MultipartForm

要访问多部分表单条目，可以使用`MultipartForm()`解析二进制文件。 这会返回一个`map[string][]string`，因此给定键的值将是一个字符串切片。

**签名**

```go
c.MultipartForm() (*multipart.Form, error)
```

**示例**

```go
app.Post("/", func(c *fiber.Ctx) {
  // Parse the multipart form:
  if form, err := c.MultipartForm(); err == nil {
    // => *multipart.Form

    if token := form.Value["token"]; len(token) > 0 {
      // Get key value:
      fmt.Println(token[0])
    }

    // Get all files from "documents" key:
    files := form.File["documents"]
    // => []*multipart.FileHeader

    // Loop through files:
    for _, file := range files {
      fmt.Println(file.Filename, file.Size, file.Header["Content-Type"][0])
      // => "tutorial.pdf" 360641 "application/pdf"

      // Save the files to disk:
      c.SaveFile(file, fmt.Sprintf("./%s", file.Filename))
    }
  }
})
```

## Next

当调用**Next**时，它将执行堆栈中与当前路由匹配的next方法。 您可以在方法中传递错误结构以进行自定义错误处理。

**签名**

```go
c.Next(err ...error)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  fmt.Printl("1st route!")
  c.Next()
})

app.Get("*", func(c *fiber.Ctx) {
  fmt.Printl("2nd route!")
  c.Next(fmt.Errorf("Some error"))
})

app.Get("/", func(c *fiber.Ctx) {
  fmt.Println(c.Error()) // => "Some error"
  fmt.Println("3rd route!")
  c.Send("Hello, World!")
})
```

## OriginalURL

包含原始请求URL。

**签名**

```go
c.OriginalURL() string
```

**示例**

```go
// GET http://example.com/search?q=something

app.Get("/", func(c *fiber.Ctx) {
  c.OriginalURL() // "/search?q=something"
})
```

## Params

方法可用于获取路由参数。

{% hint style="info" %}
如果参数**不存在**，则默认为空字符串\(`“”`\)。
{% endhint %}

**签名**

```go
c.Params(param string) string
```

**示例**

```go
// GET http://example.com/user/fenny

app.Get("/user/:name", func(c *fiber.Ctx) {
  c.Params("name") // "fenny"
})
```

## Path

包含请求URL的路径部分。

**签名**

```go
c.Path() string
```

**示例**

```go
// GET http://example.com/users?sort=desc

app.Get("/users", func(c *fiber.Ctx) {
  c.Path() // "/users"
})
```

## Protocol

包含请求协议字符串：**TLS**请求的`http`或`https`。

**签名**

```go
c.Protocol() string
```

**示例**

```go
// GET http://example.com

app.Get("/", func(c *fiber.Ctx) {
  c.Protocol() // "http"
})
```

## Query

此属性是一个对象，其中包含路由中每个查询字符串参数的属性。

{% hint style="info" %}
如果**没有**查询字符串，则返回**空字符串**。
{% endhint %}

**签名**

```go
c.Query(parameter string) string
```

**示例**

```go
// GET http://example.com/shoes?order=desc&brand=nike

app.Get("/", func(c *fiber.Ctx) {
  c.Query("order") // "desc"
  c.Query("brand") // "nike"
})
```

## Range

{% hint style="info" %}
快来了！ 随时创建PR！
{% endhint %}

## Redirect

重定向到具有指定状态的从指定路径派生的URL，该状态为与HTTP状态代码相对应的正整数。

{% hint style="info" %}
如果**未指定**，则状态默认为**302 Found**。
{% endhint %}

**签名**

```go
c.Redirect(path string, status ...int)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Redirect("/foo/bar")
  c.Redirect("../login")
  c.Redirect("http://example.com")
  c.Redirect("http://example.com", 301)
})
```

## Render

渲染带有数据的模板并发送`text/html`响应。 我们支持以下模板引擎：

| Keyword | Engine |
| :--- | :--- |
| `html` | [golang.org/pkg/html/template/](https://golang.org/pkg/html/template/) |
| `amber` | [github.com/eknkc/amber](https://github.com/eknkc/amber) |
| `handlebars` | [github.com/aymerick/raymond](https://github.com/aymerick/raymond) |
| `mustache` | [github.com/cbroglie/mustache](https://github.com/cbroglie/mustache) |
| `pug` | [github.com/Joker/jade](https://github.com/Joker/jade) |

**签名**

```go
c.Render(view string, data interface{}, engine ...string) error
```

个带有****`mustache` ****语法的简单模板文件。

{% code title="/views/home.tmpl" %}
```go
<html>
    <head>
        <title>{{title}}</title>
    </head>
<body>
    I wish it was {{year}}
</body>
</html>
```
{% endcode %}

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  // fiber.Map == map[string]interface{}
  data := fiber.Map{
    "title": "Homepage",
    "year":  1999,
  }
  c.Render("./views/home.tmpl", data, "mustache")
})
```

{% hint style="success" %}
如果您全局设置模板设置，则更加容易！
{% endhint %}

```go
app := fiber.New(&fiber.Settings{
  ViewEngine:    "mustache",
  ViewFolder:    "./views",
  ViewExtension: ".tmpl",
})

app.Get("/", func(c *fiber.Ctx) {
  c.Render("home", fiber.Map{
    "title": "Homepage",
    "year":  1999,
  })
})
```

## Route

包含匹配的[Route](https://pkg.go.dev/github.com/gofiber/fiber?tab=doc#Route)结构。

**签名**

```go
c.Route() *Route
```

**示例**

```go
// http://localhost:8080/hello

app.Get("/hello", func(c *fiber.Ctx) {
  r := c.Route()
  fmt.Println(r.Method, r.Path, r.Prefix, r.Regex, r.Params, r.HandlerCtx)
})

app.Post("/:api?", func(c *fiber.Ctx) {
  c.Route()
  // => {POST /:api?  ^(?:/([^/]+?))?/?$ [api] 0x7b49e0}
})
```

## SaveFile

方法用于将任何部分文件保存到磁盘。

**签名**

```go
c.SaveFile(fh *multipart.FileHeader, path string)
```

**示例**

```go
app.Post("/", func(c *fiber.Ctx) {
  // Parse the multipart form:
  if form, err := c.MultipartForm(); err == nil {
    // => *multipart.Form

    // Get all files from "documents" key:
    files := form.File["documents"]
    // => []*multipart.FileHeader

    // Loop through files:
    for _, file := range files {
      fmt.Println(file.Filename, file.Size, file.Header["Content-Type"][0])
      // => "tutorial.pdf" 360641 "application/pdf"

      // Save the files to disk:
      c.SaveFile(file, fmt.Sprintf("./%s", file.Filename))
    }
  }
})
```

## Secure

一个布尔属性，如果建立**TLS**连接，则为true。

**签名**

```go
c.Secure() bool
```

**示例**

```go
// Secure() method is equivalent to:
c.Protocol() == "https"
```

## Send
设置HTTP响应体。 **发送**主体可以是任何类型。

{% hint style="warning" %}
像[Write](https://fiber.wiki/context#write)方法一样发送**不**追加。
{% endhint %}

**签名**

```go
c.Send(body ...interface{})
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Send("Hello, World!")         // => "Hello, World!"
  c.Send([]byte("Hello, World!")) // => "Hello, World!"
  c.Send(123)                     // => 123
})
```

Fibre还为原始输入提供了`SendBytes`和`SendString`方法。

{% hint style="success" %}
如果**不需要**类型断言，请使用此选项，建议使用**更好**的性能。
{% endhint %}

**签名**

```go
c.SendBytes(b []byte)
c.SendString(s string)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.SendByte([]byte("Hello, World!"))
  // => "Hello, World!"

  c.SendString("Hello, World!")
  // => "Hello, World!"
})
```

## SendFile

从给定的路径传输文件。 根据**filenames**扩展名设置[Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type)响应HTTP头字段

{% hint style="info" %}
方法默认使用**gzipping**，将其设置为**false**以禁用。
{% endhint %}

**签名**

```go
c.SendFile(path string, gzip ...bool)
```

**示例**

```go
app.Get("/not-found", func(c *fiber.Ctx) {
  c.SendFile("./public/404.html")

  // Disable gzipping:
  c.SendFile("./static/index.html", false)
})
```

## SendStatus

如果响应主体为**空**，则在主体中设置状态代码和正确的状态消息。

{% hint style="success" %}
您可以找到所有使用过的状态代码和消息[here](https://github.com/gofiber/fiber/blob/dffab20bcdf4f3597d2c74633a7705a517d2c8c2/utils.go#L183-L244).
{% endhint %}

**签名**

```go
c.SendStatus(status int)
```

**示例**

```go
app.Get("/not-found", func(c *fiber.Ctx) {
  c.SendStatus(415)
  // => 415 "Unsupported Media Type"

  c.Send("Hello, World!")
  c.SendStatus(415)
  // => 415 "Hello, World!"
})
```

## Set

将响应的HTTP头字段设置为指定的`键`，`值`。

**签名**

```go
c.Set(field, value string)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Set("Content-Type", "text/plain")
  // => "Content-type: text/plain"
})
```

## Stale

[https://expressjs.com/en/4x/api.html\#req.fresh](https://expressjs.com/en/4x/api.html#req.fresh)

{% hint style="info" %}
尚未执行，欢迎提出请求！
{% endhint %}

## Status

设置HTTP响应的状态。

{% hint style="info" %}
方法是**可链接的**。
{% endhint %}

**签名**

```go
c.Status(status int)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Status(200)
  c.Status(400).Send("Bad Request")
  c.Status(404).SendFile("./public/gopher.png")
})
```

## Subdomains

请求的域名中的子域数组。

应用程序属性的子域偏移量(默认为`2`)，用于确定子域段的开头。

**签名**

```go
c.Subdomains(offset ...int) []string
```

**示例**

```go
// Host: "tobi.ferrets.example.com"

app.Get("/", func(c *fiber.Ctx) {
  c.Subdomains()  // ["ferrets", "tobi"]
  c.Subdomains(1) // ["tobi"]
})
```

## Type

将[Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type)HTTP头设置为[此处](https://github.com/nginx/nginx/blob/master/conf/mime.types)列出的MIME类型，由**扩展名**指定。

**签名**

```go
c.Type(t string) string
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Type(".html") // => "text/html"
  c.Type("html")  // => "text/html"
  c.Type("json")  // => "application/json"
  c.Type("png")   // => "image/png"
})
```

## Vary

将给定的标头字段添加到[Vary](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary)响应标头中。 如果标题尚未列出，则将附加该标题，否则将其保留在当前位置列出。

{% hint style="info" %}
允许使用多个字段。
{% endhint %}

**签名**

```go
c.Vary(field ...string)
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Vary("Origin")     // => Vary: Origin
  c.Vary("User-Agent") // => Vary: Origin, User-Agent
  
  // No duplicates
  c.Vary("Origin") // => Vary: Origin, User-Agent

  c.Vary("Accept-Encoding", "Accept")
  // => Vary: Origin, User-Agent, Accept-Encoding, Accept
})
```

## Write

将**any**输入追加到HTTP正文响应中。

**签名**

```go
c.Write(body ...interface{})
```

**示例**

```go
app.Get("/", func(c *fiber.Ctx) {
  c.Write("Hello, ")         // => "Hello, "
  c.Write([]byte("World! ")) // => "Hello, World! "
  c.Write(123)               // => "Hello, World! 123"
})
```

## XHR

如果请求的[X-Requested-With](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)头字段为[XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)，表明请求是由客户端库\(例如[jQuery](https://api.jquery.com /jQuery.ajax/)\)。

**签名**

```go
c.XHR() bool
```

**示例**

```go
// X-Requested-With: XMLHttpRequest

app.Get("/", func(c *fiber.Ctx) {
  c.XHR() // true
})
```