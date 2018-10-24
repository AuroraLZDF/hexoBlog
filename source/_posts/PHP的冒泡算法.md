---
title: PHP的冒泡算法
date: 2017-03-14 14:28:30
tags: [PHP, 数据结构]
categories: [数据结构]
toc: true
---

```php
<?php
/* 冒泡算法
 * @para $arr 传人进去排序的数组
 * @return $newArr 排序之后的数组
 */
 
function maopao($arr){
    //一共是多少趟
    for($i = count($arr)-1; $i>0; $i--){
        $flag = 0;
        //每一趟进行相邻两个数进行比较
        for($j = 0; $j < $i; $j++){
            if($arr[$j]>$arr[$j+1]){
                $temp = $arr[$j];
                $arr[$j] = $arr[$j+1];
                $arr[$j+1] =$temp;
                $flag = 1;
            }
        }
        if($flag == 0){
            break;
        }
    }
    return $arr;
}
$arr=array(30,40,10,50,20,60);
print_r(maopao($arr));
?>
```
--------------------
print:
```php
Array
(
    [0] => 10
    [1] => 20
    [2] => 30
    [3] => 40
    [4] => 50
    [5] => 60
)
//冒泡算法：
原理是临近的数字两两进行比较,按照从小到大或者从大到小的顺序进行交换,
这样一趟过去后,最大或最小的数字被交换到了最后一位,
```