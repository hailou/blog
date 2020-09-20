title: java递归list变成树结构的形式
categories: 递归处理JSON
tags:

  - 递归
date: 2020-09-20 19:32:25

---

###### 实体类

```}
public class ObjectJSONFormat {

    private String id;
    private String name;
    private String type;
    private String parentId;

    private List<ObjectJSONFormat> children = new ArrayList<>();
    
}
```

###### 树结构的工具类

```
/**
 * @author hailou
 * @Description 树结构的工具类
 * @Date on 2020/9/17 9:27
 */
public class ObjectJSONFormatTree {

    private List<ObjectJSONFormat> list = new ArrayList<>();

    public ObjectJSONFormatTree(List<ObjectJSONFormat> list) {
        this.list = list;
    }

    //建立树形结构
    public List<ObjectJSONFormat> builTree() {
        List<ObjectJSONFormat> tree = new ArrayList<>();
        for (ObjectJSONFormat node : getRootNode()) {
            node = buildChilTree(node);
            tree.add(node);
        }
        return tree;
    }

    //递归，建立子树形结构
    private ObjectJSONFormat buildChilTree(ObjectJSONFormat pNode) {
        List<ObjectJSONFormat> childrens = new ArrayList<>();
        for (ObjectJSONFormat node : list) {
            if (node.getParentId().equals(pNode.getId())) {
                childrens.add(buildChilTree(node));
            }
        }
        pNode.setChildren(childrens);
        return pNode;
    }

    //获取根节点
    private List<ObjectJSONFormat> getRootNode() {
        List<ObjectJSONFormat> rootLists = new ArrayList<>();
        for (ObjectJSONFormat node : list) {
            if (node.getParentId().equals("0")) {
                rootLists.add(node);
            }
        }
        return rootLists;
    }

}
```

###### 测试

```
//业务代码获取formatList，这里直接new一个，生产是获取的
List<ObjectJSONFormat> formatList = new ArrayList<>();
formatList.add(...) //若干ObjectJSONFormat
//创建树
ObjectJSONFormatTree formatTree = new ObjectJSONFormatTree(formatList);
formatList = formatTree.builTree();
String jsonOutput = JacksonUtils.getInstance().pojo2Json(formatList);
System.out.println(jsonOutput);
```