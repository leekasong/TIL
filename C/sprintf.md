# sprintf 사용시 주의사항

<br />

```
char buf[255] = {0,};
strcpy(buf, "abcdef");
sprintf(buf, "./%s", buf);

```

위 코드의 의도는 buf에 ./abcdef를 저장하는 것이다.

그러나 buf에 들어간 문자열은 ./abcdef가 아니라 ././def이다.

sprintf의 동작은 두 번째 인자의 % 문법을 만날 때까지 문자열을 첫 번째 인자에 복사한다.

세번째 인자인 buf가 %s에 들어갈 때에는 인덱스 0, 1에 각각 '.', '/'가 복사된 상태이다.

따라서 원하는 결과가 나오지 않았다.

해결하려면 변수를 하나 더 쓸 수밖에 없다.

```
char buf[255] = { 0, };
strcpy(buf, "abcdef");
char *p_temp = malloc(strlen(buf) + 1 + 2);
sprintf(p_temp, "./%s", buf);
free(p_temp);

```
