---
title: PHP遍历文件夹下子目录和文件
date: 2017-03-14 10:58:04
tags: [PHP]
categories: [PHP]
toc: true
---
```php
<?php

function read_all_dir ( $dir )
{
	$result = array();
	$handle = opendir($dir);
	if ( $handle )
	{
		while ( ( $file = readdir ( $handle ) ) !== false )
		{
			if ( $file != '.' && $file != '..')
			{
				$cur_path = $dir . DIRECTORY_SEPARATOR . $file;
				if ( is_dir ( $cur_path ) )
				{
					$result['dir'][$cur_path] = read_all_dir ( $cur_path );
				}
				else
				{
					$result['file'][] = $cur_path;
				}
			}
		}
		closedir($handle);
	}
	return $result;
}

$result = read_all_dir('/home/www/hexo');
var_dump($result);
```
-----
```php
print:

array(2) {
  ["file"]=>
  array(4) {
    [0]=>
    string(27) "/home/www/hexo/package.json"
    [1]=>
    string(22) "/home/www/hexo/db.json"
    [2]=>
    string(25) "/home/www/hexo/.npmignore"
    [3]=>
    string(26) "/home/www/hexo/_config.yml"
  }
  ["dir"]=>
  array(7) {
    ["/home/www/hexo/.idea"]=>
    array(2) {
      ["file"]=>
      array(7) {
        [0]=>
        string(34) "/home/www/hexo/.idea/encodings.xml"
        [1]=>
        string(30) "/home/www/hexo/.idea/blade.xml"
        [2]=>
        string(42) "/home/www/hexo/.idea/jsLibraryMappings.xml"
        [3]=>
        string(34) "/home/www/hexo/.idea/workspace.xml"
        [4]=>
        string(29) "/home/www/hexo/.idea/hexo.iml"
        [5]=>
        string(29) "/home/www/hexo/.idea/misc.xml"
        [6]=>
        string(32) "/home/www/hexo/.idea/modules.xml"
      }
      ["dir"]=>
      array(2) {
        ["/home/www/hexo/.idea/libraries"]=>
        array(1) {
          ["file"]=>
          array(1) {
            [0]=>
            string(52) "/home/www/hexo/.idea/libraries/hexo_node_modules.xml"
          }
        }
        ["/home/www/hexo/.idea/copyright"]=>
        array(1) {
          ["file"]=>
          array(1) {
            [0]=>
            string(52) "/home/www/hexo/.idea/copyright/profiles_settings.xml"
          }
        }
      }
    }
    ... ...
```
