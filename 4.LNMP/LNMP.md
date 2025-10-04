# 网站程序部署

## 一、完成度与前文

| 程序      | 访问地址                                     | 数据库名称 | 使用 PHP 版本 | 状态   |
| --------- | -------------------------------------------- | ---------- | ------------- | ------ |
| Discuz    | [dz.mxdx.com](http://dz.mxdx.com/)           | discuz     | PHP 8.2       | 已完成 |
| DedeCMS   | [dedecms.mxdx.com](http://dedecms.mxdx.com/) | dedecms    | PHP 7.4       | 已完成 |
| MediaWiki | [wiki.mxdx.com](http://wiki.mxdx.com/)       | debianwiki | PHP 8.2       | 已完成 |
| WordPress | [blog.mxdx.com](http://blog.mxdx.com/)       | wordpress  | PHP 8.2       | 已完成 |

+ 运行在LNMP环境
+ 通过DNS指向web服务地址
+ 使用二进制安装包安装的mysql
+ 由于虚拟机里下载压缩包有点慢，因此本文采取主机下载然后ssh上传的方式

## 二、各站点的安装与配置

+ 权限统一设置为

  ```
  chown -R www-data:www-data /var/www/目录名
  ```

### 1. Discuz

1. 下载并移动

   ```sh
   git clone https://gitee.com/Discuz/DiscuzX.git
   
   // 下载一些插件
   apt install php-fpm php-mysql
   apt install php8.2-xml
   // 将其移动到/var/www/dz.mxdx.com
   // 赋权
   chown -R www-data:www-data /var/www/dz.mxdx.com
   ```

2. 数据库配置

   ```sh
   CREATE DATABASE discuz;
   CREATE USER 'discuz'@'localhost' IDENTIFIED BY '123456';
   GRANT ALL PRIVILEGES ON discuz.* TO 'discuz'@'localhost';
   FLUSH PRIVILEGES;
   ```

3. nginx配置

   ```sh
   root@debian:/etc/nginx/conf.d# cat dz.mcdx.com.conf 
   server {
       listen 80;
       server_name dz.mxdx.com;
   
       root /var/www/dz.mxdx.com;
       index index.php index.html index.htm;
   
       location / {
           try_files $uri $uri/ /index.php?$args;
       }
   
       location ~ \.php$ {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/run/php/php8.2-fpm.sock;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           include fastcgi_params;
       }
   
       location ~ /\.ht {
           deny all;
       }
   
       error_log /var/log/nginx/dz_error.log;
       access_log /var/log/nginx/dz_access.log;
   }
   ```

4. 权限问题

   ```
   chown -R www-data:www-data /var/www/dz.mxdx.com
   chmod -R 777 /var/www/dz.mxdx.com
   ```

   ![image-20250717082541027](https://img.lhjeong.cn/20250717082541166.png)

### 2. DedeCms

+ 前置（PHP 7.4）

  ```sh
  // dedecms不兼容8.2，因此我们使用7.4
  // 1. 添加源
  	apt install -y lsb-release ca-certificates apt-transport-https software-properties-common
  	wget -qO /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
  	sh -c 'echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
  // 2. 安装本体及扩展
  	apt install -y php7.4 php7.4-fpm php7.4-mysql php7.4-xml php7.4-mbstring php7.4-gd php7.4-curl php7.4-zip
  ```

1. 下载

   ```sh
   从官网下载后，上传到/var/www/dedecms.mxdx.com
   
   // 赋权
   chown -R www-data:www-data /var/www/dedecms.mxdx.com
   chmod -R 777 /var/www/dedecms.mxdx.com
   ```

2. 数据库配置

   ```sh
   CREATE DATABASE dedecms;
   CREATE USER 'dedecms'@'localhost' IDENTIFIED BY '123456';
   GRANT ALL PRIVILEGES ON dedecms.* TO 'dedecms'@'localhost';
   FLUSH PRIVILEGES;
   ```

3. nginx配置

   ```sh
   root@debian:/etc/nginx/conf.d# cat dedecms.mxdx.com.conf 
   server {
       listen 80;
       server_name dedecms.mxdx.com;
   
       root /var/www/dedecms.mxdx.com;
       index index.php index.html;
   
       location / {
               try_files $uri $uri/ /index.php?$args;
       }
   
       location ~ \.php$ {
           include fastcgi_params;
           fastcgi_pass unix:/run/php/php7.4-fpm.sock;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
       }
   
       location ~ /\.ht {
           deny all;
       }
   }0
   ```
   
   ![image-20250717104238806](https://img.lhjeong.cn/20250717104238953.png)

### 3. MediaWiki

1. 下载

   ```sh
   从官网下载后，在主机上解压，上传到/var/www/wiki.mxdx.com
   
   // 赋权
   chown -R www-data:www-data /var/www/wiki.mxdx.com
   chmod -R 777 /var/www/dedecms.mxdx.com
   ```

2. 数据库配置

   ```sh
   CREATE DATABASE debianwiki;
   CREATE USER 'wiki'@'localhost' IDENTIFIED BY '123456';
   GRANT ALL PRIVILEGES ON debianwiki.* TO 'wiki'@'localhost';
   FLUSH PRIVILEGES;
   ```

3. nginx配置

   ```sh
   root@debian:/etc/nginx/conf.d# cat wiki.mxdx.com.conf 
   server {
       listen 80;
       server_name wiki.mxdx.com;
   
       root /var/www/wiki.mxdx.com;
       index index.php index.html;
   
       location / {
           try_files $uri $uri/ @rewrite;
       }
   
       location @rewrite {
           rewrite ^/(.*)$ /index.php?title=&1&$args;
       }
   
       location ~ \.php$ {
           include fastcgi_params;
           fastcgi_pass unix:/run/php/php8.2-fpm.sock;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
       }
   
       location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
           expires max;
           log_not_found off;
       }
   }
   ```
   
   ![image-20250717090309744](https://img.lhjeong.cn/20250717090309884.png)

### 4. WordPress

1. 下载

   ```
   下载解压后，上传到/var/www/blog.mxdx.com
   
   // 赋权
   chown -R www-data:www-data /var/www/blog.mxdx.com
   ```

2. 数据库配置

   ```sh
   CREATE DATABASE wordpress;
   CREATE USER 'wp'@'localhost' IDENTIFIED BY '123456';
   GRANT ALL PRIVILEGES ON wordpress.* TO 'wp'@'localhost';
   FLUSH PRIVILEGES;
   ```

3. nginx配置

   ```sh
   root@debian:/etc/nginx/conf.d# cat blog.mxdx.com.conf 
   server {
       listen 80;
       server_name blog.mxdx.com;
   
       root /var/www/blog.mxdx.com;
       index index.php index.html index.htm;
   
       location / {
           try_files  $uri $uri/ /index.php?$args;
       }
   
       location ~ \.php$ {
           include fastcgi_params;
           fastcgi_pass unix:/run/php/php8.2-fpm.sock; 
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           include fastcgi.conf;
       }
   
       location ~ /\.ht {
           deny all;
       }
   }
   ```

   ![image-20250717090336932](https://img.lhjeong.cn/20250717090337017.png)

## 三、使用FTP管理站点内容

### 1. 在DNS服务器上

1. 安装nfs服务

   ````sh
   sudo apt update
   sudo apt install nfs-kernel-server
   ````

2. 配置nfs共享

   ```sh
   vim /etc/exports
   添加
   /srv/ftp_uploads 192.168.231.2(rw,sync,no_subtree_check)
   ```

3. 重新加载nfs服务

   ```sh
   sudo exportfs -a
   sudo systemctl restart nfs-kernel-server
   ```

### 2. 在FTP服务器上

1. 安装nfs客户端

   ```sh
   sudo apt update
   sudo apt install nfs-common
   ```

2. 创建挂载点并挂载共享目录

   ```sh
   mkdir -p /mnt/ftp_uploads
   mount 192.168.231.1:/srv/ftp_uploads /mnt/ftp_uploads
   ```

![image-20250717091634834](https://img.lhjeong.cn/20250717091634912.png)

## 四、邮件功能

### 1、discuz

+ 要在管理后台开启会员和邮件功能

![image-20250717105922932](https://img.lhjeong.cn/20250717105923027.png)

### 2、dedecms

+ 本文在大量试验中选择了重写的方法，故不建议使用该方法

1. 下载插件并移动

   ```
   git clone https://github.com/PHPMailer/PHPMailer.git;
   // 移动
   mv PHPMailer/ /var/www/html/dedecms/include/
   ```

2. 修改index_do.php

   ```sh
   <?php
   /**
    * @version        $Id: index_do.php 1 8:24 2010年7月9日 $
    * @package        DedeCMS.Member
    * @founder        IT柏拉图, https://weibo.com/itprato
    * @author         DedeCMS团队
    * @copyright      Copyright (c) 2004 - 2024, 上海卓卓网络科技有限公司 (DesDev, Inc.)
    * @license        http://help.dedecms.com/usersguide/license.html
    * @link           http://www.dedecms.com
    */
   define('DEDEINC', dirname(__FILE__) . '/../include');
   require_once(DEDEINC.'/mail.class.php');
   require_once(dirname(__FILE__)."/config.php");
   if(empty($dopost)) $dopost = '';
   if(empty($fmdo)) $fmdo = '';
   
   /*********************
   function check_email()
   *******************/
   if($fmdo=='sendMail')
   {
       require_once(DEDEINC.'/mail.class.php');
   	// 测试代码
   	file_put_contents('/tmp/phpmailer_check.log', date('Y-m-d H:i:s')." | mail.class.php 加载完毕\n", FILE_APPEND);
   file_put_contents('/tmp/dedeinc_path.log', DEDEINC);
   if (!function_exists('send_via_phpmailer')) {
       file_put_contents('/tmp/phpmailer_check.log', date('Y-m-d H:i:s')." | send_via_phpmailer 未定义！\n", FILE_APPEND);
   } else {
       file_put_contents('/tmp/phpmailer_check.log', date('Y-m-d H:i:s')." | send_via_phpmailer 函数已找到\n", FILE_APPEND);
   }
   file_put_contents('/tmp/mail_debug.log', "DEDEINC=".DEDEINC."\n", FILE_APPEND);
   require_once(DEDEINC.'/mail.class.php');
   file_put_contents('/tmp/mail_debug.log', "mail.class.php included\n", FILE_APPEND);
   
   if (!function_exists('send_via_phpmailer')) {
       file_put_contents('/tmp/mail_debug.log', "send_via_phpmailer() not found\n", FILE_APPEND);
   } else {
       file_put_contents('/tmp/mail_debug.log', "send_via_phpmailer() found!\n", FILE_APPEND);
   }
       
       if(!CheckEmail($cfg_ml->fields['email']) )
       {
           ShowMsg('你的邮箱格式有错误！', '-1');
           exit();
       }
       if($cfg_ml->fields['spacesta'] != -10)
       {
           ShowMsg('你的帐号不在邮件验证状态，本操作无效！', '-1');
           exit();
       }
       $userhash = md5($cfg_cookie_encode.'--'.$cfg_ml->fields['mid'].'--'.$cfg_ml->fields['email']);
       $url = $cfg_basehost.(empty($cfg_cmspath) ? '/' : $cfg_cmspath)."/member/index_do.php?fmdo=checkMail&mid={$cfg_ml->fields['mid']}&userhash={$userhash}&do=1";
       $url = preg_replace("#http:\/\/#i", '', $url);
       $url = 'http://'.preg_replace("#\/\/#", '/', $url);
       $mailtitle = "{$cfg_webname}--会员邮件验证通知";
       $mailbody = '';
       $mailbody .= "尊敬的用户[{$cfg_ml->fields['uname']}]，您好：\r\n";
       $mailbody .= "欢迎注册成为[{$cfg_webname}]的会员。\r\n";
       $mailbody .= "要通过注册，还必须进行最后一步操作，请点击或复制下面链接到地址栏访问这地址：\r\n\r\n";
       $mailbody .= "{$url}\r\n\r\n";
       $mailbody .= "Power by http://www.dedecms.com 织梦内容管理系统！\r\n";
   
       $fromname = $cfg_webname;
       $fromemail = $cfg_smtp_usermail;
       $mailtype = 'TXT';
   
       require_once(DEDEINC.'/mail.class.php'); // 使用 PHPMailer 封装文件
   
       // 在 mail.class.php 中定义的函数
       if(!send_via_phpmailer($cfg_ml->fields['email'], $mailtitle, $mailbody, $fromname, $fromemail, $mailtype))
       {
           // 可选：使用 PHP mail() 作兜底
           $headers = "From: ".$fromemail."\r\nReply-To: ".$fromemail;
           @mail($cfg_ml->fields['email'], $mailtitle, $mailbody, $headers);
       }
   
       ShowMsg('成功发送邮件，请稍后登录你的邮箱进行接收！', '/member');
       exit();
   }
   ```

3. 修改mail.class.php

   ```sh
   <?php
    file_put_contents('/tmp/mail_marker.log', __FILE__ . " | loaded\n", FILE_APPEND);
   require_once(dirname(__FILE__).'/PHPMailer/src/Exception.php');
   require_once(dirname(__FILE__).'/PHPMailer/src/PHPMailer.php');
   require_once(dirname(__FILE__).'/PHPMailer/src/SMTP.php');
   
   use PHPMailer\PHPMailer\PHPMailer;
   use PHPMailer\PHPMailer\Exception;
   
   class smtp
   {
       var $smtp_port;
       var $time_out;
       var $host_name;
       var $log_file;
       var $relay_host;
       var $debug;
       var $auth;
       var $user;
       var $pass;
       var $sock;
   
       function smtp($relay_host = "", $smtp_port = 25, $auth = FALSE, $user, $pass)
       {
           $this->debug = FALSE;
           $this->smtp_port = $smtp_port;
           $this->relay_host = $relay_host;
           $this->time_out = 30;
           $this->auth = $auth;
           $this->user = $user;
           $this->pass = $pass;
           $this->host_name = "localhost";
           $this->log_file = "";
           $this->sock = FALSE;
       }
   }
   
   /**
    * 定义的全局邮件发送函数，供 index_do.php 使用。
    */
   function send_via_phpmailer($email, $mailtitle, $mailbody, $fromname, $fromemail, $mailtype)
   {
       file_put_contents('/tmp/mail_marker.log', __FILE__ . " | loaded\n", FILE_APPEND);
       global $cfg_sendmail_bysmtp, $cfg_smtp_server, $cfg_smtp_port,
              $cfg_smtp_usermail, $cfg_smtp_password, $cfg_webname;
   
       try {
           $mail = new PHPMailer(true);
   
           $mail->SMTPOptions = [
               'ssl' => [
                   'verify_peer'       => false,
                   'verify_peer_name'  => false,
                   'allow_self_signed' => true,
               ],
           ];
   
           if ($cfg_sendmail_bysmtp === 'Y') {
               $mail->isSMTP();
               $mail->Host       = $cfg_smtp_server;
               $mail->SMTPAuth   = false;
               $mail->Username   = $cfg_smtp_usermail;
               $mail->Password   = $cfg_smtp_password;
               $mail->Port       = $cfg_smtp_port;
           }
   
           $mail->setFrom($fromemail, $fromname);
           $mail->addAddress($email);
   
           $mail->isHTML(false);
           $mail->CharSet = 'UTF-8';
           $mail->Subject = $mailtitle;
           $mail->Body    = $mailbody;
   
           return $mail->send();
   
       } catch (Exception $e) {
           file_put_contents('/tmp/mail_error.log', date('Y-m-d H:i:s')." | ".$e->getMessage()."\n", FILE_APPEND);
           return false;
       }
   }
   ```

   ![image-20250717105955623](https://img.lhjeong.cn/20250717105955722.png)

### 3、wiki

1. 权限设置

   ```sh
   # 禁止所有用户编辑。
   $wgGroupPermissions['*']['edit'] = false;
   # 禁止注册用户编辑（默认情况下'user'用户组将被允许编辑，即便'*'被设置为禁止）。
   $wgGroupPermissions['user']['edit'] = false;
   # # 将已确认电子邮箱的用户加入用户组中。
   $wgAutopromote['emailconfirmed'] = APCOND_EMAILCONFIRMED;
   # 在用户列表中隐藏该组。
   $wgImplicitGroups[] = 'emailconfirmed';
   # 最后，赋予该用户组编辑权限。
   $wgGroupPermissions['emailconfirmed']['edit'] = true;
   ```

2. 加入smtp

   ```sh
   $wgSMTP = [
           'host' => 'mail.mxdx.com',   // SMTP服务器地址
           'IDHost' => 'mxdx.com',      // 用于识别主机名
           'port' => 25,                // 端口号
           'username' => 'wiki',        // SMTP用户名
           'password' => '123456',      // SMTP密码
           'auth' => false,             // 是否需要身份验证
           'verify_peer' => false
   ];
   ```

   ![image-20250717110127500](https://img.lhjeong.cn/20250717110127613.png)

### 4、blog

+ 下载插件

![image-20250717110238389](https://img.lhjeong.cn/20250717110238496.png)

## 五、数据库备份

### 1. 命令行备份

```sh
mysqldump -u su -p discuz > /data/db_backups/manual/db_discuz_$(date +%F).sql

mysqldump -u su -p dedecms > /data/db_backups/manual/db_dedecms_$(date +%F).sql

mysqldump -u su -p debianwiki > /data/db_backups/manual/db_wiki$(date +%F).sql

mysqldump -u su -p blogdb > /data/db_backups/manual/db_blogdb$(date +%F).sql 
```

### 2. 自动化备份

```sh
#!/bin/bash
# 当前日期
DATE=$(date +%F)

# 备份目录
BACKUP_DIR="/data/db_back/$DATE"
mkdir -p "$BACKUP_DIR"

# 数据库登录信息
MYSQL_USER="su"
MYSQL_PASS="123456"

# 要备份的数据库列表
DB_LIST=("discuz" "dedecms" "debianwiki" "blogdb")

echo "开始数据库备份..."

for db in "${DB_LIST[@]}"; do
    echo "备份..$db"
    mysqldump -u "$MYSQL_USER" -p"$MYSQL_PASS" "$db" > "$BACKUP_DIR/${db}.sql"
done

# 删除 7 天前的旧备份
echo "清理 7 天前的备份..."
find /data/db_back/ -maxdepth 1 -type d -mtime +7 -exec rm -rf {} \;

echo "所有数据库已备份至：$BACKUP_DIR"
```

![image-20250716092225971](https://img.lhjeong.cn/20250716092226104.png)

