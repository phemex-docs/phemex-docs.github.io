# t(:predict)
t(:predict_para)

## t(:order_buy)
> t(:codequote_curlExample)

```java
class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!"); 
    }
}
```

> t(:codequote_responseExample)

```java
{
    "code": 0,
    "msg": "",
    "data": {
      "transactionTime": 1660702505000,
      "transferAmountEv": 1000000000,
      "status": "Ok"
    }
}
```


<p class="fake_header">t(:httprequest)</p>
GET
<code><span id=svAccount>/phemex-predict/order/buy</span></code>
<button class="clipboard_button" data-clipboard-action="copy" data-clipboard-target="#svAccount"><img src="/images/copy_to_clipboard.png" height=15 width=15></img></button>

<p class="fake_header">t(:requestparameters)</p>
|t(:column_parameter)|t(:column_required)|t(:column_type)|t(:column_comments)|
|:----- |:-------|:-----|----- |
|productId| true | number | t(:predict_product_id)|
|currency| true | string | t(:currency)|
|amountEv| true | number | t(:amount_ev)|
|optionId| true | number | t(:predict_option_id)|

<p class="fake_header">t(:responseparameters)</p>
|t(:column_parameter)|t(:column_type)|t(:column_comments)|
|:----- |:-----|----- |
|code| number | t(:ret_code)|
|msg| string | t(:ret_msg)|
|data| Object | t(:ret_data)|
