---
title: PHP json_decode 遇到的坑
date: 2017-08-03 01:11:47
categories:
    - PHP
tags:
    - PHP
---

场景：某项目客户反馈，输出的结果 JSON 中有个要求为对象的数据字段，在某些情况下返回的是 `[]` 而不是 `{}`；数据由公司其他部门提供，查看原始数据的时候，没有发现任何问题；后来因为要加入某些预处理，在获取到其他部门的 JSON 数据之后进行解码并对某个字段进行处理；然而，在处理完之后再次使用 JSON 输出，发现结果已经不是我们想要的了。

原始数据
```json
{
  ...,
  "foo": "",
  "bar": {},
  ...
}
```
其中 `foo` 是我要进行处理的字段，处理完成之后再次使用 `json_encode($data)` 进行 JSON 编码。

编码完成之后的结果却是这样的

```json
{
  ...,
  "foo": "",
  "bar": [],
  ...
}
```
空对象编程了空数组，而且我并没有处理过字段 `bar`


通过对模拟数据的实测，发现是因为在对 JSON 进行解码的时候，是这么解的

```php
$data = json_decode($jsonString, true)
```

问题就出在这里，由于 PHP 自身的特性，在 PHP 中 `array` 是可以代表强类型语言，如 Java 中的 List 和 Map 的。

来看 PHP 中 `json_decode()` 方法是如何定义的

```php
mixed json_decode ( string $json [, bool $assoc = false [, int $depth = 512 [, int $options = 0 ]]] )
```  

> 来源 [http://php.net/manual/zh/function.json-decode.php](http://php.net/manual/zh/function.json-decode.php)

当第二个参数 `$assoc` 为 `true` 时，返回的类型是 `array`，所以问题就来了，当 JSON 中空对象 `{}` 和 `[]` 空数组，使用这种方式解码出来的结果表现是一致的；即 `array()`；当再次 `json_encode()` 编码的时候就出现了 `{}` 变 `[]` 了。

所以正确的做法是在解码 JSON 的时候 `json_decode` 不要传递第二个参数；让解码结果是一个对象，然后操作对象的属性，操作完成之后再次编码就不会出现偏差。

这种问题最突出在于强类型语言和弱类型语言的 API 对接上，由于项目的下家是使用 Java 语言，所以导致了 BUG

<!--more-->