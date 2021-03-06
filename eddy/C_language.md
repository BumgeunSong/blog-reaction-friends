# C 언어

## C 언어를 왜 배우지?

- 컴퓨터에 대한 이해
    - 고급 언어에 속하면서도 하드웨어를 직접 제어할 수 있다.
    - C 언어를 활용하려면 컴퓨터에 대한 이해와 숙련도가 있어야 한다.
- 다른 언어에 끼친 영향
    - C 언어는 오래되었다. Java, C#, Obj-c 등 C 언어를 기반으로 만들어진 언어가 많다.
- C 언어는 시스템 프로그래밍을 하는데 사용하기 때문에 컴퓨터와 프로그래밍의 원리를 이해하기에 적합한 언어다.

## ANSI 표준 - C99

C 언어가 발전하면서, 다양한 C 언어 컴파일러가 등장함.

ANSI에 의해서 C언어 표준화가 등장함. 99년도에 만들어진 C99를 대부분 사용한다.

C99부터 한줄 주석 및 자유로운 변수 선언, for 문에서 변수 선언 등이 가능해졌다.

## 기본 개념

`#`은 전처리기다. 전처리기는 컴파일 전에 처리해야하는 명령어다. `#include` 는 컴파일 전에 괄호 안의 것을 추가.

`.h` 는 헤더 파일이다. 헤더파일은 만들어진 함수가 어떤 게 있는지 정리한 목차다. stdio.h는 표준 입출력 헤더 파일이다. 입출력, 반복문, 조건문 등이 있다.

`main()` 은 가장 기본 함수다. 실행하면 main 코드가 실행된다.

main은 `return 0` 을 해야한다.

C 언어에서는 코드가 끝날 때 세미 콜론을 사용해줘야 한다.

## 변수 선언

```c
#include <stdio.h>

//선언 후 대입

int main()
{
  int level;
  int hp;
  int damage;
  int defense;
  
  level = 1;
  hp = 50;
  damage = 5;
  defense = 2;
  
  return 0;
}
```

```c
#include <stdio.h>

//선언과 동시에 대입. 초기화

int main()
{
  int level = 1;
  int hp = 50;
  int damage = 5;
  int defense = 2;
  
  defense = 5; // 대입을 통해 변수의 값을 바꿀 수 있다
  
  return 0;
}
```

## C의 기본 자료형

### 정수

- char 1바이트 ~128개 값.
- short 2바이트
- int 4바이트
- long 4바이트
- long long 4바이트
- (unsigned의 경우 표현 범위가 2배 늘어남. 양수만 표현 가능)

### 실수형

- float, double (8바이트), long double. (16바이트)
- unsigned 없음.

## 변수 프린트하기

```c
printf("a 는 %d이고 b 는 %d이고 c 는 %d입니다.", a, b, c);
```

```c
#include <stdio.h>

int main()
{  
  float a = 1.432f;
  double b = 3.14;

  printf("a 는 %.2f입니다.", a);
  printf("b 는 %.1f 입니다.", b);

  return 0;
}
```

- C의 경우 실수로 값을 적으면 모두 double 타입으로 인식한다. f를 붙여줘야 float로 인식한다. 만약 double 타입으로 인식한 값을 float에 넣으려고 하면 작은 float에 넣으므로 숫자가 잘릴 수 있다는 메시지를 출력한다.
- `%2.f`는 소수점 2자리까지 표시한다는 것을 의미하는 `format specifier`다.
- `const` 는 상수 선언

```c
#include <stdio.h>

int main()
{
	const double PI = 3.1415;

	PI = 5;

	return 0;
}
```

## 전위 연산자와 후위 연산자.

`a++`가 후위, `++a`가 전위. 

후위 연산자는 코드를 먼저 실행하고, 그 줄이 끝나면 그때 값을 변경 시킨다.

## 비교 연산자

C 언어에서는 참이면 1, 거짓이면 0을 반환한다.

## **비트 연산자**

비트 연산은 정수 타입만 가능하며, 실수나 포인터는 비트 연산을 할 수 없다.

AND 연산 &

OR 연산 |

XOR 연산 ^

& 단항 연산자로 쓰이면 (`&b`) 주소값을 가리킨다.

a & b 로 쓰이면 AND 연산자를 나타낸다.

## 비트 이동 연산자

<< 연산자는 지정한 횟수대로 비트 자리를 왼쪽으로 이동시킨다.

왼쪽으로 이동해서 생기는 오른쪽 빈 비트는 0으로 채워진다.

왼쪽에서 밀려나는 비트는 버려진다. 

비트가 한자리 왼쪽으로 이동할 때마다 정수의 값은 2배가 된다. 

