# 내부 클래스에서 static 안되는 이유

```

class outer{
    class inner{
        static int data;
    }
    
    public void func(){
        inner i = new inner();
        inner j = new inner();
    }

}

main(){
    outer a = new outer();
    outer b = new outer();
}

```

static은 클래스가 공유하는 멤버이다. 그렇다면 위 코드에서 어떤 클래스들이 data를 공유하는가?

답은 다양하다. a와 b일 수도 있고, i와 j일 수도 있고, 그 외에 여러 가지 답이 나올 수 있다.

결국 정하기 나름이라는 것이다. 자바 문법을 만드는 사람의 생각에 따라 달라질 수 있다는 뜻이다.

자바를 만든 사람의 생각은 '애매모호하기 때문에 아예 금지한다' 인 것 같다.

그래서 내부 인스턴스 클래스에서 static 멤버를 사용할 수 없게 만들었다.


#### note
> static 상수형 변수는 가능하다. 일반 변수나 메서드는 변경할 수 있는 것들이다. 변수는 값을 바꿀 수 있고, 메서드는 실행 흐름이 달라질 수 있다.
따라서 이들을 누가 소유하고 있는지가 분명하게 정해져야 한다. 하지만 상수는 변경이 불가능하고, 누구나 사용하는 것이기 때문에 어디서 선언하든 상관이 없다.


### reference
https://stackoverflow.com/questions/12727252/why-static-fields-not-final-is-restricted-in-inner-class-in-java