---
title: webuploder+PHP实现大文件断点（切片）续传
date: 2017-05-08 11:18:07
tags: [webuploader, js, PHP]
toc: true
---
# 前言
做手机游戏运营难免会遇到上传超大APP，但受限于宽带、http响应时间等因素，导致文件上传不成功。在网上搜罗了一圈，终于在github上找到了[kazaff](https://github.com/kazaff)大神对于`webuploader`[大文件上传](http://blog.kazaff.me/2014/11/14/聊聊大文件上传/)的详细介绍。

*下面整理一下自己的实现过程吧（kazaff已做了详细介绍，这里只是借鉴一下）：*
文件目录结构：
<img src='/images/webuploader/webuploader_php_menu.png' />

`index.html`

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>WebUploader</title>
        <script type="text/javascript" src="js/jquery-1.11.2.min.js" charset="utf-8"></script>
        <script type="text/javascript" src="js/webuploader.js" charset="utf-8"></script>
        <script type="text/javascript" src="js/md5.js"  charset="utf-8"></script>
        <link rel="stylesheet" href="css/webuploader.css" />
        <style type="text/css">
            .itemDel, .itemStop, .itemUpload{
                margin-left: 15px;
                color: blue;
                cursor: pointer;
            }
            #theList{
                width: 80%;
                min-height: 100px;
                border: 1px solid red;
            }
            #theList .itemStop{
                display: none;
            }
        </style>
    </head>
    <body>
        <div id="uploader">
            <ul id="theList"></ul>
            <div id="picker">选择文件</div>
        </div>
        <p style="color:red">已实现大文件分段，FTP上传需要连接ftp服务器。</p>
    </body>
    <script type="text/javascript" src="js/bigFileUploader.js" charset="utf-8"></script>
</html>
```
<img src='/images/webuploader/index_html.png' />

`bigFileUploader.js`

```js
var userInfo = {userId:"up_file", md5:""};   //用户会话信息
var chunkSize =  2 * 1024 * 1024;        //分块大小
var uniqueFileName = null;          //文件唯一标识符
var md5Mark = null;

var backEndUrl = 'serverPHP/upload.php';

WebUploader.Uploader.register({
    "before-send-file": "beforeSendFile"
    , "before-send": "beforeSend"
    , "after-send-file": "afterSendFile"
}, {
    beforeSendFile: function (file) {
        //秒传验证
        var task = new $.Deferred();
        var start = new Date().getTime();
        (new WebUploader.Uploader()).md5File(file, 0, 10 * 1024 * 1024).progress(function (percentage) {
            console.log(percentage);
        }).then(function (val) {
            console.log("总耗时: " + ((new Date().getTime()) - start) / 1000);

            md5Mark = val;
            userInfo.md5 = val;

            $.ajax({
                type: "POST",
                url: backEndUrl,
                data: {
                    status: "md5Check",
                    md5: val
                },
                cache: false,
                timeout: 1000, //todo 超时的话，只能认为该文件不曾上传过
                dataType: "json"
            }).then(function (data, textStatus, jqXHR) {
                console.log(data, 1111);
                if (data.ifExist) {   //若存在，这返回失败给WebUploader，表明该文件不需要上传
                    task.reject();

                    uploader.skipFile(file);
                    file.path = data.path;
                    UploadComlate(file);
                } else {
                    task.resolve();
                    //拿到上传文件的唯一名称，用于断点续传
                    uniqueFileName = md5('' + userInfo.userId + file.name + file.type + file.lastModifiedDate + file.size);
                }
            }, function (jqXHR, textStatus, errorThrown) {    //任何形式的验证失败，都触发重新上传
                task.resolve();
                //拿到上传文件的唯一名称，用于断点续传
                uniqueFileName = md5('' + userInfo.userId + file.name + file.type + file.lastModifiedDate + file.size);
            });
        });
        return $.when(task);
    },
    beforeSend: function (block) {
        //分片验证是否已传过，用于断点续传
        var task = new $.Deferred();
        $.ajax({
            type: "POST",
            url: backEndUrl,
            data: {
                status: "chunkCheck",
                name: uniqueFileName,
                chunkIndex: block.chunk,
                size: block.end - block.start
            },
            cache: false,
            timeout: 1000, //todo 超时的话，只能认为该分片未上传过
            dataType: "json"
        }).then(function (data, textStatus, jqXHR) {
            if (data.ifExist) {   //若存在，返回失败给WebUploader，表明该分块不需要上传
                task.reject();
            } else {
                task.resolve();
            }
        }, function (jqXHR, textStatus, errorThrown) {    //任何形式的验证失败，都触发重新上传
            task.resolve();
        });

        return $.when(task);
    },
    afterSendFile: function (file) {
        var chunksTotal = 0;
        if ((chunksTotal = Math.ceil(file.size / chunkSize)) > 1) {
            //合并请求
            var task = new $.Deferred();
            $.ajax({
                type: "POST",
                url: backEndUrl,
                data: {
                    status: "chunksMerge",
                    name: uniqueFileName,
                    chunks: chunksTotal,
                    ext: file.ext,
                    md5: md5Mark
                },
                cache: false,
                dataType: "json"
            }).then(function (data, textStatus, jqXHR) {

                //todo 检查响应是否正常
                //console.log(data);
                task.resolve();
                file.path = data.path;
                UploadComlate(file, data);

            }, function (jqXHR, textStatus, errorThrown) {
                task.reject();
            });

            return $.when(task);
        } else {
            UploadComlate(file);
        }
    }
});

