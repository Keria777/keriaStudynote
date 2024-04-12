# 处理HTTP请求数据

在Web开发中，处理从客户端发送到服务器的数据对于动态交互至关重要。这些数据可以通过多种方式捕获，常见的方式有`FormQuery`（查询字符串参数）、`FormBody`（请求体）和`FormRoute`（URL路径参数）。下面，我们将探讨每种方法及其在应用中的用途。

## FormQuery（查询字符串参数）

**定义：** 查询字符串参数是URL的一部分，用于向服务器传递额外的数据。它们从问号（`?`）开始，并由和号（`&`）分隔。

**用途：**
- 在GET请求中过滤结果。
- 传递不敏感且不会改变服务器状态的数据。

**示例URL：** 
`http://example.com/api/products?category=books&price=20`

## FormBody（请求体）

**定义：** 请求体是在HTTP请求中发送的数据，通常与POST或PUT方法一起使用。它可以包括各种格式的数据，如表单数据、JSON或XML。

**用途：**
- 提交表单数据。
- 发送将改变服务器状态或存储在数据库中的数据。

**示例JSON请求体：**
```json
{
    "name": "John Doe",
    "email": "john@example.com"
}