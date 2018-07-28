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



##### 登录流程

- `FormAuthenticationFilter.createToken(request, response)`
  - 通过解析`request`获取用户名密码，设置到返回的`token`中
- 调用`SecurityUtils.getSubject().login(token)`
  - 没抛`AuthenticationException` 就登陆成功
  - 目前先自己手动控制跳转页面吧



##### SQL查询优化

``` sql
# 全表扫描, filesort, 临时表（temporary）, 嵌套循环
select
          a.activity_name,
          a.activity_status,
          count(b.id)
        from act_activity_detail a left join act_activity_attendance b
            on a.id = b.activity_detail_id and b.appeared_status = 1
        group by a.activity_name, a.activity_status
        
# 改成子查询, 运用索引(id, phone), b+树结构, 先按第一个字段排序, 再按第二个字段排序
select
          activity_name,
          activity_status,

          (select count(*)
           from act_activity_attendance b
           where b.activity_detail_id = a.id and b.appeared_status = 1)

        from act_activity_detail a



```

