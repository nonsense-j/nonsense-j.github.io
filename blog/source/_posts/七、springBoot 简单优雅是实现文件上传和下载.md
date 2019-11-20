layout: post
title: 七、springBoot 简单优雅是实现文件上传和下载
author: QuellanAn
categories: 
  - springBoot
tags:
  - springboot
  - java
---
# 前言
好久没有更新spring Boot 这个项目了。最近看了一下docker 的知识，后期打算将spring boot 和docker 结合起来。刚好最近有一个上传文件的工作呢，刚好就想起这个脚手架，将文件上传和下载整理进来。

# 配置
在application.properties 中增加上传文件存放的路径配置
```
#文件上传目录
file.upload.url=E:/test
```

# controller 层
上传文件和下载文件都比较简单，我们就直接在controller层来编写。也不用在pom.xml 中增加什么依赖。所以直接上代码。
在controller 包下创建一个file包，在file 包下创建一个FileController 类。
```
@RestController
@RequestMapping("file")
@Slf4j
public class FileController {
    @Value("${file.upload.url}")
    private String uploadFilePath;
		
		@RequestMapping("/upload")
    public String httpUpload(@RequestParam("files") MultipartFile files[]){
        JSONObject object=new JSONObject();
        for(int i=0;i<files.length;i  ){
            String fileName = files[i].getOriginalFilename();  // 文件名
            File dest = new File(uploadFilePath  '/'  fileName);
            if (!dest.getParentFile().exists()) {
                dest.getParentFile().mkdirs();
            }
            try {
                files[i].transferTo(dest);
            } catch (Exception e) {
                log.error("{}",e);
                object.put("success",2);
                object.put("result","程序错误，请重新上传");
                return object.toString();
            }
        }
        object.put("success",1);
        object.put("result","文件上传成功");
        return object.toString();
    }

}
```
上面的代码看起来有点多，其实就是一个上传的方法，首先通过 MultipartFile 接收文件。这里我用的是file[] 数组接收文件，这是为了兼容多文件上传的情况，如果只用file 接收，然后在接口上传多个文件的话，只会接收最后一个文件。这里大家注意一下。看自己的需求，我这里兼容多文件所以用数组接收。

然后遍历files 获取文件，下面这段代码是判断文件在所在目录是否存在，如果不存在就创建对应的目录。
```
 File dest = new File(uploadFilePath  '/'  fileName);
            if (!dest.getParentFile().exists()) {
                dest.getParentFile().mkdirs();
            }
```

```
 files[i].transferTo(dest);
```
就是将文件存放到对应的服务器，这里有一点需要说明一下，如果我们上传重复的文件会怎么样么？上传重复的文件不会报错，后上传的文件会直接覆盖已经上传的文件。

整体代码就是这样。现在就可以实现文件的上传操作。

# 测试
我们写好之后，基本上传功能就已经实现了，我们现在来测试一下。启动项目后我们用postman 请求，因为我们需要上传文件，用get 方式请求不了。
![file](https://img-blog.csdnimg.cn/20191028180801278.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
![file](https://img-blog.csdnimg.cn/20191028180801508.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

可以看到文件上传成功了，由此可见，springboot文件上传一个方法就搞定了。

# 文件下载
其实文件下载，不太建议用接口做，因为文件下载一般都是下载一些静态文件，我们可以先将文件处理好，然后通过Nginx 服务下载静态文件，这样速度会快很多。但是这里我们还是写一下。代码也很简单，就一个方法，也写在fileController 类中

```
 @RequestMapping("/download")
    public String fileDownLoad(HttpServletResponse response, @RequestParam("fileName") String fileName){
        File file = new File(downloadFilePath  '/'  fileName);
        if(!file.exists()){
            return "下载文件不存在";
        }
        response.reset();
        response.setContentType("application/octet-stream");
        response.setCharacterEncoding("utf-8");
        response.setContentLength((int) file.length());
        response.setHeader("Content-Disposition", "attachment;filename="   fileName );

        try(BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file));) {
            byte[] buff = new byte[1024];
            OutputStream os  = response.getOutputStream();
            int i = 0;
            while ((i = bis.read(buff)) != -1) {
                os.write(buff, 0, i);
                os.flush();
            }
        } catch (IOException e) {
            log.error("{}",e);
            return "下载失败";
        }
        return "下载成功";
    }

```
 代码也很简单，就是根据文件名判断是否存在文件，不存在就提示没有文件，存在就将文件下载下来。response设置返回文件的格式，以文件流的方式返回，采用utf-8 字符集，设置下载后的文件名。然后就是以文件流的方式下载文件了。
 
 测试的话也简单，我们启动项目，访问接口
 ```
 http://localhost:9090/zlflovemm/file/download?fileName=11
 http://localhost:9090/zlflovemm/file/download?fileName=1.rar
 ```
 ![file](https://img-blog.csdnimg.cn/20191028180801787.jpeg)
 ![file](https://img-blog.csdnimg.cn/20191028180800424.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
 可以看到如果文件存在，会直接下载，不会提示下载成功或者失败。
 
 # 删除文件
 
 删除文件是很简单的，我这里讲一下删除文件下所有文件夹和文件。并做一个定时任务，每天清理一次。
 ```
 
    @Scheduled(cron="0 0 3 * * ?")
    private void deleteFiles(){
        deleteFile(new File(deleteFilePath));
    }

    public void deleteFile(File file){
        //判断文件不为null或文件目录存在
        if (file == null || !file.exists()){
            log.info("暂无文件");
            return;
        }
        //取得这个目录下的所有子文件对象
        File[] files = file.listFiles();
        //遍历该目录下的文件对象
        for (File f: files){
            //打印文件名
            String name = f.getName();
            log.info(name);
            //判断子目录是否存在子目录,如果是文件则删除
            if (f.isDirectory()){
                deleteFile(f);
            }else {
                f.delete();
            }
        }
        //删除空文件夹  for循环已经把上一层节点的目录清空。
        file.delete();
    }

 ```
 
 # 番外
到此为止，我们常用的镜像和容器的操作就会使用啦。都是一些命令。忘记的可以--help 查看一下。

好了，就说这么多啦
代码上传到github：
https://github.com/QuellanAn/zlflovemm

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤

![file](https://img-blog.csdnimg.cn/20191015213334732.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)




