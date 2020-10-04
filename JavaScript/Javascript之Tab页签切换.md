---
title: Javascript之Tab页签切换
date: 2017-11-13 19:34:05
updateed: 2017-11-13 19:34:05
tags: [javascript, Tab]
---

<center>
  闲来无事，研究了一下 JavaScript 插件的写法，今天就将自己写的一个小插件记录下来。
本文介绍了此款插件的基本用法，实现的功能以及代码。
<center>
</br>
</center>
  标签：#javascript, #Tab
</center>

<!-- more -->

首先，来看看最终效果：
![eTab](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTEzMTkwOTEyMTA2)

这是一款普通的 Tab 选项卡插件，下面来讲讲它实现了哪些功能：

1、支持不同鼠标事件触发选项卡切换效果；
2、支持不同切换效果的配置，例如淡入淡出/直接切换；
3、支持默认展示第几个选项卡的配置；
4、支持选项卡的自动切换效果。

例子很简单，需要用到的知识包括：
1、html、css 的基础知识；
2、对 this，prototype，new 等关键词的理解。

简而言之，就是通过参数配置的形式来完成不同效果的展示。

下面先看一看如何使用：

```
1、$(".js-tab").etab();
2、$(".js-tab").etab({
                triggerType: "click",
                effect: "fade",
                invoke: 2,
                auto: 3000
            });
3、Tab.init($(".js-tab"));
```

本插件支持几种不同的初始化方式，代码很简单，类似于 BootStrap 插件的使用方法。下面奉上完整的代码：

```
index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Tab选项卡</title>
    <link href="tab.css" rel="stylesheet">
    <style>
        * {
            margin:0;
            padding:0;
        }
        body {
            background-color: #323232;
            font-size:12px;
            font-family:微软雅黑;
            padding:100px;
        }
        ul, li {
            list-style-type: none;
        }
    </style>
    <script src="../lib/jquery-1.11.3.js"></script>
    <script type="text/javascript" src="tab.js"></script>
</head>
<body>
    <div class="js-tab tab">
        <ul class="tab-nav">
            <li class="active"><a href="#">新闻</a> </li>
            <li><a href="#">电影</a> </li>
            <li><a href="#">娱乐</a> </li>
            <li><a href="#">科技</a> </li>
        </ul>

        <div class="content-wrap">
            <div class="content-item current">
                <h3>新闻</h3>
            </div>
            <div class="content-item">
                <h3>电影</h3>
            </div>
            <div class="content-item">
                <h3>娱乐</h3>
            </div>
            <div class="content-item">
                <h3>科技</h3>
            </div>
        </div>
    </div>
    <script>
        $(function() {
//            Tab.init($(".js-tab"));
            $(".js-tab").etab({
                triggerType: "click",
                effect: "fade",
                invoke: 2,
                auto: 3000
            });
            $(".js-tab").etab();
        });
    </script>
</body>
</html>
```

```
.tab {
    width: 300px;
}

.tab .tab-nav {
    height: 30px;
}

.tab .tab-nav li {
    float: left;
    margin-right:5px;
    background-color:#767676;
    border-radius:3px 3px 0 0;
}

.tab .tab-nav li a{
    display:block;
    height:30px;
    padding:0 20px;
    color: white;
    line-height:30px;
    text-decoration: none;
}

.tab .tab-nav .active {
    background-color: #fff;

}

.tab .tab-nav .active a{
    color: #777;
}

.tab .content-wrap{
    background-color: white;
    padding:5px;
    height:200px
}

.tab .content-wrap .content-item {
    position:absolute;
    height: 200px;
    display: none;
}

.tab .content-wrap .current {
    height: 200px;
    display: block;
}
```

最后将插件代码列出来，在代码里面已经写了很详细的注释：

```
/**
 * Created by Wu.lin on 2017/11/12.
 */
(function($){

    var Tab = function(tab, _params) {
        var _this = this;

        //保存单个Tab组件
        this.tab = tab;

        this.params = _params;

        //默认配置参数
        this.config = {
            //用来定义鼠标的出发类型   "click"/mouseover
            "triggerType": "mouseover",

            //用来定义内容切换效果，直接切换/淡入淡出
            "effect": "default",

            //默认展示第几个Tab
            "invoke": "1",

            //用来定义Tab是否自动切换，当指定了事件间隔，就表示自动切换，并指定了切换间隔
            "auto": false
        };

        //如果配置参数存在，就扩展默认的配置参数
        if(this.params){
            $.extend(this.config, this.params);
        }

        //保存Tab标签列表，以及对应的内容列表
        this.tabItem = this.tab.find("ul.tab-nav li");
        this.contentItem = this.tab.find("div.content-wrap .content-item");

        //保存配置参数
        var config = this.config;

        if(config.triggerType === "click") {
            this.tabItem.bind(config.triggerType, function() {
                _this.invoke($(this));
            });

        } else {
            this.tabItem.mouseover(function(){
                _this.invoke($(this));
            });
        }

        //自动切换功能
        if(config.auto) {
            this.timmer = null;

            //计数器
            this.loop = 0;

            this.autoPlay();

            this.tab.hover(function() {
                window.clearInterval(_this.timmer);
            }, function() {
                _this.autoPlay();
            });
        }

        //设置默认显示第几个Tab
        if(config.invoke > 1) {
            this.invoke(this.tabItem.eq(config.invoke - 1));
        }


    };

    Tab.prototype = {

        //事件驱动函数
        invoke: function(currentTab) {

            /**
             * 1、执行Tab选中状态，当前选中Tab加上Active,
             * 2、切换对应Tab内容，根据配置参数effect参数default|fade
             */

            var index = currentTab.index();
            var conItem = this.contentItem;

            //Tab切换
            currentTab.addClass("active").siblings().removeClass("active");

            //内容区域切换
            var effect = this.config.effect;

            if(effect === "fade") {
                conItem.eq(index).fadeIn().siblings().fadeOut();
            } else {
                conItem.eq(index).addClass("current").siblings().removeClass("current");
            }

            //注意，如果配置了自动切换，记得把当前的loop值设置为当前的Tab的index
            if(this.config.auto) {
                this.loop = index;
            }
        },

        //自动间隔切换
        autoPlay: function() {

            var _this_ = this,
                tabItems = this.tabItem,    //临时保存Tab列表
                tabLength = tabItems.size(),
                config = this.config;

            this.timmer = window.setInterval(function() {
                _this_.loop++;
                if(_this_.loop >= tabLength) {
                    _this_.loop = 0;
                }

                tabItems.eq(_this_.loop).trigger(config.triggerType);
            }, config.auto);

        }
    };

    Tab.init = function(tabs) {
        var _this_ = this;
        tabs.each(function() {
            new _this_($(this));
        });
        // var tab = new Tab($(".js-tab").eq(0));
    };

    //注册成JQuery方法
    $.fn.extend({
        etab: function(_param) {
            this.each(function () {
                new Tab($(this), _param);
            });
            return this;
        }
    });

    window.Tab = Tab;

})(jQuery);
```

如此看来，是不是很简单，一起来动手试试吧！

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