3칸을 이동하면 2^3배가 된다. 

하지만 자료형에 따라서 비트 연산은 잘릴 수도 있기 때문에 주의.

## for 문

- 초기식. 시작하는 값
- 조건식. 반복하는 조건 (참인 동안 반복)
- 증감식. 어떻게 변화시킬것인가.
    - 보통 후위 증감 연산자 i++로 표현한다.
- 들여쓰기를 잘하면 중괄호가 없어도 된다.

## while 문

```c
// while 문으로 "Hello, world!\n" 를 5번 출력

int main()
{
	int i = 0;
	while(i<5)
	{
		printf("Hello, world!\n");
		i++;
	}

	return 0;
}
```

## 배열 초기화

```c
int main()
{
  int arr1[5] = {1, 33 , 47, 102, 155}; // 선언과 동시에 초기화
  int arr2[5] = {5}; // 0 번째 값을 5 로 초기화하고 나머지는 모두 0 으로 초기화
  int arr3[5] = {5, 10}; // 0 번째 값을 5, 1 번째 값을 10으로 초기화하고 나머지는 모두 0 으로 초기화
  int arr4[5] = {}; // 모두  0 으로 초기화
  int arr5[5]; // 초기화 하지 않음
  int arr6[] = {11, 22 , 33, 44}; // 배열의 크기가 4로 정해지면서 자동으로 초기화

  return 0;
}
```

배열은 선언한 크기만큼 연속적으로 연결된다.

arr가 int일 경우, 배열 한 원소당 4바이트 크기다. 따라소 주소는 4씩 증가한다.

0번째 원소가 1000이라면, 그 다음 주소는 1004, 1008이 된다.

배열의 크기를 구하려면, 전체 배열의 메모리 길이를 원소 하나의 길이로 나눠줘야 한다.

```c
printf("%d\n", sizeof(arr1) / sizeof(arr1[0]));
```

하나의 글자는 char 자료형으로 표현한다. 1바이트로 1글자만 표현할 수 있다. ascii 코드로 표현한다.

변수에 여러 글자를 담으려면, char 형 배열을 만들어주면 된다. 

char의 배열은 4개의 글자가 들어가면 크기가 5가 된다.

왜냐하면 컴퓨터가 인식할 때 char의 길이를 알기 위해서 마지막 남는 자리에 종료 문자를 넣는다.

그러면 프린트를 할때 종료문자가 있는데까지만 프린트한다.

```c
#include <stdio.h>

int main()
{
  char ch[7] = { 'a', 'b', 'c', 'd', 0, 'e', 'f' };
  
  printf("ch 는 %s", ch);
  
  return 0;
}

> ch 는 abcd
```

**배열의 이름은 주소를 담고 있다.** 

```c
#include <stdio.h>

int main()
{
	char ch[31];
	scanf("%s", ch);
	

	int size = 0;
	
	for (int i=0; ch[i]!= 0; i++) {
		size += 1;
	}
	printf("%d", size);

	return 0;
}
```

## 함수

- 함수에서 전달받은 인자는 매개변수에 복사된다. 즉, 원래 매개변수를 변경해도 원래 인자는 바뀌지 않는다.
- 함수의 형태

```c
[반환형] [함수명] (매개변수)
{
    [호출 시 작동될 함수 내부 코드]
}
```

- 만약 반환형이 없다면 `void` 라는 반환형을 사용한다.
- 함수의 순서
    - 함수 선언이 메인 함수 아래에 있다면, 인식을 하지 못한다.
    - 하지만 실행을 해보면 정상적으로 컴파일 및 실행이 완료된다. 이는 컴파일러의 확장 기능 때문이다.
    - 컴파일러마다 에러를 낼 수도 있다.
- 함수의 원형
    - 반환형, 함수 이름, 인자를 함수 원형이라고 한다.
    - 이것들만 선언하게 되면, 함수 정의하는 코드는 main 함수 아래쪽에 써도 상관없다.
    

## 포인터

- 포인터는 주소를 가리킨다.
- 이름만 포인터이고, int, char와 같이 특정 값을 저장하는 변수다.
- **포인터에는 ‘변수의 주소값'이 저장**된다.

## 포인터의 선언 `*`

- 포인터 변수는 담고자하는 자료형에 * 연산자를 붙여서 선언한다.  변수 이름에 붙여도 되고, 타입 선언에 붙여도 된다.
- 포인터 변수 크기 자체는 담겨있는 자료형과 관련없이 똑같다. 같은 OS라면.
- 하지만 자료형을 알아야하는 이유는, 포인터에 담긴 주소값으로 가서 값을 어떤 단위로 읽어야 하는지 알아야하기 때문이다.
- 아래는 모두 똑같이 포인터를 선언하는 코드
    - `int *p`
    - `int* p`
    - `int * p`

