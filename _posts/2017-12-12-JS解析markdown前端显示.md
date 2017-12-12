---
layout: post
title:  JS解析markdown to html页面显示(IE8兼容)
date: 2017-12-12
tag: markdown
---

## JS解析markdown在前端页面显示(IE8兼容)  

**最近内部系统需要一个答疑论坛,其中需要在线浏览归总的问题手册，所以考虑用markdown来写,这样也比较好解析。比较难受的就是必须兼容IE8及以上**  

**IE8及以下要支持H5特性需要提前引入两个JS插件**  

````
<!--IF under IE9 :需要引入的JS文件-->
<script src="html5shiv.js"></script>
<script src="respond.js"></script>

````

**IE8要支持bootstrap需要除引入上方两个JS外还需1.9.1版本的jquery.js**  

````
<!DOCTYPE HTML>
<html>
    <head>
        <!--需要的配置-->
        <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="IE=8">
        <link href="bootstrap.css" rel="stylesheet"/>
        <script src="jquery-1.9.1.js"></script>
        <script src="bootstrap.js"></script>
        
        <!--under IE9-->
        <script src="html5shiv.js"></script>
        <script src="respond.js"></script>
        
    </head>
</html>

````


### 方案一是用HyperDown,不兼容IE8(放弃)   

````

<!--需要引入的JS文件-->
<script src="jquery.js"></script>
<script src="Parser.js"></script>

<!--AJAX获取后台的JSON格式markdown数据returndata，解析插入页面-->
var parser = new HyperDown();
var htm = parser.makeHtml(returndata);
document.getElementById("book").innerHTML = htm;
````

<br>

### 方案二是用Marked,兼容亲测可用  

**下面给出完整的前后台实现**  

````
<!--需要引入的JS文件-->
<script src="jquery.js"></script>
<script src="marked.min.js"></script>
````
<br>
````
//前端用AJAX请求markdown文件数据
dataMap = {
  ExeMethod:"getxxxx"
}

$.ajax({
  type:"post",
  url:path+"/xxxx.do",
  async:false,
  dataType: "text", //这里返回text格式数据
  success: function(returndata){
    if(returndata){
      document.getElementById("book").innerHTML = marked(returndata); //调用marked.js的marked()将数据转换为HTML显示
    }
  },
  error: function(){
    //do something
  }
  
});

````
<br>

````
//后端springmvc模式
@Controller
@RequestMapping("/xxxx.do")
public class xxxAction{
  
  @RequestMapping(params = "ExeMethod=getxxxx")
  public void getxxx(HttpServletRequest req,HttpServletResponse res){
    String path = "xxxx";
    File file = new File(path);
    StringBuilder builder = new StringBuilder();
    BufferedReader reader = null;
    try{
      reader = new BufferReader(new InputStreamReader(new FileInputStream(file),"UTF-8"));
      String temp = null;
      while((temp=reader.readLine())!=null){
        bulider.append(temp+"\n");
      }
      reader.close();
    }catch(IOException e){
      e.printStackTrace();
    }finally{
      try{
        reader.close();
      }catch(IOException e){
        e.printStackTrace();
      }
    }
    
    try{
      res.setCharcaterEncoding("UTF8");
      PrintWriter pw = res.getWriter();
      pw.write(bulider.toString());
      pw.close();
    }catch(IOException e)
    e.printStackTrace();
  }
}
````


**不能COPY项目代码,以上代码纯手打,未验证**  
**感觉Editor.md不错,留着以后试试**

<br>
<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)
````

````