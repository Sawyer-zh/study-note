# 通过PHP的邮件函数发送邮件是出现的问题

PHP环境下，是提供了发送邮件的函数mail()的，不过该函数要求服务器支持sendmail或者必须设置一台不需要中继的邮件发送服务器，但现在要找到一台不需要身份验证的邮件发送中继几乎不可能，所以使用mail函数往往无法成功发送电子邮件



什么是域名邮箱



1、Warning: mail() [function.mail]: Failed to connect to mailserver at "localhost" port 25, verify your "SMTP" and "smtp_port" setting in php.ini or use ini_set()



2、SMTP server response:530 5.7.0 Must issue a STARTTLS command first

​	首先确保PHP安装有SSL支持（通过在php脚本执行phpinfo()函数，然后在浏览器查看“openssl”选项）

