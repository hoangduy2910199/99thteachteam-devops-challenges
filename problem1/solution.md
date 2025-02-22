- command 
```bash
    grep "TSLA" ./transaction-log.txt | awk '{print $2}' | tr -d '",' | while read order_id; do 
    echo "-------------------------------------------" >> ./output.txt
    echo "Response from order number: $order_id" >> ./output.txt
    curl -s "https://example.com/api/$order_id" >> ./output.txt
    echo -e "\n" >> ./output.txt 
    done