## 포인터의 초기화?

- 변수를 초기화하지 않으면 그냥 메모리 주소만 가리키기 때문에 쓰레기값이 들어가 있다.
- 포인터 변수의 값은 주소값이기 때문에 어떤 특정한 숫자로 할 수 없다. NULL(0) 으로 없다는 것을 표현해준다.
- 되도록 초기화를 해주는 걸 권장한다.
- 주소값을 사용할 떄는 `&`를 붙인다.
- 선언했을 때는 `*p`로 하지만, 주소값을 넣을 때는 `p = &num` 형태로 넣어준다.

```c
#include <stdio.h>

int main()
{
	int *p = NULL;  // int* p == int * p 모두 같음
	int num = 15;

	p = &num;

	printf("포인터 p의 값 : %d \n", p);
	printf("int 변수 num의 주소 : %d \n", &num);

	return 0;
}
```

## `*` 연산자

- `*` 는 피연산자가 하나이면 참조 연산자, 두개이면 곱하기 연산자.
- `&` 는 피연산자가 하나이면 주소 연산자, 두개이면 and 연산자.
- 참조 연산자는 포인터가 가리키는 주소값에 ‘저장된 값'을 반환한다.

```c
#include <stdio.h>

int main()
{
	int *p = NULL;  
	int num = 15;

	p = &num;

	printf("int 변수 num의 주소 : %d \n", &num);
	printf("포인터 p의 값 : %d \n", p);
	printf("포인터 p가 가리키는 값 : %d \n", *p);

	return 0;
}
```

```c
> int 변수 num의 주소 : -142560612 
포인터 p의 값 : -142560612 
포인터 p가 가리키는 값 : 15
```

따라서 포인터 변수 그 자체는 주소값을 의미한다. 

이미 선언한 포인터 변수에 `*` 을 붙이면, **그 주소에 들어있는 실제 값을 의미**한다.

## 연산자 우선순위

`*p += 5`

주소로 가서 들어있는 값을 5 증가시킨다. 

`(*p)++`

p 주소로 가서 들어있는 값을 1 증가시킨다.

`*p++`

연산자에는 우선순위가 있다. 증감 연산자가 참조 연산자보다 우선순위가 높다. 따라서 주소값이 들어있는 변수를 먼저 증가시킨 다음에, 그 주소를 찾아가게 된다.

증가한 주소에는 아무것도 초기화하지 않았으니 쓰레기값이 출력된다.

포인터를 함수에 넘기게 되면, 함수에서도 주소값을 알고 찾아갈 수 있다. 

넘겨진 변수 값을 바로 수정하는 것이 가능하다. (swift의 `inout` 과 같음)

C 언어는 기본적으로 값으로 매개변수를 복사해서 넘겨줌.

하지만 인자에 포인터를 선언하면 복사가 아닌 참조를 넘겨줌.

```c
#include <stdio.h>

void swap(int *a, int *b)
{
	int temp;

	temp = *a;
	*a = *b;
	*b = temp;
}

int main()
{
	int a, b;

	a = 10;
	b = 20;

	printf("swap 전 : %d %d\n", a, b);

	swap(&a, &b);

	printf("swap 후 : %d %d\n", a, b);

	return 0;
}
```

- 매개변수에는 * 연산자를 사용해서 포인터를 인자로 받겠다고 표현.
- 넘겨줄 때는 & 연산자를 사용해서 주소값을 인자로 넘김.
- 그러면 *a = &a 로 할당이 일어나면서, 함수 내부에서 외부 변수의 주소값을 갖게 됨.

## 배열의 이름은 주소값

- 배열의 이름은 포인터와 같은 기능을 한다. 배열의 첫번재 원소를 가리키는 포인터다.
- 따라서 `int *arrPtr = arr;` 로 바로 포인터에 주소값을 할당 가능.
- 일반 변수였다면 &arr 로 해줘야했을 것.
- **배열은 겉으로 보기엔 일반 변수같지만, 사실 포인터다.**

## 포인터 연산

- 포인터 변수도 일반 변수처럼 증감 연산 가능. ++, — 같은 전후위연산, 일반 덧셈 뻴셈 가능. 다만 곱셈 나눗셈은 못함.
- 다만 포인터 변수의 증감 연산은, 1씩이 아니다. 포인터 변수는 ++ 연산자 사용했을 때 증가하는 크기가 자료형에 따라 달라진다.
- int 형 포인터는 4씩 증가, double 형 포인터는 8씩 증가.
- 따라서 i번째 요소의 값을 가리키는 `arr[i]`는, 곧 `*(arr+i)` 와 같다.
- 포인터의 이름은 배열의 첫번째 요소의 주소값이므로, i * 자료형 크기만큼 더해지면 결국 배열 i와 같다.

