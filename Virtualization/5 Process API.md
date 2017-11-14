
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
From the above, we now know that both the child and parent can access the file descriptor returned by open().
According to this example, what happens when they are writing to the file concurrently is that parent will write first then the child write. (Is this concurrent though??)

*Extended Reading: "The Linux Programming Interface" Chapter 5.5*
