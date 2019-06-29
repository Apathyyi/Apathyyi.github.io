---
layout: post
title: 表单提交一般数据和文件数据及获取.
category: java
keywords: form,fileupload.
excerpt: 学海无涯
---
`form`表单提交的一般数据后台可使用`request.getParameter()`来接收,可是当表单内有文件数据时就不能这样获取到文件数据.
如何才能分别获取一般数据和文件数据.
## jsp页面
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<html>
<head>
    <title>发布二手</title>
    <link rel="stylesheet" href="${pageContext.request.contextPath}/static/css/jquery-ui.css">
    <link rel="stylesheet" href="${pageContext.request.contextPath}/static/css/xiaoyuantao.css">
    <script src="${pageContext.request.contextPath}/static/js/jquery-3.4.0.js"></script>
    <script src="${pageContext.request.contextPath}/static/js/jquery-ui.js"></script>
</head>
<body>
<form action="/release.action"  id="release_form" method="post" class="post-form" enctype="multipart/form-data">
                        <table>
                            <tr><td><lable>.</lable><input type="text" name="goods_title" placeholder="请输入标题"></td></tr>
                            <tr><td><lable>.</lable><input type="text" name="goods_detail" placeholder="请输入商品详情"></td></tr>
                            <tr><td class="money"><lable>.</lable><input type="text" name="goods_price" placeholder="请输入商品价格" class="post-goods-pirce" onchange="showPic()"><span class="money-logo">￥</span></td></tr>
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
                                    <input type="file" name="file" class="webuploader-element-invisible" multiple="multiple" accept="image/png,image/jpg,image/jpeg" >
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
</body>
</html>                    
```
需要注意的是上传文件需要改变`form`表单的`enctype="multipart/form-data"` </br>
同时如果想传多个文件的话需要在上传文件的`input`加上`multiple="multiple"` </br>
这样就可以在选择文件时选择多个,不过只能一次选择多个,没有叠加的效果,这里我限制了上传文件类型为图片.
## 后端接收数据的`servlet`
需要引入两个`jar`包,`commons-io-2.6.jar`和`commons-fileupload-1.3.2.jar`
`maven`依赖
```
<!--文件上传-->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.4</version>
        </dependency>
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.2</version>
        </dependency>
```
也可以自行下载，地址:http://commons.apache.org/proper/commons-io/ </br>
http://commons.apache.org/proper/commons-fileupload/download_fileupload.cgi
```
package com.schoolTao.controller.sy;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import com.schoolTao.model.TCategory;
import com.schoolTao.model.TSaleGoods;
import com.schoolTao.service.ICategeoryService;
import com.schoolTao.service.StySaleGoodService;
import com.schoolTao.service.impl.CategeoryServiceImpl;
import com.schoolTao.service.impl.StySaleGoodServiceImpl;
import com.schoolTao.utils.UploadUtils;
import org.apache.commons.fileupload.*;
import org.apache.commons.fileupload.servlet.*;
import org.apache.commons.fileupload.disk.*;

