---
layout: post
title: jquery-validation进行表单校验
category: java
keywords: jQuery-validation,form
excerpt: 学海无涯.
---
`form`表单一般都需要对用户填写的数据进行校验，符合一些规则才能够让用户进行提交，今天使用`jQuery`的表单校验插件`jQuery-validation`.</br>
### 下载
下载地址：https://jqueryvalidation.org/
##使用
下载好插件后在`jsp`页面引入`jquery-validate.js`（记得`jQuery.js`也得引入）
```
 <script src="${pageContext.request.contextPath}/static/js/jquery.validate.js"></script>
 <script src="${pageContext.request.contextPath}/static/js/jquery-3.4.0.js"></script>
 ```
### 编写`jsp`页面
给`form`表单一个`id`,并给每个`input`标签对应的`name`.
```
 <form action="/release.action"  id="release_form" method="post" class="post-form" enctype="multipart/form-data">
                        <table>
                            <tr><td><lable>.</lable><input type="text" name="goods_title" placeholder="请输入标题"></td></tr>
                            <tr><td><lable>.</lable><input type="text" name="goods_detail" placeholder="请输入商品详情"></td></tr>
                            <tr><td class="money"><lable>.</lable><input type="text" name="goods_price" placeholder="请输入商品价格" class="post-goods-pirce"><span class="money-logo">￥</span></td></tr>
                            <tr><td><lable>.</lable><select class="select-postgoods" name="goods_category_name">
                                <option value="">选择商品类型</option>
                                <option>代步工具</option>
                                <option>手机</option>
                                <option>电脑</option>
                                <option>数码</option>
                                <option>电脑</option>
                                <option>衣鞋伞帽</option>
                                <option>书籍教材</option>
                                <option>体育健身</option>
                                <option>乐器</option>
                                <option>自行设计</option>
                                <option>其他</option>
                            </select></td></tr>
                            <tr><td><lable>.</lable><input type="text" placeholder="请输入手机号" name="goods_contact_phone"></td></tr>
                            <tr><td><lable>.</lable><input type="text" placeholder="请输入QQ" name="goods_contact_qq"></td></tr>
                            <tr><td><lable>.</lable>
                                <div class="uploader-img">
                                    <input type="file" name="file" id="file" class="webuploader-element-invisible" multiple="multiple" onchange="showPic()"  accept="image/png,image/jpg,image/jpeg" >
                                    <div class="adduploader-img"><img src="${pageContext.request.contextPath}/static/images/xiaoyuantao/add-images.png" alt=""></div>
                                    <div>点击进入添加图片</div>
                                </div>
                                </td></tr>
                            <tr><td>
                               <div id = "showimage"></div>
                            </td></tr>
                            <tr><td><input type="submit" value="发布" class="submit-secondhand"></td></tr>
                        </table>
                    </form>
```
### 接下来编写校对规则的js
新建一个`check.js`（名字随意）
```
$(function(){
      //对表单提交进行拦截，校验不合格不能够提交
    $("#release_form").validate({
        submitHandler: function() {
            form.submit();
        },
        //开启隐藏项的校验，如果有下拉列表
        ignore:":hidden:not(select)",
        //校验规则名字与表单name一致
        rules: {
            goods_title: {
                required: true,
                minlength:5
            },
            goods_detail: {
                required: true,
                minlength: 10
            },
            goods_price: {
                required:true,
                digits:true
            },
            goods_category_name: {
                required: true
            },
            goods_contact_phone: {
                required: true,
                digits:true,
                rangelength:[11,11]
            },
            goods_contact_qq: {
                required: true,
                digits:true
            },
            file:{
                required:true,
                minlength:3
            }

        },
        //对应的提示信息
        messages: {
            goods_title: {
                required: "标题不能为空",
                minlength: "标题长度至少是5个字符"
            },
            goods_detail: {
                required: "商品详情不能为空",
                minlength: "商品详情长度至少是10个字符"
            },
            goods_price: {
                required: "价格不能为空",
                digits:"价格必须为数字"
            },
            goods_category_name: {
                required: "必须为商品选择类别"
            },
            goods_contact_phone: {
                required: "电话不能为空",
                digits:"请填数字",
                rangelength:"请填十一位电话号码"
            },
            goods_contact_qq:{
                required:"qq不能为空",
                digits:"请输入数字"
            },
            file:{
                required:"请至少选择一张图片",
                minlength:"请最少上传3张图片"
            }
        },
        //校验提示的样式
         errorClass:"error"
    });
});
```
上面js需要注意的是：必须对提交进行拦截，同时校验规则和提示信息也必须和`input`的`name`一致.同时如果有下拉列表的话需要在第一个`option`的加上`value`=""
这样可以才可以在用户不选择选项时判断是空.
### 关于校验规则
* 名字               ->                 规则                                   
* (1)required:true   ->                必输字段                                  
* (2)email:true      ->                必须输入正确格式的电子邮件                  
* (3)url:true        ->                必须输入正确格式的网址                      
* (4)date:true       ->                必须输入正确格式的日期                      
* (5)dateISO:true    ->                必须输入正确格式的日期                      
* (6)number:true     ->                必须输入合法的数字(负数，小数)               
* (7)digits:true     ->                必须输入整数                                
* (8)creditcard:     ->                必须输入合法的信用卡号                      
* (9)equalTo:"#field"->                输入值必须和#field相同                       
* (10)accept:        ->                输入拥有合法后缀名的字符串（上传文件的后缀）   
* (11)maxlength:5    ->                输入长度最多是5的字符串(汉字算一个字符)       
* (12)minlength:10   ->                输入长度最小是10的字符串(汉字算一个字符)      
* (13)rangelength:[5,10] ->            输入长度必须介于 5 和 10 之间的字符串")(汉字算一个字符)
* (14)range:[5,10]   ->                输入值必须介于 5 和 10 之间                     
* (15)max:5          ->                输入值不能大于5 
* (16)min:10         ->                输入值不能小于10</br>
使用默认的校验提示效果是英文并且样式不是很好看
想改变校验提示的样式则需自己编写`.erro`r的`css`
例如:
```
.error{
    color: #e60012;
    font-size: 2px;
    font-weight: bolder;
    display: block;
}
```
或者直接在源码里面改变默认的样式以后都可以使用,最后记得在`jsp`页面引入`css`
效果如下：
![](https://github.com/Apathyyi/Apathyyi.github.io/blob/master/image/jquery-validation.png)
如果想添加校验成功的样式可参考下列代码：
```
   success: function(label) {
           //label指向上面那个错误提示信息标签
           label.html("");
           label.addClass("success").html("Ok!");  //加上自定义的success类
         }
```
直接加在`errorClass`后面即可，同时编写`.success`的`css`
```
.success {
    color: greenyellow;
    font-size: 2px;
    font-weight: bolder;
    display: block;
}
```
不过上面代码有个`bug`,就是在成功之后再次输入不合格的类型会造成提示标签同时有`error`和`success`的`class`属性,有一定冲突，尝试在成功后移除`error`未成功.
所以我选择不增加成功的样式.

