---
title: Writeup for pwn.college
tags: [ctf]

---


![image](https://hackmd.io/_uploads/BJBJvdQZ0.png)

> Date: XX/11/2023
Flow control
Sure, here are some common registers in the x86 and x86-64 architectures and their typical usages:

**EAX/RAX (Accumulator):** This register is often used in arithmetic and logic operations. It’s also used for I/O port access, and interrupt calls4. In the context of function calls, it’s used to store the return value4.

**EBX/RBX (Base):** This register is often used as a base pointer for memory access1.

**ECX/RCX (Counter):** This register is often used as a loop counter and in shift/rotate instructions3.

**EDX/RDX (Data):** This register is used in arithmetic and I/O operations3. It’s also used with EAX/RAX for multiply and divide operations involving large values2.

**ESI/RSI (Source Index) and EDI/RDI (Destination Index):** These registers are used for string and array operations1. In the context of function calls, they’re used to store the second and first parameters respectively1.

**EBP/RBP (Base Pointer) and ESP/RSP (Stack Pointer):** These registers are used in stack operations2. EBP/RBP is often used as a base pointer for accessing local variables and function parameters on the stack2. ESP/RSP is used as a stack pointer, pointing to the top of the stack2.

**EIP/RIP (Instruction Pointer):** This register points to the next instruction to be executed2.

>Please note that the specific use of these registers can vary depending on the context and the specific assembly instructions being used.

>Date:2X/11/2023 
#### *Tra mạng về system call*

In the context of *Linux system calls* on an x86 architecture, **mov eax, 4** is used to specify the write system call1. Here’s a breakdown of what each part does:

**mov eax, 4:** This sets the value of the eax register to 4, which is the system call number for write.

**mov ebx, 1:** This sets the ebx register to 1, which specifies that the output should go to stdout (the terminal).

**mov ecx, variable:** This *sets* the ecx register to point to the data that you want to write.

**mov edx, 1:** This *sets* the edx register to the length of the data that you want to write.

**int 0x80:** This *triggers* the system call interrupt, which transfers control to the kernel1.

Please note that this is specific to Linux on an **x86 architecture1**. Other operating systems or architectures may use different conventions1.

#### Linux command
 
     let pc=4498
     
![image](https://hackmd.io/_uploads/SJ35d_mWA.png)

    
>23/11/2023
Viết writeup cho pwn.college 

### Template python: 
```python
    import pwn
    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
    int 3
    """
    ))
    print(process.readallS())
```
> Linux command : **ipython**
### Level 2: Set multiple registers
**Problem**:
![image](https://hackmd.io/_uploads/r1EGLsm-A.png)

    import pwn
    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
        mov rax, 0x1337
        mov r12, 0xCAFED00D1337BEEF
        mov rsp, 0x31337
    """))   
    print(process.readallS())


### Level 3: Tìm hiểu về phép cộng - trừ - nhân

 ![image](https://hackmd.io/_uploads/BJ16udX-C.png)

### Level 4: Multipilcation 

![image](https://hackmd.io/_uploads/Hk29tiXWC.png)


![image](https://hackmd.io/_uploads/SyHFFjQ-C.png)
```python
    import pwn
    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
        imul rdi,rsi
        add rdi,rdx
        mov rax, rdi
    """))   
    print(process.readallS())
```
flag: pwn.college{Qi24oRY5i2oAk0HiL0m0N_hgfgL.0lN5EDL1MDOxQzW}

### Level 5: Tìm hiểu về phép chia
	Thông tin chung:
 ![image](https://hackmd.io/_uploads/HJ86udmWC.png)

Từ đây ta biết: 
     
1. Ban đầu số bị chia nằm ở lower 64 bits (hay **rax**).Kết quả trả về cũng chính là ở **rax** 
2. Phần dư sẽ được lưu trong **rdx**


>Chat GPT:
**64-bit division DIV operand64** ; 
Divides RDX:RAX by operand64, result in RAX (quotient/ thương), remainder (số dư) in RDX

*Note: You must be careful about what is in rdx and rax before you call div.*

#### Problem:
![image](https://hackmd.io/_uploads/H1z_Fd7WC.png)

 
#### Solution:

Đề yêu cầu tính *speed = **dis** / time* thì ta đã biết **rdx** (upper 64-bits/remainder) và **rax**(lower 64-bits/quotient) lần lượt là phần dư và thương (hay dividend/số bị chia). Vì vậy rax = rdi(**dis**) còn rdx = 0 

    import pwn
    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
    mov rdx, 0
    mov rax, rdi
    div rsi
    """
    ))
    print(process.readallS())
### Level 6: Modulus

![image](https://hackmd.io/_uploads/rkt6joXZ0.png)
```python
    import pwn
    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
        mov rax, rdi
        div rsi
        mov rax, rdx
    """))   
    print(process.readallS())
```
 Tương tự với chia thông thường vì nó sẽ rdx là remainder.
> Note: rax là số bị chia/thương
>  
flag: pwn.college{kpb00-Ll4mu2PXrKDZKMx-GE6f9.0FO5EDL1MDOxQzW} 
### Level 7:
#### Problem:
![image](https://hackmd.io/_uploads/BkF_VhmbA.png)
![image](https://hackmd.io/_uploads/HkjK4hmbA.png)
#### Solution:
    import pwn
    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
        mov ah, 0x42

    """))   
    print(process.readallS())


