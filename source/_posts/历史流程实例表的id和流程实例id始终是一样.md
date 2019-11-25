title: 历史流程实例表的id和流程实例id始终是一样
date: 2019-11-21 10:36:40
categories: 开发总结
tags: 
	- activiti
---
##### 历史流程实例表的id和流程实例id始终是一样
---
历史流程实例表的id和流程实例id始终是一样的。所以Activiti没有提供获取流程实例id的接口；因为直接getId()获取的值和流程实例Id是一样的；

<!-- more -->

##### 代码如下
```
@Test
public void getHistoryProcessInstance(){
    HistoricProcessInstance hpi= processEngine.getHistoryService() // 历史任务Service
    .createHistoricProcessInstanceQuery() // 创建历史流程实例查询
    .processInstanceId("2501") // 指定流程实例ID
     .singleResult();
    System.out.println("流程实例ID:"+hpi.getId());
    System.out.println("创建时间："+hpi.getStartTime());
    System.out.println("结束时间："+hpi.getEndTime());
}
```
