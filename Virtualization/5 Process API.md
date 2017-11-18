
>1. Write a program that opens a file (with the **open()** system call) and then calls **fork()** to create a new process. Can both the child and parent access the file descriptor returned by **open()**? What happens when they are writing to the file concurrently, i.e., at the same time?


Code1:
```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

int main(int argc, char *argv[])
{
    printf("hello world(pid:%d)\n",(int) getpid());
    int x = 2017;
    int rc = fork();
    if(rc < 0)
    {
        fprintf(stderr, "fork failed\n");
        exit(1);
    }
    else if(rc == 0)//child
    {
        printf("hello, I am child(pid:%d),now x = %d\n", (int) getpid(),x);
    }
    else//parent
    {
        printf("hello, I am parent of %d (pid:%d), now x = %d\n", rc, (int) getpid(),x);
    }
return 0;
}
```
Results1:
```
Peggys-MacBook-Air:~ Peggy$ gcc -o s1 s1.c
Peggys-MacBook-Air:~ Peggy$ ./s1
hello world(pid:11694)
hello, I am parent of 11695 (pid:11694), now x = 2017
hello, I am child(pid:11695),now x = 2017
```
Thus, the value of **x** in the child process is still 2017, the same as that in the main process *(parent)*.

Code2:
```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main(int argc, char *argv[])
{
    printf("hello world(pid:%d)\n",(int) getpid());
    int x = 2017;
    int rc = fork();
    if(rc < 0)
    {
        fprintf(stderr, "fork failed\n");
        exit(1);
    }
    else if(rc == 0)//child
    {
        int x = 2018;
        printf("hello, I am child(pid:%d),now x = %d\n", (int) getpid(),x);
    }
    else
    {   int x = 2019;
        printf("hello, I am parent of %d (pid:%d), now x = %d\n", rc, (int) getpid(),x);
    }
    printf("Actually, the value of x =%d\n",x);
return 0;
}

```
Results2:
```
Peggys-MacBook-Air:~ Peggy$ gcc -o s1 s1.c
Peggys-MacBook-Air:~ Peggy$ ./s1
hello world(pid:11712)
hello, I am parent of 11713 (pid:11712), now x = 2019
Actually, the value of x =2017
hello, I am child(pid:11713),now x = 2018
Actually, the value of x =2017
```
We can see that even though both the child and the parent have changed the value of **x**, while child changed it to 2018 and parent changed it to 2019 (inside the respective braces), the actual value of **x** still remains 2017. Thus, I assume that any changes made to the variable by the child and the parent are only effective within the respective braces as written above.

>2. Write a program that opens a file (with the **open()** system call) and then calls **fork()** to create a new process. Can both the child and parent access the file descriptor returned by **open()**? What happens when they are writing to the file concurrently, i.e., at the same time?


Code:
```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

int main(int argc, char *argv[])
{
    int fd,f_size;
    char str[5]="12345";
    int num =5;
    fd = open("/Users/apple/Desktop/file.txt", O_RDWR | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR);
    if(fd == -1)
       fprintf(stderr,"open failed\n");
    else
    {
        printf("Open/Create file successful.\n");

        int rc = fork();//create a new process
        if(rc < 0)
        {
            fprintf(stderr, "fork failed\n");
            exit(1);
        }
        else if(rc == 0)//child
        {
            printf("hello, I am child, the file descriptor is: %d\n",fd);//child tried to access fd
            char buff[100]="CCCCC";
            write(fd,buff,100);
        }
        else//parent
        {
            printf("hello, I am parent, the file descriptor is: %d\n",fd);//parent tried to access fd
            char buff[100]="PPPPP";
            write(fd,buff,100);
        }
        f_size = write(fd,str,num);//write to the file concurrently
        close(fd);
    }
return 0;
}

```
Results:
```
//Terminal
Peggys-MacBook-Air:~ Peggy$ gcc -o s2 s2.c
Peggys-MacBook-Air:~ Peggy$ ./s2
Open/Create file successful.
hello, I am parent, the file descriptor is: 3
hello, I am child, the file descriptor is: 3


//file.txt
PPPPP...(95 "\u0")12345CCCCC...(95 "\u0")12345  
/*Because the number of bytes to be written into the file was assigned "100", 
while at the end of the program it's assigned exactly "5".*/
```
From the above, we now know that both the child and parent can access the file descriptor returned by **open()**.
According to this example, what happens when they are writing to the file concurrently is that parent will write first then the child write. (Is this concurrent though??)

**Extended Reading: "The Linux Programming Interface" Chapter 5.5*

>3. Write another program using **fork()**. The child process should print *“hello”*; the parent process should print *“goodbye”*. You should try to ensure that the child process always prints first; can you do this without calling **wait()** in the parent?

