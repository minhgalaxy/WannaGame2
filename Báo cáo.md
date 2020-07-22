### Copy input file sang 1 file khác
```python
import shutil

input_name = "calc.exe"
output_name = "calc-injected.exe"

shutil.copy2(input_name, output_name)
```
