title: JSON增加节点或者删除指定key的值
categories: 递归处理JSON
date: 2020-09-20 1909:04
tags:
  - 递归
---


###### 工具类

```
/**
 * @author hailou
 * @Description JSON增加节点或者删除指定key的值
 * @Date on 2020/9/18 11:09
 */

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;

import java.util.Iterator;
import java.util.Map;
import java.util.Set;
public class JsonAddOrRemoveUtils {
    public static JSONObject addOrRemoveNodeByKeys(String json,String[] removeKeys, Map<String,Object> addNodes) {
        JSONObject objTem=JSON.parseObject(json);
        JSONObject objRel=JSON.parseObject(json);
        return dealOrCreate(objTem,objRel,removeKeys,addNodes);
    }
    private static JSONObject dealOrCreate(JSONObject objTem, JSONObject objRel, String[] removeKeys, Map<String,Object> addNodes) {
        /**
        下面这段代码是批量增加节点时从当前的某个节点获取一个值塞进去
        如:addNodes.put("key1","type");type就是当明节点的的一个key
        把当前的key的值付给自定义节点的值，可能会做页面映射用
        */
        /*if(null != addNodes && !addNodes.keySet().isEmpty()){
            Set<String> keys = addNodes.keySet();
            Iterator<String> iteratorKeys = keys.iterator();
            while(iteratorKeys.hasNext()) {
                String key =  iteratorKeys.next();
                if(objTem.get(addNodes.get(key))!=null){
                    objRel.put(key,objTem.get(addNodes.get(key)));
                }
            }
        }*/

        /**
         * 这个就是把自定义节点插入进去，和上面代码取其一
         */
        if(null != addNodes && !addNodes.keySet().isEmpty()){
            objRel.putAll(addNodes);
        }

        Set<String> keySet = objTem.keySet();
        Iterator<String> iterator = keySet.iterator();
        while(iterator.hasNext()) {
            String temp =  iterator.next();//存key
            Object objR = objTem.get(temp);//存value
            for (int i = 0;  i<removeKeys.length ; i++) {
                if(removeKeys[i].equals(temp)) {
                    objRel.remove(temp);
                    continue;
                }
            }
            if(objR instanceof JSONObject) {
                JSONObject j=(JSONObject)objR;
                JSONObject object2 = (JSONObject)objRel.get(temp);
                dealOrCreate(j,object2,removeKeys,addNodes);
                continue;
            }
            if(objR instanceof JSONArray) {
                JSONArray jsonArray = objTem.getJSONArray(temp);
                JSONArray jsonArray2 = objRel.getJSONArray(temp);
                for(int i=0;i<jsonArray.size();i++) {
                   dealOrCreate(jsonArray.getJSONObject(i),jsonArray2.getJSONObject(i),removeKeys,addNodes);
                }
            }
        }
        return objRel;
    }
}

```

###### 测试

```
//原始json，注意这里的json只是对象开头的，如果是[]开头的会报错
String json = "{\"aa\":[{\"name\":\"aaa\",\"type\":\"list\",\"parentId\":\"0\",\"seat\":\"1\",\"children\":[{\"name\":\"bbb\",\"type\":\"base\",\"parentId\":\"1\",\"seat\":\"1-1\",\"children\":[]},{\"name\":\"ccc\",\"type\":\"base\",\"parentId\":\"1\",\"seat\":\"1-2\",\"children\":[]}]},{\"name\":\"ddd\",\"type\":\"map\",\"parentId\":\"0\",\"seat\":\"2\",\"children\":[{\"name\":\"eee\",\"type\":\"base\",\"parentId\":\"2\",\"seat\":\"2-1\",\"children\":[]},{\"name\":\"fff\",\"type\":\"map\",\"parentId\":\"2\",\"seat\":\"2-3\",\"children\":[{\"name\":\"ggg\",\"type\":\"base\",\"parentId\":\"2-3\",\"seat\":\"2-3-1\",\"children\":[]}]}]}]}";
System.out.println("json = " + json);
//要删除的节点
String[] removeNames = {"seat","parentId"};
//调用删除方法
Map<String,Object> addNodes = new HashMap<>();
addNodes.put("key1","key1-value");
addNodes.put("key2","key2-value");
JSONObject node = JsonAddOrRemoveUtils.addOrRemoveNodeByKeys(jsonOutput,removeNodeNames,addNodes);
System.out.println("jsonOutput = " + node);
```