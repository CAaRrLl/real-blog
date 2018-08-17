#### 概述

> 一个rest api被建模为可单独寻址的资源的集合。资源以其resource name作为该资源的唯一标识，并通过一小组method（也称为动词或操作）来对资源进行操作
标准的method，如GET、POST、DELETE、PUT等，但是当某些功能不能简单与标准的method做映射时，可以使用自定义的method

#### 设计流程

* 确定API提供的资源类型
* 确定资源之间的关系
* 根据资源类型和资源之间的关系确定资源名称方案
* 确定资源模式，将最少的method集附加到资源

#### 资源

一组同类型的资源组成一个集合，比如一个用户有个联系人列表的集合。一个资源拥有一些状态以及零个或者多个子资源，每个子资源要么是一个资源，要么是一个集合
例如，Gmail API有一组用户，每个用户都有一组消息，一组线程，一组标签，一个配置文件资源和几个设置资源
Gmail API
Gmail API服务实现了Gmail API并公开了大多数Gmail功能。 
它具有以下资源模型：

``` javascript
API服务：gmail.googleapis.com
用户集合：users / *。
每个用户都有以下资源。
消息集合：users / * / messages / *。
线程集合：users / * / threads / *。
标签集合：users / * / labels / *。 
更改历史记录的集合：users / * / history / *。
表示用户配置文件的资源：users / * / profile。 
表示用户设置的资源：users / * / settings。
```

综上所述，rest api要么是一个资源要么是一个集合

#### method

rest api的关键特性在于它强调资源而不是资源上执行的method， 典型的rest api使用少量的method公开大量的资源。这些method可以是标准的也可以是自定义的。自定义method提供与传统RPC API相同的设计自由度，可用于实现常见的编程模式，例如数据库事务或数据分析

#### 资源名

资源名称由资源本身的ID，任何父资源的ID及其API服务名称组成

示例
|API服务名|集合id|资源id|集合id|资源id|
|----|---|---|----|---|
|//storage.googleapis.com|/buckets|/bucket-id|/objects|/object-id|
|//mail.googleapis.com|/users|/*@example.com|/settings|/customFrom

#### 自定义method
自定义method可以与资源，集合或服务相关联。它可能需要任意请求并返回任意响应，并且还支持流请求和响应
可以使用以下请求格式
```
https://service.name/v1/some/resource/name:customVerb
```
示例
| Method Name | Custom verb | HTTP verb | Note |
| --- | --- | --- | --- |
| Cancel | `:cancel` | `POST` | Cancel an outstanding operation (build, computation etc.) |
| BatchGet<plural noun> | `:batchGet` | `GET` | Batch get of multiple resources. (See details in [the description of List](https://cloud.google.com/apis/design/standard_methods#list)) |
| Move | `:move` | `POST` | Move a resource from one parent to another. |
| Search | `:search` | `GET` | Alternative to List for fetching data that does not adhere to List semantics. |
| Undelete | `:undelete` | `POST` | Restore a resource that was previously deleted. The recommended retention period is 30-day. |

***
#### 参考
[Google API Design](https://cloud.google.com/apis/design/)
