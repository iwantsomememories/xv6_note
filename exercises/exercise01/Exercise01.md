## 1.6 练习

编写一个使用UNIX系统调用的程序，通过一对管道在两个进程之间“ping-pong”一个字节（也就是像打乒乓球一样来回传递），每个方向一个管道。以每秒的交换次数为单位，测量程序的性能。

代码实现如下：

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/time.h>
#include <sys/wait.h>

#define NUM_ITERATIONS 100000 // 定义交换次数

int 
main(int argc, char const *argv[])
{
    int pipe1[2], pipe2[2];
    int cpid;
    char read_buf;
    char write_buf;
    struct timeval start,end;

    if (pipe(pipe1) == -1 || pipe(pipe2) == -1)
    {
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    cpid = fork();
    if (fork() == -1)
    {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (cpid == 0)
    {
        write_buf = 'b';
        for (int i = 0; i < NUM_ITERATIONS; i++)
        {
            read(pipe1[0], &read_buf, 1);
            printf("DEBUG: Child process received %c\n", read_buf);
            write(pipe2[1], &write_buf, 1);
        }
        close(pipe1[0]);
        close(pipe2[0]);
        close(pipe1[1]);
        close(pipe2[1]);

        exit(EXIT_SUCCESS);
    } else {
        gettimeofday(&start, NULL); // 记录开始时间

        write_buf = 'a';
        for (int i = 0; i < NUM_ITERATIONS; i++)
        {
            write(pipe1[1], &write_buf, 1);
            read(pipe2[0], &read_buf, 1);
            printf("DEBUG: Parent process received %c\n", read_buf);
        }

        gettimeofday(&end, NULL); // 记录结束时间

        close(pipe1[0]);
        close(pipe2[0]);
        close(pipe1[1]);
        close(pipe2[1]);

        double elapsed_time = (end.tv_sec - start.tv_sec) * 1000.0; // 时间差（毫秒）
        elapsed_time += (end.tv_usec - start.tv_usec)/1000.0; // 加上微秒部分

        // 计算平均每秒交换次数
        int ping_pong_per_sec = (int)(NUM_ITERATIONS / elapsed_time) *1000.0;
        printf("Exchange %d times in one second\n", ping_pong_per_sec);
    }

    return 0;
}

```

经过测试，每秒交换次数约为11000次。

<img src='.\Figure\Figure-1.png'>