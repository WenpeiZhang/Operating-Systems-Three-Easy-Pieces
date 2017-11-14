1. Write a program that calls **fork()**.Before calling **fork()**,have the main process access a variable (e.g., **x**) and set its value to something (e.g., 100). What value is the variable in the child process? What happens to the variable when both the child and parent change the value of **x**?

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
Thus, the value of **x** in the child process is still 2017, the same as what is in the main process*(parent)*.

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
We can see that even though both the child and the parent have changed the value of **x**, while child changed it to 2018 and parent changed it to 2019,(inside the respective braces), the actual value of **x** still remains 2017. Thus, I assume that any changes made to the variable by the child and the parent are only effective within the respective braces as written above.
