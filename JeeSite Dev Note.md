##### 框架选型

- 开发时间比较紧迫，所以选择开源框架
- `JEPLUS` 
  - 优点 能生成 安卓及`IOS`端代码，但是我们有这方面的开发人员，不需要
- `AOSuite` `JeeCG`  `JeeSite` 
  - 就界面样式`JeeSite`更漂亮，我们一些需求点基本都已经覆盖到了，了解到有导师使用过，确定`JeeSite`了



##### 活动签到时间

- 用户是否关心自己的签到状态
  - 关心（本系统）
    - 必须实时给出签到反馈：签到成功，迟到，已结束
    - 一个活动有多次签到，防止早退
    - 活动时间跨度较长，上午下午，都要签到
    - 增加一张额外的表，记录签到时间区间
    - 签到状态：未开始；成功；重复签到；本次已结束，下轮未开始；签到时间已过
  - 不关心
    - 不设定活动签到时间区间，最后管理员查看时，通过查询来区分出多少人或者谁迟到



##### 数据库优化

- **表结构优化**
  - 活动详情表，活动状态，由`enum`改成另一张状态表的外键，便于活动状态的增加或修改，提高可扩展性
- **`SQL`查询优化**

``` sql
# 全表扫描, filesort, 临时表（temporary）, 嵌套循环
select
          a.activity_name,
          a.activity_status,
          count(b.id)
        from act_activity_detail a left join act_activity_attendance b
            on a.id = b.activity_detail_id and b.appeared_status = 1
        group by a.activity_name, a.activity_status
        
# left join 改成子查询, 运用索引(id, phone), b+树结构, 先按第一个字段排序, 再按第二个字段排序
select
          activity_name,
          activity_status,

          (select count(*)
           from act_activity_attendance b
           where b.activity_detail_id = a.id and b.appeared_status = 1)

        from act_activity_detail a
```

- **子查询慢查优化** 
  - 上文优化后的`SQL`带上排序，筛选条件后超时
  - `exlpain`发现使用了全表扫描 `filesort` 而子查询 `DEPENDENT SUBQUERY` 依赖父查询的查询结果
  - 尝试将子查询改成与临时表链接查询，那要用到`LEFT JOIN`，签到人数为`0`的话，子查询结果集中不会出现活动相应的`id`
  - 拆开来查询，业务要求查询出活动详情及其相关的签到人数
  - `reference:`[慢查优化慎用MySQL子查询，尤其是看到DEPENDENT SUBQUERY标记时](https://www.cnblogs.com/zhengyun_ustc/p/slowquery3.html) ，[MySQL · 性能优化 · 条件下推到物化表](http://mysql.taobao.org/monthly/2016/07/08/)



#####  业务

- 创建活动，成功后返回一个二维码，二维码的内容是一个关于该活动的签到界面
- 生成文件路径问题
  - 在`main`方法中生成和`web`项目中生成有差别，`web`项目`java`文件结构和编译出的`target/**/*.class`文件结构又有所不同
  - `web`项目中需要`request.getSession().getServletContext().getRealPath("/")`获取当前项目根路径的绝对路径，也就是如果打成`war`包的根路径，是可以直接对应到浏览器的根路径（域名加上项目名称，以`Tomcat`举例，如果`war`包叫`migu.war`，那么放到`webapp`文件夹内，`Tomcat`启动后会自动解压成`migu`文件夹，那么访问的时候，浏览器地址栏域名根路径即对应到`webapp/`，所以访问我们自己的项目需要在`URL`中加上`migu`关键字）的
- 扫码问题
  - 我们生成的文件是一张二维码图片，二维码的内容是一串网址，也就是说，我们拿手机扫码后，就会跳转到该`URL`，而我们的项目部署在公司内网，手机如果是数据的情况下是无法访问的，于是向申请做了一个端口映射，将`8080`映射到一台公网服务器的端口上，这样二维码的`URL`直接指向该公网服务器
  - 随之而来的问题就是，我们是无法通过`Java`代码去获取到映射后的`ip:port`，所以我选择了在配置文件中配置该公网`ip:port`，实际表现为和具体部署的`ip`不存在耦合关系，二维码内容直接读取配置文件中的域名，路径是固定的，正确对应到签到页面`Controller`上，返回签到界面
- 非登录状态访问问题
  - 项目使用`JeeSite`快速开发框架，其中集成了`Shiro`权限框架，如果用户处于非登录状态，是无法访问签到界面的，这与我们实际业务不相符，拥有账号的应当是活动创建人，参与活动的人员不一定需要账号，也就是说，签到界面要求不登录也可以访问，那就需要在过滤器中把访问签到页面的请求放过去，不做拦截，网上查询了解到，`shiro `在配置文件中定义了权限过滤器，在`<constructor-arg>`的`<value>`标签中，可以配置指定路径使用指定的过滤器，我们只要将指定的`Controller`请求过滤器指定为`anon`即匿名过滤器，但是当我配置完成之后，我发现还是会拦截并跳转到登录界面，后来算是根据经验吧，猜测这个`xml`文件的解析流程，应该是存在一个顺序，从上往下匹配，如果路径匹配完成，那么就走相应的过滤器，所以我需要把我自定义的配置往上调，然后问题解决



##### 数据库

- 注意 `enum`类型存储的是 `'1'`字符，而不是 数字`1`

##### 扫码

- 奇怪的问题，浏览器（设置了`UA=PC`）扫描二维码可以成功跳转，微信扫描扫描却报错`404`
- `Jeesite`有一个`mobileInterceptor`拦截器实现了`postHandle`，通过`UA`判断，如果是手机则在`view`前加上一个`mobile/`路径