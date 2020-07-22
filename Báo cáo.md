### Copy file

Để quá trình thêm shellcode không làm mất file exe gốc, chúng ta sẽ tiến hành copy file này ra thành 1 file khác. File mới sẽ có gắn thêm _-injected_
```python
import shutil

input_name = "calc.exe"
output_name = "calc-injected.exe"

shutil.copy2(input_name, output_name)
```

### Resize file size tăng lên 0x1000 bytes

Ehihi
```python
```
