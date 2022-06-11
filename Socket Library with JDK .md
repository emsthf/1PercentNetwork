# Socket Library with JDK

Socket은 클라이언트와 서버의 특정 포트를 연결하여 양방향 데이터 통신을 하는 지원하는 운영체제의 API입니다.

자바에서 지원하는 Socket 도 이와 같은지 궁금해졌습니다.

[오라클]([https://docs.oracle.com/javase/tutorial/networking/sockets/definition.html](https://docs.oracle.com/javase/tutorial/networking/sockets/definition.html))에 이렇게 나와 있습니다.

~~해석을 해주신 분이 계시니 해석 본으로 보시죠~~

[해석 출처]([https://www.daleseo.com/what-is-a-socket/](https://www.daleseo.com/what-is-a-socket/))

**소켓의 정의**

> 소켓은 네트워크 상에서 돌아가는 두 개의 프로그램 간 양방향 통신의 하나의 엔트 포인트입니다. 소켓은 포트 번호에 바인딩되어 TCP 레이어에서 데이터가 전달되야하는 어플리케이션을 식별할 수 있게 합니다.
> 

**Socket 클래스**

> 자바 플랫폼에서 `java.net` 패키지는 네트워크 상에서 두개의 프로그램 간 양방향 통신에서 한쪽 지점을 구현하는 `Socket` 클래스를 제공합니다. `Socket` 클래스는 특정 시스템의 세부사항은 감추면서 플랫폼 독립적인 구현의 최상단에 위치합니다. 네이트브 코드에 의존하는 대신에 `java.net.Socket` 클래스를 이용해서 플랫폼 독립적인 방식으로 네트워크 상에서 통신을 할 수 있습니다.
> 
> 
> 추가적으로 `java.net` 패키지는 서버가 클라이언트로 부터 연결을 리스팅하고 수락하는데 사용되는 소켓을 구현하는 `ServerSocket` 클래스도 포함합니다. 본 튜토리얼에서는 `Socket`과 `ServerSocket` 클래스를 어떻게 사용하는지 살펴볼 것입니다.
> 

JVM의 **`Write Once Run Anywhere`** 와 같은 맥락이네요.

그럼 OS의 소켓 라이브러리를 먼저 알아보고, 이 소켓 인터페이스를 JDK는 어떻게 플랫폼 독립적으로 지원하는지 알아보겠습니다.

## OS Socket Library

OS의 소켓 라이브러리는 구현체마다 상이 할 수 있겠지만 보통 아래와 같은 순서로 통신을 한다고 합니다.

![출처 : [https://gnutec.net/socket-programming-in-php-part-2/socket-programming/](https://gnutec.net/socket-programming-in-php-part-2/socket-programming/)](img/Socket%20Library%20with%20JDK/Untitled.png)

출처 : [https://gnutec.net/socket-programming-in-php-part-2/socket-programming/](https://gnutec.net/socket-programming-in-php-part-2/socket-programming/)

## **TCP Socket[¶](https://os.mbed.com/handbook/Socket#tcp-socket)**
![출처 : [https://os.mbed.com/handbook/Socket](https://os.mbed.com/handbook/Socket)](img/Socket%20Library%20with%20JDK/Untitled%201.png)

## UD**P Socket[¶](https://os.mbed.com/handbook/Socket#tcp-socket)**
![출처 : [https://os.mbed.com/handbook/Socket](https://os.mbed.com/handbook/Socket)](img/Socket%20Library%20with%20JDK/Untitled%202.png)


TCP 소켓 기준으로 설명하겠습니다.

먼저 서버에서 TCP 소켓 연결을 위한 아이피와 포트를 바인드하고 대기 가능한 수를 지정하여 대기열을 생성합니다.

클라이언트는 서버로 연결 요청을 보내고 서버가 이를 수용하면 연결이 됩니다.

이후 클라이언트가 서버로 데이터 요청을 보내고 서버는 이에 대한 응답을 보냅니다.

이 과정을 반복한 뒤 클라이언트가 연결을 종료합니다.

 

이 처럼 서버와 클라이언트가 서로 아이피와 포트를 점유한 채로 입출력 스트림을 열고 읽기, 쓰기를 반복하는 작업을 소켓 통신이라고 합니다.

자바에서는 `[java.net](http://java.net)` 패키지에서 `Socket` 클래스를 지원합니다.

주석을 보면 무려 JDK1.0부터 지원을 했습니다.

![Untitled](img/Socket%20Library%20with%20JDK/Untitled%203.png)

자바에서의 소켓 사용법은 위에서 알아본 방법과 다르지 않습니다.

실제로는 여러 Try - Catch와 소켓을 지속 사용하기 위한 While(true)와 같은 방법들을 사용합니다.

순서 상으로 간단히 적는다면 아래와 같습니다.

```java
1. 서버 소켓 생성
ServerSocket serverSocket = new ServerSocket(SERVER_PORT);

2. 클라이언트로부터 연결 요청이 될 경우 연결 수락 
final Socket socketForServer = serverSocket.accept();

3. 클라이언트 소켓 생성
Socket socketForClient= new Socket(SERVER_IP, SERVER_PORT);

4. 서버 소켓으로부터 인풋/아웃풋 스트림을 얻는다.
InputStream inputStreamForServer = socketForServer .getInputStream();
OutputStream outputStreamForServer = socketForServer .getOutputStream();

4-1. 클라이언트 소켓으로부터 인풋/아웃풋 스트림을 얻는다.
InputStream inputStreamForClient = socketForClient.getInputStream();
OutputStream outputStreamForClient = socketForClient.getOutputStream();

5. 클라이언트 아웃풋 스트림으로 서버에 메시지 전송
outputStreamForClient.write(MESSAGE);
outputStreamForClient.flush();

6. 서버 인풋 스트림에서 클라이언트가 보낸 메시지 읽기
byte[] dataFromClient = new byte[100];
int readCount = inputStreamForServer.read(dataFromClient);
String data = new String(dataFromClient , 0, readCount , "UTF-8");

7. 서버 아웃풋 스트림으로 클라이언트에 메시지 전송 후 스트림 종료
outputStreamForServer.write(MESSAGE);
outputStreamForServer.flush();
outputStreamForServer.close();
inputStreamForServer.close();

8. 클라이언트 인풋 스트림에서 서버가 보낸 메시지 읽은 후 스트림 종료
byte[] dataFromServer = new byte[100];
int readCount = inputStreamForClient.read(dataFromServer );
String data = new String(dataFromServer , 0, readCount , "UTF-8");
inputStreamForClient.close();
outputStreamForClient.close();
```
[참고 블로그]([https://wonos.tistory.com/388](https://wonos.tistory.com/388))
[참고 블로그]([https://coding-factory.tistory.com/270](https://coding-factory.tistory.com/270))

그럼 이 과정이 어떻게 서버와 클라이언트의 시스템에서 소켓 통신을 가능하게 해주는지 알기 위해 예시로 ServerSocket의 `accept()` 메서드를 열어보았습니다.

accept() 메서드는 간단하게 생겼습니다.

이 메서드는 소켓이 닫혔는지, 그리고 바인딩 되지 않았는지를 판단한 뒤 문제가 없으면 새로운 소켓을 반환합니다.

```java
public Socket accept() throws IOException {
        if (isClosed())
            throw new SocketException("Socket is closed");
        if (!isBound())
            throw new SocketException("Socket is not bound yet");
        Socket s = new Socket((SocketImpl) null);
        implAccept(s);
        return s;
    }
```

이때 `implAccept(socket)` 를 보면 `securityManager`를 호출하여 실행하고자 하는 민감한 메서드에 문제가 없는지 체크하고 허용하거나 예외를 발생 합니다.

```java
getImpl().accept(si);
SecurityManager security = System.getSecurityManager();
if (security != null) {
    security.checkAccept(si.getInetAddress().getHostAddress(),
                         si.getPort());
}
```

*`System.getSecurityManager();`* 는 이미 어플리케이션이 획득한 SecurityManager가 있다면 이를 반환하고 없다면 null을 반환합니다.

```java
public void checkAccept(String host, int port) {
  if (host == null) {
      throw new NullPointerException("host can't be null");
  }
  if (!host.startsWith("[") && host.indexOf(':') != -1) {
      host = "[" + host + "]";
  }
  checkPermission(new SocketPermission(host+":"+port,
      SecurityConstants.SOCKET_ACCEPT_ACTION));
}
```

```java
public void checkPermission(Permission perm) {
    java.security.AccessController.checkPermission(perm);
}
```

`checkPermission(perm)` 하위로 굉장히 많은 체크 메서드들이 있는데, 이러한 과정을 거치면서 문제가 있으면 예외가 발생하고 없을 경우 무사 통과 됩니다.

```java
// allow if all of them allowed access
if (dumpDebug) {
		debug.println("access allowed "+perm);
}
```

이것만 보자면 조금 와닿지 않으니 소켓을 생성하는 부분을 보겠습니다.

소켓은 생성한 후에 아이피, 포트를 바인딩해도 되지만 생성 시점에 바로 바인딩을 할 수도 있는데요, 바로 생성하는 경우 bind 메서드를 바로 호출합니다.

```java
public ServerSocket(int port, int backlog, InetAddress bindAddr) throws IOException {
    setImpl();
    if (port < 0 || port > 0xFFFF)
        throw new IllegalArgumentException(
                   "Port value out of range: " + port);
    if (backlog < 1)
      backlog = 50;
    try {
        bind(new InetSocketAddress(bindAddr, port), backlog);
    } catch(SecurityException e) {
        close();
        throw e;
    } catch(IOException e) {
        close();
        throw e;
    }
}
```

때문에 중요한건 `bind()`인데요

```java
public void bind(SocketAddress endpoint, int backlog) throws IOException {
    if (isClosed())
        throw new SocketException("Socket is closed");
    if (!oldImpl && isBound())
        throw new SocketException("Already bound");
    if (endpoint == null)
        endpoint = new InetSocketAddress(0);
    if (!(endpoint instanceof InetSocketAddress))
        throw new IllegalArgumentException("Unsupported address type");
    InetSocketAddress epoint = (InetSocketAddress) endpoint;
    if (epoint.isUnresolved())
        throw new SocketException("Unresolved address");
    if (backlog < 1)
      backlog = 50;
    try {
        SecurityManager security = System.getSecurityManager();
        if (security != null)
            security.checkListen(epoint.getPort());
        getImpl().bind(epoint.getAddress(), epoint.getPort());
        getImpl().listen(backlog);
        bound = true;
    } catch(SecurityException e) {
        bound = false;
        throw e;
    } catch(IOException e) {
        bound = false;
        throw e;
    }
}
```

여기서도 `SecurityManager` 를 호출하여 체크를 한뒤 문제가 없으면 `bind()`와 `listen()`을 호출하는 것을 볼 수 있습니다.

여기서 `bind()` 를 타고 들어가면 아래와 같은 추상 메서드가 가득한 추상 클래스`SocketImpl` 가 나옵니다.

![Untitled](img/Socket%20Library%20with%20JDK/Untitled%204.png)

그리고 이를 구현한걸 찾아가면 여러 구현체가 있는데, 이 중 `PlainSocketImpl` 을 보면 이런 필드들이 있습니다.

```java
class PlainSocketImpl extends AbstractPlainSocketImpl
{
    private AbstractPlainSocketImpl impl;

    /* the windows version. */
    private static float version;

    /* java.net.preferIPv4Stack */
    private static boolean preferIPv4Stack = false;

    /* If the version supports a dual stack TCP implementation */
    private static boolean useDualStackImpl = false;

    /* sun.net.useExclusiveBind */
    private static String exclBindProp;

    /* True if exclusive binding is on for Windows */
    private static boolean exclusiveBind = true;

    static {
        java.security.AccessController.doPrivileged( new PrivilegedAction<Object>() {
                public Object run() {
                    version = 0;
                    try {
                        version = Float.parseFloat(System.getProperties().getProperty("os.version"));
                        preferIPv4Stack = Boolean.parseBoolean(
                                          System.getProperties().getProperty("java.net.preferIPv4Stack"));
                        exclBindProp = System.getProperty("sun.net.useExclusiveBind");
                    } catch (NumberFormatException e ) {
                        assert false : e;
                    }
                    return null; // nothing to return
                } });

        // (version >= 6.0) implies Vista or greater.
        if (version >= 6.0 && !preferIPv4Stack) {
                useDualStackImpl = true;
        }

        if (exclBindProp != null) {
            // sun.net.useExclusiveBind is true
            exclusiveBind = exclBindProp.length() == 0 ? true
                    : Boolean.parseBoolean(exclBindProp);
        } else if (version < 6.0) {
            exclusiveBind = false;
        }
    }
```

이 곳에서 시스템에 관한 정보를 가져와 소켓을 사용하기 위한 사전 준비를 해줍니다.

그리고 아래 생성자에서 시스템에 알맞는 소켓 구현체를 주입해주는 것으로 보입니다.

```java
/**
 * Constructs an empty instance.
 */
PlainSocketImpl() {
    if (useDualStackImpl) {
        impl = new DualStackPlainSocketImpl(exclusiveBind);
    } else {
        impl = new TwoStacksPlainSocketImpl(exclusiveBind);
    }
}

/**
 * Constructs an instance with the given file descriptor.
 */
PlainSocketImpl(FileDescriptor fd) {
    if (useDualStackImpl) {
        impl = new DualStackPlainSocketImpl(fd, exclusiveBind);
    } else {
        impl = new TwoStacksPlainSocketImpl(fd, exclusiveBind);
    }
}
```

이는 JDBC를 이용한 DB를 연결할때와 같은 방식이죠!

자바는 인터페이스를 통해 각 OS에 맞는 소켓을 사용 할 수 있도록 제공하고 있습니다.
