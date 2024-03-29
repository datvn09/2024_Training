Source:
```
#include <iostream>
#include <string>
#include <Windows.h>
#include <unordered_map>

using namespace std;

#define NT_GLOBAL_FLAG_DEBUGGED (0x10 | 0x20 | 0x40)

int sub4(const string& flag) {
    int answer[18] = { 0 };
    for (size_t i = 0; i < flag.size(); ++i) {
        answer[i] = (int)flag[i];
    }
    return answer[6] + answer[7] + answer[8] - answer[5] + answer[13] == 228 &&
        answer[6] + answer[5] + answer[5] - answer[2] == 179 &&
        answer[9] + answer[3] + answer[2] + answer[5] == 384 &&
        answer[7] + answer[3] - answer[10] - answer[11] == 14 &&
        answer[1] - answer[9] - answer[5] + answer[4] == -40 &&
        answer[2] - answer[3] + answer[5] - answer[12] == 109 &&
        answer[8] - answer[7] - answer[6] + answer[5] - answer[11] == -81 &&
        answer[0] + answer[8] - answer[5] - answer[3] == 23 &&
        answer[5] + answer[6] + answer[8] + answer[2] == 400 &&
        answer[5] - answer[4] - answer[5] + answer[9] == 46 &&
        answer[2] - answer[9] + answer[5] - answer[0] == 34 &&
        answer[2] - answer[5] + answer[4] - answer[9] == -42 &&
        answer[8] + answer[3] + answer[7] - answer[6] - answer[12] == 155 &&
        answer[6] - answer[5] - answer[0] - answer[5] == -242 &&
        answer[2] - answer[5] - answer[6] - answer[4] == -118;
}

std::string custom(int test) {
    const std::string base64_chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789_+@/";
    int n = base64_chars.length();
    std::string custom_chars(n, ' ');

    for (int i = 0; i < n; i++) {
        custom_chars[i] = base64_chars[(i * test + i + n + i * 4) % n];
    }
    return custom_chars;
}

std::string custom_base64_encode(const std::string& input, int test) {
    const std::string base64_chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789_+@/";
    const std::string custom_chars = custom(test);

    std::unordered_map<char, char> encode_map;
    for (size_t i = 0; i < base64_chars.size(); ++i) {
        encode_map[base64_chars[i]] = custom_chars[i];
    }

    std::string encoded;
    for (char c : input) {
        encoded += encode_map[c];
    }
    return encoded;
}

bool sub3(string& flag, int test) {
    string enc = "BbhuN4bn+";
    CONTEXT context = {};
    context.ContextFlags = CONTEXT_DEBUG_REGISTERS;
    GetThreadContext(GetCurrentThread(), &context);
    if (context.Dr0 || context.Dr1 || context.Dr2 || context.Dr3) {
        MessageBox(NULL, L"Debug detection. End program!", L"REV_2024", MB_OK);
        return -1;
    }

    string flag2 = custom_base64_encode(flag, test);
    return flag2 == enc;
}

bool sub2(string& flag, int test) {
    const int SHUFFLE[] = { 8, 3, 2, 6, 1, 5, 0, 4, 7 };
    const int ADD32[] = { 0xEF, 0xBE, 0xFD, 0xDE, 0xAD, 0xDE, 0xE1, 0xFE, 0x37 };
    const int XOR[] = { 0x76, 0x58, 0x44, 0x49, 0x8D, 0x1A, 0x5F, 0x38, 0xD4 };
    int cipher[] = { 108, 109, 55, 123, 72, 43, 10, 66, 16 };
    for (int i = 0; i < 9; i++) {
        cipher[i] = cipher[i] ^ test;
    }
    for (int i = 0; i < 9; i++) {
        flag[SHUFFLE[i]] = (flag[SHUFFLE[i]] + ADD32[i]) ^ XOR[i];
    }

    for (int i = 0; i < 9; i++) {
        if (int(flag[i]) != cipher[i]) {
            return false;
        }
    }
    return true;
}

void sub1(string& flag, int test) {
    for (size_t i = 0; i < flag.size(); i++) {
        flag[i] = flag[i] ^ (test + 1);
    }
}

bool check(string& flag) {
    PDWORD pNtGlobalFlag = (PDWORD)(__readgsqword(0x60) + 0xBC);
    int test = (*pNtGlobalFlag) & NT_GLOBAL_FLAG_DEBUGGED;

    if (test) {
        sub1(flag, test);
        sub2(flag, test);
    }
    else {
        if (sub2(flag, test)) {
            string flag1 = flag.substr(9, 9);
            if (sub3(flag1, test)) {
                string flag2 = flag.substr(18);
                return sub4(flag2);
            }
        }
    }
}

int main() {
    string flag;
    cout << "Enter the flag: ";
    getline(cin, flag);

    if (IsDebuggerPresent()) {
        MessageBox(NULL, L"Debug detection. End program!", L"REV_2024", MB_OK);
        return 1;
    }

    if (flag.length() != 32) {
        MessageBox(NULL, L"Invalid flag length", L"REV_2024", MB_OK);
        return 1;
    }

    if (check(flag)) {
        MessageBox(NULL, L"Success. Flag is correct!", L"REV_2024", MB_OK);
    }
    else {
        MessageBox(NULL, L"Incorrect!", L"REV_2024", MB_OK);
    }

    return 0;
}
```

