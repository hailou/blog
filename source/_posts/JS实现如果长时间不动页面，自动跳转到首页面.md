title: JS实现如果长时间不动页面，自动跳转到首页面
date: 2019-11-21 10:45:57
categories: 开发总结
tags: 
	- javascript
---
 
```
        /* 如果5分钟不动页面，自动提示 */
        var maxTime = 300; // seconds
        var time = maxTime;
        $('body').on('keydown mousemove mousedown', function(e) {
            time = maxTime; // reset
        });
        var intervalId = setInterval(function() {
            time--;
            if (time <= 0) {
                ShowInvalidLoginMessage();
                clearInterval(intervalId);
            }
        }, 1000)
        function ShowInvalidLoginMessage() {
            alert("您已经长时间没操作了，即将退出系统");
            //TODO 做需要做的操作
            //exp:关闭页面
            window.close();
        }
```