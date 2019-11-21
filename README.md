# php fastcgi_finish_request
记录这个好用的方法
php 利用fastcig 来实现异步执行操作
比如系统记录一些业务日志，系统邮件，第三方接口等 不用利用redis 直接可以异步执行不影响返回
以下是日志手机的方法
<?php

namespace Ku;
/**日志记录
 * Class Logger
 * @package Ku
 */
class Logger
{

    public static function write($fileName, $data, $path = false)
    {
        register_shutdown_function(['\Ku\Logger', 'asyncWrite'], $fileName, $data, $path);
    }

    public static function asyncWrite($fileName, $data, $path)
    {
        if (function_exists('fastcgi_finish_request')) {
            fastcgi_finish_request();
        }
        $logPath = APPLICATION_PATH . '/logs/';
        $folder = $path == false ? $logPath . date("Y-m-d") . '/' : $logPath . $path . '/' . date("Y-m-d") . '/';
        self::checkFolderExists($folder);
        if (!chmod($folder . $fileName . '.log', 0777)) {
            error_log(date('Y-m-d H:i:s') . $folder . $fileName . '.log', 3, '/tmp/xf_' . date('Y-m-d') . '.log');
        }
        error_log(json_encode($data) . '     date:' . date("Y-m-d H:i:s") . "   \r\n", 3, $folder . $fileName . '.log');
    }

    /**
     * 判断文件夹是否存在，如果不存在则创建
     * @param string $folder
     */
    private static function checkFolderExists($folder)
    {
        if (!is_dir($folder)) {
            mkdir($folder, 0777, true);
        }
    }
}
