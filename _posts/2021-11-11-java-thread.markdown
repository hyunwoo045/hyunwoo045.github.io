---
title: "Java로 Thread 만들기"
excerpt: "Java 로 Thread 를 만들어보자."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Java

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

## 자바로 Thread 만들기

프로그램은 실행되기 전에는 하드 디스크에 저장되어 있다가 실행되면 메모리에 올라간다. 메모리에 올라간 프로그램을 '프로세스' 라고 한다. 프로세스가 수행되기 위해서는 CPU를 점유해야 하는데, 프로세스 자체가 CPU를 점유 하는 것이 아니라 동작 단위로 CPU를 점유하게 된다. 이 때 이 동작을 'Thread' 라고 한다. 간혹 한 프로그램이 여러 동작을 하는 것처럼 보일 때가 있는데, 이는 한 프로그램에서 여러 Thread 를 동시에 수행하고 있는 것이며 이를 '멀티쓰레딩' 이라고 한다.

Thread 는 각각 자신만의 작업 공간을 가지며 이를 'Context' 라고 한다.

여러 Thread 끼리 공유하는 자원이 있을 수 있다. 이 때 이 자원을 서로 사용하려는 Race Condition 이 발생할 수 있고, 이 때 경쟁이 발생하는 부분을 임계 영역 (Critical Section) 이라고 한다.

임계 영역에 대한 동기화를 구현하지 않으면 오류가 발생할 수 있다.

```java
class MyThread extends Thread {

	public void run() {
		int i;
		for ( i = 1; i<=200; i++ ) {
			System.out.print(i + "\t");
		}
	}
}


public class ThreadTest {
	public static void main(String[] args) {

		System.out.println( Thread.currentThread() + "start");
		MyThread th1 = new MyThread();
		MyThread th2 = new MyThread();

		th1.start();
		th2.start();
		System.out.println( Thread.currentThread() + "end");
	}
}
```

출력 결과를 보면 숫자가 뒤죽박죽 섞여 있는 것을 볼 수 있음.

자바는 다중 상속을 지원하지 않는다. 어떠한 객체가 하위 클래스면서도 Thread 로 만들고 싶다면 'Runnable' 을 implements 하여 사용하자.

```java
class MyThread implements Runnable {

	@Override
	public void run() {
		int i;
		for ( i = 1; i<=200; i++ ) {
			System.out.print(i + " ");
		}
	}
}


public class ThreadTest {
	public static void main(String[] args) {

		System.out.println( Thread.currentThread() + "start");

		MyThread runnable = new MyThread();

		Thread th1 = new Thread(runnable);
		Thread th2 = new Thread(runnable);
		th1.start();
		th2.start();
		System.out.println( Thread.currentThread() + "end");
	}
}
```

Thread 는 여러 상태를 가진다.

- Runnable: 프로세스가 실행되어 CPU 를 점유할 수 있는 상태
- Run: CPU 를 점유한 상태
- Not Runnable: CPU 를 점유할 수 없는 상태. Java 에서는 `sleep()`, `wait()`, `join()`이 호출되었을 때 Not Runnable 상태가 된다.
  - sleep: 프로그래머가 지정한 시간만큼 대기.
  - wait: 임계 구역을 다른 Thread가 이미 점유하고 있어서 대기.
  - join: 특정 thread 가 다른 thread 의 결과를 기다려야 하는 경우.

Join 예제

```java
public class JoinTest extends Thread{

	int start;
	int end;
	int total;

	public JoinTest(int start, int end){
		this.start = start;
		this.end = end;
	}

	public void run(){

		int i;
		for(i = start; i <= end; i++){
			total += i;
		}
	}
	public static void main(String[] args) {

		JoinTest jt1 = new JoinTest(1, 50);
		JoinTest jt2 = new JoinTest(51, 100);


		jt1.start();
		jt2.start();

		int lastTotal = jt1.total + jt2.total;

		System.out.println("jt1.total = " + jt1.total);
		System.out.println("jt2.total = " + jt2.total);

		System.out.println("lastTotal = " + lastTotal);

	}
}
```

위 프로그램은 돌릴 때 마다 다른 결과가 나옴. `int lastTotal = jt1.total + jt2.total` 할 때, 아직 각 Thread 가 계산 중에 있고 결과를 도출해내지 못한 상태에서 lastTotal 으로 더해 버린 것이다. 이 때 `join` 메서드를 사용할 수 있음.