@WebServlet(name = "ReleaseServlet",urlPatterns = {"/release.action"})
public class ReleaseServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
        //保存文件到磁盘uploadFile目录下
        String savepath = ("D://uploadFile");
        //创建一个文件工厂对象
        DiskFileItemFactory factory = new DiskFileItemFactory();
        //创建一个servlet文件上传对象
        ServletFileUpload upload = new ServletFileUpload(factory);
        String image= "";
        //用于保存非文件数据的集合
        List<String> pList = new ArrayList<>();
        TSaleGoods tSaleGoods = new TSaleGoods();
        //form表单数据的集合
        List<FileItem> list = null;
        String filename;
        try {
            //获取form表单提交的数据
            list = upload.parseRequest(request);
        } catch (Exception e) {
            e.printStackTrace();
        }
        if(list!=null) {
            for (FileItem item : list) {
                //如果fileitem中封装的是普通输入项的数据
                if (item.isFormField()) {
                    //String name = item.getFieldName();
                    String value = item.getString("UTF-8");
                    //将非文件的其他参数放到一个list中，可按表单顺序取
                    pList.add(value);
                    continue;
                } else {
                    //如果fileitem中封装的是上传文件
                    //上传文件需要的文件流参数
                    InputStream stream = item.getInputStream();
                     //获取文件名
                    filename = item.getName();  
                    if (filename == null || filename.trim().equals("")) {
                        //判空处理
                        }
                        continue;
                    }
                    //获得随机的文件名
                    String randomfilename =  UploadUtils.getrandomFilename(filename);
                    //获得分离目录
                    String realpath = UploadUtils.getpath(randomfilename);
                    String url = savepath + realpath;
                    //文件保存的路径
                    File file = new File(url);  
                    if(!file.exists()){
                        file.mkdirs();
                    }
                     //调用工具类方法
                    UploadUtils.uploadFile(stream, file, randomfilename);  
                    //设置发布单图片,使用#分割每张图片
                    String imgpath = (realpath+randomfilename).replace("\\","//");
                    image += imgpath+"#";
                }
            }
        }
        image = image.substring(0,image.length()-1);
        //设置图片路径
        image = image.substring(0,image.length()-1);
        tSaleGoods.setGoodsImg(image);
        //处理表单数据
        processFormdata(pList, tSaleGoods);
        tSaleGoods.setGoodsByUserId(1);
        StySaleGoodService releseService = new StySaleGoodServiceImpl();
        //保存发布的订单
        releseService.save(tSaleGoods);
        request.getRequestDispatcher("/saleGoods.action").forward(request,response);
}
    protected void doGet(HttpServletRequest request, HttpServletResponse response) {

    }
    //处理表单提交的其他数据
    public void processFormdata( List<String> list ,TSaleGoods tSaleGoods) {
        //发布单title
        String goods_title = list.get(0);
        System.out.println(goods_title);
        //发布单详情
        String goods_detail = list.get(1);
        //价格
        String goods_price = list.get(2);
        //类别
        String goods_categeory_name = list.get(3);
        System.out.println(goods_categeory_name);
        //电话
        String goods_contact_phone = list.get(4);
        //qq
        String goods_contact_qq = list.get(5);
        tSaleGoods.setGoodsTitle(goods_title);
        tSaleGoods.setGoodsDetails(goods_detail);
        tSaleGoods.setGoodsPrice(Double.parseDouble(goods_price));
        tSaleGoods.setGoodsContactPhone(goods_contact_phone);
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd");//设置日期格式
        String publishDate = df.format(new Date());
        //将日期横杠去掉
        String publishtime = publishDate.replace("-", "");
        //转换为int,设置发布日期
        tSaleGoods.setGoodsPublishTime(Integer.parseInt(publishtime));
        //设置联系人qq
        tSaleGoods.setGoodsContactQq(goods_contact_qq);
        //设置商品名称
        tSaleGoods.setGoodsCategoryName(goods_categeory_name);
        //通过名字查找id
        ICategeoryService iCategeoryService = new CategeoryServiceImpl();
        TCategory tCategory = iCategeoryService.findCategoryByname(goods_categeory_name);
        System.out.println(tCategory.getCategoryId());
        //设置商品id
        tSaleGoods.setGoodsCategoryId(tCategory.getCategoryId());
        //默认热度为0
        tSaleGoods.setGoodsHot(0);
        //默认有有效期
        tSaleGoods.setGoodsStatus(1);
    }
}
```
具体的步骤代码里已经很清楚了,主要有几步 </br>
1.创建`servlet`文件上传对象并传入一个文件工厂对象 </br>
2.获取表单提交的数据封装成一个`FileItem`集合 </br>
3.判断是否为文件数据 </br>
4.分别处理一般数据和文件数据 </br>
我这里是将上传的图片存到磁盘并将路径存到数据库(如果想要在`jsp`页面显示需要配置虚拟路径(这里就不说了)) </br>
需要注意的是用户可能上传的图片名字相同和一个目录下文件过多 </br>
解决这两个问题通常采用随机文件名(UUID和当前系统时间戳作为文件名)和目录分离 </br>
## UploadUtils文件上传工具
```
package com.schoolTao.utils;


import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.UUID;

import org.apache.commons.fileupload.disk.DiskFileItemFactory;
import org.apache.commons.fileupload.servlet.ServletFileUpload;
/**
 *文件上传（文件流，要上传的路径，文件名）三个参数
 */
public class UploadUtils {
    public static void uploadFile(InputStream filestream,File savaPath,String filename){
        //1、创建一个DiskFileItemFactory工厂
        DiskFileItemFactory factory = new DiskFileItemFactory();
        ServletFileUpload upload = new ServletFileUpload(factory);
        //解决上传文件名的中文乱码
        upload.setHeaderEncoding("UTF-8");
        factory.setSizeThreshold(4 * 1024);
        // 设置允许上传的最大文件大小 4M
        upload.setSizeMax(4 * 1024 * 1024);
        //创建一个文件输出流,有些浏览器提交上来的文件名是带有路径的
        //分离出文件名
        filename = filename.substring(filename.lastIndexOf("\\")+1);
        String realSavePath = savaPath+"\\"+filename;
        //创建一个输出流
        FileOutputStream out = null;
        try {
            out = new FileOutputStream(realSavePath);
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        //创建一个缓冲区
        byte buffer[] = new byte[1024];
        //判断输入流中的数据是否已经读完的标识
        int len = 0;
        //循环将输入流读入到缓冲区当中，(len=in.read(buffer))>0就表示in里面还有数据
        try {
            while((len=filestream.read(buffer))>0){
                //使用FileOutputStream输出流将缓冲区的数据写入到指定的目录(savePath + "\\" + filename)当中
                out.write(buffer, 0, len);
            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println("这才是真正的目录:"+realSavePath);
        //关闭输入流
        try {
            filestream.close();
            out.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        //关闭输出流
    }
    //生成随机文件名
    public static String getrandomFilename(String filename){
        int sub=filename.lastIndexOf(".");
        if(sub==-1){
            return UUID.randomUUID().toString().replace("-", "")+".png";
        }else {
            String extions = filename.substring(sub);
            return UUID.randomUUID().toString().replace("-", "") + extions;
        }
    }
    //根据文件名生成二级目录
    public static String getpath(String uuidFilename){
        int code1 = uuidFilename.hashCode();
        int dir1 = code1 & 0xf;//一级目录
        int code2= code1>>>4;
        int dir2=code2 & 0xf;//二级目录
        return "/"+dir1+"/"+dir2;
    }
}

```
随机文件名和目录分离比较简单,然后将流数据写入对应文件.如果是使用`Ajax`传的图片数据有不同，下次会有讲解.
如果有什么问题希望指正,大家共同进步.




