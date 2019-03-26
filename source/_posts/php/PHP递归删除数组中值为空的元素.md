---
title: PHP递归删除数组中值为空的元素.md
date: 2018-10-23 17:47:06
tag: [PHP]
categories: [PHP]
toc: true
cover: '/images/categories/php.jpeg'
---

```php
/**
 * 递归删除数组中值为空的元素
 * @param $arr
 * @return array
 */
function array_remove_empty($arr)
{
    $_arr = array();

    foreach($arr as $key => $val)
    {
        if (is_array($val))
        {
            $val = array_remove_empty($val);
            if (count($val) != 0)
            {
                $_arr[$key] = $val;
            }
        }
        else {
            if (trim($val) != ""){
                $_arr[$key] = $val;
            }
        }
    }
    unset($arr);
    return $_arr;
}
```