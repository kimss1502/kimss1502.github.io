---
layout: single
categories: 
  - Java
title: "Collection과 Multi thread"
---

 자바의 Collection 클래스 대부분은 thread-safety 하지 않다.

 아래 예의 ArrayList는 thread-safety 하지 않다. 따라서 위 코드는  `ConcurrentModificationException` 이 발생한다.

> 참고로 이런 Multi thread 가 아니라도 ConcurrentModificationException은 발생할 수 있다 자세한 내용은 `[Java] ConcurrentModificationException.md' 참고 
 
```
public class Main {
	public static void main(String[] args) {
		List<integer> list = new ArrayList<integer>();
		new WriterThread(list).start();
		new ReaderThread(list).start(); 
    }
}
```

```
public class ReaderThread extends Thread {
	private final List<integer> list;
  
	public ReaderThread(List<integer> list) {
		super("ReaderThread");
		this.list = list;
	}
  
	public void run() {
		while (true) {
		
			// foreach문은 내부적으로 iterator를 사용한다.
			for (int n : list) {	
				System.out.println(n);
			}
			
		}
	}
}
``` 

```
public class WriterThread extends Thread {
	private final List<integer> list;
	
	public WriterThread(List<integer> list) {
		super("WriterThread");
		this.list = list;
	}
  
	public void run() {
		for (int i = 0; true; i++) {
			list.add(i);     // list에 write
			list.remove(0);  // list에 write
		}
	}
}
```

WriterThread가 list를 변경시키기 때문에 ReaderThread의 foreach 문에서 예외가 발생한다.

이런 Collection의 동기화 문제를 해결하는 2가지 방법이 있다.

## 1. Collections.synchronizedList()
 Collections.synchronizedList() 메서드를 이용하면 Collection 인스턴스를 동기화 시킬 수 있다. <br/>
 동기화되면 Collection 클래스의 모든 메서드는 동기화되어 동작한다. <br/>
 (내부 코드를 보면 parameter로 전달받는 list를 `new SynchronizedList(list)`로 만들고 내부적으로 호출을 synchronized로 호출하게 된다.)
 
 
 **주의해야 할 점은 구현에 따라 이게 모든 동기화 문제를 해결한다고 생각하면 안된다는 것이다.**

 참고로 Collection의 인터페이스에 따라서 `Collections.synchronizedList()`, `Collections.synchronizedSet()`, `Collections.synchronizedMap()` 등이 있다.

### 1.1. 예 1
 위의 코드를 예를 들면 아래와 같이 ReaderThread는 synchronized로 list를 동기화 처리 해야 한다.

```
final List<integer> list = Collections.synchronizedList( new ArrayList<integer>() );
```
 
```
public class ReaderThread extends Thread {
  
	private final List<integer> list;

	public ReaderThread(List<integer> list) {
		super("ReaderThread");
		this.list = list;
	}
  
	public void run() {
		while (true) {
		
			// list 객체로 동기화
			synchronized (list) {
				for (int n : list) {
					System.out.println(n);
				}
			}
		}
	}
}
```

 WriterThread에서 list 객체의 add(), remove()는 동기화 되어 여러 쓰레드에서 동시에 접근할 수 없다. ReaderThread의 foreach문의 경우 내부적으로 Collection을 다룰때 iterator 객체로 다루기 때문에 Collection의 동기화와 상관이 없다. 따라서, 루프문을 돌때는 list 변화가 일어나지 않도록 루프문 전체를 list 객체로 synchronized를 건 것이다. 


### 1.2. 예 2

```
final List<String> list = Collections.synchronizedList(new ArrayList<String>());
final int nThreads = 2;

ExecutorService es = Executors.newFixedThreadPool(nThreads);
for (int i = 0; i < nThreads; i++) {
	es.execute(new Runnable() {
		public void run() {
			while(true) {
				try {
					list.clear();
					list.add("888");
					list.remove(0);
				} catch(IndexOutOfBoundsException ioobe) {
					ioobe.printStackTrace();
				}
			}
		}
	});
}
```

 위 예는 여전히 문제가 발생한다. list의 각 메서드 동작이야 동기화가 되겠지만 Thread 1이 list.clear() 한 직후 Thread 2가 list.remove(0)을 하는 순간 빈 list를 지우려 하기 때문에 예외가 발생한다.

따라서 아래와 같이 동작 전체를 synchronized로 묶어야 한다.

```
sychronized(list) {
	list.clear();
	list.add("888");
	list.remove(0);
}
```

## 2. CopyOnWriteArrayList 클래스
 `java.util.concurrent.CopyOnWriteArrayList` 클래스는 thread-safety 하다. 

```
final List<Integer> list = new CopyOnWriteArrayList<Integer>();
```

 copy-on-write는 ‘write 할 때 copy 한다’는 의미.  <br/>
 컬렉션에 대하여 write(추가, 수정)를 할 때마다, 내부에 확보된 배열을 통째로 복사한다. 이렇게 통째로 복사를 하면 iterator를 사용하여 element들을 순서대로 읽어가는 도중에 element가 변경될 염려가 없으므로  ConcurrentModificationException가 발생하지 않는다.<br/>
 
 Collections.synchronizedList()를 통해서도 thread-safety한 collection을 사용할 수 있겠지만 이 경우 read에 대해서도 동기화 되어 성능상 손해가 발생한다. <br/>
 
 이 클래스는 read에 대한 성능상 손해가 없는 대신 write할때마다 배열을 통째로 copy하기 때문에 성능저하가 심하다. <br/>
 즉, 적은 write와 잦은 read가 발생할때 좋은 클래스이다.


## 3. BlockingQueue
 보통 생산자 - 소비자 패턴에서 활용되는 큐로 많이 사용된다. <br/>
 이 큐는 멀티쓰레드환경에서 대표할만한 컬렉션이다. 
 소비자가 꺼내어 사용할동안 생산자는 멈춰있고, 생산자가 넣을동안 소비자는 멈춰있어야한다. 


## 4. ConcurrentHashMap
 HashTable은 java2에서 Collection framework가 나오기 이전부터 있었던 클래스로 기본적으로 모든 메서드가 synchronized 로 동기화 되어 있다. <br/>
 이는 `Collections.synchronizedMap()`을 이용해 Map을 동기화 시키는 경우도 마찬가지다.
 
 이렇게 모든 동작이 동기화되어 있을 경우 이 객체를 참조하는 thread의 개수가 많아질수록 경쟁이 심해져 성능이 기하급수적으로 떨어진다. 
 
 반면에 ConcurrentHashMap 에서는 내부적에 여러개의 세그먼트를 두고 각 세그먼트마다 별도의 락을 가지고 있다. 따라서 여러 쓰레드에서 ConcurrentHashMap 객체에 동시에 데이터를 삽입, 참조하더라도 그 데이터가 다른 세그먼트에 위치하면 서로 락을 얻기 위해 경쟁하지 않아 성능이 좋다.
  
--
[참고 문서]
> [코드 예](http://so-blog.net/2015/01/04/collection_with_multi_thread/) <br/> 
> [ConcurrentModificationException 참고](http://suein1209.tistory.com/323) <br/>
> [좋은 자료](http://okky.kr/article/279692) <br/>
> [ConcurrentHashmap과 Hashtable 비교](http://agbird.egloos.com/4849046) <br/>