var uploader = WebUploader.create({
    swf: "Uploader.swf",
    server: backEndUrl,
    pick: "#picker",
    resize: false,
    dnd: "#theList",
    paste: document.body,
    disableGlobalDnd: true,
    thumb: {
        width: 100,
        height: 100,
        quality: 70,
        allowMagnify: true,
        crop: true
        //, type: "image/jpeg"
    },
    /*, compress: {
     quality: 90
     , allowMagnify: false
     , crop: false
     , preserveHeaders: true
     , noCompressIfLarger: true
     ,compressSize: 100000
     },*/
    compress: false,
    prepareNextFile: true,
    chunked: true,
    chunkSize: chunkSize,
    formData: function (){
        return $.extend(true, { }, userInfo);
    },
    threads:1,
    fileNumLimit: 1,
    fileSingleSizeLimit: 1000 * 1024 * 1024,
    duplicate: true
});

uploader.on("fileQueued", function (file) {

    $("#theList").append('<li id="' + file.id + '">' +
        '<img /><span>' + file.name + '</span><span class="itemUpload">上传</span><span class="itemStop">暂停</span><span class="itemDel">删除</span>' +
        '<div class="percentage"></div>' +
        '</li>');

    var $img = $("#" + file.id).find("img");

     uploader.makeThumb(file, function (error, src) {
     if (error) {
     $img.replaceWith("<span>不能预览</span>");
     }

     $img.attr("src", src);
     });

});

$("#theList").on("click", ".itemUpload", function () {
    uploader.upload();
    //"上传"-->"暂停"
    $(this).hide();
    $(".itemStop").show();
});

$("#theList").on("click", ".itemStop", function () {
    uploader.stop(true);
    //"暂停"-->"上传"
    $(this).hide();
    $(".itemUpload").show();
});

//todo 如果要删除的文件正在上传（包括暂停），则需要发送给后端一个请求用来清除服务器端的缓存文件
$("#theList").on("click", ".itemDel", function () {
    uploader.removeFile($(this).parent().attr("id"));   //从上传文件列表中删除
    $(this).parent().remove();  //从上传列表dom中删除
});

uploader.on("uploadProgress", function (file, percentage) {
    $("#" + file.id + " .percentage").text(percentage * 100 + "%");
});

