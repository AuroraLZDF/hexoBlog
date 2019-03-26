---
title: PHP-数组根据某键值，把相同键值的合并最终生成一个新的二维数组.md
date: 2019-02-21 15:28:07
tags: [PHP、笔记]
categories: [PHP]
toc: true
cover: '/images/categories/php.jpeg'
---

**说明**：

将原数组中的有某个相同键值的二维数组合并到一个数组中重新组成一个二维数组。

要处理的PHP数组：

```php
$infos = array(    
    array(
        'gid' => 36,
        'name' => '高二佳木斯',         
        'start_time' => '2015-08-28 00:00:00',           
        'pic' => '2015/08/438488a00b3219929282e3652061c2e3.png'                   
    ),    
    array(          
        'gid' => 36,
        'name' => '高二佳木斯',    
        'start_time' => '2015-08-20 00:00:00',            
        'pic' => '2015/08/438488a00b3219929282e3652061c2e3.png'    
    ),   
    array(           
        'gid' => 36,
        'name' => '高二佳木斯',        
        'start_time' => '2015-08-28 00:00:00',  
        'pic' => '2015/08/438488a00b3219929282e3652061c2e3.png'     
    ),    
    array(          
        'gid' => 36,
        'name' => '高二佳木斯',        
        'start_time' => '2015-08-27 00:00:00',  
        'pic' => '2015/08/438488a00b3219929282e3652061c2e3.png'    
    ),   
    array(           
        'gid' => 18,           
        'name' => '天书',          
        'start_time' => '2015-08-24 00:00:00',           
        'pic' => 'dev/2015/08/438488a00b3219929282e3652061c2e3.png'       
    ),   
    array(         
        'gid' => 17,           
        'name' => '晒黑西游',          
        'start_time' => '2015-08-06 00:00:00',       
        'pic' => ''
    )    
    array(           
       'gid' => 17,           
       'name' => '晒黑西游',           
       'start_time' => '2015-08-24 00:00:00',         
       'pic' => 
    )
);
```
**处理要求**：将数组中 `gid` 相同的二维数组合并到一个数组中，生成一个新的二维数组

代码：

```php
$result= array();
foreach ($infos as $key => $info) {
    $result[$info['gid']][] = $info;
} 
print_r($result);
```

输出：

```php
Array(  
  [36] => Array(            
      [0] => Array(                   
          [gid] => 36                   
          [name] => 高二佳木斯            
          [start_time] => 2015-08-28 00:00:00        
          [pic] => dev/2015/08/438488a00b3219929282e3652061c2e3.png                
      )            
     [1] => Array(
          [gid] => 36 
          [name] => 高二佳木斯                   
          [start_time] => 2015-08-20 00:00:00              
          [pic] => dev/2015/08/438488a00b3219929282e3652061c2e3.png                
      )           
      [2] => Arra(                    
          [gid] => 36                   
          [name] => 高二佳木斯               
          [start_time] => 2015-08-28 00:00:00           
          [pic] => dev/2015/08/438488a00b3219929282e3652061c2e3.png               
      )          
      [3] => Array(                    
          [gid] => 36                  
          [name] => 高二佳木斯               
          [start_time] => 2015-08-27 00:00:00           
          [pic] => dev/2015/08/438488a00b3219929282e3652061c2e3.png               
      ) 
  )   
  [18] => Array(            
      [0] => Array(                   
          [gid] => 18               
          [name] => 天书             
          [start_time] => 2015-08-24 00:00:00    
          [pic] => dev/2015/08/438488a00b3219929282e3652061c2e3.png                
      )               
  )        
  [17] => Array(            
      [0] => Arra(                 
          [gid] => 17           
          [name] => 晒黑西游      
          [start_time] => 2015-08-06 00:00:00        
          [pic] => 
      )            
     [1] => Array(            
         [gid] => 17       
         [name] => 晒黑西游       
         [start_time] => 2015-08-24 00:00:00            
         [pic] => 
     )        
  )
);
```