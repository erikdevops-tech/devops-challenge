Write a bash command for parsing the transaction-log -> getting order-id to call to http://example and append to ouputs.txt

Testing:
```
cat transaction-log.txt | jq -c 'select(.symbol == "TSLA", .side == "sell") |  .order_id'| xargs -I{} echo 'curl -s "https://example.com/api/{}"'               
curl -s "https://example.com/api/12346"
curl -s "https://example.com/api/12346"
curl -s "https://example.com/api/12348"
curl -s "https://example.com/api/12350"
curl -s "https://example.com/api/12352"
curl -s "https://example.com/api/12354"
curl -s "https://example.com/api/12356"
curl -s "https://example.com/api/12358"
curl -s "https://example.com/api/12360"
curl -s "https://example.com/api/12362"
curl -s "https://example.com/api/12362"
curl -s "https://example.com/api/12364"
```

Cli running
```
cat transaction-log.txt | jq -c 'select(.symbol == "TSLA", .side == "sell") |  .order_id'| xargs -I{} curl -s "https://example.com/api/{}" >>
./output.txt
```