function UploadComlate(file, data) {
    console.log(file, data);
    if(file && data && data.type == 1){
        $.ajax({
            url: backEndUrl,
            type: "POST",
            dataType: "json",
            cache: false,
            data:{
                act: "upload",
                real_name: file.name,
                tmp_name: data.path,
                code: data.type,
                size: file.size
            },
            success:function (data) {
                if(data.code == 1){
                    /* $("#app_url").val(data.url);
                    $("#app_size").val(data.size); */

                    $("#" + file.id + " .percentage").text("上传完毕").css({'color':'red'});
                    $(".itemStop").hide();
                    $(".itemUpload").hide();
                    $(".itemDel").hide();
                }
                if(data.code == 2){
                    $("#" + file.id + " .percentage").text(data.mes).css({'color':'red'});
                }
            },
            error:function () {
                $("#" + file.id + " .percentage").text('网络错误！').css({'color':'red'});
            }
        });
    }
}
```
`upload.php`
```php
<?php
@define('ROOT', dirname(dirname(__FILE__)));
include_once('./FileUpload.php');
include_once('./Ftp.php');


if(@$_POST['act'] == 'upload'){
    $data = $_POST;
    $arr = uploadRunApk($data, 'upload/' . date("Y") . '/' . date('m') . '/');
    echo json_encode($arr);
    exit;
}

$arr = uploadChunked();
echo json_encode($arr);
exit;

//通过FTP上传到指定位置(这里以上传apk文件为例)
function uploadRunApk($file, $save_path)
{
    global $_SC;
    $max_file_size_in_bytes = 1073741824;                // 1024M in bytes
    //var_dump($file);exit;
    if (!isset($file)) {
        return array('mes' => '找不到文件', 'code' => 2);
    } else if (!isset($file["tmp_name"])) {     //is_uploaded_file()
        return array('mes' => '文件无法上传', 'code' => 2);
    } else if (!isset($file['real_name'])) {
        return array('mes' => '文件名不存在', 'code' => 2);
    }
    //$file_size = @filesize($file["tmp_name"]);
    $file_size = $file['size'];
    if (!$file_size || $file_size > $max_file_size_in_bytes) {
        return array('mes' => '文件size太大', 'code' => 2);
    }
    if ($file_size <= 0) {
        return array('mes' => '文件大小不能为0', 'code' => 2);
    }
    $path_info = pathinfo($file['real_name']);
    $file_extension = $path_info["extension"];

    if($file_extension != 'apk'){
        return array('mes' => '文件类型错误:', 'code' => 2);
    }
    //$fmd5 = md5_file($file["tmp_name"]);
    //$file_name = $save_path . $fmd5 . '.' . $file_extension;
    $file_name = $save_path . $path_info['filename'] . '.' .$file_extension;
    $ftp = new Ftp($ftp_host, $ftp_port, $ftp_user, $ftp_pass);
    if ($ftp->up_file($file["tmp_name"], $file_name)) {
        $show_name = substr($file_name, strrpos($file_name, '/') + 1);
        return array('mes' => '文件上传成功', 'code' => 1, 'url' => $file_name, 'size' => $file_size, 'show_name' => $show_name);
    } else {
        return array('mes' => $file_name, 'code' => 2);
    }
}

function uploadChunked()
{
    //关闭缓存
    header("Expires: Mon, 26 Jul 1997 05:00:00 GMT");
    header("Last-Modified: " . gmdate("D, d M Y H:i:s") . " GMT");
    header("Cache-Control: no-store, no-cache, must-revalidate");
    header("Cache-Control: post-check=0, pre-check=0", false);
    header("Pragma: no-cache");

    global $_SC;
    $uploader = new FileUpload(array('apk', 'jpg', 'png', 'zip'));

    //用于断点续传，验证指定分块是否已经存在，避免重复上传
    if (isset($_POST['status'])) {
        if ($_POST['status'] == 'chunkCheck') {
            $target = $uploader->path . $_POST['name'] . '/' . $_POST['chunkIndex'];
            if (file_exists($target) && filesize($target) == $_POST['size']) {
                return array('ifExist' => 1);
            }
            return array('ifExist' => 0);

        } elseif ($_POST['status'] == 'md5Check') {
            //todo 模拟持久层查询
            $dataArr = array(
                'b0201e4d41b2eeefc7d3d355a44c6f5a' => 'kazaff2.jpg'
            );

            if (isset($dataArr[$_POST['md5']])) {
                return array('ifExist' => 1, 'path' => $dataArr[$_POST['md5']]);
            }
            return array('ifExist' => 0);
        } elseif ($_POST['status'] == 'chunksMerge') {

            if ($path = $uploader->chunksMerge($_POST['name'], $_POST['chunks'], $_POST['ext'])) {
                //todo 把md5签名存入持久层，供未来的秒传验证
                return array('status' => 1, 'path' => $path, 'type' => 1);
            }
            return array('status' => 0, 'chunksMerge' => 0);
        }
    }

    if (($path = $uploader->upload('file', $_POST)) !== false) {
        return array('status' => 1, 'path' => $path, 'type' => 2);
    }
    return array('status' => 0, 'upload' => 0);
}

