# find the command path using 'whereis' and exec system call

### exec familly
use exec family functions. put 'whereis' in the arguments.
<br />
### pipe
use pipe to pass execution results to the parent

```
#include <unistd.h>
#include <stdlib.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>

void main(){
    char *argv[10] = {"/usr/bin/whereis", "whereis", "sort"};
    char buf[100] = {0,};
    int fd[2];
    if(pipe(fd) == -1){
          perror("pipe()");
          return;
    }
    int pid = fork();
    if(pid == 0){
         dup2(fd[1], 1);
         execv(argv[0], &argv[1]);
     }else{
         wait(NULL);
         int nRet = read(fd[0], buf, 100);
         strtok(buf, " ");
         char *p_temp = strtok(NULL, " ");
         printf("read : %s\n", p_temp);
    }
    close(fd[0]);
    close(fd[1]);
}


```
