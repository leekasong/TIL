# How to convert char \* to char \*[]

### use standard function 'strtok()'

strtok function breaks string into a series of tokens.  <br />
so, add the tokens to array of char pointer.

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void func(char *src, char **cmd, char *argv[]){
	*cmd = strtok(src, " ");
	int i;
	for(i = 0; i < 20; i++){
		argv[i] = malloc(20);
	}
	char *p_temp;
	for(i = 0; (p_temp = strtok(NULL, " ")) != NULL; i++){
		strcpy(argv[i], p_temp);
	}


}

void main(){

	char *argv[20];
	memset(argv, 0, sizeof(char *) * 20);
	char *cmd;
	char *temp = "ps -al | grep 1";
	char *src = (char *)malloc(30);
	strcpy(src, temp);
	func(src, &cmd, argv);
	int i;

	printf("cmd : %s arg : ", cmd);
	for(i = 0; i < 20 && argv[i] != NULL; i++){
		printf("%s ", argv[i]);
	}
	printf("\n");
	free(src);

	for(i = 0; i < 20 && argv[i] != NULL; i++){
		free(argv[i]);
	}

}

```

#### reference
> https://www.tutorialspoint.com/c_standard_library/c_function_strtok.htm