```

`FileUpload.php`

```php
<?php
/**
 *  PHP通用文件上传类
 *
 *  支持单文件和多文件上传
 */
class FileUpload
{
    //要配置的内容
    public $path;
    private $allowtype;
    private $maxsize;
    private $israndname;

    private $originName;
    private $tmpFileName;
    private $fileType;
    private $fileSize;
    private $newFileName;
    private $errorNum = 0;
    private $errorMess = "";

    private $isChunk = false;
    private $indexOfChunk = 0;

    public function __construct($type)
    {
        $this->path = ROOT . '/upload';
        //$this->allowtype = array('jpg', 'jpeg', 'gif', 'png', 'mp4', 'mp3', 'zip', 'apk', 'pdf', 'rar');
        $this->allowtype = $type;
        $this->maxsize = 1073741824;                // 1024M in bytes
        $this->israndname = true;
    }

    /**
     * 用于设置成员属性($path, $allowtype, $maxsize, $israndname)
     * 可以通过连贯操作一次设置多个属性值
     * @param $key  成员属性（不区分大小写）
     * @param $val  为成员属性设置的值
     * @return object 返回自己对象$this, 可以用于连贯操作
     */
    function set($key, $val)
    {
        $key = strtolower($key);
        if (array_key_exists($key, get_class_vars(get_class($this)))) {
            $this->setOption($key, $val);
        }
        return $this;
    }

    //为单个成员属性设置值
    private function setOption($key, $val)
    {
        $this->$key = $val;
        return $val;
    }

    /** 调用该方法上传文件
     * @param $fileField
     * @param $info
     * @return bool
     * @throws Exception
     */
    function upload($fileField, $info)
    {
        //判断是否为分块上传
        $this->checkChunk($info);

        if (!$this->checkFilePath($this->path))
        {
            //$this->errorMess = $this->getError();
            //return false;
            show_message($this->getError(), '', 2);
        }

        //将文件上传的信息取出赋给变量
        $name = $_FILES[$fileField]['name'];
        $tmp_name = $_FILES[$fileField]['tmp_name'];
        $size = $_FILES[$fileField]['size'];
        $error = $_FILES[$fileField]['error'];

        //设置文件信息
        if ($this->setFiles($name, $tmp_name, $size, $error))
        {
            //如果是分块，则创建一个唯一名称的文件夹用来保存该文件的所有分块
            if ($this->isChunk) {
                $uploadDir = $this->path;
                $tmpName = $this->setDirNameForChunks($info);
                if (!$this->checkFilePath($uploadDir . '/' . $tmpName))
                {
                    //$this->errorMess = $this->getError();
                    //return false;
                    show_message($this->getError(), '', 2);
                }
                //创建一个对应的文件，用来记录上传分块文件的修改时间，用于清理长期未完成的垃圾分块
                touch($uploadDir . '/' . $tmpName . '.tmp');
            }

            if ($this->checkFileSize() && $this->checkFileType()) {
                $this->setNewFileName();
                if ($this->copyFile()) {
                    return $this->path . $this->newFileName;
                }
            }
        }

        //$this->errorMess = $this->getError();
        return false;
    }

