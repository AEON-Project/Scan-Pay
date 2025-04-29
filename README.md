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

```html
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
