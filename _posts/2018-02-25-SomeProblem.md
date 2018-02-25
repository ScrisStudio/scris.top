---
layout: post
title: "Mixbank有关的一些遇到的问题"
image: '/assets/img/'
description: 'Unity和iOS'
tags:
- 广告
- 自适应

categories:
- 笔记

twitter_text: ''

---

    以下内容，就是摘录了别人的内容，整合一下。

## 广告

iOS中，VungleAds经常会没有广告，必须也把UnityAds加上去。

关于接入，文档中讲的很清楚了。UnityAds有时候会显示找不到函数，那就重启Unity，再不行删除Library文件夹。

## iPhoneX自适应

![Mixbank界面](https://i.loli.net/2018/02/25/5a929800b0ed3.png)

这款游戏基本上做完了，先拿开始界面吊一吊胃口，也是展示一下适应的结果。

[点击我](https://bitbucket.org/p12tic/iossafeareasplugin/src)

在里面下载SetCanvasBounds.cs 和 SafeAreaImpl.mm，倒入进你的工程（SafeAreaImpl.mm路径要和bitbucket里面一样）

接着，在Scene里创建一个Panel，挂在Canvas下面，倒入SetCanvasBounds，挂在MainCamera上（挂在哪个GameObject都一样），Panel选刚刚那个Panel，Canvas选你的Canvas，注意_Panel必须是原来的设置_，再把内容挂进去，内容就不会进安全区了。背景图片放在这个Panel之外或者放在Panel里面并且做额外的拉伸（不然安全区里没有背景了）

导出以后Xcode里就不用改了。

不过ShareSDK for Unity对于横屏的iPhoneX支持不好，已经反馈了。

## ShareSDK

ShareSDK脚本推荐挂Main Camera上，设置好appKey之类的，不然有可能会Xcode里Log说找不到（还是打得开的）。

Unity里面测试出现*EntryPointNotFoundException: __iosShareSDKRegisterAppAndSetPltformsConfig*不用管，导出了就好啦。

Unity里面出现appKey等内容找不到定义等问题，将Target改成iOS或者Android就好了。

如果要删SDK，最好不要按官方文档那样，应该挂一个ManagePlatforms脚本。（那脚本倒是有时候很鬼畜地直接又全部选中了……）而且删除的时候不要删FacebookMessanger的SDK不然Xcode会报错的。

有几款SDK需要不启用bitcode，如果用了那几个平台，必须关掉bitcode。（Xcode项目文件BuildSettings里面）

## Dotween

淡入淡出用这玩意挺好的

首先引用namespace

        using DG.Tweening;
        
init：
        
        DOTween.Init();
        
然后用这个函数就好了

    	void Fade(GameObject fader)
    	{
    		DOTween.ToAlpha(() => fader.GetComponent<Image>().color, x => fader.GetComponent<Image>().color = x, 0f, 2f);
    	}

fader为要淡出的GameObject，要淡入的放ta后面。

## GameCenter

    using UnityEngine.SocialPlatforms;

-- 开始的时候

    Social.localUser.Authenticate(HandleAuthenticated);
    
-- 引用的时候

	private void HandleAuthenticated(bool success)  
	{  
		GameCenterState = success;  
		//Debug.Log("*** HandleAuthenticated: success = " + success);  
		//初始化成功  
		if (success) {   
			userInfo = "Username: " + Social.localUser.userName +   
			           " && User ID: " + Social.localUser.id +   
			           " && IsUnderage: " + Social.localUser.underage;  
			//Debug.Log (userInfo);  
		} else {  
			//初始化失败  
          
		}  
	}

	public void OpenGameCenter()
	{
		if (Social.localUser.authenticated) {  
			Social.ShowLeaderboardUI();  
		}
		else
		{
			//无法读取信息时执行这里
		}
	}
	

-- 调用OpenGameCenter方法打开面板

    Social.ReportScore(/*分数*/, /*面板ID*/, HandleScoreReported);

回调函数：

    void HandleScoreReported(bool success)  
    {  
    	//Debug.Log("*** HandleScoreReported: success = " + success);  
    }  
    
-- 读取数据，并加载到本地最高分

	if (Social.localUser.authenticated) {  
		Social.LoadScores(/*面板ID*/,HandleGetGcScore);
	}
		
这个的回调：

*gcscore是long类型，即获得的GameCenter中分数*

        void HandleGetGcScore(IScore[] iscore)
        {
            foreach (var va in iscore)
            {
                if (va.userID != Social.localUser.id) continue;
                gcscore = va.value;
                if(PlayerPrefs.GetInt("Best") < Convert.ToInt32(gcscore))
                {
                    PlayerPrefs.SetInt("Best", Convert.ToInt32(gcscore));
                    PlayerPrefs.Save();
                    //显示在最高分的Text中
                }
                return;
            }
        }

## 跨场景播放音乐

挂AudioSource上，AudioSource放在开始音乐的场景

    public class MusicScript : MonoBehaviour {
        static MusicScript _instance;
        public static MusicScript instance    {
            get
            {
                if (_instance == null)
                {
                    _instance = FindObjectOfType<MusicaliaScript>();
                    DontDestroyOnLoad(_instance.gameObject);
                }
                return _instance;
            }
        }
        void Awake()
        {
    
            if (_instance == null)
            {
                _instance = this;
                DontDestroyOnLoad(this);
            }
            else if (this != _instance)
            {
                Destroy(gameObject);
            }
    
        }
    }   

## 杂

建议导出之后再自己搞Appiconset，Unity自动导出的少一个AppStore iCon。

FixedUpdate是好东西。

## 应用下载地址

看下一篇文章吧。