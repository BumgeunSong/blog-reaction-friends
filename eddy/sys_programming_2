## systemd란?

- init 프로세스란?
    - 리눅스의 부팅 단계를 간단하게 설명해보자.
        - 1) 하드웨어 단계: 전원을 켜고 시스템에 이상이 없는지 체크한다. 디스크에서 부트로더를 읽어온다.
        - 2) 부트로더 단계: 운영체제를 선택할 수 있다. 커널을 실행한다.
        - 3) 커널 단계: 하드웨어 사용에 필요한 드라이버, 모듈을 점검한다. init 프로세스를 실행시킨다.
        - 4) init 단계: 네트워크, 호스트 설정, 디스크 최적화 및 각종 시스템 서비스를 실행.
    - **Init 프로세스**는 커널이 부팅된 뒤에 가장 먼저 실행되는 프로세스다.
        - PID가 1번이다.
        - init만 커널이 실행시키고, 다른 프로세스들은 모두 init, 혹은 init의 자식 프로세스가 실행시키기 때문에 모든 프로세스의 조상이다.
        - 다양한 시스템 및 프로세스의 초기화 및 관리를 수행한다.
        - 네트워킹, 파일 시스템 등 코어 기능을 위한 각종 시스템 서비스를 실행한다.
        - 컴퓨터 사용중 백그라운드에서 돌아가는 서비스를 실행한다.
- system 5 init과 **systemd**
    - 리눅스에는 다양한 init 프로세스가 있다. 그 중 가장 오래된 것은 SysV이고, 현대에 가장 널리 쓰이는 것은 systemd다.
    - 상당수의 배포판들이 오래전부터 사용하던 init을 버리고 systemd로 갈아탔다.
    - Init 시스템은 다양한 시스템 서비스를 실행시켜야 한다.
    - 이 서비스를 순서대로 어떻게 실행시키도록 할 수 있을까?
    - 예를 들어, 마운트가 가장 먼저, 그 다음 네트워킹 그 다음에 로깅 혹은 알람 순으로 시작을 해야한다고 치자.
    - 프로세스 자체에 이 순서를 코딩할 수 있겠지만, 서비스가 추가되거나 순서가 바뀔 때 매번 컴파일을 해야하니 유지보수성이 아주 떨어진다.
    - system 5 init은 서비스를 실행시키기 위해 미리 쉘 스크립트를 적어두고 거기에 맞게 서비스를 차례대로 실행시켰다. 별도의 config 파일에 순서 정보를 기록한다. 해당 파일을 읽어와서 실행한다. 다만 서비스 갯수가 많아지면 config 파일이 매우 복잡해졌고 실행도 느렸다.
    - systemd는 이벤트 방식으로 관리한다.  ‘마운트 서비스가 완료되면, 로깅 서비스를 실행한다' 같은 식으로 설정한다. 가독성이 좋고 관리가 편하다. 서비스 개수가 많거나 새로운 서비스가 추가되어도 다른 서비스에 영향을 주지 않는다.
    - 또 systemd는 병렬 처리를 사용해서 부팅이 빠르다. 조건을 만족한 서비스는 바로 처리하는 식으로 병렬적으로 시작 프로그램을 실행시켜 부팅 속도를 끌어올렸다. 조건을 보고 ‘네트워킹이 끝났으니 로깅과 알람은 동시에 실행하는 식’이다.
    - systemd는 그 외 에도 네임서버 주소 설정이나 DHCP 서버에서 IP를 받아오는 등의 기능까지 추가되어 강력하다.
    - systemd는 ‘유닛’ 단위로 시스템을 관리한다.
- systemctl
    - systemd의 일부인 ***systemctl 유틸을 많이 사용한다.***
    - 서비스의 상태를 보거나 활성화, 시작, 종료 등을 할 수 있다.
    - 그 외에도 부팅 관련 모니터링, 시스템 로그 등 수많은 다양한 기능이 있지만 필요할 때 찾아보도록 하자.
- 데몬이란?
    - 데몬이란 백그라운드 프로세스다.
    - 보통 시스템이 켜질 때부터 실행되어서, 메모리에 상주하면서 네트워크 요청이나 하드웨어 동작등 특정 요청이 오면 즉시 대응할 수 있도록 대기한다.
    - 보통 데몬을 뜻하는 ‘d’를 이름끝에 달고 있다.
    - 모든 데몬은 systemd (init)의 하위 프로세스가 된다.

## 2️⃣ Copy `cp` 명령어 구현