For starters, let's recall how it can be done using **wait()** ！

Code1:
```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

int main(int argc, char *argv[])
{
    int rc = fork();
    if(rc < 0)
    {
        fprintf(stderr,"fork failed\n");
        exit(1);
    }
    else if(rc == 0)//child
    {
        printf("hello\n");
    }
    else//parent
    {
        wait(NULL);//if parent happens to run first then wait for the child
        printf("goodbye\n");
    }
    return 0;
}

```
Result1:
```
Peggys-MacBook-Air:~ Peggy$ vim s3.c
Peggys-MacBook-Air:~ Peggy$ gcc -o s3 s3.c
Peggys-MacBook-Air:~ Peggy$ ./s3
hello
goodbye
```
In this way, we can ensure the child process always prints first!

Now let's make it more interesting! How can we achieve it w/o calling **wait()** in parent?

Code2:
```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

int main(int argc, char *argv[])
{
    int rc = fork();
    if(rc < 0)
    {
        fprintf(stderr,"fork failed\n");
        exit(1);
    }
    else if(rc == 0)//child
    {
        printf("hello\n");
    }
    else//parent
    {  
       usleep(1000);//1000 microseconds (10^(-3)s) 
       //sleep(1)  1 second  (1s)
       printf("goodbye\n");
    }
    return 0;

```
Results2:
```
Peggys-MacBook-Air:~ Peggy$ gcc -o s3 s3.c
Peggys-MacBook-Air:~ Peggy$ ./s3
hello
goodbye
```
Without using **wait()** , we can use **sleep()** or **usleep()** in the parent procss. In this case, in the parent process, **usleep()** was called. It stopped parent for a certain amount of time, during which the child process started running. As long as child can finish its process within the set length of time, we can ensure that child prints first because once time's up it will switch back to run parent process.(***Actually this depends on the scheduling mechanism of the system***) 

**Extended Reading: "The Linux Programming Interface" Chapter 24*


 >4. Write a program that calls **fork()** and then calls some form of **exec()** to run the program */bin/ls*. See if you can try all of the variants of **exec()**,including **execl()**, **execle()**, **execlp()**, **execv()**, **execvp()**, and **execvP()**. Why do you think there are so many variants of the same basic call?  
 
 Let's try **excvp()** first!

