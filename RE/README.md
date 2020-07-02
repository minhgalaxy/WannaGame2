# \[First Blood Challenge\] Crack me if you can!
>The algorithm isn't hard, The hard is how to thoroughly understood algorithm

[Link tải challenge](re4)

Kiểm tra file bằng Detect It Easy

![Screenshot](../screenshots/re_1.png?raw=true "Screenshot")

=> File đã bị pack bằng UPX

Tiến hành unpack upx

![Screenshot](../screenshots/re_2.png?raw=true "Screenshot")

Sau đó mở file bằng **IDA Pro**, nhấn F5 để xem mã giả

Hàm main:
```c++
int __cdecl main(int argc, const char **argv, const char **envp)
{
  unsigned __int64 v3; // rsi
  int v4; // ecx
  void *v5; // rsp
  unsigned __int64 v7; // rbx
  int j; // [rsp+0h] [rbp-500h]
  int i; // [rsp+4h] [rbp-4FCh]
  unsigned int v10; // [rsp+8h] [rbp-4F8h]
  int v11; // [rsp+Ch] [rbp-4F4h]
  __int64 v12; // [rsp+10h] [rbp-4F0h]
  unsigned __int64 v13; // [rsp+18h] [rbp-4E8h]
  int v14[288]; // [rsp+20h] [rbp-4E0h]
  char v15[40]; // [rsp+4A0h] [rbp-60h]
  unsigned __int64 v16; // [rsp+4C8h] [rbp-38h]

  v16 = __readfsqword(0x28u);
  qmemcpy(v14, "R", 1144uLL);
  printf((unsigned __int64)"Key: ");
  v3 = (unsigned __int64)v15;
  _isoc99_scanf((unsigned __int64)"%32s");
  v10 = ((unsigned __int64)j_strlen_ifunc(v15, v15) >> 1) - 2;
  if ( (unsigned __int64)j_strlen_ifunc(v15, v15) > 32 )
  {
    printf((unsigned __int64)"missing length!");
  }
  else
  {
    for ( i = 0; ; ++i )
    {
      v7 = i;
      if ( v7 >= j_strlen_ifunc(v15, v3) )
        break;
      v4 = v10;
      v12 = (signed int)v10 - 1LL;
      v5 = alloca(16 * ((4LL * (signed int)v10 + 15) / 16uLL));
      v13 = 4 * (((unsigned __int64)&j + 3) >> 2);
      for ( j = 0; j < (signed int)v10; ++j )
        *(_DWORD *)(v13 + 4LL * j) = v14[v10 * i + j];
      v11 = (unsigned __int64)(4LL * v4) >> 2;
      something(v13, 0LL, (unsigned int)(v11 - 1));
      v3 = (unsigned int)v15[i];
      if ( !(unsigned int)checkkey(v13, v3, v10) )
      {
        printf((unsigned __int64)"wrong key!");
        return 0;
      }
    }
    printf((unsigned __int64)"Right! Your flag: flag{%s}");
  }
  return 0;
}
```

Đầu tiên chương trình gọi hàm `qmemcpy(v14, "R", 1144uLL)` để copy `1144/4 = 286` số nguyên từ địa chỉ của `"R"` vào biến `v14` (_có thể double click vào "R" để xem data tại vùng nhớ đó_)

Tiếp theo, chương trình yêu cầu người dùng nhập key (tối đa 32 ký tự) và lưu vào biến `v15`.

Ở đây, biến `v10` được tính bằng cách lấy `(strlen(key) >> 1) - 2)`, nghĩa là biến `v10` này chỉ thay đổi khi độ dài key thay đổi mà thôi.

Sau đó, chương trình sẽ duyệt lần lượt từng ký tự trong key người dùng đã nhập.

Mỗi vòng lặp, chương trình tạo 1 vùng nhớ có kích thước ```16 * ((4 * v10 + 15) / 16) = 4 * v10 + 15```

Dòng code `v13 = 4 * (((unsigned __int64)&j + 3) >> 2)` mục đích là gán địa chỉ vùng nhớ vừa cấp phát vào biến `v13`.

Vòng lặp
```c++
for ( j = 0; j < (signed int)v10; ++j )
	*(_DWORD *)(v13 + 4LL * j) = v14[v10 * i + j];
```
dùng để lấy `v10` phần tử từ `v14` và copy vào `v13`.

Tại đây, hàm `something` sẽ được gọi, tham số đầu vào của `something` là biến `v13`, 0 và `v10`.

Hàm something:
```c++
__int64 __fastcall something(__int64 a1, unsigned int a2, unsigned int a3)
{
  __int64 result; // rax
  int v4; // ST1C_4
  unsigned int v5; // [rsp+0h] [rbp-20h]

  v5 = a3;
  result = a2;
  if ( (signed int)a2 < (signed int)a3 )
  {
    v4 = choice(a1, a2, a3);
    something(a1, a2, (unsigned int)(v4 - 1));
    result = something(a1, (unsigned int)(v4 + 1), v5);
  }
  return result;
}
```

Hàm choice:
```c++
__int64 __fastcall choice(__int64 a1, int a2, int a3)
{
  int v4; // [rsp+0h] [rbp-20h]
  int v5; // [rsp+14h] [rbp-Ch]
  int i; // [rsp+18h] [rbp-8h]
  int v7; // [rsp+1Ch] [rbp-4h]

  v4 = a3;
  v7 = *(_DWORD *)(4LL * a3 + a1);
  v5 = a2 - 1;
  for ( i = a2; v4 > i; ++i )
  {
    if ( v7 > *(_DWORD *)(4LL * i + a1) )
      swap(4LL * ++v5 + a1, a1 + 4LL * i);
  }
  swap(4 * (v5 + 1LL) + a1, a1 + 4LL * v4);
  return (unsigned int)(v5 + 1);
}
```
Như vậy có nghĩa là hàm `something` sẽ swap các phần tử trong mảng `v13` theo 1 cách nào đó.

