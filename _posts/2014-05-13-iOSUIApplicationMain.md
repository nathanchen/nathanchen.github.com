---
layout: post
title: "iOS UIApplicationMain"
category: Reading Notes
tags: ["读文章", "iOS"]
---
{% include JB/setup %}

1、`- (void)applicationWillResignActive:(UIApplication *)application`

说明：当应用程序将要入非活动状态执行，在此期间，应用程序不接收消息或事件，比如来电话了

2、`- (void)applicationDidBecomeActive:(UIApplication *)application`

说明：当应用程序入活动状态执行，这个刚好跟上面那个方法相反

3、`- (void)applicationDidEnterBackground:(UIApplication *)application`

说明：当程序被推送到后台的时候调用。所以要设置后台继续运行，则在这个函数里面设置即可

4、`- (void)applicationWillEnterForeground:(UIApplication *)application`

说明：当程序从后台将要重新回到前台时候调用，这个刚好跟上面的那个方法相反。

5、`- (void)applicationWillTerminate:(UIApplication *)application`

说明：当程序将要退出是被调用，通常是用来保存数据和一些退出前的清理工作。这个需要要设置UIApplicationExitsOnSuspend的键值。

6、`- (void)applicationDidReceiveMemoryWarning:(UIApplication *)application`

说明：iPhone设备只有有限的内存，如果为应用程序分配了太多内存操作系统会终止应用程序的运行，在终止前会执行这个方法，通常可以在这里进行内存清理工作防止程序被终止

7、`- (void)applicationSignificantTimeChange:(UIApplication*)application`

说明：当系统时间发生改变时执行

8、`- (void)applicationDidFinishLaunching:(UIApplication*)application`

说明：当程序载入后执行

9、`- (void)application:(UIApplication)application willChangeStatusBarFrame:(CGRect)newStatusBarFrame`

说明：当StatusBar框将要变化时执行

10、`- (void)application:(UIApplication*)application willChangeStatusBarOrientation:`

`(UIInterfaceOrientation)newStatusBarOrientation`

`duration:(NSTimeInterval)duration`

说明：当StatusBar框方向将要变化时执行

11、`- (BOOL)application:(UIApplication*)application handleOpenURL:(NSURL*)url`

说明：当通过url执行

12、`- (void)application:(UIApplication*)application didChangeStatusBarOrientation:(UIInterfaceOrientation)oldStatusBarOrientation`

说明：当StatusBar框方向变化完成后执行

13、`- (void)application:(UIApplication*)application didChangeSetStatusBarFrame:(CGRect)oldStatusBarFrame`

说明：当StatusBar框变化完成后执行


### UIApplication的一些功能

1.设置`icon`上的数字图标

    //设置主界面icon上的数字图标，在2.0中引进， 缺省为0
    
    [UIApplication sharedApplication].applicationIconBadgeNumber = 4;
    
2.设置摇动手势的时候，是否支持`redo`,`undo`操作

    //摇动手势，是否支持redo undo操作。

	//3.0以后引进，缺省YES

    [UIApplication sharedApplication].applicationSupportsShakeToEdit =YES;

3.判断程序运行状态

    //判断程序运行状态，在2.0以后引入

    /*

	     UIApplicationStateActive,
	
	     UIApplicationStateInactive,
	
	     UIApplicationStateBackground

     */

	if([UIApplication sharedApplication].applicationState == UIApplicationStateInactive){

        NSLog(@"程序在运行状态");

    }
    
4.阻止屏幕变暗进入休眠状态

    //阻止屏幕变暗，慎重使用,缺省为no 2.0

    [UIApplication sharedApplication].idleTimerDisabled =YES;

5.显示联网状态

    //显示联网标记 2.0
    [UIApplication sharedApplication].networkActivityIndicatorVisible =YES;

6.在`map`上显示一个地址

	NSString* addressText = @"1 Infinite Loop, Cupertino, CA 95014";

	// URL encode the spaces

    addressText =  [addressText stringByAddingPercentEscapesUsingEncoding:NSASCIIStringEncoding];

	NSString* urlText = [NSString stringWithFormat:@"http://maps.google.com/maps?q=%@", addressText];

    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:urlText]];

7.发送电子邮件

	NSString *recipients = @"mailto:first@example.com?cc=second@example.com,third@example.com&subject = Hello from California!";

	NSString *body = @"&body=It is raining in sunny California!";

    NSString *email = [NSStringstringWithFormat:@"%@%@", recipients, body];

    email = [email stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];

    [[UIApplicationsharedApplication] openURL:[NSURLURLWithString:email]];
    
8.打电话到一个号码

    // Call Google 411
    [[UIApplicationsharedApplication] openURL:[NSURLURLWithString:@"tel://8004664411"]];

9.发送短信

    // Text to Google SMS
    [[UIApplication sharedApplication] openURL:[NSURLURLWithString:@"sms://466453"]];

10.打开一个网址

	// Lanuch any iPhone developers fav site
	[[UIApplication sharedApplication] openURL:[NSURLURLWithString:@"http://itunesconnect.apple.com"]];


#### Reference

http://blog.csdn.net/wanna_love/article/details/25073879