---
title: http post 数据分析
categories:
- JavaScript
tags: 
- Golang Http
- 模块化
date: "2019-04-20"
ShowToc: true
TocOpen: true
---

在一次POST图片的base64编码时，后端（golang）始终无法正确获取提交的base64，排查发现后端net包的url模块在parseQuery提交的数据时，把参数拆成了2部分，一番尝试，把提交的数据编码后就可以了；猜测应该是特殊字符问题，遂整理了以下知识点：

<!--more-->

## 理论先行

下面摘录自[RFC 1738](https://www.ietf.org/rfc/rfc1738.txt)

>
>  Reserved:
>
>  Many URL schemes reserve certain characters for a special meaning:
>  their appearance in the scheme-specific part of the URL has a
>  designated semantics. If the character corresponding to an octet is
>  reserved in a scheme, the octet must be encoded.  The characters ";",
>  "/", "?", ":", "@", "=" and "&" are the characters which may be
>  reserved for special meaning within a scheme. No other characters may
>  be reserved within a scheme.
>
>  Usually a URL has the same interpretation when an octet is
>  represented by a character and when it encoded. However, this is not
>  true for reserved characters: encoding a character reserved for a
>  particular scheme may change the semantics of a URL.
>
>  Thus, only alphanumerics, the special characters "$-_.+!*'(),", and
>  reserved characters used for their reserved purposes may be used
>  unencoded within a URL.
>
>  On the other hand, characters that are not required to be encoded
>  (including alphanumerics) may be encoded within the scheme-specific
>  part of a URL, as long as they are not being used for a reserved
>  purpose.
>

概括下：

1. 字符**; / ? : @ = &**是保留字符，在url中有特殊语义；
2. 只有**数字字母**、**特定字符$,_.+!*()**、**保留字符**可以不经过编码直接用于URL，
3. 也就是说**汉字、阿拉伯文等非英语语种，以及在http中`提交的参数值包含保留字符`的都需要经过编码**([js url编码](https://www.haorooms.com/post/js_escape_encodeURIComponent))

>
> In addition, after the <hsoname>, optional fields and values
> associated with a Prospero link may be specified as part of the URL.
> When present, each field/value pair is separated from each other and
> from the rest of the URL by a ";" (semicolon). The name of the field
> and its value are separated by a "=" (equal sign). If present, these
> fields serve to identify the target of the URL.  For example, the
> OBJECT-VERSION field can be specified to identify a specific version
> of an object.
>

大意：hostname后面可以带`field=value`形式的参数，每对field/value通过`&`或`;`分割

## 分析代码

golang net/http的parsePostForm方法使用url.ParseQuery来解析POST的数据，也就是根据上面协议来解析数据

```go
// net/http下的parsePostForm方法
...
switch {
  case ct == "application/x-www-form-urlencoded":
    var reader io.Reader = r.Body
    maxFormSize := int64(1<<63 - 1)
    if _, ok := r.Body.(*maxBytesReader); !ok {
      maxFormSize = int64(10 << 20) // 10 MB is a lot of text.
      reader = io.LimitReader(r.Body, maxFormSize+1)
    }
    b, e := ioutil.ReadAll(reader)
    if e != nil {
      if err == nil {
        err = e
      }
      break
    }
    if int64(len(b)) > maxFormSize {
      err = errors.New("http: POST too large")
      return
    }
    vs, e = url.ParseQuery(string(b))			// <- 看这里，解析POST的body
    if err == nil {
      err = e
    }
...
```
```go
// net/url下的parseQuery方法
...
if i := strings.IndexAny(key, "&;"); i >= 0 { // <- 看这里，每对field/value通过&或;分割
  key, query = key[:i], key[i+1:]
} else {
  query = ""
}
if key == "" {
  continue
}
value := ""
if i := strings.Index(key, "="); i >= 0 {
  key, value = key[:i], key[i+1:]
}
key, err1 := QueryUnescape(key)
if err1 != nil {
  if err == nil {
    err = err1
  }
  continue
}
value, err1 = QueryUnescape(value)
...
```

## 得出结论

回到问题，POST提交参数如：

Content-Type: application/x-www-form-urlendcoded

Body:

```
img_name=gratification.jpg&img=图片base64编码值
```

图片base64编码格式如：

```
data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEASABIAAD/2wBDAAMCAgMCAgMDAwMEAwMEBQgFBQQEBQoHBwYIDAoMDAsKCwsNDhIQDQ4RDgsLEBYQERMUFRUVDA8XGBYUGBIUFRT/2wBDAQMEBAUEBQkFBQkUDQsNFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBT/wAARCAKaBAADAREAAhEBAxEB/8QAHgAAAAcBAQEBAAAAAAAAAAAAAAECAwQFBgcICQr/xABgEAACAQMCBAQDBQUEBAgJBBMBAgMABBEFIQYSMUEHE1FhCCJxFDKBkaEJFSNCsVJiwdEWcrLwJDNDRIKSs+EXJTRTc6K0wvEmJ1R0dpOj0hg1NjdFVWRldWNng5Sk4v/EABwBAAIDAQEBAQAAAAAAAAAAAAABAgMEBQYHCP/EAEsRAAICAQMDAQQIAwYEBQEGBwABAhEDBCExBRJBURMiYXEGMoGRobHB0RTh8CMzQlJyshUkYnMHJTSC8TUWQ1NUkqLC0iajNkSD/9oADAMBAAIRAxEAPwD1tyn618dR4cMDJ6U07JUOBCKlQvIoJ61OiQePWmARFAuRJHtRyLYQ4pcAxFMQlsH/ABoAIgdSKl5FQkjsaV+omhBA7UX6CC...
```

`:/;`是保留字符，`+,`是特定字符，根据RFC的说明，这些字符放在URL上的话不需要编码，但我们是POST这些数据，要想后端在net/url能正确parseQuery数据**(分号会导致错误解析img为：img=data:image/jpeg和base64=xxxxxx)**，我们要编码图片base64

**那是否POST的数据都要经过编码呢？**

如果Content-Type是`application/x-www-form/urlencoded`则需要像GET里的参数一样进行URL encode；如果是`multipart/form-data`则不需要，它使用内容分隔符分割数据，数据不需要编码

另外POST二进制或提交特别大的数据，Content-Type建议使用`multipart/form-data`

## 参考

[RFC Uniform Resource Locators (URL)](https://www.ietf.org/rfc/rfc1738.txt)

[MDN POST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)

[MDN URL encoding(Percent-encoding)](<https://developer.mozilla.org/en-US/docs/Glossary/percent-encoding>)

[POST时Content-Type该用application/x-www-form-urlencoded还是multipart/form-data](https://stackoverflow.com/questions/4007969/application-x-www-form-urlencoded-or-multipart-form-data)

[Should I URL-encode POST data?](https://stackoverflow.com/questions/6603928/should-i-url-encode-post-data)

[axios 使用post方式传递参数，后端接收不到](https://segmentfault.com/a/1190000012635783)