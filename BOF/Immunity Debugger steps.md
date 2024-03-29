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
some.spk will look like:
```bash
s_readline();
s_string("Some command name to test ");
s_string_variable("0");
```
## Step 2: Fuzzing (Send bunch of charecter to break the program)

## Step 3: Finding the offset (At what point it breaks)

## Step 4: Overwriting the EIP


## Step 5: Finding the Bad Charecters


## Step 6: Finding the right module

## Step 7: Generating shellcode

## Step 8: Gain Root





