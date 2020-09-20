title: Java删除数据库中的树形数据及下面的子数据
categories: 递归处理JSON
date: 2020-09-20 1909:04
tags:
  - 递归
---
    
```
  	public void deleteContentCategory(Long parentId, Long id) {
        // 声明存放需要删除的节点的容器
        List<Object> ids = new ArrayList<>();
        // 把自己的id放到ids中
        ids.add(id);
        // 使用递归获取所有需要删除的节点的id
        this.getIds(id, ids);
        // 使用父类的方法，批量删除
        super.deleteByIds(ids);
        // 查询兄弟节点，声明查询条件
        ContentCategory param = new ContentCategory();
        param.setParentId(parentId);
        // 执行查询
        int count = super.queryCountByWhere(param);
        // 判断是否没有兄弟节点
        if (count == 0) {
            // 如果没有兄弟节点
            ContentCategory parent = new ContentCategory();
            parent.setId(parentId);
            // 修改父节点的isParent为false
            parent.setIsParent(false);
            // 执行修改
            super.updateByIdSelective(parent);
        }
        // 如果还有兄弟节点，神马都不做
    }

    // 使用递归的方式查询所有的子节点的id
    private void getIds(Long id, List<Object> ids) {
        // 根据条件查询当前节点的所有的子节点
        ContentCategory param = new ContentCategory();
        param.setParentId(id);
        List<ContentCategory> list = super.queryListByWhere(param);
        // 使用递归的方式，必须设置递归的停止条件，否则会一直自己调用自己，直到内存溢出
        // 判断是否还有子节点
        if (list.size() > 0) {
            // 如果有子节点，遍历结果集
            for (ContentCategory son : list) {
                // 1.把子节点的id放到ids容器中
                ids.add(son.getId());
                // 2.执行递归，自己调用自己，查询子节点的子
                this.getIds(son.getId(), ids);
            }
        }
    }
```

