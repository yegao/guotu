国家图书馆寻根网前端代码总结
----
目录结构及作用如下
+ template --- 模块(html文件之中可以引用这里面的模块)
    * basejs.tpl  --- require.js引用和配置模块
    
+ require  --- require.js引用根目录，里面存放的大多数是.es6文件，编译到build之后猴嘴会变成.js
+ general  --- 游客或者用户进来的时候默认进来看的就是这里的东西
    * index.html  --- 首页
    * news.html   --- 新闻中心
    * jpxg.html   --- 家谱寻根
    * xsyy.html   --- 姓氏源流
    * qxdt.html   --- 迁徙地图
    * xgwd.html   --- 寻根问答
    * 其他的文件应该都是上面6个页面中进行某些操作(点击,提交等等)跳转进入的界面
+ work     --- 用户查看自己的个人信息和管理员管理的地方
    * user.html   --- 我的账户   （普通用户级别可以进入）
    * wdqy.html   --- 我的亲友   （普通用户级别可以进入）
    * rjgs.html   --- 日记故事   （普通用户级别可以进入）
    * scdjp.html  --- 收藏的家谱 （普通用户级别可以进入）
    * scdpy.html  --- 收藏的谱页 （普通用户级别可以进入）
    * jpjz.html   --- 家谱捐赠   （普通用户级别可以进入）

    * jpgl.html   --- 家谱管理   （管理员级别可以进入）
    * jzsh.html   --- 捐赠审核   （管理员级别可以进入）
    * zj.html     --- 专辑      （管理员级别可以进入）
    * wz.html     --- 文章      （管理员级别可以进入）
    * glzh.html   --- 管理账户   （管理员级别可以进入）
    * glrz.html   --- 管理日志   （管理员级别可以进入）

    * ptgl.html   --- 平台管理    （qingtime超管可以进入）
+ static   --- 静态资源(图片,字体,样式,引用的js脚本)
    * css     --- 样式
    * fonts   --- 字体
    * images  --- 图片
    * js      --- 自己写的js脚本
    * libs    --- 引用的js脚本
    
general文件夹中的html文件大致的写法
----
```html
<link rel="import" href="../template/head.tpl?__inline"> <!--head模块 这里引用了所有general文件夹下界面共有的样式文件-->
<style>当前页面会用到的特有的样式</style>
<link rel="import" href="../template/nav.tpl?__inline">  <!--顶部的切换面板-->
<div id="main">主界面</div>
<link rel="import" href="../template/foot.tpl?__inline"> <!--foot模块-->
<link rel="import" href="../template/basejs.tpl?__inline"> <!--require.js配置模块-->
<script>
    require(['easy','common'],function(){
        //这里写require的脚本已经加载完之后的回调处理
    });
</script>
</html>
```
work文件夹中html文件大致的写法
```html
<link rel="import" href="../template/work_head.tpl?__inline"> <!--head模块 这里引用了所有general文件夹下界面共有的样式文件-->
<style>当前页面会用到的特有的样式</style>
<link rel="import" href="../template/work_nav.tpl?__inline"> <!--左侧的切换面板-->
<div id="wrapper">主界面</div>
<link rel="import" href="../template/work_basejs.tpl?__inline"> <!--require.js配置模块-->
<script>
    require(['news'],function(){
        //这里写require的脚本已经加载完之后的回调处理
    });
</script>
</body>
</html>
```
关于权限
----
`注：以下为我接手项目的时候公司内部定义的，其实需求书上和这一块不同，后来这一块的定义有很大的改变`
游客、用户、协管、管理员、qingtime超管

+ 游客   --- 指的是没有注册那些
+ 用户   --- 指的是注册了账号,登录进来了的那些人
+ 协管   --- 由管理员指定的
+ 管理员 --- （比如国家图书馆的管理员，他可以创建协管）
+ qingtime超管 --- qingtime超管可以创建机构(比如国家图书馆就是一个机构)