flag: pwn.college{EM1GJiQ8tAdcNc08CTJoAGSZELY.dFTM4MDL1MDOxQzW}

### Level 8: Tìm hiểu về mod 2^n và các sub register của rax, rdi, rsi 

#### Thông tin chung

Hóa ra sử dụng div để tính *“%”* thì khá chậm. Khi tính x % y với y là ở dạng  2^n thì kết quả của phép tính sẽ là n bits thấp nhất của x.
![image](https://hackmd.io/_uploads/BkZBJYX-C.png)


	Tại sao lại như vậy? 

> **Chat GPT:**
![image](https://hackmd.io/_uploads/S1AhJKQbC.png)

Ta biết một số y = 8  ở dạng  2^3 biểu diễn bits của nó sẽ là: 1000 thì trong trường hợp x = 23 nó sẽ được biểu diễn theo dạng  x=2.2^3+7 thì khi ta lấy ra 2^3 từ 23 phần còn lại là phần nguyên 2 và phần dư 7. Biểu diễn trên bits thì đó chính là 111 trong 10111 của 23. 


**Problem**: 
 ![image](https://hackmd.io/_uploads/Hk_Mtr7bR.png)
![image](https://hackmd.io/_uploads/HJhGFS7bR.png)

#### Solution: 
    import pwn

    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
    mov al, dil
    mov bx, si 
    """))
    print(process.readallS())

Quay lại với bài này, vì chỉ có thể dùng “mov” để lấy n bits sau một hồi search trên mạng về register rdi và rsi thì tôi nhận ra nó có một số điểm đáng chú ý: 
 ![image](https://hackmd.io/_uploads/ryIeKBmbA.png)

> Note: Tưởng tượng các register 64 bits khác cũng như thế này được lưu chồng lên nhau
 
![image](https://hackmd.io/_uploads/B1ASZY7Z0.png)

> 256 và 65536 tương đương với 2^8 và 2^16 có nghĩa là cần lấy 8 bits và 16 bits cuối. 

Để lấy 8 bits hoặc 16 bits cuối theo đề bài ta chỉ cần quan tâm đến 2 register 8 bits và 16 bits *dil* và *si* của *rsi* và *rdi*.  

Thì lúc này ta đơn giản chỉ là chuyển 8 bits cuối(dil) của rdi sang 8 bit cuối của rax (al) và 16 bits cuối (sil) của rsi sang 8 bits cuối (dl) của rdx.

### Level 9: Bitwise shift
Trong bài này ta nói về instructions shl (shift left) và shr(shift right) 
![image](https://hackmd.io/_uploads/ByX1YrXZC.png)

    

Khi ghi như shl/shr reg1, reg2 có nghĩa là dịch reg2 sang trái/phải đúng giá trị reg1 bits (lưu ý 1 byte = 8 bits)
 ![image](https://hackmd.io/_uploads/r1_ktSmZR.png)


Trong đề yêu cầu ta lấy byte LSB (least significant byte) thứ 5 (nghĩa là từ phải sang trái đếm qua). Nhìn vào rdi ta thấy nó có 8 byte (1 byte biểu diễn 2 số) thì ta sẽ shift sang trái 4 byte để xóa 4 byte bên trái và shift sang trái 7 byte!
	Tại sao lại 7 byte nhỉ chẳng phải là chỉ có 3 byte thôi sao?
 ![image](https://hackmd.io/_uploads/ryLLOSQbR.png)

Có lẽ là bởi vì bộ nhớ có 8 byte trống nên khi dịch qua phải 4 byte thì phần còn lại vẫn giữ nguyên là 7 byte + 1 byte đáp án. Giống như 1 cái ống nhựa chứa 8 quả bóng vậy, dù ta đã lấy ra bớt  ra 4 quả ở “bên phải” (thì ta phải nghiêng để tất cả bóng đều chạy sang phải) thì kích thước nó vẫn không thay đổi, nên khi ta muốn lấy ra bớt tiếp 4 quả ở bên trái ta phải đi 1 quảng đường trong ổng tương đương 7 quả bóng.  
### Level 10: Bitwise and
#### Solution
    import pwn

    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
    and rdi, rsi
    xor  rax, rax
    or rax, rdi  
    """))
    print(process.readallS())

Ý tưởng của tôi là sẽ lưu đáp án trong rdi = rdi and rsi sau đó xóa giá trị trong rax bằng xor (not or) rax với chính nó. Cuối cùng là lưu đáp án bằng or  
### Level 11: Bitwise logic
 ![image](https://hackmd.io/_uploads/rJr4drQ-A.png)

#### Solution

    import pwn

    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
    xor rdi, 1
    and rdi, 1
    xor rax, rax
    or rax, rdi
    """))
    print(process.readallS())

Bài này yêu cầu ta tìm chẳn lẻ thông qua duy nhất and, or, xor. Ở đây tôi nghĩ tới tính chất chẳn lẻ của số biểu diễn dưới dạng nhị phân vì 2^0=1 nên ta chỉ quan tâm bit đầu tiên này(nếu bit này tắt những trường hợp khác chắc chắn chẳn!). Bây giờ ta sẽ xor nó với 1 (00001). 
>	Vì sao lại như vậy?

Hãy tưởng tượng nếu bài toán là Chẳn thì y=0 Lẻ thì y=1 thì nó sẽ dễ hơn rất nhiều vì ta chỉ cần and nó với 000001

VD: Tôi có số 8 là 1000 thì tôi chỉ cần and nó với 0001  thì y sẽ bằng kết quả đó vì 8 chẳn nên y = 0

Mà giờ thì nó không như vậy, ta sẽ đảo các bit bằng xor sau đó làm như bình thường.

### Level 11: Memory reads and writes

Bài này chắc cho ta học về addq [address], value

    import pwn
    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
    mov rax, [0x404000]
    addq [0x404000], 0x1337
    """))
    print(process.readallS())
### Level 12:
![image](https://hackmd.io/_uploads/Syqgu27WR.png)
![image](https://hackmd.io/_uploads/B1u-_3X-C.png)

    mov rax, [0x404000] 
flag: pwn.college{8L2pSXIPKPHTT5tH9ij4xMDxHMd.dJTM4MDL1MDOxQzW}
### Level 16: Read multiple data sizes
#### Problem: 
 ![image](https://hackmd.io/_uploads/ryrAvSmZ0.png)
 
 

#### Solution:
Bài này đang nhắc ta về các phân bố bộ nhớ chồng chéo nhau của bộ nhớ register và cách chia bộ nhớ theo bộ 8 bits 16 bits, 32 bits, 64 bits.

 
![image](https://hackmd.io/_uploads/S1B3wSmbA.png)


 
### Level 17: Dynamic address memory writes
#### Thông tin chung:
 ![image](https://hackmd.io/_uploads/rJ-LwB7ZR.png)
![image](https://hackmd.io/_uploads/r12BwBmbA.png)
![image](https://hackmd.io/_uploads/BklHPHXbR.png)
  

Bài nhắc tới cách lưu dữ liệu trong x86, khi đó nó sẽ bị đảo ngược(nhưng từng bit ở trong byte thì không). Cái này được gọi là **“Little Endian”**
Trong ví dụ khi lưu [0x1330] = 0x00000000deadc0de nó sẽ lần lượt bị đảo thành như hình trên. 

#### Problem:
Bài này yêu cầu ta lưu  
Lưu [rdi] = 0xdeadbeef00001337
Lưu [rsi] = 0xc0ffee0000

 ![image](https://hackmd.io/_uploads/SJmovr7W0.png)


#### Solution:
Ta nhận thấy ta phải giá trị big constant qua rax. Sau đó chuyển ngược lại qua [rdi].

>Big Constant: The constant in question is large, meaning it has a high numerical value or takes up a significant amount of memory.
  

    import pwn
    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
    mov rax,  0xdeadbeef00001337
    mov [rdi], rax
    mov rax, 0xc0ffee0000
    mov [rsi], rax
    """))   
    print(process.readallS())
>code

    ---------------- CODE ----------------
    0x400000:    movabs    rax, 0xdeadbeef00001337
    0x40000a:    mov       qword ptr [rdi], rax
    0x40000d:    movabs    rax, 0xc0ffee0000
    0x400017:    mov       qword ptr [rsi], rax
    --------------------------------------

### Level 18: Consecutive memory reads
Dựa vào tên bài ta cũng phần nào đoán ra được chúng ta phải làm gì trong bài này. 

Tảng mạn: Khi lưu trữ dữ liệu ở một địa chỉ nào đó là chúng sẽ được lưu “trải” trên 1 vùng bộ nhớ chung nên xảy chuyện bị ghi đè là hoàn toàn có thể…

#### Thông tin chung 
Nhắc lại một số kiến thức về little endian nó sẽ lưu ngược dữ liệu input (nhưng 2 bits ở trong thì không ngược) . Nó làm được gì cho chúng ta?
  ![image](https://hackmd.io/_uploads/ry6RISm-A.png)

Bài đã cho thông tin khá đầy đủ, nó giúp ta truy cập những thứ kế bên nhau thông qua **offsets**. 

Ở đây ví dụ khi muốn truy cập byte thứ 5th của từ 1 địa chỉ chúng ta ghi **[address+4]** bởi vì nó đếm từ 0. Đơn giản nhỉ  ;-;
![image](https://hackmd.io/_uploads/H137UrX-R.png)

 
**Problem**: 
 ![image](https://hackmd.io/_uploads/BkKH8H7bA.png)

Rõ ràng là không như vậy rồi, bây giờ đề yêu cầu ta truy cập *2 quad words*(1 quad word là 8 bytes) liên tiếp trong rdi, rồi tính tổng của nó lưu trong rsi. 

Vấn đề là mới ban đầu đọc tôi không hiểu “Load” ở đây là đề muốn ta làm gì với 2 cái quad words đó chẳng phải đều cho sẳn rồi sao? Nhưng sau khi học kiến trúc máy tính thì tôi nhận ra đề đang yêu cầu chúng ta lưu dữ liệu từ 1 register sang memory. 

    ---------------- CODE ----------------
    0x400000:    mov       rax, qword ptr [rdi]
    0x400003:    add       rax, qword ptr [rdi +8]
    0x400007:    mov       qword ptr [rsi], rax
    --------------------------------------
Rõ ràng là phần code trên workspace mà tôi đã dùng là chưa hoàn chỉnh, nó đã bị giản lược 1 số chỗ? Tôi nghĩ từ giờ nên để ý tới phần code chính hơn. 
### Level 19:  
**Problem**: 
 ![image](https://hackmd.io/_uploads/Hk3O4H7bA.png)


Ở bài này ta nói về stack một khái niệm khá quen trong ngôn ngữ bậc cao. 
 ![image](https://hackmd.io/_uploads/SkNYVB7bR.png)
![image](https://hackmd.io/_uploads/rJRK4HXW0.png)
Mô tả về stack 

 ![image](https://hackmd.io/_uploads/rJ7qESmZR.png)

Thì nó giống như chồng đĩa vậy cái nào bỏ lên sau thì phải lấy nó ra trước. 
Liên quan tới stack ta còn có rsp (Stack Pointer Register)

#### Solution 

    pop reg1 -> là lấy giá trị top ra(xóa luôn) lưu trong reg1
    push reg1 -> là đẩy giá trị reg1 vào
    ---------------- CODE ----------------
    0x400000:    pop       rax
    0x400001:    sub       rax, rdi
    0x400004:    push      rax
    --------------------------------------
flag: pwn.college{gFvtFBnzLX9e8nPvaX-ofON3M-H.01NwIDL1MDOxQzW}

### Level 20:
#### Problem and Solution: 
![image](https://hackmd.io/_uploads/Bka4VH7bA.png)
![image](https://hackmd.io/_uploads/SkHHVr7b0.png)

 
 
### Level 21: 
#### Thông tin chung & Problem:

In the previous levels you used push and pop to store and load data from the stack.

However you can also access the stack directly using the stack pointer.

On x86, the stack pointer is stored in the special register, rsp.
rsp always stores the memory address of the top of the stack,
i.e. the memory address of the last value pushed.

Similar to the memory levels, we can use [rsp] to access the value at the memory address in rsp.
 
Chúng ta sẽ dùng [rsp+?] để truy cập vị trí trong stack pointer mà không dùng pop.
 
Đếm trong stack thì mỗi value là dài 1 quad word = 8 byte

#### Solution:

    ---------------- CODE ----------------
    0x400000:    mov       rax, qword ptr [rsp]
    0x400004:    add       rax, qword ptr [rsp + 8]
    0x400009:    add       rax, qword ptr [rsp + 0x10]
    0x40000e:    add       rax, qword ptr [rsp + 0x18]
    0x400013:    shr       rax, 2
    0x400017:    push      rax
    --------------------------------------

flag: pwn.college{wdsAH9gG7GHrVNGzP8-f8Ru5HZQ.0VOwIDL1MDOxQzW}

    ---------------- CODE ----------------
    0x400000:    mov       rax, qword ptr [rsp]
    0x400004:    add       rax, qword ptr [rsp + 8]
    0x400009:    add       rax, qword ptr [rsp + 0x10]
    0x40000e:    add       rax, qword ptr [rsp + 0x18]
    0x400013:    mov       rdi, 4
    0x40001a:    div       rdi
    0x40001d:    push      rax
    --------------------------------------

Rõ ràng thì cách để lấy tổng 4 quad words là giống nhau, khi chia ta có 2 cách: 
1. Đó là phép dịch bits
2. Hoặc chia bằng div  
    
        import pwn
        pwn.context.update(arch="amd64")
        process = pwn.process("/challenge/run")
        process.write(pwn.asm("""
        movq rax, [rsp]
        addq rax, [rsp+8]
        addq rax, [rsp+16]
        addq rax, [rsp+24]
        mov rdi, 4
        div rdi
        push rax
        """))   
        print(process.readallS())
### Level 22:

#### Problem:
![image](https://hackmd.io/_uploads/rJJH_67ZA.png)
![image](https://hackmd.io/_uploads/H1TIOpXW0.png)
![image](https://hackmd.io/_uploads/Byc_dpmb0.png)
#### Solution:

    push 0x403000 
    ret 

flag: pwn.college{kwRjrB7CxR4_maawHAbFgpnlPZ6.dVTM4MDL1MDOxQzW}

### Level 24: 
#### Thông tin chung:
>Ta sẽ tìm hiểu về “rip” - the instruction pointer
 ![image](https://hackmd.io/_uploads/HJ8p52Q-A.png)

#### Problem: 
1. Đề yêu cầu ta tạo instruction đầu tiên là jmp tới 1 vị trí cách vị trí hiện tại 0x51 bytes.
2. Ở vị trí $+0x51 đó thì thực hiện đặt rdi bằng giá trị đầu của stack.
3. Sau đó thì jmp tới 0x403000, 1 địa chỉ cố định.   
 ![image](https://hackmd.io/_uploads/B1HCq2m-0.png)


#### Thông tin cần biết:
1. Absolute jump
>In order to perform an absolute jump, we have to specify the address to jump to instead of a label.

#### Code:
    -----------------
    jmp 0x10
    .
    .
    .
    0x10
    -----------------
Vấn đề của cách này là “endianness” nó sẽ lưu ngược các bits địa chỉ nhập vào.  Có 2 cách xử lý.

We can first copy the address in a register and then provide the register as the operand. (Chúng ta có thể copy địa chỉ trong 1 register rồi sử dụng register như một toán hạng)

#### Code:
    -----------------
    mov rax, 0x403000
    jmp rax
    -----------------
For the second method we have to understand the how the ret instruction works in tandem with the instruction pointer. (Ở cách thứ 2 thì chúng ta phải hiểu được cách instruction ret hoặc động song song với instruction pointer (rip) )

2. Instruction pointer: 

The instruction pointer is a register that holds the address of the instruction to be executed next, thus pointing to it.

--------------------------------------------------------------
                0x00    Instruction 1    
                0x01    Instruction 2     Already executed
                0x02    Instruction 3    
    rip ------> 0x03    Instruction 4    ## To be executed

--------------------------------------------------------------

#### Ret Instruction: 
When we use the ret instruction, it pops the latest value on the stack into the instruction pointer rip.

                +------------------+
    RSP+0x18    |       0x00       | <------ rbp 
                +------------------+                                                       
    RSP+0x10    |       0x01       | 
                +------------------+                                                           
    RSP+0x08    |       0x02       |
                +------------------+
    RSP         |       0x03       | <------ rsp  
                +------------------+
   
                    after ret
                        
                +------------------+
    RSP+0x10    |       0x00       | <------ rbp 
                +------------------+                                                       
    RSP+0x08    |       0x01       | 
                +------------------+                                                           
    RSP         |       0x02       | <------ rsp
                +------------------+
    Repeat Loop:

#### Lặp lại code nhiều lần theo ý muốn.
--------------------------------------
    .rept (number of times to be repeated)

    code

    .endr

--------------------------------------


#### Solution & Code: 

	First line, we will jump to the 0x51 position. 

    +-----------+
    jmp thejump
    +-----------+

	And then we can define thejump bellow. 
    +-----------+
    thejump:
        pop rdi
    push 0x403000
    ret
    +-----------+

	However, remember to create the “filter” first by using repeat loop.
    +-----------+
        .rept 0x51
        nop
        .endr
    +-----------+
#### Code:
    import pwn
    pwn.context.update(arch="amd64")
    process = pwn.process("/challenge/run")
    process.write(pwn.asm("""
    jmp thejump
    .rept 0x51
    nop
    .endr
    thejump:
        pop rdi
        push 0x403000
        ret

    """))   
    print(process.readallS())


