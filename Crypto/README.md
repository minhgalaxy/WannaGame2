# The Return of Fox
>It seems that the fox refused to give us a gift. The gift will only be given to the one who deserves.

[Link tải challenge](foxreturn.rar)

Nội dung file cipher.py
```python
#!/usr/bin/env python3

import flag
import binascii

def sxor(str1, str2):
	res = ""
	for a, b in zip(str1, str2):
		res += chr(ord(a)^ord(b))
	return res.encode("utf-8").hex()

def main():
	msg = flag.msg
	key = flag.key
	msg_len = len(msg)
	nkey = (key*(int(msg_len/len(key))+1))[:msg_len]
	cipher = sxor(msg, nkey)
	open("ciphertext.txt","w").write(str(cipher))

if __name__ == '__main__':
	main()
```

Nội dung file ciphertext.txt
```text
0502120415041c1a0d0a493536193122014016103c361c11333a5e282e503a445a3c5f0416455c5b000f4001575d411257021e
```

Đây là thuật toán **xor**, đặc điểm của **xor** là:

Nếu a ^ b = c thì:
- c ^ a = b
- c ^ b = a

Cho nên trong bài này ta có **flag ^ key = cipher**
=> Có thể tính được flag bằng cách lấy **cipher ^ key**

Dựa vào các challege trước, ta đã biết flag có dạng **flag{ahihi...}**, nên ta có thể tính được 10 ký tự đầu của key như sau

```
cipher: 05 02 12 04 15 04 1c 1a 0d 0a 49 35 36 19 31 22 01 40 16 10 3c 36 1c 11 33 3a 5e 28 2e 50 3a 44 5a 3c 5f 04 16 45 5c 5b 00 0f 40 01 57 5d 41 12 57 02 1e
flag:    f  l  a  g  {  a  h  i  h  i                                                                                                                          }
key:     c  n  s  c  n  e  t  s  e  c                                                                                                                          c
```
Thử xor cipher bằng key **cnscnetsec** thì ra flag luôn :smile:

```python
#!/usr/bin/env python3

import binascii

def sxor(str1, str2):
	res = ""
	for a, b in zip(str1, str2):
		res += chr(ord(a)^ord(b))
	return res

def main():
	cipher = str(bytes.fromhex('0502120415041c1a0d0a493536193122014016103c361c11333a5e282e503a445a3c5f0416455c5b000f4001575d411257021e'), 'utf-8')
	key = 'cnscnetsec'
	msg_len = len(cipher)# length của msg bằng length của cipher
	nkey = (key*(int(msg_len/len(key))+1))[:msg_len]
	msg = sxor(cipher, nkey)
	print('Flag: %s' % msg)

if __name__ == '__main__':
	main()

```

Flag: `flag{ahihi*[Ez_Gu3ss_Xor]_*[K3Y*)_1ab698ca3b985a2a}`