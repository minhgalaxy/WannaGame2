# Corrupted file
>I need your help! Can you recover it?

[Link tải challenge](6iy5ny7KT.zip)

Sau khi tải file về, các bạn sẽ thấy đuôi file là .zip nhưng không mở được

![Screenshot](../screenshots/corrupted_file_1.png?raw=true "Screenshot")

Mình đoán file này thật ra không phải file zip, mà là tác giả cố tình đổi đuôi thành .zip. 
Để kiểm tra suy luận, mình dùng lệnh ```file 6iy5ny7KT.zip``` và kết quả đây là file jpeg

```bash
root@Vo-Van-Minh:/mnt/d/CTF/WannaGame2/Writeups/Forensic# file 6iy5ny7KT.zip
6iy5ny7KT.zip: JPEG image data, JFIF standard 1.01, resolution (DPI), density 300x300, segment length 16, Exif Standard: [TIFF image data, big-endian, direntries=1, orientation=upper-left], baseline, precision 8, 2360x1777, frames 3
```
Đổi đuôi file lại thành .jpg và mở lên là có ngay flag :blush:

![Screenshot](../screenshots/corrupted_file_2.jpg?raw=true "Screenshot")

##Flag: ```flag{ch3ck_3xt3n510n_f1r5t}```


# Magic Wallet
>It is not small as you think

[Link tải challenge](130e423cf6ea37874a01ae502bfff92n.jpg)

Dùng tool **binwalk*** để extract những file bị ẩn trong file ***130e423cf6ea37874a01ae502bfff92n.jpg***

```
root@Vo-Van-Minh:/mnt/d/CTF/WannaGame2# binwalk --dd='.*' 130e423cf6ea37874a01ae502bfff92n.jpg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
188702        0x2E11E         PDF document, version: "1.7"
194383        0x2F74F         Unix path: /ColorSpace/DeviceRGB/Filter/FlateDecode/Height 1500/Interpolate false/Length 11744/SMask 1939 0 R/Subtype/Image/Type/XObject/Wi
206324        0x325F4         Unix path: /ColorSpace/DeviceRGB/Filter/DCTDecode/Height 2000/Interpolate true/Length 311216/Subtype/Image/Type/XObject/Width 3000>>stream
569441        0x8B061         Unix path: /ColorSpace/DeviceGray/Filter/FlateDecode/Height 1500/Interpolate false/Length 3936/Matte[0 0 0]/Subtype/Image/Type/XObject/Widt
610668        0x9516C         Unix path: /ColorSpace/DeviceGray/Filter/FlateDecode/Height 357/Interpolate false/Length 2336/Matte[0 0 0]/Subtype/Image/Type/XObject/Width
...
5890885       0x59E345        Unix path: /ColorSpace/DeviceGray/Filter/FlateDecode/Height 140/Interpolate false/Length 1664/Matte[0 0 0]/Subtype/Image/Type/XObject/Width
5892741       0x59EA85        Unix path: /ColorSpace/DeviceRGB/Filter/FlateDecode/Height 140/Interpolate false/Length 432/SMask 708 0 R/Subtype/Image/Type/XObject/Width
5913959       0x5A3D67        Unix path: /Subtype/XML/Type/Metadata>>stream
```
**binwalk** sẽ extract những file nó biết vào thư mục **_130e423cf6ea37874a01ae502bfff92n.jpg.extracted**.
Dùng tiếp lệnh ```file *``` để kiểm tra loại file của những file này

```
root@Vo-Van-Minh:/mnt/d/CTF/WannaGame2# cd _130e423cf6ea37874a01ae502bfff92n.jpg.extracted/
root@Vo-Van-Minh:/mnt/d/CTF/WannaGame2/_130e423cf6ea37874a01ae502bfff92n.jpg.extracted# file *
0:      JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 1000x1000, frames 3
1038B1: data
103B40: data
...
2DECDC: data
2E0ECC: data
2E11E:  PDF document, version 1.7
2E36EA: data
...
B10F1:  data
B1567:  data
```
Trong mớ file này có 1 file PDF tên là **2E11E**, đổi đuôi thành .pdf và mở file này lên bằng trình duyệt (_Chrome chẳng hạn_). 

Nhấn tổ hợp **Ctrl + F** và tìm với từ khóa **flag{** sẽ thấy flag ```flag{m0r3_v4lu4bl3_th4n_y0u_th1nk}```