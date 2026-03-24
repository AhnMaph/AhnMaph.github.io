---
title: CHÈN MÃ VÀO TẬP TIN PE
date: 2026-03-24 3:49:00 +0700
categories: [ctf]
tags: [reverse, ctf]
pin: false
toc: true
---

Link source code: Source Code

## **Bài 1:**
Viết chương trình tìm số nhỏ nhất trong một mảng gồm N số nguyên (giá trị có thể lớn hoặc nhỏ hơn 1 chữ số).
Như bài có nói em sử dụng các bước:
Đầu tiên sẽ cho min_result = phần tử đầu tiên, sau đó duyệt tuần tự qua các thành phần còn lại coi cái nào nhỏ hơn thì ta cập nhật min_result.
 ![image](https://hackmd.io/_uploads/Syh181FCxx.png)

Sau đó là duyệt qua từng phần tử
 ![image](https://hackmd.io/_uploads/SJ4gU1KRlx.png)


## **Bài 3**
Khai báo
 

 ![image](https://hackmd.io/_uploads/r1qxUJtAlg.png)
Chương trình chính:
![image](https://hackmd.io/_uploads/S1G-LJK0gx.png)

Bài này ta coi số nhập vào như 1 sâu, duyệt đầu cuối từ  so sánh ở vị trí i và n-i coi có giống nhau không nếu có thì là sâu đổi xứng.  


## Bài 5
 
-	Đầu tiên mở rộng section .rsrc 0x1000 sau đó chèn code đã được lấy từ file cpp debug bằng IDA.  AddressOfEntryPoint được cập nhật thành 13C00. Jmp được tính từ khoảng cách từ jmp tới hàm _WinMain 
 ![image](https://hackmd.io/_uploads/B1Oz8kFRee.png)

-	Sau đó thêm các thông tin Info và Caption vào .rsrc cập nhật vị trí tương ứng trong code. 
 ![image](https://hackmd.io/_uploads/H1Fm8kFRxg.png)
![image](https://hackmd.io/_uploads/S1n781F0ge.png)
![image](https://hackmd.io/_uploads/S1gN8JYAel.png)

 
 


## Bài 7
Thay vì đổi AddressOfEntryPoint thì ta jmp ngay từ đầu bằng cách chèn jmp tới shellcode của ta vào _WinMain tuy nhiên do không có 5 byte trống nên thay thế 3 lệnh đầu  2+1+5 để đổi thành lệnh jmp sau đó bù lại 3 lệnh đó ở cuối shellcode
Các tình trạng ban đầu
 ![image](https://hackmd.io/_uploads/S1IV81KCgg.png)

Đổi lệnh jmp trong _Winmain
 ![image](https://hackmd.io/_uploads/SycE8kYCgg.png)

Shell code injected
 
 ![image](https://hackmd.io/_uploads/Sy1BIJtAge.png)

**Bài 8:**
Bài này em làm được phân nửa: 
Đầu tiên khi runtime,  shell code chạy đầu tiên sẽ đổi iat thành iat của shell code, sử dụng VirtualProtect để có quyền write vào IAT.
![image](https://hackmd.io/_uploads/rJEUIyFAle.png)

 
Nhưng trong NOTEPAD.EXE không có import VirtualProtect nên em sử dụng pebear để import thêm vào
 ![image](https://hackmd.io/_uploads/Bk9UL1F0lx.png)

Sau đó viết script  khi gọi sẽ nhận tham số của WriteFile sau đó thực hiện gọi POP UP sau đó gọi ngược lại WriteFile  
![image](https://hackmd.io/_uploads/ry-v81KCll.png)

## **Bài 9:**
Bài này ta sẽ viết 2 code 1 code là code bị encoded: 
Em dựa vào bài 6 để làm tiếp tục bài 9, đầu tiên ta mã shellcode ban đầu bằng python trong IDA sau đó chèn ngược lại vào trong NOTEPAD.EXE
![image](https://hackmd.io/_uploads/rkeuUkKRlx.png)

 Ta thấy code đã bị mã hóa và không đọc ra đang làm gì cả
  ![image](https://hackmd.io/_uploads/HkVdUyKRgx.png)

-	Đoạn code thứ 2 dùng giải mã đoạn code này  - đoạn này sẽ được chạy đầu tiên: 

  ![image](https://hackmd.io/_uploads/HJ8t8ytCxx.png)

Đoạn này được thực hiện bằng cách tạo 1 đoạn giải mã cpp sau đó inject nó vào sau đó gọi nó để giải mã -> shell code được chèn vào chỉ thực hiện và decode lúc runtime. 
Lưu ý: sau khi chạy đoạn này sẽ báo không có quyền truy cập lý do là .rsrc không có quyền write nên em chỉnh quyền trong CFF cho section .rsrc để nó có quyền Write
 ![image](https://hackmd.io/_uploads/S1k9UyKAex.png)
![image](https://hackmd.io/_uploads/ryNqLkYAlx.png)

 
Trước khi giải mã:
 ![image](https://hackmd.io/_uploads/Hk5cIJYCel.png)

Sau khi giải mã:
 ![image](https://hackmd.io/_uploads/B1Ns8yKRlg.png)



