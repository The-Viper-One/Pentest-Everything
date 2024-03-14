# Enumeration

## Tools

WhatWeb: [https://github.com/urbanadventurer/WhatWeb](https://github.com/urbanadventurer/WhatWeb)

```python
# Scan example.com.
whatweb example.com

# Scan reddit.com and slashdot.org with verbose plugin descriptions.
whatweb -v reddit.com slashdot.org

# Perform an aggressive scan of wired.com to detect the exact version of WordPress.
whatweb -a 3 www.wired.com

# Quickly scan the local network and suppress errors.
whatweb --no-errors 192.168.0.0/24

# Scan the local network for HTTPS websites.
whatweb --no-errors --url-prefix https:// 192.168.0.0/24

# Scan for crossdomain policies in the Alexa Top 1000.
whatweb -i plugin-development/alexa-top-100.txt --url-suffix /crossdomain.xml -p crossdomain_xml
```