    /**
     * @param $uniqueFileName
     * @param $chunksTotal
     * @param $fileExt
     * @return bool
     */
    public function chunksMerge($uniqueFileName, $chunksTotal, $fileExt)
    {
        $targetDir = $this->path . '/' . $uniqueFileName;

        //检查对应文件夹中的分块文件数量是否和总数保持一致
        if ($chunksTotal > 1 && (count(scandir($targetDir)) - 2) == $chunksTotal)
        {
            //同步锁机制
            $lockFd = fopen($this->path . '/' . $uniqueFileName . '.lock', "w");
            if (!flock($lockFd, LOCK_EX | LOCK_NB))
            {
                fclose($lockFd);
                return false;
            }

            //进行合并
            $this->fileType = $fileExt;
            $finalName = $this->path . '/' . ($this->setOption('newFileName', $this->proRandName()));
            $file = fopen($finalName, 'wb');

            for ($index = 0; $index < $chunksTotal; $index++)
            {
                $tmpFile = $targetDir . '/' . $index;
                $chunkFile = fopen($tmpFile, 'rb');
                $content = fread($chunkFile, filesize($tmpFile));
                fclose($chunkFile);
                fwrite($file, $content);

                //删除chunk文件
                unlink($tmpFile);
            }

            fclose($file);

            //删除chunk文件夹
            rmdir($targetDir);
            unlink($this->path . '/' . $uniqueFileName . '.tmp');

            //解锁
            flock($lockFd, LOCK_UN);
            fclose($lockFd);
            unlink($this->path . '/' . $uniqueFileName . '.lock');

            return $this->path . '/' . $this->newFileName;
        }
        return false;
    }

    //获取上传后的文件名称
    public function getFileName()
    {
        return $this->newFileName;
    }

    //上传失败后，调用该方法则返回，上传出错信息
    /*public function getErrorMsg()
    {
        return $this->errorMess;
    }*/

    //设置上传出错信息
    public function getError()
    {
        $str = "上传文件<span color='red'>{$this->originName}</span>时出错：";
        switch ($this->errorNum) {
            case 4:
                $str .= "没有文件被上传";
                break;
            case 3:
                $str .= "文件只有部分被上传";
                break;
            case 2:
                $str .= "上传文件的大小超过了HTML表单中MAX_FILE_SIZE选项指定的值";
                break;
            case 1:
                $str .= "上传的文件超过了php.ini中upload_max_filesize选项限制的值";
                break;
            case -1:
                $str .= "未允许的类型";
                break;
            case -2:
                $str .= "文件过大， 上传的文件夹不能超过{$this->maxsize}个字节";
                break;
            case -3:
                $str .= "上传失败";
                break;
            case -4:
                $str .= "建立存放上传文件目录失败，请重新指定上传目录";
                break;
            case -5:
                $str .= "必须指定上传文件的路径";
                break;

            default:
                $str .= "未知错误";
        }
        return $str . "<br>";
    }

    //根据文件的相关信息为分块数据创建文件夹
    //md5(当前登录用户的数据库id + 文件原始名称 + 文件类型 + 文件最后修改时间 + 文件总大小)
    private function setDirNameForChunks($info)
    {
        $str = '' . $info['userId'] . $info['name'] . $info['type'] . $info['lastModifiedDate'] . $info['size'];
        return md5($str);
    }

    //设置和$_FILES有关的内容
    private function setFiles($name = "", $tmp_name = "", $size = 0, $error = 0)
    {
        $this->setOption('errorNum', $error);
        if ($error) {
            //return false;
            show_message($this->getError(), '', 2);
        }
        $this->setOption('originName', $name);
        $this->setOption('tmpFileName', $tmp_name);
        $aryStr = explode(".", $name);
        $this->setOption("fileType", strtolower($aryStr[count($aryStr) - 1]));
        $this->setOption("fileSize", $size);
        return true;
    }

    private function checkChunk($info)
    {
        if (isset($info['chunks']) && $info['chunks'] > 0) {
            $this->setOption("isChunk", true);

            if (isset($info['chunk']) && $info['chunk'] >= 0) {
                $this->setOption("indexOfChunk", $info['chunk']);

                return true;
            }

            throw new Exception('分块索引不合法');
        }

        return false;
    }