- Sử dụng UPX (Ultimate Packer for eXecutables) để pack file thực thi đơn giản.
- Ý tưởng:

    Nhập chuỗi kí tự và kiểm tra.
  
    Sử dụng một số kĩ thuật antidebug để phát hiện thay đổi luồng chương trình, thay đổi giá trị mảng giá trị.
  
    Vận dụng các phép toán ADD XOR, mã hóa base64 (với bảng mã custom), giải phương trình với Z3...

Solution:

```
#9 ki tu dau
SHUFFLE = [8, 3, 2, 6, 1, 5, 0, 4, 7]
ADD32 = [0xEF, 0xBE, 0xFD, 0xDE, 0xAD, 0xDE, 0xE1, 0xFE, 0x37]
XOR = [0x76, 0x58, 0x44, 0x49, 0x8D, 0x1A, 0x5F, 0x38, 0xD4]
cipher = [108, 109, 55, 123, 72, 43, 10, 66, 16]
flag = [0] * 9

for i in range(9):
    flag[SHUFFLE[i]] = ((cipher[SHUFFLE[i]] ^ XOR[i]) - ADD32[i]) & 0xFF

for i in range(9):
    print(chr(flag[i]), end = '')

#9 ki tu tiep theo
#tim chuoi base 64 custom test = 0 --> tuong ung voi chuong trinh khong debug
def custom(test):
    base64_chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789_+@/"
    n = len(base64_chars)
    custom_chars = [' '] * n  

    for i in range(n):
        custom_chars[i] = base64_chars[(i * test + i + n + i * 4) % n]

    return ''.join(custom_chars)
#decode
def custom_base64_decode(input_string):
    base64_chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789_+@/"
    custom_chars = custom(0)

    decode_map = {}
    for i in range(len(custom_chars)):
        decode_map[custom_chars[i]] = base64_chars[i]

    decoded = ""
    for c in input_string:
        if c in custom_chars and c != '=':
            decoded += decode_map[c]

    return decoded

print(custom_base64_decode("BbhuN4bn+"), end = '')

#su dung z3 tim cac ki tu cuoi cung
from z3 import *
answer = [Int(f"answer{i}") for i in range (14)]

s = Solver()

s.add(answer[6] + answer[7] + answer[8] - answer[5] + answer[13] == 228)
s.add(answer[6] + answer[5] + answer[5] - answer[2] == 179)
s.add(answer[9] + answer[3] + answer[2] + answer[5] == 384)
s.add(answer[7] + answer[3] - answer[10] - answer[11] == 14)
s.add(answer[1] - answer[9] - answer[5] + answer[4] == -40)
s.add(answer[2] - answer[3] + answer[5] - answer[12] == 109)
s.add(answer[8] - answer[7] - answer[6] + answer[5] -answer[11] == -81)
s.add(answer[0] + answer[8] - answer[5] - answer[3] == 23)
s.add(answer[5] + answer[6] + answer[8] + answer[2] == 400)
s.add(answer[5] - answer[4] - answer[5] + answer[9] == 46)
s.add(answer[2] - answer[9] + answer[5] - answer[0] == 34)
s.add(answer[2] - answer[5] + answer[4] - answer[9] == -42)
s.add(answer[8] + answer[3] + answer[7] - answer[6] -answer[12] == 155)
s.add(answer[6] - answer[5] - answer[0] - answer[5] == -242)
s.add(answer[2] - answer[5] - answer[6] - answer[4] == -118)

if s.check() == sat:
    model = s.model()
    for i in range (14):
        print(f"{chr(model[answer[i]].as_long())}", end = '')
else:
    print("No solution")

#R3verSe_w1th_d@tVn_trA1nIng_2o24
```


      

