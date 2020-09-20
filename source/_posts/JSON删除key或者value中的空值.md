title: JSON删除key或者value中的空值
categories: 递归处理JSON
date: 2020-09-20 1909:04
tags:
  - 递归
---

###### maven中的依赖

```
<!-- 阿里fastjson包JSON转换-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>
```

工具类

```
/**
 * @author hailou
 * @Description JSON删除key或者value中的空值
 * @Date on 2020/9/18 11:09
 */
import java.util.Iterator;
import java.util.Set;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
public class JsonDealUtils {
    public static JSONObject getNoNullValue(String json) {
        JSONObject objTem=JSON.parseObject(json);
        JSONObject objRel=JSON.parseObject(json);
        return deal(objTem,objRel);
    }
    private static JSONObject deal(JSONObject objTem,JSONObject objRel) {
        Set<String> keySet = objTem.keySet();
        Iterator<String> iterator = keySet.iterator();
        while(iterator.hasNext()) {
            String temp =  iterator.next();//存key
            Object objR = objTem.get(temp);//存value
            if(temp==null||"".equals(temp)||"null".equals(temp)) {
                objRel.remove(temp);
                continue;
            }
            if(objR==null||"".equals(objR.toString())||"null".equals(objR.toString())||"[]".equals(objR.toString())||"{}".equals(objR.toString())) {
                objRel.remove(temp);
                continue;
            }
            if(objR instanceof JSONObject) {
                JSONObject j=(JSONObject)objR;
                JSONObject object2 = (JSONObject)objRel.get(temp);
                deal(j,object2);
                continue;
            }
            if(objR instanceof JSONArray) {
                JSONArray jsonArray = objTem.getJSONArray(temp);
                JSONArray jsonArray2 = objRel.getJSONArray(temp);
                for(int i=0;i<jsonArray.size();i++) {
                    deal(jsonArray.getJSONObject(i),jsonArray2.getJSONObject(i));
                }
            }
        }
        return objRel;
    }
}
```