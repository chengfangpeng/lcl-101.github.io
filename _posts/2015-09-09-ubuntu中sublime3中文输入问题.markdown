---
layout:     post
title:      "Ubuntu中sublime3中问输入问题"
subtitle:   " \"Ubuntu中sublime3中问输入问题\""
date:       2015-09-09 12:00:00
author:     "X-ray"
header-img: "img/post-bg-2015.jpg"
tags:
    - Ubuntu
---

## 前言 ##

Sublime Text３是一个跨平台的编辑器，但是在ubuntu上安装以后发现不能输入中文。google了一下记录一下解决步骤（注：我的系统是Ubuntu14.04）。

 

 - **创建sublime_imfix.c文件**
		 在HOME目录（或者其他目录）下创建sublime_imfix.c文件，文件内容如下：

	

    
```
#include <gtk/gtkimcontext.h>
 
void gtk_im_context_set_client_window (GtkIMContext *context,
         GdkWindow    *window)
{
 GtkIMContextClass *klass;
 g_return_if_fail (GTK_IS_IM_CONTEXT (context));
 klass = GTK_IM_CONTEXT_GET_CLASS (context);
 if (klass->set_client_window)
     klass->set_client_window (context, window);
 g_object_set_data(G_OBJECT(context),"window",window);
 
 if(!GDK_IS_WINDOW (window))
     return;
 int width = gdk_window_get_width(window);
 int height = gdk_window_get_height(window);
 if(width != 0 && height !=0)
     gtk_im_context_focus_in(context);
}
```

 - **编译文件**
	  编译上面创建的sublime_imfix.c文件，命令如下：

	

  ```
    # 拷贝文件
    sudo cp libsublime-imfix.so /opt/sublime_text/
   ```
    修改文件sudo vim /usr/bin/subl，将文件内容替换成下面的代码：

	```
		#!/bin/sh
	LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text "$@"
	```
	修改之后，通过命令行执行subl启动sublime text，已经可以输入中文了。为了使用鼠标右键打开文件时能够使用中文输入，还需要修改文件sublime_text.desktop的内容。使用下面的内容替换
	```
		[Desktop Entry]
	Version=1.0
	Type=Application
	Name=Sublime Text
	GenericName=Text Editor
	Comment=Sophisticated text editor for code, markup and prose
	Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text %F"
	Terminal=false
	MimeType=text/plain;
	Icon=sublime-text
	Categories=TextEditor;Development;
	StartupNotify=true
	Actions=Window;Document;
	 
	[Desktop Action Window]
	Name=New Window
	Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text -n"
	OnlyShowIn=Unity;
	 
	[Desktop Action Document]
	Name=New File
	Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text --command new_file"
	OnlyShowIn=Unity;
	```
现在就可以了，但是有事后会有失灵的问题，找了半天也没找到解决方法，但是如果在终端中用命令：

	```
    $ subl filename
    ```
    打开是没问题的。