- `cp {source_file} {destination_file}`
- source를 열어서 읽은 다음, destination을 열어서 쓰기 작업을 하면 된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b8fc3ec7-4d1f-4b6d-946b-421346c6c6cd/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/34585d03-830d-4a1b-b4da-00f30441ab23/Untitled.png)

## 3️⃣ 논블록 입출력이 효과적이지 않은 이유: 다중 입출력을 사용하는 이유

- 블로킹이란?
    - 파일에 읽기를 시도했을 때 지금 당장 읽을 수 없으면 보통 대기(block)한다.
    - 다시 말해, 현재 접근불가능하지만, 꼭 얻어야 하는 자원을 기다리는 현상이다.
    - 만약에 파일을 여러개 다뤄야 할 때, 특정 파일에서 블록이 걸리면, 다른 파일 디스크립터의 내용은 읽을 수가 없게 된다.
    - 아주 짧은 시간이라고 하더라도 사용자한테는 불편하고 반응성이 좋지 않다.
    - 예를 들어 하나의 프로세스에서 소켓이나 파이프의 데이터를 읽어야할 경우. 어떤 fd에서 데이터가 먼저 올지 아무도 알지 못한다.
    - 이럴 때 한 곳에 블록이 된다면 불편할 것이다.
- 논블로킹 입출력
    - 하지만 접근불가능하면, 대기하지 않고 그냥 바로 다음으로 넘어가고 싶을 수 있다.
    - 이 경우에는 논블로킹 입출력을 사용한다.
    - 접근불가능하면 바로 return 해버린다.
- 여러 개 파일을 다룰 때 논블록 입출력이 효과적이지 않은 이유
    - 논블록 입출력은 비록 블록은 일어나지 않지만, 입출력 접근이 가능해지는 시점을 알 수가 없다. 논블록 입출력은 프로그래밍 작업이 까다롭다.
    - 여러 개의 파일 중 어떤 것이 접근가능해지는지 알 수는 없다. 따라서 파일에 접근하려면 계속해서 임의로 재시도를 해보는 수밖에 없다.
    - 이런 방법보다는 입출력 준비가 되면, 알아서 그 시점을 알려주는 입출력이 더 좋을 것이다.
    - 바로 다중 입출력이다.
- 다중 입출력 Multiplexed I/O
    - 다중 입출력은 여러개의 파일 디스크립터를 다룰 때 유용하다.
    - 특정한 파일 디스크립터가 입출력 준비가 되었는지 확인하고 아니라면 대기하다가, 준비가 되면 블로킹 없이 입출력을 처리한다.
    - 작업할 준비가 된 파일에 대해서만 작업을 하면 되기 때문에 편리하다.
    - Linux엔 대표적으로 `select`, `poll`, `epoll` 시스템 콜이 있다.

## 4️⃣ 다중 입출력: Select, poll 사용 (named pipe)

## Select()

```swift
#include <sys/select.h>

int select(int n,
		fd_set *readfds,
		fd_set *writefds,
		fd_set *exceptfds,
		struct timeval *timeout);

```

- select 호출은 파일 디스크립터가 입출력을 수행할 준비가 되거나, 옵션으로 정해진 시간이 지날 때까지만 블록된다.
- select가 지켜보는 파일 디스크립터의 집합
    - 1) read file descriptor set: 여기 있는 파일은 읽기가 가능한지 지켜본다.
    - 2) write file descriptor set: 쓰기가 가능한지 지켜본다.
    - 3) except fd set: 예외가 발생하는지 지켜본다.
- select 호출을 하고 나면, 인자로 넘긴 각 집합은, 각 그룹의 조건이 가능한 fd만 남도록 값이 변경된다.
    - 예를 들어, readfds에 7,9 를 넣었다면, 호출 후 7만 남아있을 수 있다.
    - 7번은 블록없이 읽기가 가능하다는 뜻이다.
- `n` 이라는 인자는 파일 디스크립터 집합에 있는 fd 중에서 가장 큰 숫자에 1을 더한 값이다.
- `timeout` 은 `timeval` 라는 구조체다. 초와 ms를 나타내는.
    - 이 시간만큼 지났는데 입출력이 준비된 fd가 없으면 그냥 return이 된다.
    - 만약 timeout이 0이면, 호출은 즉시 return되고 호출한 그 시점에서 준비된 fd만 알려준다.
