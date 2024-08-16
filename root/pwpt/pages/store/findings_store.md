# endpoint
- /checkout



# request


```
POST /checkout HTTP/1.1
Host: race1
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://race1/store
Content-Type: application/json
Content-Length: 231
Origin: http://race1
Connection: keep-alive
Cookie: connect.sid=s%3A_6s7p0RL8S1u757qnENZ_yop4pF5uYP7.GxavXvYIrs0jvPuGda6qeXGerSfMSif8FyRkCXmGagM; token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2NmI4NDhhZDA1YjMwYWUzYzE5ZjI2Y2QiLCJlbWFpbCI6InBtdGVzdEB0ZXN0LmNvbSIsImFjY291bnRfdHlwZSI6InN1YnNjcmliZXIiLCJpYXQiOjE3MjMzNTQ1NTIsImV4cCI6MTcyMzQ0MDk1Mn0.VR-fp7vf8owpYKFA9Vi63QvTZNYu-0RQPjFPLCCz-lg

{"items":[{"product":"66b7fdcc8c5a247f79f5e107","name":"Racing Gloves","price":29.99,"quantity":1,"variant":"Red"}],"total":23.99,"discount":20.00666888962988,"cardNumber":"4242 4242 4242 4242","cardExpiry":"06/34","cardCvc":"123"}
```

![[deleteheader.png]]