```c
#include <stdio.h>

// arr[]는 int다. 포인터로 받는다.
void bubbleSort(int arr[])
{
	int temp;	
	for(int i=0; i<9; i++)
	{
		for(int j=0; j<9-i; j++)
		{
			if(arr[j] > arr[j+1])
			{
				temp = arr[j];
				arr[j] =  arr[j+1];
				 arr[j+1] = temp;
			}
		}		
	}
}

int main()
{
	int arr[10];
	for(int i=0; i<10; i++)
	{
		scanf("%d", &arr[i]);
	}

	bubbleSort(arr);

	for(int i=0; i<10; i++)
	{
		printf("%d ", arr[i]);
	}
	
	return 0;
}
```

## 상수 포인터

- 포인터를 한번 선언하면, 주소값을 바꿀 수 없는 상수 포인터가 있다.
- `const int *ptr` (맨 앞에 Const → 포인터 사용 상수화)
    - 포인터를 ‘사용해' 변수의 값을 변경하는 것을 막는다. 오직 포인터가 가리키는 값을 읽을 수만 있다.
    
    ```c
    const int *ptr2 = &num;
    *ptr2 = 40; // error: assignment of read-only location ‘*ptr2’
    ```
    
    - const를 맨 앞에 붙여서 선언한 ptr2 는 이것을 사용해 값을 변경할 수 없음.
    - 하지만 num 자체를 변화시키는 것은 가능하다. num을 변경하는 코드는 에러가 나지 않는다.
    - 이 포인터를 통해서만 변경할 수 없게 되는 것.
- `int* const ptr`  (포인터 연산자 다음에 const → 포인터 자체가 상수화)
    - 포인터 선언시 * 은 어디 붙여도 상관없었다.
    - 하지만 포인터를 상수화시키려면, const 전에 *를 써줘야 한다.
    - 포인터의 주소값을 바꿀 수 없게 된다.
    - 포인터가 가리키는 값을 바꿀 수는 있지만, 포인터가 가리키는 주소는 바꿀 수 없다.
- `const int* const ptr`  (둘 다 사용)
    - 둘 다 사용하게 되면, 주소값도 바꿀 수 없고, 변수값도 바꿀 수 없도록 만든다.

## 이중 포인터

- 이중 포인터는 포인터의 주소값을 담는다.

```c
#include <stdio.h>

int main()
{
	int num = 10;
	int *ptr;
	int **pptr;

	ptr = &num;
	pptr = &ptr;

	printf("num : %d, *ptr : %d, **ptr : %d\n", num, *ptr, **pptr);
	printf("num 주소 : %d, ptr 값 : %d, **ptr 값 : %d\n", &num, ptr, *pptr);
	printf("ptr 주소 : %d, pptr 값 : %d", &ptr, pptr);

	return 0;
}
```

- num에 10이 담겨있고,
- `ptr`는 포인터로 num의 주소값이 담겨있고,
- `pptr`은 이중포인터로 ptr의 주소값이 담겨있다.
- num의 값 = *ptr (ptr이 가리키는 값) = **pptr (ptr이 가리키는 값이 가리키는 값)
- &num (num 주소값) = ptr = *ptr (ptr이 가리키는 값)
- 이렇게 3개가 일치하게 된다.
- 참조연산자를 3개 이상 찍는 것도 가능하지만 현실적으로 거의 사용되지 않는다.

## 포인터 배열

```c
#include <stdio.h>

int main()
{
		int num1 = 10, num2 = 20, num3 = 30;
		int *parr[3];

		parr[0] = &num1;
		parr[1] = &num2;
		parr[2] = &num3;

		for(int i=0; i<3; i++)
		{
			printf("parr[%d] : %d\n", i, *parr[i]);
		}

		return 0;
}
```

- `int parr[3];` 자체는 이미 배열의 주소값을 담은 포인터이므로, 여기에 * 를 붙이면 이중 포인터가 된다.
- 따라서 궁극적으로 가리키는 값은 int지만 그 안에 요소는 포인터들이 담겨있다.
- 포인터를 각각 num1, num2, num3의 주소값으로 저장해준다음, *를 가지고 프린트한다.

arrPtr는 arr의 2번째 요소 주소값을 가리키는 포인터가 된다.

땡. arr의 &을 해준게 아니라, 그냥 arr를 해줬으므로, 그냥 arr와 똑같은 ptr이 된다.

*arrPtr = 3

arrPtr = 1004

arrPtr2++