- 파일 디스크립터 집합 조작하기
    - 직접 조작하지 않고 매크로를 사용한다.
    - FD_ZERO
        - 집합 내 fd를 모두 제거한다.
        
        ```c
        fd_set wrtiefds;
        FD_ZERO(&writefds);
        ```
        
    - FD_SET
        - 집합에 fd를 추가한다.
    - FD_ISSET
        - 집합에 fd가 존재하는지 검사한다.
        - select 호출이 끝나고 fd가 입출력이 가능한 상태인지 확인하기 위해 사용된다.
        
    
    ```c
    #include <stdio.h>
    #include <unistd.h>
    #include <sys/time.h>
    #include <sys/types.h>
    
    #define TIMEOUT 5
    #define BUF_LEN 1024
    
    int main(void) {
    	struct timeval tv;
    	fd_set readfds;
    	int ret;
    
    	FD_ZERO(&readfds);
    	FD_SET(STDIN_FILENO, &readfds);
    
    	
    	/* Wait up to five seconds. */
    	tv.tv_sec = TIMEOUT;
    	tv.tv_usec = 0;
    	
    	/* All right, now block! */
    	ret = select (STDIN_FILENO + 1,&readfds,NULL,NULL,&tv);
    	if (ret == −1) {
    		perror ("select");
    		return 1;
    	} else if (!ret) {
    		printf ("%d seconds elapsed.\n", TIMEOUT);
    		return 0;
    	}
    	
    	/** Is our file descriptor ready to read?
    	* (It must be, as it was the only fd that
    	* we provided and the call returned
    	* nonzero, but we will humor ourselves.)*/
    	
    	if (FD_ISSET(STDIN_FILENO, &readfds)) {
    		char buf[BUF_LEN+1];
    		int len;
    		
    		/* guaranteed to not block */
    		len = read (STDIN_FILENO, buf, BUF_LEN);
    		if (len == −1) {
    			perror ("read");
    			return 1;
    		}
    		
    		if (len) {
    			buf[len] = '\0';
    			printf ("read: %s\n", buf);
    		}
    		return 0;
    	}
    
    	fprintf (stderr, "This should not happen!\n");
    	return 1;
    	}
    }
    ```
    
    ## Select 사용해보기
    
    ```c
    // sender.c
    
    #include <fcntl.h>
    #include <unistd.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <string.h>
    
    #define PIPENAME "./named_pipe"
    #define MSG_SIZE 1000
    
    int main() {
    
    	char input[MSG_SIZE];
    	int fd_text = open("./message.txt", O_RDONLY);
    	if (fd_text == -1) {
    		perror("Open text file");
    		exit(1);
    	}
    
    	int ret = read(fd_text, input, MSG_SIZE);
    
    	if (ret == -1) {
    		perror("Read text file");
    		exit(1);
    	}
    
    	input[ret] = '\0';
    
    	printf("Sender Sent: %s\n", input);
    
    	close(fd_text);
    
    	mkfifo(PIPENAME, 0666);
    	int fd_writer = open(PIPENAME, O_WRONLY);
    	write(fd_writer, input, strlen(input)+1);
    
    	close(fd_writer);
    
    	return 0;
    }
    ```
    
    ```c
    // receiver.c
    
    #include <fcntl.h>
    #include <unistd.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <string.h>
    #include <time.h>
    
    #define TIMEOUT 5
    #define MSG_SIZE 100
    #define BUF_LEN 1024
    
    int main(int argc, char *argv[]) {
    
    	struct timeval tv;
    	fd_set readfds;
    	int ret;
    
    	int fd_reader = open("named_pipe", O_RDONLY | O_NONBLOCK);
    	if (fd_reader == -1) {
    		perror("Open reader");
    		exit(1);
    	}
    
    	FD_ZERO(&readfds);
    	FD_SET(fd_reader, &readfds);
    
    	tv.tv_sec = TIMEOUT;
    	tv.tv_usec = 0;
    
    	ret = select(fd_reader+1, &readfds, NULL, NULL, &tv);
    
    	if (ret == -1) {
    		perror("select");
    		return 1;
    	} else if (!ret) {
    		printf("%d seconds elapsed.\n", TIMEOUT);
    		return 0;
    	}
    
    	if (FD_ISSET(fd_reader, &readfds)) {
    		char buf[BUF_LEN+1];
    		int len = read (fd_reader, buf, BUF_LEN);
    
    		if (len == -1) return 1;
    
    		if (len) {
    			buf[len] = '\0';
    			printf("Received: %s\n", buf);
    		}
    
    	}
    
    	unlink("named_pipe");
    	return 0;
    }
    ```
    
    ### 결과
    
    → 왼쪽에서 보낸 메시지가 오른쪽에서 읽어짐
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d03cf792-8ee9-4440-9bf7-326837d80e73/Untitled.png)
    

## Poll()

- 시스템 V에서 제공하는 다중 입출력이다. `poll()` 은 select()의 단점을 보완한다.

