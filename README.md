# CryptoWall Analysis
## Download
1. Packed Version -> `47363b94cee907e2b8926c1be61150c7`
2. Unpacked Version -> `919034c8efb9678f96b47a20fa6199f2`

## About CryptoWall 3.0
A strain of a Crowti ransomware emerged, the variant known as CryptoWall, was spotted by researchers in early 2013. The interesting spin to these infections is that the malware communicates over the I2P anonymity network. This typical attack will demand Bitcoins and direct its C2 (command and control) over the Tor network, and send victims to darknet websites to decrypt the corrupted files once a key has been bought. A file named HELP_DECRYPT (ransom note) will appear in the form of either an html, png, or txt file that will direct the victims to the sites mentioned above. The victim's files are encrypted using a RSA-2048 bit algorithm. CryptoWall's initial attack is a loader executable that goes through various stages of code, data, and resource segment decryption processes to ultimately load the main PE executable (which contains the actual malicious code) and inject the file into its own process.

The malware's first few steps after injecting threads into explorer and svchost are to generate a 63 byte campaign string which consists of campaign name/ID, computer host's info that gets MD5 hashed, privilege information, and the public IP address of the victim; spotted like this: `{1|crypt1|C6B359277232C8E248AFD89C98E96D65|0|2|1||55.45.75.254}`. The ransomware does not send it back to the C2 in this context, it will transform that text blob into an encrypted string by using an RC4 encryption algorithm that will end up looking like this (which is 63 bytes in size as well, a big RC4 hint): `85b088216433863bdb490295d5bd997b35998c027ed600c24d05a55cea4cb3deafdf4161e6781d2cd9aa243f5c12a717cf64944bc6ea596269871d29abd7e2`. The RC4 algorithm used to transform this blob is exactly the same one used to decrypt the C2 IP addresses.  

The research analysis can be seen at: {medium article coming soon}

