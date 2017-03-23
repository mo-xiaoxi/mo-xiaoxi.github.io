---
title: 代码审计之普通命令注入
time: 2016.06.03 10:11:00
layout: post
catalog: true
tags:
- Security
- Web
excerpt: 基于DVWA学习SQL注入
    


---

# Command injection

## low

### 目标
执行其它命令

### 源码
	<?php 

	if( isset( $_POST[ 'Submit' ]  ) ) { 
		// Get input 
		$target = $_REQUEST[ 'ip' ]; 
	
		// Determine OS and execute the ping command. 
		if( stristr( php_uname( 's' ), 'Windows NT' ) ) { 
	    	// Windows 
	    	$cmd = shell_exec( 'ping  ' . $target ); 
		} 
		else { 
	    	// *nix 
	    	$cmd = shell_exec( 'ping  -c 4 ' . $target ); 
		} 
	
		// Feedback for the end user 
		echo "<pre>{$cmd}</pre>"; 
	} 
	
	?> 


### 过程

-  这里只需要输入&& 隔开即可或者；

   ![image](https://momomoxiaoxi.com/img/post/DVWA/9.png)


## Medium

### 源码


	<?php 

	if( isset( $_POST[ 'Submit' ]  ) ) { 
		// Get input 
		$target = $_REQUEST[ 'ip' ]; 
	
		// Set blacklist 
		$substitutions = array( 
	    	'&&' => '', 
	    	';'  => '', 
		); 
	
		// Remove any of the charactars in the array (blacklist). 
		$target = str_replace( array_keys( $substitutions ), $substitutions, $target );
	
		// Determine OS and execute the ping command. 
		if( stristr( php_uname( 's' ), 'Windows NT' ) ) { 
	    	// Windows 
	    	$cmd = shell_exec( 'ping  ' . $target ); 
		} 
		else { 
	    	// *nix 
	    	$cmd = shell_exec( 'ping  -c 4 ' . $target ); 
		} 
	
		// Feedback for the end user 
		echo "<pre>{$cmd}</pre>"; 
	} 
	
	?> 

### 	过程

- 这里过滤了&&，；  但是，我们还有一种方式绕过 通过 ｜ 或者｜｜

![image](momomoxiaoxi.com/img/post/DVWA/10.png)

## High

### 源码


	<?php 

	if( isset( $_POST[ 'Submit' ]  ) ) { 
		// Get input 
		$target = trim($_REQUEST[ 'ip' ]); 
	
		// Set blacklist 
		$substitutions = array( 
	    	'&'  => '', 
	    	';'  => '', 
	    	'| ' => '', 
	    	'-'  => '', 
	    	'$'  => '', 
	    	'('  => '', 
	    	')'  => '', 
	    	'`'  => '', 
	    	'||' => '', 
		); 
	
		// Remove any of the charactars in the array (blacklist). 
		$target = str_replace( array_keys( $substitutions ), 	$substitutions, $target ); 
	
		// Determine OS and execute the ping command. 
		if( stristr( php_uname( 's' ), 'Windows NT' ) ) { 
	    	// Windows 
	    	$cmd = shell_exec( 'ping  ' . $target ); 
		} 
		else { 
	    	// *nix 
	    	$cmd = shell_exec( 'ping  -c 4 ' . $target ); 
		} 
	
		// Feedback for the end user 
		echo "<pre>{$cmd}</pre>"; 
	} 
	
	?> 


### 过程

这里其实多了一些过滤条件吧。
暂时不会😂

