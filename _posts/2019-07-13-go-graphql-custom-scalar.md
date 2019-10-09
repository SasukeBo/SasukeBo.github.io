---
layout: post
title:  "go-graphql 自定义 scalar"
date:   2019-07-13 16:11:00 +0800
categories: golang
---

学习笔记，写这篇笔记时才刚入门golang，请多指教。

### 场景

假设有一个 `Role` 模型
```go
type Role struct {
	Id     int
	Name   string
	Status RoleStatus
}
```

其中`RoleStatus`类型表示角色的状态：
```go
type RoleStatus int

const (
	RDefault RoleStatus = iota
	RPublish
	RBlock
	RDeleted
)
```
所以数据库在存储角色状态时是`Int`型，而使用`graphql`开发接口想以`string`类型输出状态
的名称，此时就可以利用自定义`Scalar`功能。

下面这个map用于存储状态名称值和RoleStatus之间的映射：
```go
type RSMapType map[interface{}]interface{}

var RoleStatusMap = RSMapType{
	"default": RDefault,
	"publish": RPublish,
	"block":   RBlock,
	"deleted": RDeleted,
}
```

### 代码及解释

假设我们有一个`custom`包用于放置所有的custom Scalar。

定义一个`Scalar`需要完成`ScalarConfig`的5个`Field`
```go
package custom

// RoleStatus is a custom GQL type
var RoleStatus = graphql.NewScalar(graphql.ScalarConfig{
	Name: "RoleStatus",
	Description: `roleStatus Scalar is represent role current status,
	it convert string type to models.RoleStatus type for DB,
	and convert models.RoleStatus type to string type for output`,
	// Serialize 用于将数据库中的 models.RoleStatus 类型值转换为 string 类型从GQL接口输出
	Serialize: serialize,
	// ParseValue 用于转换通过 variables 形式传递给GQL query string的变量值
	ParseValue: parseValue,
	// ParseLiteral 用于转换GQL inline query string的参数值
	ParseLiteral: parseLiteral,
})
```

- Name 字段用于命名自定义的Scalar，你会在GQL IDE中看到这个类型的名称。


- Description 描述Scalar。
- Serialize 是一个`func`，用于处理输出到GQL前的转换
```go
// 输出到GQL之前值为models.RoleStatus类型，需要用上面定义的map来获取对应的状态名称。
var serialize = func(value interface{}) interface{} {
	rs, ok := value.(models.RoleStatus)
	if !ok {
		return nil
	}
	key := RMap(models.RoleStatusMap, rs)
	return key
},
```
- ParseValue 也是一个`func`，当GQL请求带有参数，而且参数值是以变量的形式传递，则会执行该方法用来转换
GQL输入为models.RoleStatus类型。
```go
// 从GQL输入的数据类型是已知的，此处我们接收到的需要是string类型，
// 然后借助map获取对应的models.RoleStatus类型值
var parseValue = func(value interface{}) interface{} {
	key, ok := value.(string)
	if !ok {
		logs.Error("value is not a string")
		return nil
	}
	if value := models.RoleStatusMap[key]; value != nil {
		return value
	}
	logs.Error("value is not a RoleStatus type value")
	return nil
},
```
- ParseLiteral 同样也是`func`，当GQL请求带有参数而且参数值是内嵌(`inline`)到query string当中时，会执行该方法转换GQL参数值为models.RoleStatus类型值。
```go
// 同样这里获取的参数值也是string类型，所以只需要判断类型是否为string，然后再转换为models.RoleStatus类型
var parseLiteral = func(valueAST ast.Value) interface{} {
	switch valueAST.(type) {
	case *ast.StringValue:
		if value := models.RoleStatusMap[valueAST.GetValue().(string)]; value != nil {
			return value
		}
	}
	logs.Error("value is not a RoleStatus type value")
	return nil
},
```

正确实现这三个方法基本上就成功定义了一个Scalar。
![rolestatus1.png](https://upload-images.jianshu.io/upload_images/14226559-e0bdc5d983355bf1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