## Decrypt C2 IP Addresses
```
➜  CryptoWall git:(master) python decrypt_c2_ip_addrs.py
[!] Searching PE sections for .data
[+] Extracted encrypted data from PE File

[+] Got plaintext key: 6hehbz4fp

[+] Generated key bytes from plaintext:
bytearray(b'6L]\x84\x80\x8b\x0c,-&"\n\x8cu3A\xb6\x18\xeajs\xe5h\x95n\xc4\xc6\x9c4\x9ag\x12Fz(*\xd3dQX\x92Y\x97\xa9PB\x8e\xd0T\x1f\x1ep\xa8M\x83\xfc\x99:\xd5\t\xdc\\\x85\xb2\xc5\x9ek\xf9)\xaa\x1c\x19\xcc\xc7I\x00\xc1\xec\x1blJ\xb3m~\xa0\'K\xf8\xafc\xb9r>$i!\x98o\x93\x0fS\xeb1\x86\x90\xcd\x9dx\x0eG\xa4\xee=\xc3\xd4fZ\xe6\xda\x10\x89%\xe2/\x91C\x1d\xac\xd7\x02\xa3\xdfqaD\xd1|y\xd2V\xf6w2H\x16\xa7_\xddR\x0b\xc0\xe8\xca \xa5\x05t\xe7\xff\xb7\xe1\xef\xc9\xe3.\xde\xe0\xb4#\r7\xbbU\xfb\xb8`\xdb\xd9\xa1\xab\xf5^\xe9\xed\x1a8\x14\xb0\x04\xbf\x019\xd6\x17\x82\xc8\xae\x13N}\xb5\xd8\xce\xfaW\xbc\xe4@\xf2\xb1e\x15\x88E\xfeO\xfdb\x9f\x085\x96\xa6\xf3\x030\x9b\x87\xad\xf7\x11\xcb\xbe{\x7f?;\xa2\x8f\xba\xf1\x06v\x07\x8a<\x94\xf0\x81\x8d+\xbd[\xc2\xf4\xcf')

Decrypted data:
209.148.85.151:8080
94.247.28.26:2525
94.247.31.19:8080
91.121.12.127:4141
94.247.28.156:8081
```

## Retrieve Ransomware Note from Unpacked binary
```
➜  CryptoWall git:(master) python2 decompress_ransomwarenote.py
[!] Searching PE sections for .data
[+] Found ransomware note
[+] Decompressed successfully
[+] Finished writing to file
```
![Ransomware Note](https://i.ibb.co/r2mk1fc/Screen-Shot-2020-03-21-at-12-43-42-PM.png)

## Decode and Decrypt shellcode loader that injects the final unpacked PE
### When decrypted, you'll notice the shellcode uses WriteProcessMemory to inject the main ransomware exe at offset 0x224b as seen in the dumped asm file. The 3rd argument on the stack (ECX) contains the PE file buffer.
### You can run the script as a shellcode-to-assembly dumper or shellcode emulator.
```
➜  CryptoWall git:(master) python decrypt_shellcode_loader.py -d
[!] Searching PE sections for .rsrc

[+] Successfully extracted encoded shellcode
[+] Successfully decoded encrypted shellcode

0x9b 0xce 0x30 0xc2 0x6 0x43 0x30 0x4c 0x4d 0x9b 0xcd 0xd 0x2 0x43 0x47 0x54
0xa9 0xa9 0xab 0x3c 0x7c 0x85 0xd9 0xd1 0x82 0x77 0x92 0x20 0x9e 0x7b 0x62 0x66
0x66 0x63 0x64 0x97 0xeb 0xab 0x84 0x7f 0xac 0xf8 0xd6 0x71 0xa0 0xf0 0xdc 0x99
0x84 0xa9 0xff 0xb5 0xa6 0x9b 0x43 0x5d 0x7f 0x42 0xb2 0x2 0xf6 0x9b 0x90 0x69


[+] Successfully decrypted shellcode

0x55 0x8b 0xec 0x81 0xc4 0x4 0xf0 0xff 0xff 0x50 0x81 0xc4 0xb8 0xfc 0xff 0xff
0x53 0x56 0x57 0xeb 0x2a 0x36 0x89 0x74 0x24 0x1c 0x36 0xc7 0x44 0x24 0xa 0x1
0x0 0x0 0x0 0x36 0x89 0x4c 0x24 0x12 0x3e 0x8d 0x6a 0x8 0x36 0x89 0x74 0x24
0xe 0x36 0x8b 0x44 0x34 0x2c 0xd3 0xe0 0x1 0xc7 0x36 0x89 0x7c 0x24 0x18 0x64

[+] Using Capstone to Disassemble shellcode to x86
[+] Successfully saved assembly dump file to extractions/pe_process_injector_dump.asm
```
### Emulate Shellcode with SCDbg.exe
```
C:\CryptoWall\>scdbg.exe /s 3200000 /bp WriteProcessMemory /f extractions\pe_process_injector_dump.bin
Loaded 10587 bytes from file ..\pe_process_injector_dump.bin
Breakpoint 0 set at 7c802213
Initialization Complete..
Max Steps: 3200001
Using base offset: 0x401000

4011cf  GetProcAddress(LoadLibraryA)
40165f  GetProcAddress(VirtualAlloc)
401b6d  GetProcAddress(CreateToolhelp32Snapshot)
401b81  GetProcAddress(Process32First)
401b95  GetProcAddress(Process32Next)
401ba9  GetProcAddress(CloseHandle)
401c46  GetProcAddress(GetCurrentProcessId)
401c52  GetCurrentProcessId() = 29
401c62  CreateToolhelp32Snapshot(2, 0) = 4823
401c81  Process32First(4823)
401cc7  Process32Next(4823)
401ccf  CloseHandle(4823)
401cd9  CreateToolhelp32Snapshot(2, 0) = 18be
401cf8  Process32First(18be)
401d3e  Process32Next(18be)
401d46  CloseHandle(18be)
401f40  VirtualAlloc(base=0 , sz=20400) = 600000
40205f  GetProcAddress(GetCommandLineA)
4020d8  GetProcAddress(GetModuleHandleA)
4020e9  GetProcAddress(GetModuleFileNameA)
4020fa  GetProcAddress(VirtualAllocEx)
40210b  GetProcAddress(CreateProcessA)
40211c  LoadLibraryA(ntdll)
402120  GetProcAddress(ZwUnmapViewOfSection)
402131  GetProcAddress(WriteProcessMemory)
402142  GetProcAddress(GetThreadContext)
402153  GetProcAddress(SetThreadContext)
402164  GetProcAddress(ResumeThread)
402175  GetProcAddress(ExitProcess)
402189  GetModuleHandleA()
        Unknown Dll - Not implemented by libemu
40218d  GetModuleFileNameA(hmod=0, buf=12f997, sz=105) = c:\Program Files\scdbg\parentApp.exe
4021b4  GetCommandLineA() = 2531d0
4021ba  CreateProcessA( scdbg.exe  /s 3200001 /bp WriteProcessMemory /f ..\pe_process_injector_dump.bin,  ) = 0x1269
4021c8  ZwUnmapViewOfSection(h=1269, addr400000)
4021e1  VirtualAllocEx(pid=1269, base=400000 , sz=25000) = 621000
        Breakpoint 0 hit at: 7c802213
4021fe  WriteProcessMemory(pid=1269, base=400000 , buf=600000, sz=400, written=12fd70)
4021fe   0FB75F06                        movzx ebx,[edi+0x6]             step: 2685957  foffset: 11fe
eax=1         ecx=600000    edx=600000    ebx=0
esp=12eaa8    ebp=12fdfc    esi=ffffffff  edi=6000b8     EFL 44 P Z

dbg> g
        Breakpoint 0 hit at: 7c802213
40224e  WriteProcessMemory(pid=1269, base=401000 , buf=600400, sz=16400, written=12fd70)
40224e   83C408                          add esp,0x8             step: 2685987  foffset: 124e
eax=1         ecx=600000    edx=401000    ebx=4
esp=12eaa8    ebp=12fdfc    esi=0         edi=6000b8     EFL 4 P
```

## Unpack CryptoWall with r2pipe (Still needs some bugs kinked out)
```
C:\CryptoWall\EMU-Scripts\>python.exe Unpacker-r2.py
Found main: 0x401100

Found Second stage loader: 0x302c940
            ; DATA XREFS from main @ 0x4024c7, 0x402dd4
            ;-- rax:
            ;-- rip:
            0x0302c940      55             push ebp
            0x0302c941      8bec           mov ebp, esp
            0x0302c943      50             push eax
            0x0302c944      b80e000000     mov eax, 0xe                ; 14
        .-> 0x0302c949      81c404f0ffff   add esp, 0xfffff004
        :   0x0302c94f      50             push eax
        :   0x0302c950      48             dec eax
        `=< 0x0302c951      75f6           jne 0x302c949
            0x0302c953      8b45fc         mov eax, dword [ebp - 4]
            0x0302c956      81c494f5ffff   add esp, 0xfffff594

Hold your horses... this may take awhile
Inside Second stage loaders call to EAX: 0x302ca57
Inside Third stage loader: 0x1912a6
Patched Third stage loader debugger check: 0x191451

Found VirtualAlloc is at: 0x76cd66b0
Inside VirtualAlloc Part I: 0x76cd66b0
Inside VirtualAlloc Part II: 0x76cd66b0
Inside WriteProcessMemory Part I: 0x76cd82f2
Inside WriteProcessMemory Part II: 0x76cd82f2

Found dumped PE:
05800000: 4d5a 9000 0300 0000 0400 0000 ffff 0000  MZ..............
05800010: b800 0000 0000 0000 4000 0000 0000 0000  ........@.......
```

## Fake C2 I2P Server
### Run this server before executing the malware (Note: still have to add functionality that fake c2 sends back correct to the ransomware to continue with the file encryption process)
```
C:\CryptoWall\> python.exe fake_c2_i2p_server.py

* Serving Flask app "fake_c2_server" (lazy loading)
* Environment: production
  WARNING: This is a development server. Do not use it in a production deployment.
  Use a production WSGI server instead.
* Debug mode: off
* Running on http://127.0.0.1:80/ (Press CTRL+C to quit)
127.0.0.1 - - [31/Mar/2020 15:10:06] "[33mGET / HTTP/1.1[0m" 404 -

Data Received from CryptoWall Binary:
------------------------------
[!] Found URI Header: 93n14chwb3qpm
[+] Created key from URI: 13349bchmnpqw
[!] Found ciphertext: ff977e974ca21f20a160ebb12bd99bd616d3690c3f4358e2b8168f54929728a189c8797bfa12cfa031ee9c2fe02e31f0762178b3b640837e34d18407ecbc33
[+] Recovered plaintext: b'{1|crypt1|C6B359277232C8E248AFD89C98E96D65|0|2|1||55.59.84.254}'

[+] Sending encrypted data blob back to cryptowall process
127.0.0.1 - - [31/Mar/2020 15:11:52] "[37mPOST /93n14chwb3qpm HTTP/1.1[0m" 200 -
```

## Check if CryptoWall will encrypt file based on file type
```
➜  CryptoWall git:(master) python tor_site_checksum_finder.py --check-file-ext "dll"
[!] Searching PE sections for compressed .data
[!] Searching PE sections for compressed extension .data

[-] '.dll' is not a valid file extension for Cryptowall

➜  CryptoWall git:(master) python tor_site_checksum_finder.py --check-file-ext "py"
[!] Searching PE sections for compressed .data
[!] Searching PE sections for compressed extension .data

[+] '.py' is a valid file extension for Cryptowall
```

## Decrypting Ransomware Infected Files
```
➜  CryptoWall git:(master) wine In_Progress/decrypt_aes_key.exe priv_key_1.pem loveme.txt
[+] Initialized crypto provider
[+] Successfully imported private key from PEM file
[!] Extracted encrypted AES keys from file
[+] Decrypted AES Key => 08020000106600002000000040b4247954af27637ce4f7fabfe1ccfc6cd55fc724caa840f82848ea4800b320
[+] Successfully decrypted key from file

➜  CryptoWall git:(master) python In_Progress/decrypt_file.py loveme.txt 40b4247954af27637ce4f7fabfe1ccfc6cd55fc724caa840f82848ea4800b320
[+] Decrypting file
[+] Found hash header => e91049c35401f2b4a1a131bd992df7a6
[+] Plaintext from file: b'"hello world" \r\n\xfc\xca\xeb\xc8H\xa0\x9c\xcd\xc6\xd4\x90\x11IZ%\xb0'
```
