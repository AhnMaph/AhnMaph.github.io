---
title: Anti Debug
date: 2026-03-24 3:49:00 +0700
categories: [ctf]
tags: [ctf]
pin: false
toc: true
---

# **anti-debug**
Tìm hiểu về các anti debug

## I. Window API
Giới thiệu các Window API có thể được sử dụng cho AntiDebug. Có nhiều lý do các người tạo ra malware không hay sử dụng Window API để anti-debug. Một trong số các lý do là bởi vì các API call này có thể bị hook và không hoạt động.  Bởi vậy thông thường người ta thường chọn các API call tự code hơn là dựa vào Window API.

Sau đây là danh sách các Window API được tìm hiểu:

### IsDebuggerPresent

#### Lý thuyết:
 ``` c
 BOOL WINAPI IsDebuggerPresent(void);
```
- Hàm này xem cấu trúc Process Environment Block (PEB) để tìm trường `IsDebugged`.
- Nếu process hiện đang trong một debugger thì giá trị trả về là khác 0.
- Ngược lại nếu không trong một debugger thì giá trị trả về là 0.

![p4](https://hackmd.io/_uploads/BkCB4lGMR.png)
Hàm này đơn giản là returns giá trị `IsDebugged` flag
Ta chỉnh jnz thành jz để nó không xuất ra "But detected debugger!"
![image](https://hackmd.io/_uploads/HJpMSzGfA.png)
### OutputDebugString
``` c
 void WINAPI OutputDebugString(_In_opt_ LPCTSTR lpOutputString);
```
- Hàm này được dùng để gửi một chuỗi đến debugger để hiển thị.

- Nếu OutputDebugString được gọi mà không có debugger nào đang được gắn (attached), thì GetLastError sẽ không còn giữ giá trị tùy ý mà ta đặt trước đó.

- Bởi vì khi OutputDebugString trả về thất bại, nó sẽ tự thiết lập error code mới thông qua hàm GetLastError.
### NtGlobalFlag
![image](https://hackmd.io/_uploads/rkACA-GMA.png)

NtGlobalFlag là một trường (DWORD) lưu tại `PEB` (process environment block) . Offset `0x68` ở máy 32-bit, và ở máy 64-bit thì lưu tại offset `0xBC`. Giá trị mặc định cho vùng này là 0. Tuy nhiên khi nếu tiến trình được tạo bởi debuger thì: 

Debugger sẽ set các flag sau:
```
FLG_HEAP_ENABLE_TAIL_CHECK (0x10)

FLG_HEAP_ENABLE_FREE_CHECK (0x20)

FLG_HEAP_VALIDATE_PARAMETERS (0x40)
```
Để phát hiện debugger thì kết hợp các flag trên lại. 

Ta thấy đầu tiên nó call function sub_401120 
![image](https://hackmd.io/_uploads/SJrDAWMzR.png)

`mov eax, large fs:30h` là Process Environment Block 
Sau đó tính NtGlobalFlag bằng cách cộng `PEB` cho 0x68. So sánh NtGlobalFlag với 0x70 (nếu debugger đã set các flag ở trên) nếu return về 0 thì có debugger đang hoạt động

![p7](https://hackmd.io/_uploads/Bk9pVgGGR.png)

Tương  tự ta sẽ sửa cờ ZF = 1 
### CheckRemoteDebuggerPresent
``` c
BOOL WINAPI CheckRemoteDebuggerPresent(
 _In_ HANDLE hProcess,
 _Inout_ PBOOL pbDebuggerPresent);
 • Parameters
 • hProcess [in] : A handle to the process.
 • pbDebuggerPresent [in, out] : A pointer to a variable that the function sets to 
TRUE if the specified process is being debugged, or FALSE otherwise.
 • Return value
 • If the function succeeds, the return value is nonzero otherwise is zero.
```
- Xác định process có đang bị debug hay không
- Từ **remote** trong hàm **CheckRemoteDebuggerPresent** không có nghĩa là trình gỡ lỗi (debugger) nhất thiết phải nằm trên một máy tính khác mà thay vào đó, nó cho biết rằng trình gỡ lỗi đang chạy trong một tiến trình riêng biệt và song song với tiến trình hiện tại.
- Sử dụng hàm **IsDebuggerPresent** để xác định process gọi nó có đang chạy debugger không.
- Hàm này nhận một **handle** của tiến trình làm tham số và sẽ kiểm tra xem tiến trình đó có trình gỡ lỗi (debugger) đang được gắn vào hay không.

![image](https://hackmd.io/_uploads/HJG_cWGMA.png)

**CheckRemoteDebuggerPresent** là hàm của kernel32 dùng để xác định process chỉ định có bị debugged không. 

Nếu có debugger thường là khi nó đang chạy, **pbDebuggerPresent** sẽ được set thành 0xffffffff. 


### ProcessMonitor
ProcMon (Process Monitor) tạo một device object ở kernel (ví dụ \\.\Global\ProcmonDebugLogger) để nhận dữ liệu/command từ usermode. Nếu chương trình thử mở device đó bằng CreateFileA và thành công (trả về handle hợp lệ), thì có nhiều khả năng ProcMon đang chạy — và chương trình có thể coi đây là dấu hiệu “bị phân tích” và thay đổi hành vi. Đó là anti-debug.

**Cách hoạt động**

Chương trình gọi 
``` c 
CreateFileA("\\\\.\\Global\\ProcmonDebugLogger",...)
```
Hệ thống trả về:

Handle hợp lệ (không INVALID_HANDLE_VALUE) nếu device object tồn tại (ProcMon driver đã đăng ký device này).

Tra cứu trên mạng về device `\\\\.\\Global\\ProcmonDebugLogger` 
![image](https://hackmd.io/_uploads/r1yN4QfzC.png)
![image](https://hackmd.io/_uploads/BJSDHmGG0.png)
![image](https://hackmd.io/_uploads/SkWQd7fGC.png)

Vậy là có nghĩa là nếu device này có tồn tại nó sẽ return về một giá trị cụ thể còn không thì nó sẽ return `INVALID_HANDLE_VALUE` (chính là 0xFFFFFFFF)

Chương trình kiểm tra kết quả (compare / test / JZ/JNZ) và quyết định: nếu device tồn tại → giả sử đang chạy ProcMon 
![image](https://hackmd.io/_uploads/HJxTCMfMA.png)
Pseudocode đầy đủ của nó
![image](https://hackmd.io/_uploads/rk4C-QMMA.png)

![image](https://hackmd.io/_uploads/SJdNfmfzR.png)



 
### Check Process Name
function đầu tiên nó chạy 
![image](https://hackmd.io/_uploads/H1NmiXfGA.png)

Chúng ta sẽ phân tích sub_401130
> Pseudocode:

![image](https://hackmd.io/_uploads/r1yLC7zf0.png)

![image](https://hackmd.io/_uploads/HJlK9QMf0.png)

Đầu tiên thì ta thấy mình sử dụng hàm **CreateToolhelp32Snapshot**, sau khi tra cứu thì em thấy đây là hàm trả về snapshot của processes. 
  
![image](https://hackmd.io/_uploads/ByLC5QMzR.png)

Tra cứu thêm về PROCESSENTRY32 nó trả về mục lục process đang được tiến hành từ snapshot.
![image](https://hackmd.io/_uploads/r1z86QfMA.png)

Vậy rõ ràng nó đang tra xem trong process có tên của các debugger "tình nghi" hay không bao gồm ollydbg, idaq, wireshark v.v 

## Anti Virtual Machine
### Vmware Detection
![image](https://hackmd.io/_uploads/r1JLgNzMA.png)

![image](https://hackmd.io/_uploads/HyFEeNGGA.png)
Pseudocode
![image](https://hackmd.io/_uploads/BJg3hB4MfR.png)

Đây là một "backdoor" trong I/O port của VMware. Chúng ta có thể phát hiện nếu có chạy Vmware khi sử dụng lệnh IN để đọc data thông qua 0x5658 port 
![image](https://hackmd.io/_uploads/HyZXSVfzR.png)

Nếu chúng ta sử dụng Vmware thì ebx = 0x564D5868h 


## Manually checking
###  Thời gian chạy khác biệt
![image](https://hackmd.io/_uploads/rycjhzGM0.png)
GetTickCount() là hàm xác định thời gian từ start cho đến thời điểm đó, nên khi ta sử dụng breakpoint quá lâu, thời gian chạy chắc chắn sẽ lớn hơn thời gian chương trình quy định là `0x3E8` mili giây. 

Để bypass thì có 2 cách một là đổi ZF, hai là sẽ không đặt breakpoint ở đó để chương trình không bị trễ thời gian.


![p9](https://hackmd.io/_uploads/rJVR4xMGA.png)
### Breakpoint
đến với anti debug tiếp theo
Pseudocode:
![image](https://hackmd.io/_uploads/B1QkOEfGC.png)

khi chương trình thực hiện phép chia 1/0 làm cho nó không hợp lệ. 
Xem kĩ chương trình ta thấy ngoại lệ xảy ra thì nó sẽ chạy `loc_4015F6` sau đó thì giá trị ở var_88 = 0 sau đó cho nó chạy bình thường thì sẽ tự qua luôn. Còn nếu ta chỉnh 1/0 chương trình sẽ dăng ra. 
![image](https://hackmd.io/_uploads/SJqdoVMGA.png)

![p18](https://hackmd.io/_uploads/BkFBBgzf0.png)

Tiếp theo đến với phần chương trình này ta thấy là nó khá rõ ràng là chương trình luôn sẽ điều hướng đến chỗ sai kết quả vì var_78 mới vào đã set bằng 0. Nên em sẽ chỉnh jnz thành jn
![image](https://hackmd.io/_uploads/rkFxJrzMA.png)


![p20](https://hackmd.io/_uploads/HJrJLgGzC.png)


sau đó thì chạy bình thường tới khi cửa sổ này hiện ra.

![p28](https://hackmd.io/_uploads/HJoyf-MMC.png)



# Bài tập thêm
## Prob.exe
Bài này là thiên về dịch ngược. 
Khi cho vào IDA em có được pseudocode sau:

![image](https://hackmd.io/_uploads/SyN5QNQzC.png)

Str được cho là "have a good day! enjoy wargame!" 

Xem vào từng hàm một ta có một vài thông tin như sau:

Dòng 4 là để tính string length của Str được cho, nên em đổi tên cho dễ nhìn.
![image](https://hackmd.io/_uploads/SkgNJHVXzC.png)

Dòng 5 là lấy flag từ file flag.txt sau đó cho nó vào *unk_405900* còn vế "%[^;]s" nghĩa là nó sẽ đọc file cho tới khi gặp dấu ";"
![image](https://hackmd.io/_uploads/r1V6VVmMC.png)

Nên em đổi tên *unk_405900* thành flag
Dòng 6 có vẻ là hàm mã hóa flag vừa mới nhập:
![image](https://hackmd.io/_uploads/SkDkIEXzA.png)

flag được mã hóa từng byte và lưu vào trong v1. Sau đó lần lượt lưu v1 32 bits thấp trong mảng int *dword_405040* (đổi tên thành bien1) và 32 bits cao của v1 vào mảng int *dword_405044* (đổi tên thành bien2). 

Sau đó mã hóa nó bằng các cách ghép bien1[2 * i] bien2[2 * i] vào 32bits thấp và cao của các v. 

Dòng 7 là tiếp tục ghép vào byte từ bien1 và bien2 vào v1, sau đó chuyển nó thành hex gắn thêm 0X rồi lưu vào Buffer sau đó là gắn string này vào Destination. 
![image](https://hackmd.io/_uploads/Hy8m9VXGA.png)

Dòng 8 sau đó là lưu string tại Destination trong file tmp.enc 
![image](https://hackmd.io/_uploads/Skvoc4XzR.png)

Chúng ta đã có được file tmp.enc vậy có thể tìm được flag rồi. 

``` python
import re
enc = "0X29AF0X24930X35A90X27290X4140X24530X4580X28EF0X2F9E0X2FFC0X26D00X4670X26EB0X24390X39140X42C0X43F0X275F0X2EDD0X2B2B0X300F0X389C0X41D0X36A60X24740X32290X29790X24A90X2E890X27560X4270X29EE0X24480X36980X27500X44E0X247D0X41F0X29670X302F0X2FCF0X26CD0X4260X26D00X24A70X391D0X46B0X42A0X28090X2F100X2BF70X302B0X39120X4160X37710X24A30X32940X296D0X24A80X2E610X27F80X4680X2A220X25130X365C0X28050X4950X25120X4970X296C0X30350X2FED0X273C0X4720X27400X24F60X39500X4BA0X47C0X28120X2F76"
Str = "have a good day! enjoy wargame!"
stringlen = len(Str)
# Khai báo các giá trị có sẵn
match = [m.start() for m in re.finditer(r'0X', enc)] # tìm vị trí của các 0X được nối vào tách ngược nó ra

# Khai báo mảng cần dùng do dùng 2*i nên nhân 2
bien1 = [0] * (0x51 * 2)
bien2 = [0] * (0x51 * 2)

# trong khoảng chạy i = 0 tới i = 0x50
for i in range(0x51):
    # tách hex trong khoảng từ vị trí 0x thứ i tới 0x thứ i+1
    if i == 0x50: 
         h = enc[match[i]+2:]
    #trường hợp khi tới 0x cuối thì lấy hết luôn
    else: h = enc[match[i]+2:match[i+1]]
    
    v1 = int(h,16)
    bien1[2*i] = v1 & 0x0000FFFF
    bien2[2*i] = v1>>32 & 0xFFFFFFFF
    #lưu 32 bits đầu cuối vào bien1[2*i] và bien2[2*i]

    # dịch ngược hoàn toàn hàm sub_401500
for i in range(0x51):
    v4 = bien1[2*i]
    v2 = v4 - i
    bien1[2*i] = v2 & 0x0000FFFF
    bien2[2*i] = (v2 >> 32) & 0xFFFFFFFF
    v3 = bien1[2*i]
    v2 = v3 + ord(Str[i % stringlen])
    bien1[2*i] = v2 & 0x0000FFFF
    bien2[2*i] = (v2 >> 32) & 0xFFFFFFFF
    v1 = bien1[2*i]
    # ta biết c = a^b thì a = c ^ b
    a1_byte = (ord(Str[i % stringlen]) * ord(Str[i % stringlen]) + i)^v1  
    print(chr(a1_byte),end="")
    # xuất từng byte ra
```
![image](https://hackmd.io/_uploads/HkZ-ANQzA.png)


