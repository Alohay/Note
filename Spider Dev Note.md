#### 项目笔记

- 爬虫，所以有一次处理大量数据的机会

- 遇到的问题
  - 线程池 `cachedThreadPool` `newFixedThreadPool` 
    - 线程开太多也不行，线程池每10分钟换一组`ip`，线程太多会被源网站`ban`掉
    - 发现不是 ↑ 问题， 是代理池不够稳定，本意通过日志记录及分析出错的请求包找出出错原因，代理设为本机`127.0.0.1` 用 抓包工具查看请求包，然后就好了

  - 两个需求，数据可以从同一页面爬取，开始时将两者数据一并抓取并存入数据库，确实在一次请求中能将两个需求的数据处理成功

  - 有一个需求，抓取历史交易记录，数据量特别大，使用`insert`插入多条，连接多个括号，发现会数据库请求超时
    - 尝试减少数据库插入线程， 认为 `insert ignore` 太慢了，搜索得知**`load data infile`会快一些，而且用 `java inputStream` 省去 存文件过程（有时间了解一下如何实现的）**，  利用页面本身的跨时间查询（因为历史交易记录），减少单次插入数据量，减少代码中插入线程，猜测数据库连接池太小了，扩大容量，顺便加入了连接池日志记录，对日志分析的时候，发现默认10条线程均为忙碌状态
    - 开启对 `hikari`连接池 `DEBUG`模式的日志记录后，发现会产生 `Connection leak`连接泄露，发现没有释放数据库连接资源
    - **解决了！！ 代码中 `Connection` 资源没有释放**

  - `https://coincheckup.com/data/prod/201806070554/coins.json ` 中间有一段`url`随日期变化且有随机数
    - 分析页面，发现定义在前端，使用 `jsoup` 解析并提取该值

  - 发现切换代理次数过于频繁， 减小次数

    - 带来的一个问题是，如果某个 代理质量较差， 线程池中的任务无法及时反应， 目前设定 100 次请求换一个 代理

- `Github Token 726ce1e730645af6370d8d30f8c8e16a82250abf `

- ``` sql
  {
    repository(owner: "bitcoin", name: "bitcoin") {
      watchers {
        totalCount
      }
      stargazers {
        totalCount
      }
      forkCount
      openPullRequests: pullRequests(states: OPEN) {
        totalCount
      }
      closedPullRequests: pullRequests(states: CLOSED) {
        totalCount
      }
      openIssues: issues(states: OPEN) {
        totalCount
      }
      closedIssues: issues(states: CLOSED) {
        totalCount
      }
    }
  }
  
  ```

- `github address` 

  - 首先要统计 一个 项目下的子项目， 可能有分页

    - 难点， 数据库中给出的 可能为 组织项目，个人项目，组织项目或者个人项目下的子项目
      - 分析 可以数 `/` 个数， 舍弃第 `4` 个之后的字符
    - 统一处理 `url`，加上所有需要的参数，多了不影响，页码 从 `1` 递增，循环遍历
      - 请求的之前，不能通过`url`进行判断到底为哪种类型，分析页面发现，需要取的元素，都是对应类型唯一的，组织项目页面上没有 个人项目那个元素， 反之一样，所以 处理方法为 先取 组织项目的 元素，没有就取 个人项目的元素，没有 就 没了，返回 告知外部循环跳出
    - 组织项目 `organization`
      - 选取 某个` class `下` li`
    - 个人项目 `user`
      - 选取 某个` id` 下` li`
  - 共要统计约 `5000` 个项目信息，每个项目需要请求 api 一次， 请求 相应页面一次， 由于两次请求的信息是 充实到同一条记录里 故 串行执行
    - 一开始全部查出放在内存里处理，但是任务太多， 后面的任务会遇到 代理池ip 切换，故导致线程池中的任务由于代理不可达而失败
      - 单次查出100条，放入线程池后 睡 `2min` （先设定睡了5分钟，发现任务会有3分钟的空窗期而确定），减少任务积压，尽量保证 在代理过期之前将任务执行完毕


