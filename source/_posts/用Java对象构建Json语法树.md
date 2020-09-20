title: 用Java对象构建Json语法树
categories: 递归处理JSON
date: 2020-09-20 1909:04
tags:
  - 递归
---

```
/**
 * @author hailou
 * @Description
 * @Date on 2020/9/18 16:30
 */

import java.util.Collections;
import java.util.LinkedList;
import java.util.List;

public class Json implements Comparable<Json> {
    // There are value types
    private int Type_String = 1;
    private int Type_Array = 2;
    private int Type_List = 3;
    // Key always is String
    private String key;
    // There are three types of value
    private int valueType;
    private String valueString;
    private List<Json> valueArray;// 本质一致，表现不同
    private List<Json> valueList;
    // indent depth
    private int depth;

    public Json(String key, String value) {
        this.key = key;
        this.valueType = Type_String;
        this.valueString = value;
        this.depth = 0;
    }
    public Json(String key, int type) {
        this.key = key;

        if (type == Type_List) {
            this.valueType = Type_List;
            this.valueList = new LinkedList<Json>();
        } else if (type == Type_Array) {
            this.valueType = Type_Array;
            this.valueArray = new LinkedList<Json>();
        }
    }

    public void addJsonToList(Json json) {
        if (valueList != null) {
            valueList.add(json);

            adjustDepth();
        }
    }

    public void addJsonToArray(Json json) {
        if (valueArray != null) {
            valueArray.add(json);

            adjustDepth();
        }
    }

    private void adjustDepth() {
        if (valueType == Type_List) {
            for (Json json : valueList) {
                json.depth = this.depth + 1;
                json.adjustDepth();
            }
        }

        if (valueType == Type_Array) {
            for (Json json : valueArray) {
                json.depth = this.depth + 1;
                json.adjustDepth();
            }
        }
    }

    public String toString() {
        StringBuilder sb = new StringBuilder();
        // key
        String tabs = getIndentSpace();
        sb.append(tabs);
        //sb.append("\""+(key==null?"":key)+"\"");
        if (key != null) {
            sb.append("\"" + key + "\"");
            sb.append(":");
        } else {

        }
        // value
        if (valueType == Type_String) {
            sb.append("\"" + valueString + "\"");
        } else if (valueType == Type_Array) {
            sb.append("[\n");
            int n = valueArray.size();
            for (int i = 0; i < n; i++) {
                Json json = valueArray.get(i);
                if (i != n - 1) {
                    sb.append(json.toString() + ",\n");
                } else {
                    sb.append(json.toString() + "\n");
                }
            }
            sb.append(tabs + "]");
        } else if (valueType == Type_List) {

            sb.append("{\n");
            Collections.sort(valueList);
            int n = valueList.size();

            for (int i = 0; i < n; i++) {
                Json json = valueList.get(i);
                if (i != n - 1) {
                    sb.append(json.toString() + ",\n");
                } else {
                    sb.append(json.toString() + "\n");
                }
            }
            sb.append(tabs + "}");
        }
        return sb.toString();
    }

    public int compareTo(Json other) {
        return this.key.compareTo(other.key);
    }

    private String getIndentSpace() {
        return String.join("", Collections.nCopies(this.depth, "    "));
    }

    public static void main(String[] args) {
        Json id1 = new Json("id", "001");
        Json name1 = new Json("name", "白菜");

        Json title = new Json("title", 3);
        title.addJsonToList(id1);
        title.addJsonToList(name1);

        Json empty1 = new Json(null, 3);
        empty1.addJsonToList(new Json("id", "001"));
        empty1.addJsonToList(new Json("id", "你好白菜"));

        Json empty2 = new Json(null, 3);
        empty2.addJsonToList(new Json("id", "001"));
        empty2.addJsonToList(new Json("id", "你好萝卜"));

        Json content = new Json("content", 2);
        content.addJsonToArray(empty1);
        content.addJsonToArray(empty2);

        Json data = new Json("data", 3);
        data.addJsonToList(title);
        data.addJsonToList(content);

        Json status = new Json("status", "0000");
        Json message = new Json("message", "success");

        Json root = new Json(null, 3);
        root.addJsonToList(status);
        root.addJsonToList(message);
        root.addJsonToList(data);

        System.out.println(root.toString());
    }
}
```