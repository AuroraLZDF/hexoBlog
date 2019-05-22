---
title: PHP-to-xml
date: 2019-04-15 17:43:02
tags: [Linux,PHP]
categories: [PHP，笔记]
toc: true
cover: '/images/categories/php.jpeg'
---

**方法**

```php
<?php
function toXml($data)
{
    $xml = '<xml>';
    foreach($data as $key => $val) {
        $xml .= is_numeric($val) ? 
            '<' . $key . '>' . $val . '</' . $key . '$>' :
        	'<' . $key . '><![CDATA[' . $val . ']]></' . $key . '>';
    }
    $xml .= '</xml>';

    return $xml;
}
```

**测试**

```php
$data = [
	'num' => 44,
	'inter' => 100,
	'json' => '{"php": "xml"}'
];
$res = toXml($data);

echo '<pre>';
var_dump($res);
echo '</pre>';
```

**结果**

```xml
<xml>
    <num>44</num>
    <inter>100</inter>
    <json><![CDATA[{"php": "xml"}]]></json>
</xml>
```