```java
public class JoinTest extends Thread{

	int start;
	int end;
	int total;

	public JoinTest(int start, int end){
		this.start = start;
		this.end = end;
	}

	public void run(){

		int i;
		for(i = start; i <= end; i++){
			total += i;
		}
	}
	public static void main(String[] args) {

		JoinTest jt1 = new JoinTest(1, 50);
		JoinTest jt2 = new JoinTest(51, 100);


		jt1.start();
		jt2.start();

		try{
			jt1.join(); // -- (1)
			jt2.join();

		}catch (InterruptedException e) {  // -- (2)
			System.out.println(e);
		}

		int lastTotal = jt1.total + jt2.total;

		System.out.println("jt1.total = " + jt1.total);
		System.out.println("jt2.total = " + jt2.total);

		System.out.println("lastTotal = " + lastTotal);

	}
}
```

`(1)` 에서 join 메서드를 호출했다. `jt1` thread 가 끝나기 전까지 `main` 함수를 Not Runnable 로 보내버린 것이다. `jt1` 의 계산이 끝나고 DEAD 되면 main 함수가 다시 Runnable 이 된다. `(2)` 에서 exception 처리를 하였는데, 이는 thread 가 무한 루프에 빠졌다거나 하는 이슈로 다시 돌아오지 못할 때의 예외 처리를 하는 것임.

<br/>

---

## 멀티 Thread 환경에서의 동기화 (Synchronized)

아주 잘못 짜여진 은행 프로그램 예시를 보자.

```java
class Bank {
	private int money = 100000;

	public void saveMoney(int money) {
		int m = getMoney();
		try {
			Thread.sleep(3000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		setMoney(m + money);
	}

	public void useMoney(int money) {
		int m = getMoney();
		try {
			Thread.sleep(200);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		setMoney(m - money);
	}

	public int getMoney() {
		return money;
	}

	public void setMoney(int money) {
		this.money = money;
	}

}

class Park extends Thread {
	public void run() {
		System.out.println("start save");
		SyncMain.myBank.saveMoney(3000);
		System.out.println("saveMoney(3000) : " + SyncMain.myBank.getMoney());
	}
}

class Kim extends Thread {
	public void run() {
		System.out.println("start use");
		SyncMain.myBank.useMoney(5000);
		System.out.println("useMoney(5000) : " + SyncMain.myBank.getMoney());
	}
}

public class SyncMain {
	public static Bank myBank = new Bank();

	public static void main(String[] args) {
		Park p = new Park();
		p.start();
		Kim k = new Kim();
		k.start();
	}
}
```

결과

```
start save
start use
useMoney(5000) : 95000
saveMoney(3000) : 103000

#마르지않는은행
```

박 씨가 최초 money 인 100,000 을 가져가 놓고 3초간 멍 때리고 있는 동안에 김 씨가 money 를 가져가서 5000을 빼고 95,000을 되돌려놨지만, 박 씨는 이를 모르고 들고 있던 100,000에 3000을 더하고 103,000을 되돌려 놨다.

일반적으로 생각하면 박 씨가 최초 money 를 들고 가버렸으면, 김 씨는 가져갈 수 있는 money 가 없어야 한다. 이를 구현해보자.

```java
class Bank {
	private int money = 100000;

	public synchronized void saveMoney(int money) {
		int m = getMoney();
		try {
			Thread.sleep(3000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		setMoney(m + money);
	}

	public synchronized void useMoney(int money) {
		int m = getMoney();
		try {
			Thread.sleep(200);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		setMoney(m - money);
	}

	public int getMoney() {
		return money;
	}

	public void setMoney(int money) {
		this.money = money;
	}
}
```

Bank 클래스를 수정한 모습. saveMoney 와 useMoney 메서드에 각각 `synchronized` 키워드를 붙혀주었다. `synchronized 메서드` 를 생성한 것이다. 해당 키워드를 붙혀주기만 해도 특정 스레드가 해당 메서드가 속해는 객체를 사용하고 있으면 다른 스레드가 접근할 수 없도록 lock 을 건다.

아래와 같은 방법으로 직접 어떤 객체에 lock 을 걸 것인지도 설정할 수 있다. 아래와 같은 방식은 `synchronized 블럭`을 생성하는 것이다.

```java
// 예시 1

class Bank {
	private int money = 100000;

	public void saveMoney(int money) {
		synchronized(this) {  // this, 즉 현재 객체에 동기화를 검.
			int m = getMoney();
			try {
				Thread.sleep(3000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			setMoney(m + money);
		}
	}

	public void useMoney(int money) {
		synchronized(this) {
			int m = getMoney();
			try {
				Thread.sleep(200);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			setMoney(m - money);
		}
	}

	public int getMoney() {
		return money;
	}

	public void setMoney(int money) {
		this.money = money;
	}
}
```

