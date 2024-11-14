---
layout: post
title: NCL Gymnasium – Password Cracking
date: 2024-11-12 20:31 -0800
tag: ctf
category: Password Cracking
---
# NCL Gymnasium – Password Cracking

## Challenge Set Overview

This document covers solutions to various password-cracking challenges from the NCL Gymnasium, classified into different difficulty levels: Easy, Medium, and Hard. For each challenge, I'll detail the tools, techniques, and strategies I used to crack the passwords efficiently. 

## Challenge Breakdown

---

### Rockyou (Easy)

#### Scenario
> "Our officers have obtained password dumps storing hacker passwords. After obtaining a few plaintext passwords, it appears they overlap with the passwords from the `rockyou` breach."

**Hashes to Crack:**
- `68a96446a5afb4ab69a2d15091771e39`
- `ec5f0b1826389df8622133014e88afde`
- `32e5f63b189b78dccf0b97ac41f0d228`
- `2233287f476ba63323e60addca1f6b64`
- `6539bbb84fe2de2628fc5e4f2a31f23a`

**Points:** 20 points per hash

#### Solution
1. **Tool Used:** [CrackStation](https://crackstation.net/)
2. **Process:**
    - **Prepare the Hashes:** Copy each hash individually or create a text file containing the list of hashes for easy copying.
    - **Enter the Hashes:** Go to [CrackStation](https://crackstation.net/) and paste the hashes into the input box. CrackStation accepts both single and batch entries.
    - **Run the Crack:** Click on the "Crack Hashes" button to start the process.
3. **Explanation:** CrackStation uses a large database of pre-computed hashes from common wordlists, including the `rockyou.txt` file, which is particularly helpful for weak or common passwords.
4. **Interpreting Results:** If CrackStation finds matches, it will display the plaintext password next to each hash. Note down the cracked password for each hash for easy reference.

<img src='assets/images/ctf_writeup_images/crackstation.png' alt='crackstation'>


---

### Windows (Medium)

#### Scenario
> "Our analysts have obtained Windows password dumps storing hacker passwords. Can you crack them?"

**Hashes to Crack (Format: Hash1:Hash2):**
- `21259DD63B980471AAD3B435B51404EE:1E43E37B818AB5EDB066EB58CCDC1823`
- `11CB3F697332AE4C4A3B108F3FA6CB6D:13B29964CC2480B4EF454C59562E675C`
- `65711C079DC4CD3CC2265B23734E0DAC:47F747C5190DC0F0B921AA4A07F06285`
- `FBBDA33FC12E83FB0C240E84A183686E:DDE9DC6E34E2E6E11EF9E51C6B27ED96`
- `21C4E7C2EFE8E8D1C00B70065ED76AA7:A7A0F9AFD4A78F531A1CF4C42E531BBF`
- `E85B4B634711A266AAD3B435B51404EE:FD134459FE4D3A6DB4034C4E52403F16`
- `BA756FB317B622DBAAD3B435B51404EE:C8405270B10B13AE8A24612BB853567A`
- `199C926FA387EAB7AAD3B435B51404EE:F196F77BF8BB15781BA8364C649C5FD4`
- `FE4AACAAAD7D986AAAD3B435B51404EE:3928E16F614E2316CA51C336FA5B3011`
- `3613F7EC15407F56AAD3B435B51404EE:C82E164316183AA3AF3EA6BAA642A237`

**Points:** 10 points per password

#### Solution
1. **Tool Used:** [ophcrack](https://ophcrack.sourceforge.io/) 
2. **Process:**
   - **Setup:** Ensure you have **ophcrack** installed (Kali Linux comes with it pre-installed). Download and add the `XP Special` table to improve success rates for cracking NTLM hashes.
     - **Adding the XP Special Table:**
       - Launch **ophcrack** and navigate to the "Tables" menu.
       - Download it from the [ophcrack tables page](https://ophcrack.sourceforge.io/tables.php) and add it to the correct directory in **ophcrack**.
   - **Input the Hashes:**
     - Go to the "File" menu in **ophcrack** and select "Load" to add your hash file, or paste the hashes directly if entering them manually.
   - **Run the Crack:**
     - Once the hashes are loaded, click the "Crack" button and wait for **ophcrack** to process each hash.
3. **Explanation:** **ophcrack** uses rainbow tables (in this case, the `XP Special` table) to reverse common NTLM hash values without brute-forcing each one. This method is effective for simpler, older passwords that the `XP Special` table was designed for.
4. **Interpreting Results:** After cracking, **ophcrack** will display the plaintext password next to each hash in the result pane. Record each cracked password for reporting.

<img src='assets/images/ctf_writeup_images/ophcrack.png' alt='ophcrack'>


---

### Mask (Medium)

#### Scenario
> "Our analysts have obtained password dumps storing hacker passwords. After obtaining a few plaintext passwords, it appears that they are all in the format: `SKY-HQNT-` followed by 4 digits. Can you crack them?"

**Hashes to Crack:**
- `71b816fe0b7b763d889ecc227eab400a`
- `674291170dffcf620bda2a604a6820ea`
- `06f03267f31077d2c4b5c728472070ae`
- `d866f4b3b34b598375149fb7661113ab`
- `d9053951a8d1c15254b46ec9fc974a6b`

**Points:** 15 points per hash

#### Solution
1. **Tool Used:** Hashcat with a custom rule/mask to match the `SKY-HQNT-####` format.
2. **Process:**
   - **Prepare the Hash File:** Save the hashes you need to crack in a text file (e.g., `hash.txt`).
   - **Run Hashcat with Custom Mask:** Use Hashcat with a mask that matches the specified format, where:
     - `SKY-HQNT-` is the fixed prefix,
     - `?d?d?d?d` represents four digits at the end.
   - **Command:**
     ```bash
     hashcat -m 0 hash.txt -a 3 SKY-HQNT-?d?d?d?d
     ```
3. **Explanation:** This mask ensures that Hashcat only attempts combinations that fit the `SKY-HQNT-####` pattern, reducing unnecessary attempts and focusing on the correct format.
4. **Interpreting Results:** Once cracking is complete, Hashcat will display the cracked passwords next to each hash in the output. Record each password for reporting.

<img src='assets/images/ctf_writeup_images/hashcat.png' alt='hashcat'>

---

### Law & Order (Hard)

#### Scenario
> "Our analysts have obtained password dumps storing hacker passwords. It appears that they are based off of Law and Order: SVU episodes and end in 2 digits."

**Hashes to Crack:**
- `6475c851b56004eb96ab1404252c3a34`
- `abe6591e06aafc3cf1b0783b120f685e`
- `1e1612db8bdeebc7e8d56f8f30b39456`
- `3dd9dd0e352df4433aadf2273e269287`
- `08038f679de74982bfb9bac43d46271a`

**Points:** 20 points per hash

#### Solution
1. **Tool Used:** Custom Python script with an episode list text file found on GitHub.
2. **Process:**
   - **Prepare the Episode List:** Obtain a text file with episode titles (e.g., from a GitHub repository).
   - **Run the Python Script:** Create a Python script to iterate through the formatted episode list and append the 2-digit suffix for each variation. Use these formatted variations to brute-force each hash.
3. **Explanation:** The script reads episode titles, formats them as lowercase with no spaces, appends each 2-digit suffix from `00` to `99`, and checks if the MD5 hash matches any of the provided hashes.
4. **Interpreting Results:** If a hash matches, the script prints the corresponding password. Record each cracked password for reporting.
   
   
- **Script:**

```python
import hashlib

hashes = [
    '6475c851b56004eb96ab1404252c3a34',
    'abe6591e06aafc3cf1b0783b120f685e',
    '1e1612db8bdeebc7e8d56f8f30b39456',
    '3dd9dd0e352df4433aadf2273e269287',
    '08038f679de74982bfb9bac43d46271a'
]

with open('law_and_order.txt') as ep_list:
    for ep in ep_list:
        clean = ep.strip()
        try:
            for_password = clean.lower().replace(' ', '')
            for i in range(100):
                try_password = f'{for_password}{i:02}'
                md5_hash = hashlib.md5(try_password.encode()).hexdigest()
                if md5_hash in hashes:
                    print(f'{md5_hash}:{try_password}')
        except:
            pass
```



<img src='assets/images/ctf_writeup_images/python_cracker.png' alt='custom python cracker'>


---
*Received permission from Cyber Skyline to post this writeup pubicly*