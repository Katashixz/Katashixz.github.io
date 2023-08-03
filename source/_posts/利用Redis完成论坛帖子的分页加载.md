---
title: 利用Redis完成论坛帖子的分页加载
date: 2022-9-15 09:18:08
tags:
    - Redis
categories: 自主学习
cover: false
---
# 利用Redis完成论坛帖子的分页加载
## 核心思路
- 利用Redis的ZSet(SortedSet)和Hash做分页。
    - SortedSet会给每个添加的元素member绑定一个用于排序的值score，SortedSet就会根据score值的大小对元素进行排序，在论坛的应用场景中可以将发布日期当作score用于排序。在RedisTemplate中提供了相关的函数来进行分页取出操作。详情见下。
    - 在Hash里对应存储ID->帖子信息，在SortedSet中存储帖子ID，分页取出每个帖子ID后在Hash中直接根据ID找到对应的帖子信息，返回数组即可。


## 实现操作
1. 设置定时任务每天在访问低峰期更新数据。
   1. 使用@EnableScheduling开启基于注解的定时任务
   2. 使用@Scheduled搭配cron表达式设置时间
2. 加载1000条帖子到本地，用getTime()取到Long类型的时间数据作为排序依据，用postID作值。
```java
   //每天凌晨四点更新缓存，把数据库里前1000条更新到缓存里。(没人能这么闲看完这么多帖子吧)
   List<CommunityPostVo> communityPostVo = postMapper.loadPostWithPage(0,999);
   Map<String, Object> postMap = new HashMap<>();
   Set<ZSetOperations.TypedTuple<Integer>> tempSet = new HashSet<>();
   for (CommunityPostVo postVo : communityPostVo) {
      //先处理图片
      if (!Objects.isNull(postVo.getImageUrl()))
          postVo.setImageUrlList(postVo.getImageUrl().split("，"));
      //根据帖子发布时间进行排序
      ZSetOperations.TypedTuple<Integer> temp = new DefaultTypedTuple<>(postVo.getPostID(),Double.valueOf(postVo.getPostDate().getTime()));
      tempSet.add(temp);
      //方便根据帖子ID查找对应的帖子信息
      postMap.put(postVo.getPostID().toString(),postVo);
   }
```
3. 将数据存入Redis
```java
   redisTemplate.opsForZSet().add("postSet", tempSet);
   redisTemplate.opsForHash().putAll("postMap", postMap);
   redisTemplate.expire("postMap",1500, TimeUnit.MINUTES);
   redisTemplate.expire("postSet",1501, TimeUnit.MINUTES);
```
4. 在业务层根据传入的页面大小(pageSize)和当前页(curPage)取数据
```java
    @Override
    public Map<String ,Object> loadPost(Integer curPage, Integer pageSize) {
        //根据页号加载帖子
        Integer left = (curPage - 1) * pageSize;
        Integer right = curPage * pageSize - 1;
        log.info("加载数据 {}-{}",left,right);
        Map<String ,Object> result = new HashMap<>();
        //帖子数据是否加载完毕
        Boolean isEnd = false;
        try{
            Set<Integer> postSet = redisTemplate.opsForZSet().reverseRange("postSet", left, right);
            Map<String ,Object> postMap = redisTemplate.opsForHash().entries("postMap");
            List<CommunityPostVo> postVos = new ArrayList<>();
            for (Integer integer : postSet) {
                postVos.add((CommunityPostVo) postMap.get(integer.toString()));
            }
            //如果Redis里的数据加载到底了，就返回已全部加载(目前只允许浏览前1000条)
            if (postSet.size() < (right - left)) {
                isEnd = true;
            }
            result.put("postList", postVos);
            result.put("isEnd", isEnd);
        }catch (Exception e){
            log.error("帖子数据分页加载出现错误：");
            e.printStackTrace();
            return null;
        }


        // IPage<Post> iPage = postMapper.selectPa;
        return result;

    }
```

## 总结
1. 减少了对数据库的访问压力，但是要注意数据同步问题，以及一开始的缓存加载多少条、缓存访问完了之后怎么办需要定义其他函数解决。
2. 还要尽量减少对Redis的读写操作，数据加载到本地再进行筛选。