Code:
 ```c
 #include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>
#include<sys/wait.h>

int main(int argc, char *argv[])
{
    int rc = fork();
    if(rc < 0)
    {
        fprintf(stderr, "fork failed\n");
        exit(1);
    }
    else if(rc == 0)//child (new process)
    {
        printf("hello, I am child(pid:%d)\n", (int) getpid());
        char *myargs[3];
        myargs[0] = strdup("ls");//program: "ls"
        myargs[1] = strdup("-l");//argument: executable file
        myargs[2] = NULL;//marks end of array
        execvp(myargs[0],myargs);
        printf("This shouldn't print out");
    }
    else//parent goes down this path (main)
    {
        int ls = wait(NULL);
    }
    return 0;
}

 ```
 
 Results:
 ```
 Peggys-MacBook-Air:~ Peggy$ su
Password:
sh-3.2# gcc -o s4 s4.c
sh-3.2# ./s4
hello, I am child(pid:36317)
total 464
-r--------    1 Peggy  staff      9 Aug  2 08:34 .CFUserTextEncoding
-rw-r--r--@   1 Peggy  staff  32772 Nov 18 16:33 .DS_Store
drwxr-xr-x    3 Peggy  staff    102 Aug  3 07:06 .IdentityService
drwx------    3 Peggy  staff    102 Aug  3 22:56 .ServiceHub
drwx------    6 Peggy  staff    204 Nov 18 15:53 .Trash
drwxr-xr-x    4 Peggy  staff    136 Nov  1  2015 .adobe
drwxr-xr-x    3 Peggy  staff    102 Sep 26 13:59 .anaconda
drwxr-xr-x    3 Peggy  staff    102 Aug  3 07:06 .android
-rw-------    1 Peggy  staff   6656 Nov 18 16:52 .bash_history
-rw-r--r--    1 Peggy  staff     84 Sep 26 13:57 .bash_profile
drwx------  133 Peggy  staff   4522 Nov 18 16:52 .bash_sessions
drwxr-xr-x    3 Peggy  staff    102 Oct 15 20:53 .conda
-rw-r--r--    1 Peggy  staff     40 Oct 15 20:52 .condarc
drwxr-xr-x    6 Peggy  staff    204 Aug  3 07:06 .config
drwxr-xr-x    3 Peggy  staff    102 Sep 26 14:04 .continuum
-rw-r--r--    1 Peggy  staff   1518 Oct 31 20:01 .drjava
drwxr-xr-x    6 Peggy  staff    204 Jan 26  2017 .eclipse
-rw-r--r--    1 Peggy  staff    188 Oct 31 11:14 .gitconfig
drwxr-xr-x    5 Peggy  staff    170 Sep 26 14:00 .ipython
drwxr-xr-x    3 Peggy  staff    102 Sep 26 14:16 .jupyter
drwxr-xr-x    3 Peggy  staff    102 Aug  3 02:10 .mono
drwxr-xr-x    9 Peggy  staff    306 Nov  8 20:31 .oracle_jre_usage
-rw-r--r--    1 Peggy  staff  12288 Nov 18 16:07 .p1.c.swo
-rw-r--r--    1 Peggy  staff  12288 Nov 14 19:37 .p1.c.swp
drwxr-xr-x    5 Peggy  staff    170 Jul 29 03:56 .p2
-rw-------    1 root   staff   1024 Oct 24 18:40 .rnd
-rw-r--r--    1 Peggy  staff  12288 Nov 15 11:14 .s3.c.swp
drwxr-xr-x   40 Peggy  staff   1360 Nov 18 16:29 .sogouinput
drwxr-xr-x    6 Peggy  staff    204 Sep 11  2015 .subversion
-rw-------    1 Peggy  staff  12288 Nov 14 13:52 .swn
-rw-------    1 Peggy  staff  12288 Nov 14 10:33 .swo
-rw-------    1 Peggy  staff  12288 Nov  6 21:02 .swp
drwxr-xr-x    3 Peggy  staff    102 Aug  3 07:06 .templateengine
drwxr-xr-x    3 Peggy  staff    102 Jan 26  2017 .tooling
-rw-------    1 Peggy  staff   8237 Nov 18 16:28 .viminfo
drwxr-xr-x    3 Peggy  staff    102 Nov  8 20:25 Applications
drwx------@  26 Peggy  staff    884 Nov 18 11:09 Desktop
drwx------@  44 Peggy  staff   1496 Nov 13 11:41 Documents
drwx------+  24 Peggy  staff    816 Nov 18 15:53 Downloads
drwx------@  86 Peggy  staff   2924 Nov 18 11:09 Library
drwxr-xr-x    2 Peggy  staff     68 May  8  2017 MacKeeper Backups
drwx------+   7 Peggy  staff    238 Aug  3 02:01 Movies
drwx------+   7 Peggy  staff    238 Aug  4 01:40 Music
drwx------+   4 Peggy  staff    136 Aug  3 02:07 Pictures
drwxr-xr-x+   6 Peggy  staff    204 Aug  4 01:39 Public
drwxr-xr-x    4 Peggy  staff    136 Nov  4 21:07 VirtualBox VMs
drwxr-xr-x   23 Peggy  staff    782 Sep 26 14:36 anaconda
-rwxr-xr-x    1 Peggy  staff   8660 Nov 14 17:04 p1
-rw-r--r--    1 Peggy  staff    399 Nov 15 10:34 p1.c
-rwxr-xr-x    1 Peggy  staff   8660 Nov 14 17:12 s1
-rw-r--r--    1 Peggy  staff    593 Nov 14 17:12 s1.c
-rwxr-xr-x    1 Peggy  staff   8920 Nov 14 20:37 s2
-rw-r--r--    1 Peggy  staff   1083 Nov 14 20:37 s2.c
-rwxr-xr-x    1 Peggy  staff   8660 Nov 15 11:12 s3
-rw-r--r--    1 Peggy  staff    436 Nov 15 11:12 s3.c
-rwxr-xr-x    1 root   staff   8920 Nov 18 16:53 s4
-rw-r--r--    1 Peggy  staff    743 Nov 18 16:28 s4.c
sh-3.2# 

 ```
At first, I encountered `-bash: ./s4.c: Permission denied`. This is because the current user does not have the privilage, we need to switch to "root user" in order to execute the program */bin/ls*. To do this, we can simply call "su" and enter the password. If it says`su: sorry`(under Mac system)，we can change the password and try it again. To change the password, just type the following:
```
sudo su
//enter password
passwd root
//Set new password
//Retype new password
//Done! Password has been changed!
```

   To use the rest of the varients, simply replace the argument：`execvp(myargs[0],myargs);` with the following arguments. See the code block below:
   ```c
   char *envp[]={"PATH=/bin:/usr/bin", "TERM=console", NULL};// This array is a must for some of the funtions listed below
   
   execv("/bin/ls",myargs);
   execl("/bin/ls", "ls","-l", NULL);
   execle("/bin/ls", "ls","-l", NULL, envp);//array envp[] is used here
   execve("/bin/ls", myargs, envp);
   execlp("ls", "ls", "-l", NULL);
   execvp("ls", myargs);
   ```
