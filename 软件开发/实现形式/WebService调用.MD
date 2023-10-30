## 背景

* 业务场景：对接三方系统，通过WebService方式发送xml格式数据到指定接口。
* 技术选型：在springboot项目中，利用hutool工具包可便捷实现该需求

## 正文

### 利用Hutool
#### pom配置
```xml
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.8.11</version>
        </dependency>
```
    注意这里用的hutool-all包，项目内原本有hutool工具但只有一部分，改了一下这里。 
此处参照[这篇博客](https://blog.csdn.net/qq_39148846/article/details/130687790#google_vignette)。
#### java代码实现
```java
/**
 * 发送数据到指定接口
 * url: http://www.webxml.com.cn/WebServices/WeatherWebService.asmx
 * param: <![CDATA[
 <nebulaData regID="3a6ea7084d1" userID="test">
 <data type="Opt">
 <row>
 <col gb="" hosp="DOC_NO">23716</col>
 <col gb="" hosp="GENDER_CODE">1</col>
 <col gb="" hosp="CREATE_CODE">1234</col>
 <col gb="" hosp="CREATE_DATE">2025-10-30 13:59:49</col>
 <col gb="" hosp="MODIFY_DATE">2025-10-30 13:59:49</col>
 </row>
 </data>
 </nebulaData>
 ]]>
 */
private static void send(String url,String param){
    new Thread(()->{
        String ans = "";
        try {
            SoapClient client = SoapClient.create(url, SoapProtocol.SOAP_1_1, "urn:hl7-:v3")
                    .setMethod("urn:AbcData", "urn:hl7-org:v3")
                    .setParam("xml", param, false);
            log.info("WebService入参:{}", client.getMsgStr(true));
            ans = client.send(false);
        } catch (Exception e) {
            log.info("webService 推送出错");
        }
        if (!ans.contains("xml")) {
            // 返回数据格式不对
            log.info("接口调用错误！" + ans);
            return;
        }
        if (ans.contains("result type=\"True\"")) {
            log.info("WebService推送成功");
        } else if (ans.contains("result type=\"False\"")) {
            log.info("WebService推送失败");
        }
        log.info("WebService返回信息：");
        log.info(ans.substring(ans.indexOf("![CDATA[") + 8, ans.indexOf("]]")));
    }).start();
}
```
本用法参照文章：[Java使用Hutool调用WebService接口详解](https://blog.csdn.net/zpwvip/article/details/127487596#三、示例)。

    注意：此处形参param中末尾拼接两个右方括号]]在实际使用时会报错，实际测试中只拼写一个，在创建SoapClient后打印其参数就正常不会报错了。

#### 参数拼接
拼接形参param的示例代码如下：

```java
private static String packingWebService(String userId, String type, String colsAndData){
    StringBuilder ans=new StringBuilder();
    ans.append("<![CDATA[\n");
    ans.append("<nebulaData regID=\"3a6ea7084d1\" userID=\"").append(userId).append("\">\n");
    ans.append("  <data type=\"").append(type).append("\">\n");
    ans.append(colsAndData);
    ans.append("  </data>\n");
    ans.append("</nebulaData>\n");
    ans.append("]>\n");
    return ans.toString();
}
```
