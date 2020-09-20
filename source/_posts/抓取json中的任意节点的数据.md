title: 抓取json中的任意节点的数据
categories: 递归处理JSON
date: 2020-09-20 1909:04
tags:
  - 递归
---

###### 依赖的jar

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.4.1</version>
</dependency>
```

工具类

```
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.IOException;
import java.io.InputStream;
import java.io.StringWriter;
import java.text.SimpleDateFormat;
import java.util.List;
import java.util.Map;

/**
 * @author hailou
 * @Description
 * @Date on 2020/9/16 15:57
 */
public class JacksonUtils {

    private static final ObjectMapper MAPPER = new ObjectMapper();
    private static final JacksonUtils CONVERSION = new JacksonUtils();
    private static final String SEPARATOR = "\\.";

    private JacksonUtils() {
        init();
    }

    private static void init() {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSSXXX");
        MAPPER.setDateFormat(dateFormat);
        MAPPER.setSerializationInclusion(JsonInclude.Include.NON_NULL);
    }


    public static JacksonUtils getInstance() {
        return CONVERSION;
    }

    public String pojo2Json(Object obj) throws IOException {
        StringWriter writer = new StringWriter();
        MAPPER.writeValue(writer, obj);

        return writer.toString();
    }

    public <T> T json2Pojo(InputStream jsonStream, Class<T> classType) throws IOException {
        T t = null;

        try (InputStream inputStream = jsonStream) {
            t = MAPPER.readValue(inputStream, classType);
        }

        return t;
    }

    public <T> T json2Pojo(String json, Class<T> classType) {
        try {
            return MAPPER.readValue(json, classType);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }

    }

    /**
     * 将JSON转化为List
     *
     * @param json 字符串
     * @param elementClasses 元素类
     * @return List 转换后的对象
     */
    public static List<?> jsonConverList(String json, Class<?>... elementClasses) {
        try {
            return MAPPER.readValue(json, getCollectionType(List.class, elementClasses));
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 获取泛型的Collection Type
     *
     * @param collectionClass 泛型的Collection
     * @param elementClasses 元素类
     * @return JavaType Java类型
     */
    public static JavaType getCollectionType(Class<?> collectionClass, Class<?>... elementClasses) {
        return MAPPER.getTypeFactory().constructParametricType(collectionClass, elementClasses);
    }

    public <T> T map2Pojo(Map<String, Object> map, Class<T> classType) throws IOException {
        return MAPPER.convertValue(map, classType);
    }

    public <T> T jsonParse(String json) throws IOException {
        return MAPPER.readValue(json, new TypeReference<T>() {});
    }

    /**
     * path: Like xpath, to find the specific value via path. Use :(Colon) to separate different key
     * name or index. For example: JSON content: { "name": "One Guy", "details": [
     * {"education_first": "xx school"}, {"education_second": "yy school"}, {"education_third":
     * "zz school"}, ... ] }
     *
     * To find the value of "education_second", the path="details:1:education_second".
     *
     * @param json
     * @param path
     * @return
     */
    public String fetchValue(String json, String path) {
        JsonNode tempNode = null;
        try {
            JsonNode jsonNode = MAPPER.readTree(json);
            tempNode = jsonNode;
            String[] paths = path.split(SEPARATOR);

            for (String fieldName : paths) {
                if (tempNode.isArray()) {
                    tempNode = fetchValueFromArray(tempNode, fieldName);
                } else {
                    tempNode = fetchValueFromObject(tempNode, fieldName);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
        if (tempNode != null) {
            String value = tempNode.asText();

            if (value == null || value.isEmpty()) {
                value = tempNode.toString();
            }
            return value;
        }
        return null;
    }

    private JsonNode fetchValueFromObject(JsonNode jsonNode, String fieldName) {
        return jsonNode.get(fieldName);
    }

    static List<String> restList;

    private JsonNode fetchValueFromArray(JsonNode jsonNode, String index) {
        try{
            return jsonNode.get(Integer.parseInt(index));
        }catch (Exception e){
            restList = jsonNode.findValuesAsText(index) ;
            return null;
        }
        //return jsonNode.get(Integer.parseInt(index));
    }

    public JsonNode convertStr2JsonNode(String json) {
        try {
            return MAPPER.readTree(json);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    public static void main(String[] args) {
        String aa = "{\n" +
                "\t\"org\": {\n" +
                "\t\t\"id\": [\"aa\", \"bb\"],\n" +
                "\t\t\"name\": \"机构树\",\n" +
                "\t\t\"children\": [{\n" +
                "\t\t\t\"name\": \"信息办\",\n" +
                "\t\t\t\"children\": [{\n" +
                "\t\t\t\t\"name\": \"网站组\",\n" +
                "\t\t\t\t\"age\": \"24\"\n" +
                "\t\t\t}, {\n" +
                "\t\t\t\t\"name\": \"OA组\",\n" +
                "\t\t\t\t\"age\": \"24\"\n" +
                "\t\t\t}]\n" +
                "\t\t}, {\n" +
                "\t\t\t\"name\": \"造林司\",\n" +
                "\t\t\t\"children\": [{\n" +
                "\t\t\t\t\"name\": \"植树组\"\n" +
                "\t\t\t}, {\n" +
                "\t\t\t\t\"name\": \"退根组\"\n" +
                "\t\t\t}]\n" +
                "\t\t}]\n" +
                "\t}\n" +
                "}";

        //String res = JacksonUtils.getInstance().fetchValue(aa, "org:children:0:children:1");
        String res = JacksonUtils.getInstance().fetchValue(aa, "org.children");
        //String res = JacksonUtils.getInstance().fetchValue(aa, "org.id");
        System.out.println("res = " + res);
        System.out.println("restList = " + restList);
    }
}
```