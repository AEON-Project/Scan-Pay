# Scan-Pay
AEON supports frictionless crypto purchases across supermarkets and retail outlets in countries like Vietnam, Philippines, Nigeria, and Brazil.

## Introduction

We provide scan code analysis to enable merchants to create orders for users to purchase by tokens. The system displays the user's location and the payment methods supported in their country. Users can choose to pay in their local currency or use crypto payment.

Redirection is the fastest and easiest way to integrate with AEON. You can redirect your website or application to the AEON page by copying and editing the following sample code.

Concatenate parameters appId and qrCode to the frontend domain path. Entering the address in the browser will display the checkout.

## API Description

**Request Method:**`GET`

**Request Domain:** `https://qr.cryptogo.com/`

**Request Path:**`/scanCode`

Request Body Parameter:

| Parameter | length | Signature | Mandatory | Description    |
| :-------- | :----- | :-------- | :-------- | :------------- |
| appId     | 256    | Y         | Y         | appid          |
| qrCode    | 256    | Y         | Y         | QR code string |

Example Link:

```text
https://qr.cryptogo.com/scanCode?qrCode=0002010211****508&appId=TEST000001
```

Response Example:

![image](https://github.com/user-attachments/assets/a02563e1-9e37-49a4-851a-90aea668af0c)



<br />

### Regular Expressions for Fiat Currency QR Code Verification

- When processing QR code payments, it's often necessary to verify the content of the QR code based on the characteristics of different countries and currencies. 
- Regular expressions provide a means to efficiently parse QR codes and extract country and currency codes. This is because QR codes commonly encode this type of information. By utilizing regular expressions, you can accurately verify the origin and currency associated with a payment QR code.
- To facilitate quick identification, we've listed regular expressions for common fiat currencies below. These expressions can help us verify whether a QR code contains specific country or currency identification information.

Supported Fiat Currency Regular Expressions:

| Fiat Currency Name    | Regular Expression |
| --------------------- | ------------------ |
| Vietnamese Dong (VND) | `^.*704.*VN.*$`    |
| Philippine Peso (PHP) | `^.*608.*PH.*$`    |

## Quick Start

Before integration, you need to obtain customerNum and secret for technical integration. 

Please contact the business department to obtain the appid and secret for the production environment.

Before obtaining your production environment account, you can first connect to our sandbox environment. 

The common test environment account information is as follows:  
appid:`6fdbaac29eb94bc6b36547ad705e9293`  
appSecret: `6fdbaac29eb94bc6b36547ad705e9298`

## Environmental Information

Sandbox Environment: `https://aeon-qrpay-sbx.alchemytech.cc`  
Production Environment:`https://aeon-qrpay.alchemypay.org`

## IP Whitelist

For security, we need to whitelist your IP addresses before your integration.

Please provide your **egress IP addresses**.

## About the sign

**The secret key used to generate a signature for API request input parameters should be securely maintained by the merchant.**

To generate a signature string:

Collect all required parameters that need to be verified into an array.  
Sort the array alphabetically by parameter names (keys) in ascending ASCII order. It's important to note that only the parameter keys are sorted, not their values.  
Exclude the 'sign' parameter itself from the array before generating the signature.

### Example

#### Example request parameters

```json
{
    "appId": "TEST000001",
    "sign": "TEST000001",
    "merchantOrderNo": "11126"
}
```

Prepare the string concatenation for sign, convert the format to parameter_name=parameter_value, link with '&', and finally append the key (secret). The result is as follows:

```
appId=TEST000001&merchantOrderNo=11126&key=9999
```

Continuing from the prepared string for signing, perform the SHA-512 algorithm to generate the final signature.And change it into capital letters. The signature result is as follows:

```
  95960053CC577FCAFC272410D5F70094DD0986F6C3266DB7D00D0B37A7CB12F6607125143987143EE168DA052C0A1FD436A0E14DBA57584CC977F82823318BDC
```

#### Java signature generation code example

```java Java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.SortedMap;
import java.util.*;
public class SHA512Utils {

public static final String ENCODE = "UTF-8";

/**
 * compute the SHA hash of the message and sign it.
 *
 * @param signParams signature parameters
 * @param key     secret
 * @return
 */
public static String SHAEncrypt(TreeMap<String, String> signParams, String key) {
    signParams.remove("sign");
    StringBuffer sb = new StringBuffer();
    Set es = signParams.entrySet();
    Iterator it = es.iterator();
    while (it.hasNext()) {
        Map.Entry entry = (Map.Entry) it.next();
        String k = (String) entry.getKey();
        String v = (String) entry.getValue();
        if (null != v && !"".equals(v) && !"sign".equals(k)
                && !"key".equals(k)) {
            sb.append(k + "=" + v + "&");
        }
    }
    sb.append("key=" + key);
    return encrypt(sb.toString(), ENCODE).toUpperCase();
}

public static String encrypt(String aValue, String encoding) {
    aValue = aValue.trim();
    byte value[];
    try {
        value = aValue.getBytes(encoding);
    } catch (UnsupportedEncodingException e) {
        value = aValue.getBytes();
    }
    MessageDigest md = null;
    try {
        md = MessageDigest.getInstance("SHA-512");
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
        return null;
    }
    return toHex(md.digest(value));
}

public static String toHex(byte input[]) {
    if (input == null)
        return null;
    StringBuffer output = new StringBuffer(input.length * 2);
    for (int i = 0; i < input.length; i++) {
        int current = input[i] & 0xff;
        if (current < 16)
            output.append("0");
        output.append(Integer.toString(current, 16));
    }
    return output.toString();
}

/**
 * verify signature
 *
 * @param signParams sign parameters
 * @param key        secret
 * @return
 */
public static boolean verifySHA(TreeMap<String, String> signParams, String key) {
    String verifySign = signParams.get("sign");
    String sign = SHAEncrypt(signParams, key);
    if (sign.equalsIgnoreCase(verifySign)) {
        return true;
    }
    return false;
}
}
```

Java signature generate code example:

```java Java
    import java.util.Map;
    import com.alibaba.fastjson.JSONObject;

    public class SignTest {
        public static void main(String[] args) {
            String jsonData="{\n" +
                                " \"appId\": \"TEST000001\",\n" +
                                " \"sign\": \"TEST000001\",\n" +
                                " \"merchantOrderNo\": \"11126\"\n" +
                                "}\n";
    //Sign the data
    TreeMap resultMap=JSONObject.parseObject(jsonData, TreeMap.class);
    String result=SHA512Utils.SHAEncrypt(resultMap,"9999");
    System.out.println(result);

    //Verify the data,true=verify success
    resultMap.put("sign","95960053CC577FCAFC272410D5F70094DD0986F6C3266DB7D00D0B37A7CB12F6607125143987143EE168DA052C0A1FD436A0E14DBA57584CC977F82823318BDC");
    System.out.println(SHA512Utils.verifySHA(resultMap,"9999"));
        }
    }
```

## Create Order

### API Description

Request Method:`POST`

Request Path:`open/api/payment`

### Parameters

#### Request Parameters

| Parameter       | Sign | Mandatory | Type   | Length | Description                                           |
| :-------------- | :--- | :-------- | :----- | :----- | :---------------------------------------------------- |
| appId           | Y    | Y         | string | 64     | AppId is unique for merchant                          |
| sign            | N    | Y         | string | 256    | Sign                                                  |
| merchantOrderNo | Y    | Y         | string | 64     | Merchant order number (must be unique)                |
| qrCode          | Y    | Y         | string | 256    | QR Code string                                        |
| userId          | Y    | Y         | string | 64     | UserId is unique for user (email / phone number)      |
| userIp          | Y    | Y         | string | 32     | User IP address                                       |
| redirectURL     | Y    | Y         | string | 128    | The redirection address after the successful purchase |
| callbackURL     | Y    | Y         | string | 128    | The address receiving order webhook                   |

Example

```json
{
    "appId": "TEST000001",
    "sign": "TEST000001",
    "merchantOrderNo": "11126"
    "qrCode": "00020101021138580010A000000727012800069704070114190360421800120208QRIBFTTA53037045802VN830084006304EDC5",
    "userId": "505884978@qq.com",
    "userIp": "127.0.0.1",
    "redirectUrl": "http://127.0.0.1:8022/open/api/payment2",
    "callbackUrl": "http://127.0.0.1:8022/open/api/callback"
}
```

#### Response Parameters

| Parameter | Type    | Description      |
| :-------- | :------ | :--------------- |
| success   | boolean | Success          |
| error     | boolean | Error            |
| code      | long    | Response code    |
| msg       | string  | Response message |
| traceId   | string  | Trace id         |
| model     | object  | Response content |

##### Model Response Parameters

| Parameter | Type   | Description          |
| :-------- | :----- | :------------------- |
| webUrl    | string | Checkout url address |
| orderNo   | string | AEON order number    |

Example

```json
{
  "code": "0",
  "msg": "success",
  "model": {
    "webUrl": "htts://127.0.0.1:8080?orderNum=300217180923663910033",
    "orderNo":"21321321321"
  },
  "traceId": "6668024eeb989c77a375967aded5d646",
  "success": true,
  "error": false
}
```
