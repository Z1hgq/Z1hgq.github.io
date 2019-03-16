---
layout: post
title:  "学生作业互评系统api"
---

### **登录**
> `address:/api/login`
> `method:post`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| userid | Number | 用户id |
| pw | String | 密码 |

{% highlight ruby %}
HTTP/1.1 200 OK
{
    code:1,//登录失败返回0
    data:{
        userid:2015111926,//学生或者老师的编号
        usertype:0,    //0表示老师，1表示学生
        username:'',//姓名
        gender:'',//性别
        school:'',//学院
        position:'',//职称,学生没有这条信息
        message:[
            {
                message_id:'',//消息id
                content:'',//消息内容
                state: 0, //消息状态,0表示未读,1表示已读
            },
            {
                message_id:'',//消息id
                content:'',//消息内容
                state: 0, //消息状态,0表示未读,1表示已读
            }
        ],//消息
    },
    token:''//返回的token
}
{% endhighlight %}

### **学生获取作业列表**

> `address:/api/get_homework_list`
> `method:get`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| userid | Number | 用户id |
{% highlight ruby %}
HTTP/1.1 200 OK
{
    code:1,//获取失败返回0
    data:{
        homework_to_be_done:[  //所有待完成的作业都放进这个数组
            {
                id:123,//教师上传作业编号
                name:'',//作业名称
                address:''//作业地址
                file_name: 'filename', //文件名字
                description:'',//作业描述
                create_time:'',//上传日期
                end_time:'',//截止日期
            },
            {
                id:123,//教师上传作业编号
                name:'',//作业名称
                address:''//作业地址
                file_name: 'filename', //文件名字
                description:'',//作业描述
                create_time:'',//上传日期
                end_time:'',//截止日期
            },
        ],
        homework_to_be_evalution:[//所有待测评的作业
            {
                id:123,//待测评的提交作业编号
                student_id:'',//该作业的学生编号
                name:'',//待测评作业文件名称
                file_name: 'filename', //文件名字
                address:''//待测评作业地址
            },
            {
                id:123,//待测评的提交作业编号
                student_id:'',//该作业的学生编号
                name:'',//待测评作业文件名称
                file_name: 'filename', //文件名字
                address:''//待测评作业地址
            },
            {
                id:123,//待测评的提交作业编号
                student_id:'',//该作业的学生编号
                name:'',//待测评作业文件名称
                file_name: 'filename', //文件名字
                address:''//待测评作业地址
            }
        ]
    }
}
{% endhighlight %}

### **提交分数**

> `address:/api/post_grade`
> `method:post`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| student_id | Number | 被批改学生学号 |
| homework_id | Number | 提交作业编号 |
| grade | Number | 分数 |

{% highlight ruby %}
HTTP/1.1 200 OK
{
    code:1,//上传失败返回0
}
{% endhighlight %}

### **老师查看作业情况**

> `address:/api/get_homework`
> `method:get`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| userid | Number | 用户id |

{% highlight ruby %}
HTTP/1.1 200 OK
{
    code:1,//获取失败返回0
    data:{
        homework:[
            {
                id:'',//教师上传作业编号
                name:'',//作业名称
                status:[
                    {
                        id:'',//学生提交作业编号
                        stu_name:'',//学生名字
                        stu_id:'',//学生编号
                        score:'',//得分
                        address:'',//作业地址
                        file_name: 'filename', //文件名字
                        evaluted:[]//三个评分人
                    },
                    {
                        id:'',//学生提交作业编号
                        stu_name:'',//学生名字
                        stu_id:'',//学生编号
                        score:'',//得分
                        address:'',//作业地址
                        file_name: 'filename', //文件名字
                        evaluted:[]//三个评分人
                    },
                ]
            }
        ],
        bad_homework:[//存在恶意评价的作业
            {
                id:'',//教师上传作业编号
                name:'',//作业名称
                status:[
                    {
                        id:'',//学生提交作业编号
                        stu_name:'',//学生名字
                        stu_id:'',//学生编号
                        address:'',//作业地址
                        file_name: 'filename', //文件名字
                        evaluted:'',//有恶意评价的学生的姓名
                        evaluted_id:''//有恶意评价的学生的编号
                    },
                    {
                        id:'',//学生提交作业编号
                        stu_name:'',//学生名字
                        stu_id:'',//学生编号
                        address:'',//作业地址
                        file_name: 'filename', //文件名字
                        evaluted:'',//有恶意评价的学生的姓名
                        evaluted_id:''//有恶意评价的学生的编号
                    },
                ]
            }
        ]
    }
}
{% endhighlight %}

### **老师上传作业**
> `address:/api/post_homework`
> `method:post`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| userid | Number | 用户id |
| name | String | 作业名称 |
| address | String | 作业地址 |
| description | String | 作业描述 |

{% highlight ruby %}
HTTP/1.1 200 OK
{
    code:1,//上传失败返回0
}
{% endhighlight %}

### **老师上传作业答案**
> `address:/api/post_answer`
> `method:post`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| homework_id | String | 作业id |
| answer_address | String | 答案地址 |

{% highlight ruby %}
HTTP/1.1 200 OK
{
    code:1,//上传失败返回0
}
{% endhighlight %}

### **文件上传**
> `address:/api/post_file`
> `method:post`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| userid | Number | 用户id |
| name | String | 作业名称 |
| description | String | 作业描述 |

{% highlight ruby %}
HTTP/1.1 200 OK
{
    code:1,//上传失败返回0
    data:{
        address:'',//文件地址
        file_name: 'filename', //文件名字
    }
}
{% endhighlight %}

### **发送消息**
> `address:/api/send_message`
> `method:post`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| userid | Number | 用户id |
| message_id| String | 消息id |
| content | String | 消息内容 |

{% highlight ruby %}
HTTP/1.1 200 OK
{
    code:1,//发送失败返回0
}
{% endhighlight %}

### **更新消息状态**
> `address:/api/update_message`
> `method:post`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| userid | Number | 用户id |
| message_id| String | 消息id |

{% highlight ruby %}
HTTP/1.1 200 OK
{
    code:1,//在读完消息之后将消息修改成已读状态,修改失败返回0
}
{% endhighlight %}





