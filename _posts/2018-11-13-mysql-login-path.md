---
layout: post
title: MySQL Login path 설정하기
author: 하얀눈길
description: null
tags: 'mysql, login, path'
featuredImage: null
img: null
categories: 'mysql'
date: '2018-11-13'
extensions:
  preset: gfm

---


> mysql 5.6부터 보안 문제로 인해 패스워드를 커맨드라인에서 직접 입력하기 어려워졌습니다.
> 대신에 login path라는 것을 사용하면 되는데요. 커맨드를 자주 잊어버리네요. 
> "Warning: using a password on the command line interface can be insecure. " 이런 메세지가 출력될 때 사용하면 되겠네요.
> 간단한 사용법을 정리해 봅니다.


# 1. login path help

    $ mysql_config_editor --help
    mysql_config_editor Ver 1.0 Distrib 5.7.18, for linux-glibc2.5 on x86_64
    Copyright (c) 2012, 2017, Oracle and/or its affiliates. All rights reserved.
    
    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.
    
    MySQL Configuration Utility.
    Usage: mysql_config_editor [program options] [command [command options]]
      -#, --debug[=#]     This is a non-debug version. Catch this and exit.
      -?, --help          Display this help and exit.
      -v, --verbose       Write more information.
      -V, --version       Output version information and exit.
    Variables (--variable-name=value)
    and boolean options {FALSE|TRUE}  Value (after reading options)
    --------------------------------- ----------------------------------------
    verbose                           FALSE
    
    Where command can be any one of the following :
       set [command options]     Sets user name/password/host name/socket/port
                                 for a given login path (section).
       remove [command options]  Remove a login path from the login file.
       print [command options]   Print all the options for a specified
                                 login path.
       reset [command options]   Deletes the contents of the login file.
       help                      Display this usage/help information.


흠.. 간단하네요..


# 2. generate login path 


    $ mysql_config_editor set --login-path=설정이름 --host=주소 --user=아이디 --port=포트 --password
    Enter password: *****
     
  
    
# 3. print login path

    $ mysql_config_editor print --login-path=설정이름
    [myroot]
    user = root
    password = *****
    host = localhost
    port = 3306

  설정된 전체 목록을 출력하려면 다음과 같이 합니다.

    mysql_config_editor print --all



# 4. remove login path

    mysql_config_editor remove --login-path=설정이름


# 5. use login path

    $ mysql --login-path=설정이름

흠.. 편하네요. 스크립트로 자동화할 때 훨씬 편하게 세팅할 수 있습니다.

# 6. location of config file
윈도우인 경우는 `%APPDATA%\MySQL` 디렉토리에 `.mylogin.cnf` 파일이 생성됩니다. unix계열은 로그인 계정의 홈 디렉토리에  파일이 생성됩니다. 
이 파일은 암호화 되어 있는 것으로 보이네요. 당연하겠지요
 
 
# Troubleshooting 
*  "ERROR 1045 (28000): Access denied for user" 에러가 발생하는 경우
	* 당황하지 말고, 패스워드 입력할 때, `"`를 앞뒤로 붙여서 입력합니다. 특수기호(#,$,! 등)이 들어가 있으면 발생할 수 있습니다.

# 참조

 - https://dev.mysql.com/doc/refman/8.0/en/mysql-config-editor.html