```java
// 예시 2
class Park extends Thread {
	public void run() {
		synchronized(SyncMain.myBank) {  // 외부에서 SyncMain.myBank 객체에 동기화를 검
			System.out.println("start save");
			SyncMain.myBank.saveMoney(3000);
			System.out.println("saveMoney(3000) : " + SyncMain.myBank.getMoney());
		}

	}
}

class Kim extends Thread {
	public void run() {
		synchronized(SyncMain.myBank){
			System.out.println("start use");
			SyncMain.myBank.useMoney(5000);
			System.out.println("useMoney(5000) : " + SyncMain.myBank.getMoney());
		}
	}
}
```

주의할 것은 자바에서는 deadlock 을 예방하는 코드를 제공하지 않는다. Synchronized 메서드 안에서 Synchronized 메서드를 호출하지 않도록 주의하자.

<br/>

---

## 멀티 Thread 환경에서의 동기화 (wait / notify)

여기 책이 4권밖에 없는 기적의 도서관이 있다.

```java
class Library {
	public ArrayList<String> shelf = new ArrayList<String>();

	public Library() {
		shelf.add("Java");
		shelf.add("Spring");
		shelf.add("C++");
		shelf.add("Python");
	}

	// 심지어 원하는 책을 빌려주는게 아니라 맨 앞에 있는 책을 빌려준다.
	public String lendBook() {
		Thread t = Thread.currentThread();

		String book = shelf.remove(0);
		System.out.println(t.getName() + " rent " + book);

		return book;
	}

	// 반납.
	public void returnBook(String book) {
		Thread t = Thread.currentThread();

		shelf.add(book);
		System.out.println(t.getName() + " return " + book);
	}
}
```

덕분에 경쟁이 치열하다.

```java
class Student extends Thread {
	public Student(String name) {
		super(name);
	}

/**
 * 1. 책을 빌린다.
 * 2. 5초간 읽는다. -> sleep(5000);
 * 3. 반납한다.
 */
	public void run() {
		String title = LibraryMain.library.lendBook();
		if ( title == null) return;
		try {
			sleep(5000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		LibraryMain.library.returnBook(title);
	}
}

public class LibraryMain {

	public static Library library = new Library();
	public static void main(String[] args) {

		Student st1 = new Student("std1");
		Student st2 = new Student("std2");
		Student st3 = new Student("std3");
		Student st4 = new Student("std4");
		Student st5 = new Student("std5");
		Student st6 = new Student("std6");

		// 가즈아!!
		st1.start();
		st2.start();
		st3.start();
		st4.start();
		st5.start();
		st6.start();
	}
}
```

결과 - 대충 길이 0 배열에서 0번째 인덱스를 찾으려 했다는 훈계

```
Exception in thread "std5" Exception in thread "std6" java.lang.IndexOutOfBoundsException: Index 0 out of bounds for length 0
	at java.base/jdk.internal.util.Preconditions.outOfBounds(Preconditions.java:64)
	at java.base/jdk.internal.util.Preconditions.outOfBoundsCheckIndex(Preconditions.java:70)
	at java.base/jdk.internal.util.Preconditions.checkIndex(Preconditions.java:266)
...
...
```

이 전 예제와는 달리 이번 문제는 공유하고 있는 구역에 자원이 없는 것이다. 에러 메시지만 보았을 때는 5번째 학생과 6번째 학생이 책을 빌리려 했지만 책이 없어 빌리지 못하고 그냥 포기한 채 프로그램이 종료되어 버린 것이다.

자원이 없으면 생길 때 까지 기다리도록 해보자.

```java
class Library {
	public ArrayList<String> shelf = new ArrayList<String>();

	public Library() {
		shelf.add("Java");
		shelf.add("Spring");
		shelf.add("C++");
		shelf.add("Python");
	}

	// synchronized 메서드로 만들어줌.
	public synchronized String lendBook() throws InterruptedException {
		Thread t = Thread.currentThread();

	if ( shelf.size() == 0 ) { // 책이 없으면
		System.out.println( t.getName() + " is waiting");
		wait();  // 기다릴께요!
		System.out.println( t.getName() + " run again");
	}

		String book = shelf.remove(0);
		System.out.println(t.getName() + " rent " + book);

		return book;
	}

	public synchronized void returnBook(String book) {
		Thread t = Thread.currentThread();

		shelf.add(book);  // 책이 반납되면
		notify();  // 책 들어왔다고 알려줌
		System.out.println(t.getName() + " return " + book);
	}
}
```