怎么研究这个代码
----
你会发现，基本上所有的html页面都会引用到common.js
require/general/common.js的代码如下（work文件夹下的html文件引用的是require/work/common.js）
```javascript
//先给jquery写了一些拓展
$.fn.extend({
/**
*   $('#xxx').enter(function(){
*     对#xxx元素回车之后的处理
*   }) 
*/
    enter:function(callback){
        $(this).on('keypress',function (e) {
            var keyCode = e.keyCode ? e.keyCode : e.which ? e.which : e.charCode;
            if (keyCode == 13){
                callback();
            }
        });
    },
    validate:function(param){
        var fields = param.fields,form = this,sub = $(form).find('.submit').eq(0),errorBlock = $('<div class="error-block"></div>');
        errorBlock.insertBefore(sub);
        sub.on('click',function(){
            for(var k in fields){
                var item = fields[k],one=$(form).find('[name="'+k+'"]').eq(0);
                if(!(item.reg.test($.trim(one.val())))){
                    if(item.message){
                        errorBlock.html(item.message);
                        errorBlock.addClass('active');
                        setTimeout(function(){
                            errorBlock.removeClass('active');
                        },1000);
                    }
                    return false;
                }
            }
            param.submit($(this).closest('form'));
        });
    },
    /**
    * 将一段json中的值对应放到指定的form表单中
    */
    json2form:function(json){
        this.get(0).reset();
        for(var k in json){
            if(0<$(this).find('[name="'+k+'"]').length){
                var one = $(this).find('[name="'+k+'"]').eq(0);
                if(one.prop('type')=='checkbox'){
                    switch(typeof json[k]){
                        case 'boolean': one.prop('checked',json[k]);break;
                        case 'number': one.prop('checked',!!(+json[k]));break;
                        default : one.prop('checked',!!(json[k]));break;
                    }
                }
                else{
                    one.val(json[k]);
                }
            }
        }
    },
    /**
     *上传文件到七牛,上传成功之后，返回的信息会放在clound中 $(it).data('clound',$(it).data('clound')+','+ key);
     */
    qiniu:function(o){
        var url = o.url||'//icaretest.lifexp.cn/icare-api/upload/getUptoken';
        var token = o.token;
        var i = o.i || 0;
        var $memo = o.$memo || null;
        var mutiple = o.mutiple || null;
        var callback = o.callback;
        var it = this.get(0),fs = it.files,len = fs.length;
        if(i<len){
            $.ajax({
                url:url,
                type: 'POST',
                data:{fileName: fs[i].name,token: token},
                dataType: "json",
                success:function(middle){
                    if(middle && middle.data){
                        var data = new FormData();
                        var key = (new Date()).getTime().toString(36)+'_'+Math.random().toString(36).substr(2) + fs[i].name.match(/\.?[^.\/]+$/);
                        data.append("file", fs[i]);
                        data.append("token", middle.data);
                        data.append("key", key);
                        if(mutiple){
                            data.append("mutiple",1);
                        }
                        $.ajax({
                            url: 'http://upload.qiniu.com/',
                            type: 'POST',
                            data: data,
                            processData: false,
                            contentType: false,
                            xhr: function(){
                                var myXhr = $.ajaxSettings.xhr();
                                if(myXhr.upload){
                                    myXhr.upload.addEventListener('progress',function(e) {
                                        if (e.lengthComputable) {
                                            var percent = i/len*100+e.loaded/(e.total*len)*100;
                                            $memo.text(percent.toFixed(2) + "%");
                                        }
                                    }, false);
                                }
                                return myXhr;
                            },
                            success: function() {
                                if(!$(it).data('clound') || !mutiple){
                                    $(it).data('clound',','+key);
                                }
                                else{
                                    $(it).data('clound',$(it).data('clound')+','+ key);
                                }
                                $(it).qiniu({
                                    url:url,
                                    token:token,
                                    i:++i,
                                    $memo:$memo,
                                    mutiple:mutiple,
                                    callback:callback
                                });
                            },
                            error: function() {
                                $memo.text('上传失败');
                            }
                        });
                    }
                }
            });
        }
        else{
            $(it).data('clound',$(it).data('clound').substr(1));
            $memo.text('上传成功');
            if(callback){
                if(typeof callback == 'string' && typeof window[callback] == 'function'){
                    (window[callback])();
                }
                else if(typeof callback == 'function'){
                    callback();
                }
            }
        }
    },
    /**
     * upload会调用上面的qiniu
     */
    upload:function(o){
        var url = o.url;
        var i = o.i||0;
        var $memo = o.$memo;
        var callback = o.callback;
        var it = this.get(0),token=userInfo.token,mutiple = !!($(this).prop('multiple'));
        var fs = it.files,len = fs.length,size=0;
        for(;i<len;i++){
            size+=fs[i].size
        }
        $(it).data('size',size);
        if(!$memo){
            $memo=$(it).siblings('.file-memo').eq(0);
        }
        if($(it).data('qiniu')){
            $(it).qiniu({url:url,token:token,i:o.i||0,$memo:$memo,mutiple:mutiple,callback:callback});
        }
        else{
            var data = new FormData();
            i = 0;
            for(;i<len;i++){
                data.append("file", fs[i]);
            }
            data.append("token", token);
            if(mutiple){
                data.append('mutiple',1);
            }
            $.ajax({
                url: url || 'https://icaretest.lifexp.cn/post-service/upload',
                type: 'POST',
                data: data,
                processData: false,
                contentType:  false,
                xhr: function () {
                    var myXhr = $.ajaxSettings.xhr();
                    if (myXhr.upload) {
                        myXhr.upload.addEventListener('progress', function (e) {
                            if (e.lengthComputable) {
                                var percent = e.loaded / (e.total * len) * 100;
                                $memo.text(percent.toFixed(2) + "%");
                            }
                        }, false);
                    }
                    return myXhr;
                },
                success: function (res) {
                    if(res.data instanceof Array){
                        $(it).data('clound',res.data.join(','));
                    }
                    else{
                        $(it).data('clound',res.data);
                    }
                    ////icaretest.lifexp.cn
                    $memo.text('上传成功');
                    if(callback){
                        if(typeof callback == 'string'){
                            (window[callback])();
                        }
                        else if(typeof callback == 'function'){
                            callback();
                        }
                    }
                },
                error: function (e) {
                    $memo.text('上传失败');
                }
            });
        }
    },
    /**
     * 对于某些元素，你想要在点击除了这个元素之外的其他地方之后就隐藏这个元素，就可以
     * $('#xxx').clickOut();
     * 上面这行代码会在你点击#xxx以外的其他地方之后隐藏#xxx元素
     */
    clickOut:function(cb){
        var it = this;
        var callback = cb || function(it){
                $(it).hide();
            }
        $(document).on('click',function(event) {
            if(!$(event.target).closest($(it)).length) {
                callback(it);
            }
        });
    }
});
$(document).ready(async function(){
    //通过url地址得到当前的模块,然后给切换面板上的这个模块加上.active
    $('.nav-top-item>a').each(function(){
        var reg = new RegExp(".*"+location.pathname+".*");
        if(reg.test($(this).prop('href'))){
            $(this).addClass('active');
            return false;
        }
    });
    //当前用户的信息都在userInfo这个全局变量里面
    window.userInfo = JSON.parse(localStorage.getItem('userinfo'));
    /**
    * 如果userInfo不是空,并且里面有token属性,就发个getAuthCode请求看看这个会话有没有过期(当前这个用户是不是处在已经登录的状态)
    * 如果过期了就在localStorage把老的userInfo清除掉，并且直接跳转到登录页（init.html）
    * 如果没有过期那么就将userInfo赋值为getAuthCode的返回结果,作为当前用户的个人信息
    */
    if(!!userInfo && userInfo.hasOwnProperty('token')){
        var result = await api.getAuthCode();
        if(result && result.error) {
            userInfo = null;
            localStorage.removeItem('userinfo');
            easy.warn('你的会话已经过期或者在别的地方被登录,请重新登录!');
            if(/\/jiapu.html/.test(location.pathname.toLowerCase())){
                location.href = 'init.html';
            }
            $('#toinit').show();
            $('#toinformation').hide();
        }
        else{
            userInfo = result.data || userInfo;
            localStorage.setItem('userinfo',JSON.stringify(userInfo));
            $('#toinit').hide();//说明已经登录了
            $('#toinformation').text(userInfo.profile.nickName).show();
            $('.nav-need-token').each(function(){
                $(this).prop('href',$(this).prop('href')+'?'+userInfo.token);
            });
            if(typeof isVerified =='function'){
                isVerified();
            }
        }
        if(typeof afterVerified == 'function'){
            afterVerified();
        }
    }
    else{
        $('#toinit').show();
        $('#toinformation').hide();
        // if(/\/[jpxg.html|news.html]/.test(location.pathname.toLowerCase())){
        //     location.href = 'init.html';
        // }
    }
    
    $('#nav-toggle').on('click',function(){
        $('#nav-top,#nav-toggle-bottom').toggleClass('active');
    });
    //公共UI事件绑定
    //隐藏面板事件绑定
    $('.hide-panel').on('click',function(){
        $(this).closest('.panel').removeClass('active');
    });
    //上传事件绑定
    $('input[type="file"]').each(function(){
        var it = this;
        $(it).siblings('.file-memo').text($(it).data('memo'));
        $(it).on('change',function(){
            if($(this).data('before') && typeof $(this).data('before')){
                (window[$(this).data('before')])();
            }
            var i =0,len = this.files.length;
            if(1==len){
                $(it).siblings('.file-memo').eq(0).text((this.files)[0].name.substr(-10));
            }
            else if(1<len){
                $(it).siblings('.file-memo').eq(0).text(len+'个文件');
            }
            for(;i<len;i++){
                var fileNameArray = (this.files)[i].name.toLowerCase().split('.');
                if(fileNameArray && 1<fileNameArray.length){
                    if(_.indexOf($(it).data('allow').split(','), fileNameArray[fileNameArray.length-1])==-1){
                        easy.resetFile(this);
                        $(it).siblings('.file-memo').text('上传的文件类型不正确');
                        return false;
                    }
                }
            }
            if(!!$(it).data('auto')){
                $(it).upload({
                    url:$(it).data('url')||void(0),
                    i:0,
                    callback:$(it).data('callback')
                });
            }
            else if(0==$(it).siblings('.file-upload').length){
                var $upload = $('<span class="file-upload"><i class="fa fa-cloud-upload" aria-hidden="true"></i></span>');
                $(it).parent().append($upload);
                $upload.on('click',function(){
                    $(it).upload({
                        url:$(it).data('url')||void(0),
                        i:0,
                        callback:$(it).data('callback')
                    });
                });
            }
        });
    });
    //返回绑定
    $('.goback').on('click',function(){
        $(this).closest('.wrap').hide();
    });
});
```
举个例子(general/news.html)
----
general/news.html最后一行引用了require/general/news.js
```html
<script type="text/javascript" src="../require/general/news.js"></script>
```
require/general/news.js的代码如下
```javascript
//、、、略 (定义一些全局函数,这个你先不管,主要看下面这段)
/**
* 下面这三行代码的意思就是说，先加载jquery，加载完jquery就加载common,$对应的就是jquery返回的module，其实就是jquery
* 你加载完了common就会执行common里面的代码了（验证了用户token是否过期，得到了最新的用户信息，对公共组件绑定了事件等等）
* 在jquery和common都已经加载完之后就开始执行回调处理(这里的例子是执行了'renderTopic();')
*/
require(['jquery','common'],function($){
    renderTopic();
});
```
