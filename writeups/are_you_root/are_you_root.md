# Are you Root?

The last time I participated in a CTF was openCTF at Defcon back in August. I decided it had been a little too long. I asked a coworker if there were any good CTFs going on and he pointed me towards PicoCTF hosted by Carnegie Mellon University. I hopped on the team and went to work. The first problem I went after was titled “Are you root?”.

For this problem you’re presented with an address and port, as well as a C source file for the service running on the server. My initial approach was to connect to the server and observe the interface that was presented. 
![image1](/images/image1_cropped.png)

Upon first look, this reminded me of previous problems I had seen that took advantage of a Use-After-Free vulnerability.

Looking through the C source, you will notice an input buffer of size 512. Additionally, user input is taken via a secure fgets. Which rules out a simple buffer overflow, reinforcing the Use-After-Free.

![code1](/images/code1.png)

One aspect of the code that brings attention is how the program creates and frees the user struct. On creation, the user struct is malloc’d with the size of the user struct. But on deletion, only the name pointer in the user struct is freed. Without the rest of the struct being nulled out. The next time a user is created, the space is malloc’d again, but the previously freed memory was not cleared and thus when a new, shorter name is created, ascii in memory is assigned to the auth level.

![image2_cropped](/images/image2_cropped.png)

We now see how we’re able to overwrite auth with a user controlled value. The primary error in the code can be seen in the next three images. A simple struct is defined, space for the struct is malloc'd, then only the name is freed. This leaves user controllable data in the heap, and is left until data fully overwrites it.

*A simple user struct.*

![code2-struct](/images/code2-struct.png)


*When user is created, space for the struct is malloc'd.*

![code2-login-malloc](/images/code2-login-malloc.png)


*When the user is deleted, only the name pointer is freed.*

![code2-free](/images/code2-free.png)

I now just need to find where to write my auth value to specifically.

I compiled auth.c and ran it with GDB to see where the auth value is stored in order to find the point where I could overwrite it.

First, I logged in, and displayed my original auth level. Next, I stopped the program, found the address to the start of the heap and examined memory for 200 words. This showed me where in the heap the user was being stored and allowed me to hone in on where I was looking.

![gdb1_cropped](/images/gdb1_cropped.png)
![gdb2_cropped](/images/gdb2_cropped.png)

Next I continued the program and changed the auth level, repeated the heap inspection, then changed the auth level again. This pointed out the offset in memory where the auth value would be stored and where I should put my user controlled input.

![gdb3](/images/gdb3.png)
![gdb4](/images/gdb4.png)

In the last image, the 6th half byte in the third column of the second row is seen changing, corresponding to the auth value I set. It only sets this one point in memory so I needed to make use of it.

I could now move forward in developing my attack.

In my initial attempt, I tried sending a single byte ‘\x05’ with a prepending 5 ‘A’s. This simply took the ascii value of ‘5’ rather than the actual value 5.
![image3_cropped](/images/image3_cropped.png)

Since no ascii besides an olde timey terminal commands would write 5 to that position, I decided to script it. 

I popped open python with pwntools for a nice socket wrapper and scripted up the commands I needed to enable the use-after-free attack. Only this time, the raw \x05’ byte encoded directly into the username string, reset the user, then recreate a new user with the pre-set authentication value. 

![image2](/images/image-2.png)