```c
#include <poll.h>

int poll (struct pollfd *fds, nfds_t nfds, int timeout);
```

- poll은 `pollfd`라는 구조체 배열을 사용한다.

```c
#include <poll.h>

struct pollfd {
	int fd;
	short events;
	short revents;
}
```

- pollfd 구조체는 지켜보려고 하는 하나의 fd를 명시한다.
- pollfd를 여러개 넘길 수도 있다.
- 각 구조체의 events 필드는 각 fd에서 감시할 이벤트의 비트마스크다.
- 예를 들어, 파일 디스크립터의 읽기와 쓰기를 감시하려면, events를 `POLLIN | POLLOUT`으로 설정한다.
- 호출이 반환되면, 원하는 파일 디스크립터의 pollfd를 찾아 `revents` 에 해당 플래그가 켜져있는지 확인한다.
- POLLIN이 설정돼있다면, 이 파일은 정상적으로 읽을 수 있다는 뜻이다.

```c
#include <stdio.h>
#include <unistd.h>
#include <poll.h>

#define TIMEOUT 5

int main(void) {
	struct pollfd fds[2]; // pollfd의 배열을 선언
	int ret;

	/* 표준 입력에 읽기가 가능한지 감시하는 follfds */

	fds[0].fd = STDIN_FILENO;
	fds[0].events = POLLIN;

	// 표준 출력에 쓰기가 가능한지 감시하는 follfds

	fds[1].fd = STDOUT_FILENO;
	fds[1].events = POLLOUT;

	// poll 호출
	ret = poll(fds, 2, TIMEOUT * 1000);
	if (ret == -1) {
		perror("poll");
	}

	if (!ret) {
		printf("%d seconds elapsed.\n", TIMEOUT);
		return 0;
	}
	
	// fds.revents에 POLLIN이 있는지 검사
	if (fds[0].revents & POLLLIN) {
		printf("stdin is readable");
	}

	// fds.revents에 POLLOUT이 있는지 검사
	if (fds[1].revents & POLLLOUT) {
		printf("stdout is writable");
	}

}
```

## Poll 사용해보기

- sender 코드는 똑같고, receiver만 약간 다름.

```c
// receiver.c

#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <time.h>

#define TIMEOUT 5
#define MSG_SIZE 100
#define BUF_LEN 1024

int main(int argc, char *argv[]) {
	struct pollfd fds[2];
	int ret;

	int fd_reader = open("named_pipe", O_RDONLY | O_NONBLOCK);
	if (fd_reader == -1) {
		perror("Open reader");
		exit(1);
	}

	// 파이프에서 읽기 이벤트를 기다림.
	fds[0].fd = fd_read;
	fds[0].events = POLLIN;

	ret = poll(fds, 2, TIMEOUT*1000);
	if (ret == -1) {
		perror("Poll");
		return 1;
	} else if (!ret) {
		printf("%d seconds elapsed.\n", TIMEOUT);
		return 0;
	}

	// Revents에 읽기 이벤트가 등장하면 읽기 시작
	if (fds[0].revents & POLLIN) {
		char buf[BUF_LEN+1];
		int len = read (fd_reader, buf, BUF_LEN);

		if (len == -1) return 1;

		if (len) {
			buf[len] = '\0';
			printf("Received: %s\n", buf);
		}
	}

	unlink("named_pipe");
	return 0;

}
```

## Poll과 Select 비교

- Poll의 좋은점
    - 파일 디스크립터 값에 1을 더해서 인자로 전달할 필요가 없다.
    - 파일 디스크립터 숫자가 많을 때 효율적이다. select()의 경우 파일 디스크립터 900 을 감시한다고 할 때, 커널이 매번 전달된 파일 디스크립터 집합에서 900번째 비트까지 검사해야 한다.
    - select의 파일 디스크립터 집합은 크기가 정해져있다. 적으면 감시할 수 있는 fd 수가 적어지고, 많으면 비트마스크 연산이 비효율적이다.
    - select()를 사용하면 파일 디스크립터 집합이 변경되므로, 호출을 다시 하려면 fd set을 다시 초기화해야 한다. poll은 events와 revents가 나누어져있어 변경 없이 재사용이 가능하.
- Select()의 좋은 점
    - select가 이식성이 높다. 몇몇 유닉스 시스템이 poll() 지원하지 않는다.
    - select는 타임아웃 값을 마이크로세컨드까지 지정할 수 있다.
- 이 둘보다 더 뛰어난 것이 리눅스의 epoll() 이다.
    - 이건 다음에 살펴보도록 하자.