    //设置上传后的文件名称
    private function setNewFileName()
    {
        if ($this->isChunk) {        //如果是分块，则以分块的索引作为文件名称保存
            $this->setOption('newFileName', $this->indexOfChunk);
        } elseif ($this->israndname) {
            $this->setOption('newFileName', $this->proRandName());
        } else {
            $this->setOption('newFileName', $this->originName);
        }
    }

    //检查上传的文件是否是合法的类型
    private function checkFileType()
    {
        if (in_array(strtolower($this->fileType), $this->allowtype)) {
            return true;
        } else {
            $this->setOption('errorNum', -1);
            //return false;
            show_message($this->getError(), '', 2);
        }
    }

    //检查上传的文件是否是允许的大小
    private function checkFileSize()
    {
        if ($this->fileSize > $this->maxsize) {
            $this->setOption('errorNum', -5);
            //return false;
            show_message($this->getError(), '', 2);
        } else {
            return true;
        }
    }

    //检查是否有存放上传文件的目录
    private function checkFilePath($target)
    {
        if (empty($target)) {
            $this->setOption('errorNum', -5);
            //return false;
            show_message($this->getError(), '', 2);
        }

        if (!file_exists($target) || !is_writable($target)) {
            if (!@mkdir($target, 0755)) {
                $this->setOption('errorNum', -4);
                //return false;
                show_message($this->getError(), '', 2);
            }
        }

        $this->path = $target;
        return true;
    }

    //设置随机文件名
    private function proRandName()
    {
        $fileName = date('YmdHis') . "_" . rand(100, 999);
        return $fileName . '.' . $this->fileType;
    }

    //复制上传文件到指定的位置
    private function copyFile()
    {
        if (!$this->errorNum) {
            $path = rtrim($this->path, '/') . '/';
            $path .= $this->newFileName;
            if (@move_uploaded_file($this->tmpFileName, $path)) {
                return true;
            } else {
                $this->setOption('errorNum', -3);
                //return false;
                show_message($this->getError(), '', 2);
            }
        } else {
            //return false;
            show_message($this->getError(), '', 2);
        }
    }
}

/**
 * 实例
 */
/**
//关闭缓存
header("Expires: Mon, 26 Jul 1997 05:00:00 GMT");
header("Last-Modified: " . gmdate("D, d M Y H:i:s") . " GMT");
header("Cache-Control: no-store, no-cache, must-revalidate");
header("Cache-Control: post-check=0, pre-check=0", false);
header("Pragma: no-cache");

$uploader = new FileUpload();

//用于断点续传，验证指定分块是否已经存在，避免重复上传
if (isset($_POST['status'])) {
    if ($_POST['status'] == 'chunkCheck') {
        $target = '../uploads/' . $_POST['name'] . '/' . $_POST['chunkIndex'];
        if (file_exists($target) && filesize($target) == $_POST['size']) {
            die('{"ifExist":1}');
        }
        die('{"ifExist":0}');

    } elseif ($_POST['status'] == 'md5Check') {

        //todo 模拟持久层查询
        $dataArr = array(
            'b0201e4d41b2eeefc7d3d355a44c6f5a' => 'kazaff2.jpg'
        );

        if (isset($dataArr[$_POST['md5']])) {
            die('{"ifExist":1, "path":"' . $dataArr[$_POST['md5']] . '"}');
        }
        die('{"ifExist":0}');
    } elseif ($_POST['status'] == 'chunksMerge') {

        if ($path = $uploader->chunksMerge($_POST['name'], $_POST['chunks'], $_POST['ext'])) {
            //todo 把md5签名存入持久层，供未来的秒传验证
            die('{"status":1, "path": "' . $path . '"}');
        }
        die('{"status":0');
    }
}

if (($path = $uploader->upload('file', $_POST)) !== false) {
    die('{"status":1, "path": "' . $path . '"}');
}
die('{"status":0}');
 */
```

# 最后
发现自己语言组织能力好差啊，各位看官还是转到[聊聊大文件上传](http://blog.kazaff.me/2014/11/14/聊聊大文件上传/)去欣赏吧