Cuối cùng chương trình sẽ kiểm tra ký tự thứ `i` của key (`v15`) bằng hàm `checkkey`.

Hàm checkkey:
```c++
_BOOL8 __fastcall checkkey(__int64 a1, char a2, int a3)
{
  return *(_DWORD *)(4LL * (a3 / 2) + a1) == a2;
}
```

Hàm này chỉ đơn giản là so sánh `v13[v10 / 2]` và `v15[i]`

Nếu tinh ý có thể nhận ra ngay hàm `something` chính là hàm `quick sort`

```c++
quickSort(arr[], low, high)
{
    if (low < high)
    {
        /* pi is partitioning index, arr[pi] is now
           at right place */
        pi = partition(arr, low, high);

        quickSort(arr, low, pi - 1);  // Before pi
        quickSort(arr, pi + 1, high); // After pi
    }
}

/* This function takes last element as pivot, places
   the pivot element at its correct position in sorted
    array, and places all smaller (smaller than pivot)
   to left of pivot and all greater elements to right
   of pivot */
partition (arr[], low, high)
{
    // pivot (Element to be placed at right position)
    pivot = arr[high];  
 
    i = (low - 1)  // Index of smaller element

    for (j = low; j <= high- 1; j++)
    {
        // If current element is smaller than the pivot
        if (arr[j] < pivot)
        {
            i++;    // increment index of smaller element
            swap arr[i] and arr[j]
        }
    }
    swap arr[i + 1] and arr[high]
    return (i + 1)
}
```

Như vậy có thể tóm tắt lại chương trình như sau:
- `v14` chứa là một mảng cho sẵn
- `v15` chứa key của người dùng đã nhập vào
- `v10` tương ứng với độ dài key và bằng `(strlen(v10) >> 1) - 2)`
- Với mỗi vòng lặp, chương trình copy lần lượt từng mảng con có độ dài `v10` lưu vào biến tạm `v13`
- Sắp xếp mảng `v13` bằng thuật toán `quick sort`
- So sánh `v15[i]` với `v13[v10 / 2]`

## Câu hỏi đặt ra là làm sao có thể biết được key có độ dài bao nhiêu?

Có thể suy luận như sau:
Nếu độ dài key là 32 (tối đa) thì ta có:
- `v10 = (32 >> 1) - 2 = 14`
- Vòng lặp chạy 32 lần => mảng `v14` phải có tối thiểu `14 * 32 = 448` phần tử, trong khi mảng `v14` chỉ có `288` phần tử
=> Loại

Với cách suy luận tương tự, ta giảm dần độ dài thì thấy độ dài key bằng `26` là hợp lý

Script `solve.py`
```python
arr = [82, 118, 86, 68, 88, 72, 120, 120, 119, 119, 116, 52, 68, 98, 81, 52, 108, 110, 117, 117, 111, 104, 112, 68, 48, 97, 69, 44, 99, 50, 33, 45, 51, 104, 55, 75, 43, 46, 77, 118, 98, 102, 111, 95, 74, 70, 97, 54, 119, 46, 50, 45, 49, 50, 52, 106, 73, 80, 114, 102, 86, 119, 113, 109, 122, 108, 77, 99, 70, 43, 97, 120, 114, 106, 113, 105, 103, 43, 65, 107, 119, 72, 78, 44, 44, 43, 44, 48, 115, 85, 45, 87, 90, 106, 119, 121, 116, 119, 114, 116, 72, 51, 89, 78, 46, 45, 46, 45, 48, 49, 71, 121, 68, 73, 89, 95, 122, 119, 118, 122, 116, 119, 48, 90, 117, 103, 72, 65, 122, 115, 105, 104, 105, 54, 43, 81, 79, 112, 122, 115, 113, 119, 109, 57, 118, 53, 84, 46, 122, 111, 89, 110, 107, 95, 103, 75, 107, 115, 77, 49, 44, 72, 68, 54, 73, 74, 106, 44, 85, 54, 121, 117, 122, 119, 116, 115, 102, 74, 97, 119, 99, 81, 103, 52, 49, 72, 95, 56, 69, 55, 115, 72, 33, 46, 48, 43, 43, 49, 56, 105, 66, 87, 44, 111, 119, 113, 121, 119, 109, 113, 56, 67, 95, 45, 71, 117, 116, 116, 116, 112, 119, 115, 103, 73, 86, 44, 43, 45, 33, 44, 48, 85, 46, 101, 101, 85, 120, 122, 121, 121, 117, 114, 87, 121, 114, 102, 81, 90, 118, 119, 118, 118, 116, 88, 95, 99, 55, 33, 33, 50, 45, 83, 44, 52, 86, 73, 118, 97, 65, 108, 120, 122, 114, 119, 110, 73, 119, 69, 68, 103, 117, 87, 119, 119, 120, 116]

flag = ''
len_key = 26
v10 = (len_key >> 1) - 2
for i in range(len_key):
	sub = arr[i * v10:i * v10 + v10]
	sub.sort(reverse=True)
	flag += chr(sub[v10 / 2])
print 'Flag: %s' % flag
```

Flag: `flag{th3_4lg0r1thm_Is_1mp0rt4nt}`