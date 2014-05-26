# Java I/O Demystified #

August 11, 2012 by Buddhika Chamith    
http://chamibuddhika.wordpress.com/2012/08/11/io-demystified/    

With all the hype on highly scalable server design and the rage behind nodejs I
have been meaning to do some focused reading on IO design patterns to which
until now couldn’t find enough time to invest. Now having done some research I
thought it’s best to jot down stuff I came across as a future reference for me
and any one who may come across this post. OK then.. Let’s hop on the I/O bus
and go for a ride.

## Types of I/O ##

There are four different ways IO can be done according to the blocking or non
blocking nature of the operations and the synchronous or asynchronous nature of
IO readiness/completion event notifications.

### Synchronous Blocking I/O ###
This is where the IO operation blocks the application until its completion
which forms the basis of typical thread per connection model found in most web servers.

When the blocking read() or write() is called there will be a context switch to
the kernel where the IO operation would happen and data would be copied to a
kernel buffer. Afterwards, the kernel buffer will be transfered to user space
application level buffer and the application thread will be marked as runnable
upon which the application would unblock and read the data in user space buffer

Thread per connection model tries to limit the effect of this blocking by confining a
connection to a thread so that handling of other concurrent connections will not
be blocked by an I/O operation on one connection. This is fine as long as the
connections are short lived and data link latencies are not that bad. However in
the case of long lived or high latency connections the chances are that threads
will be held up by these connections for a long time causing starvation for new
connections if a fixed size thread pool is used since blocked threads cannot be
reused to service new connections while in the state of being blocked or else it
will cause a large number of threads to be spawned within the system if each
connection is serviced using a new thread, which can become pretty resource
intensive with high context switching costs for a highly concurrent load.

**Simple thread per connection server:**

```java
ServerSocket server = new ServerSocket(port);
while(true) {
  Socket connection = server.accept();
  spawn-Thread-and-process(connection);
}
```

### Synchronous Non Blocking I/O ###

In this mode the device or the connection is configured as non blocking so that
read() and write() operations will not be blocked. This usually means if the
operation cannot be immediately satisfied it would return with an error code
indicating that the operation would block (EWOULDBLOCK in POSIX) or the device
is temporarily unavailable (EAGAIN in POSIX). It is up to the application to
poll until the device is ready and all the data are read. However this is not
very efficient since each of these calls would cause a context switch to kernel
and back irrespective of whether some data was read or not.

### Asynchronous Non Blocking I/O with Readiness Events ###

The problem with the earlier mode was that the application had to
poll and busy wait to get the job done. Wouldn’t it be better that some how the
application was notified when the device is ready to be read/ written? That is what
exactly this mode provides you with. Using a special system call (varies
according to the platform – select()/poll()/epoll() for Linux,
kqueue() for BSD, /dev/poll for Solaris) the application registers the interest
of getting I/O readiness information for a certain I/O operation (read or write)
from a certain device (a file descriptor in Linux parlance since all sockets are
abstracted using file descriptors). Afterwards this system call is invoked, which
would block until at least on of the registered file descriptors become ready.
Once this is true the file descriptors ready for doing I/O will be fetched as the
return of the system call and can be serviced sequentially in a loop in the
application thread.

The ready connection processing logic is usually contained
within a user provided event handler which would still have to issue non blocking
read()/write() calls to fetch data from device to kernel and ultimately to the
user space buffer incurring a context switch to the kernel. More ever there is
usually no absolute guarantee that it will be possible to do the intended I/O
with the device since what operating system provides is only an indication that
the device might be ready to do the I/O operation of interest but the non blocking
read() or write() can bail you out in such situations. However this should be
the exception than the norm.

So the overall idea is to get readiness events in an asynchronous fashion and
register some event handlers to handle once such event notifications are
triggered. So as you can see all of these can be done in a single thread while
multiplexing among different connections primarily due to the nature of the
select() (here I choose a representative system call) which can return readiness
of multiple sockets at a time. This is part of the appeal of this mode of
operation where one thread can serve large number of connections at a time. This
mode is what usually known as the “Non Blocking I/O” model.

Java has abstracted out the differences between platform specific system call
implementations with its NIO API. The socket/file descriptors are abstracted
using Channels and Selector encapsulates the selection system call. The
applications interested in getting readiness events registers a Channel (usually
a SocketChannel obtained by an accept() on a ServerSocketChannel) with the
Selector and get a SelectionKey which acts as a handle for holding the Channel
and registration information. Then the blocking select() call is made on
Selector which would return a set of SelectionKeys which then can be processed
one by one using the application specified event handlers.

**Simple non blocking server:**  
  
```java
Selector selector = Selector.open();
 
channel.configureBlocking(false);
 
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
 
while(true) {
 
  int readyChannels = selector.select();
 
  if(readyChannels == 0) continue;
 
  Set<SelectionKey> selectedKeys = selector.selectedKeys();
 
  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
 
  while(keyIterator.hasNext()) {
 
    SelectionKey key = keyIterator.next();
 
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
 
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
 
    } else if (key.isReadable()) {
        // a channel is ready for reading
 
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
 
    keyIterator.remove();
  }
}
```

### Asynchronous and Non Blocking I/O with Completion Events ###

Readiness events only go so far to notify you that the device/ socket is ready do
something. The application still has to do the dirty work of reading the data
from the device/ socket (more accurately directing the operating system to do so
via a system call) to the user space buffer all the way from device. Wouldn’t it
be nice to delegate this job to the operating system to run in the background
and let it inform you once it’s completed the job by transferring all the data
from device to kernel buffer and finally to the application level buffer?
That is the basic idea behind this mode usually known as the “Asynchronous I/O”
mode. For this it is required the operating system support AIO operations.
In Linux this support is present in aio POSIX API from 2.6 and for Windows this
is present in the form of “I/O Completion Ports”.

With NIO2 Java has stepped up its support for this mode with its
AsynchronousChannel API.

## Operating System Support ##

In order to support readiness and completion event notifications different
operating systems provide varying system calls. For readiness events select()
and poll() can be used in Linux based systems. However the newer epoll() variant
is preferred due to its efficiency over select() or poll(). select() suffer from
the fact that the selection time increases linearly with the number of
descriptors monitored. It is appearently notorious for overwriting the file
descriptor array references. So each time it is called the descriptor array is
required to be repopulated from a separate copy. Not an elegant solution at any rate.

The epoll() variant can be configured in two ways. Namely edge-triggered and
level-triggered. In edge-triggered case it will emit a notification only when
an event is detected on the associated descriptor. Say during an
event-triggered notification your application handler only read half of the
kernel input buffer. Now it won’t get a notification on this descriptor next
time around even when there are some data to be read unless the device is ready
to send more data causing a file descriptor event. Level-triggered
configuration on the other hand will trigger a notification each time when there
is data to be read.

The comparable system calls are present in the form of *kqueue* in BSD flavours
and */dev/poll* or *Event Completion* in Solaris depending on the version. The
Windows equivalent is *I/O Completion Ports*.

The situation for the AIO mode however is bit different at least in the Linux
case. The aio support for sockets in Linux seems to be shady at best with some
suggesting it is actually using readiness events at kernel level while providing
an asynchronous abstraction on completion events at application level. However
Windows seems to support this first class again via “I/O Completion Ports”.

## Design I/O Patterns 101 ##

There are patterns every where when it comes to software development. I/O is no
different. There are couple I/O patterns associated with NIO and AIO models
which are described below.

### Reactor Pattern###

There are several components participating in this pattern. I will go through
them first so it would be easy to understand the diagram.

**Reactor Initiator:** This is the component which would initiate the non blocking
server by configuring and initiating the dispatcher. First it would bind the
server socket and register it with the demultiplexer for client connection accept
readiness events. Then the event handler implementations for each type of
readiness events (read/ write/ accept etc..) will be registered with the
dispatcher. Next the dispatcher event loop will be invoked to handle event
notifications.

**Dispatcher**: Defines an interface for registering, removing, and dispatching
Event Handlers responsible for reacting on connection events which include
connection acceptance, data input/output and timeout events on a set of
connections. For servicing a client connection the related event handler (e.g:
accept event handler) would register the accepted client channel (wrapper for
underlying client socket) with the demultiplexer along side with the type of
readiness events to listen for that particular channel. Afterwards the
dispatcher thread will invoke the blocking readiness selection operation on
demultiplexer for the set of registered channels. Once one or more registered
channels are ready for I/O the dispatcher would service each returned “Handle”
associated with the each ready channel one by one using registered event
handlers. It is important that these event handlers don’t hold up dispatcher
thread since it will delay dispatcher servicing other ready connections. Since
the usual logic within an event handler includes transferring data to/from the
ready connection which would block until all the data are transferred between
user space and kernel space data buffers normally it is the case that these
handlers are run in different threads from a thread pool.

**Handle**: A handle is returned once a channel is registered with the
demultiplexer which encapsulates connection channel and readiness information.
A set of ready Handles would be returned by demultiplexer readiness selection
operation. Java NIO equivalent is SelectionKey.

**Demultiplexer**: Waits for readiness events of in one or more registered
connection channels. Java NIO equivalent is Selector.

**Event Handler**: Specifies the interface having hook methods for dispatching
connection events. These methods need to be implemented by application specific
event handler implementations.

**Concrete Event Handler**: Contains the logic to read/write data from underlying
connection and to do the required processing or initiate client connection
acceptance protocol from the passed Handle.


Event handlers are typically run in separate threads from a thread pool as shown in below diagram.


A simple echo server implementation for this pattern is as follows (without event handler thread pool).

```java
public class ReactorInitiator {
 
private static final int NIO_SERVER_PORT = 9993;
 
  public void initiateReactiveServer(int port) throws Exception {
 
    ServerSocketChannel server = ServerSocketChannel.open();
    server.socket().bind(new InetSocketAddress(port));
    server.configureBlocking(false);
 
    Dispatcher dispatcher = new Dispatcher();
    dispatcher.registerChannel(SelectionKey.OP_ACCEPT, server);
 
    dispatcher.registerEventHandler(
      SelectionKey.OP_ACCEPT, new AcceptEventHandler(
      dispatcher.getDemultiplexer()));
 
    dispatcher.registerEventHandler(
      SelectionKey.OP_READ, new ReadEventHandler(
      dispatcher.getDemultiplexer()));
 
    dispatcher.registerEventHandler(
    SelectionKey.OP_WRITE, new WriteEventHandler());
 
    dispatcher.run(); // Run the dispatcher loop
 
 }
 
  public static void main(String[] args) throws Exception {
    System.out.println("Starting NIO server at port : " +
      NIO_SERVER_PORT);
    new ReactorInitiator().
      initiateReactiveServer(NIO_SERVER_PORT);
  }
 
}
 
public class Dispatcher {
 
  private Map<Integer, EventHandler> registeredHandlers =
    new ConcurrentHashMap<Integer, EventHandler>();
  private Selector demultiplexer;
 
  public Dispatcher() throws Exception {
    demultiplexer = Selector.open();
  }
 
  public Selector getDemultiplexer() {
    return demultiplexer;
  }
 
  public void registerEventHandler(
    int eventType, EventHandler eventHandler) {
    registeredHandlers.put(eventType, eventHandler);
  }
 
  // Used to register ServerSocketChannel with the
  // selector to accept incoming client connections
  public void registerChannel(
    int eventType, SelectableChannel channel) throws Exception {
    channel.register(demultiplexer, eventType);
  }
 
  public void run() {
    try {
      while (true) { // Loop indefinitely
        demultiplexer.select();
 
        Set<SelectionKey> readyHandles =
          demultiplexer.selectedKeys();
        Iterator<SelectionKey> handleIterator =
          readyHandles.iterator();
 
        while (handleIterator.hasNext()) {
          SelectionKey handle = handleIterator.next();
 
          if (handle.isAcceptable()) {
            EventHandler handler =
              registeredHandlers.get(SelectionKey.OP_ACCEPT);
              handler.handleEvent(handle);
           // Note : Here we don't remove this handle from
           // selector since we want to keep listening to
           // new client connections
          }
 
          if (handle.isReadable()) {
            EventHandler handler =
              registeredHandlers.get(SelectionKey.OP_READ);
            handler.handleEvent(handle);
            handleIterator.remove();
          }
 
          if (handle.isWritable()) {
            EventHandler handler =
              registeredHandlers.get(SelectionKey.OP_WRITE);
            handler.handleEvent(handle);
            handleIterator.remove();
          }
        }
      }
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
 
}
 
public interface EventHandler {
 
   public void handleEvent(SelectionKey handle) throws Exception;
 
}
 
public class AcceptEventHandler implements EventHandler {
  private Selector demultiplexer;
  public AcceptEventHandler(Selector demultiplexer) {
    this.demultiplexer = demultiplexer;
  }
 
  @Override
  public void handleEvent(SelectionKey handle) throws Exception {
    ServerSocketChannel serverSocketChannel =
     (ServerSocketChannel) handle.channel();
    SocketChannel socketChannel = serverSocketChannel.accept();
    if (socketChannel != null) {
      socketChannel.configureBlocking(false);
      socketChannel.register(
        demultiplexer, SelectionKey.OP_READ);
    }
  }
 
}
 
public class ReadEventHandler implements EventHandler {
 
  private Selector demultiplexer;
  private ByteBuffer inputBuffer = ByteBuffer.allocate(2048);
 
  public ReadEventHandler(Selector demultiplexer) {
    this.demultiplexer = demultiplexer;
  }
 
  @Override
  public void handleEvent(SelectionKey handle) throws Exception {
    SocketChannel socketChannel =
     (SocketChannel) handle.channel();
 
    socketChannel.read(inputBuffer); // Read data from client
 
    inputBuffer.flip();
    // Rewind the buffer to start reading from the beginning
 
    byte[] buffer = new byte[inputBuffer.limit()];
    inputBuffer.get(buffer);
 
    System.out.println("Received message from client : " +
      new String(buffer));
    inputBuffer.flip();
    // Rewind the buffer to start reading from the beginning
    // Register the interest for writable readiness event for
    // this channel in order to echo back the message
 
    socketChannel.register(
      demultiplexer, SelectionKey.OP_WRITE, inputBuffer);
  }
 
}
 
public class WriteEventHandler implements EventHandler {
 
  @Override
  public void handleEvent(SelectionKey handle) throws Exception {
    SocketChannel socketChannel =
      (SocketChannel) handle.channel();
    ByteBuffer inputBuffer = (ByteBuffer) handle.attachment();
    socketChannel.write(inputBuffer);
    socketChannel.close(); // Close connection
  }
 
}
```

### Proactor Pattern ###

This pattern is based on asynchronous I/O model. Main components are as follows.

**Proactive Initiator**: This is the entity which initiates Asynchronous Operation
accepting client connections. This is usually the server application’s main
thread. Registers a Completion Handler along with a Completion Dispatcher to
handle connection acceptance asynchronous event notification.

**Asynchronous Operation Processor**: This is responsible for carrying out I/O
operations asynchronously and providing completion event notifications to
application level Completion Handler. This is usually the asynchronous I/O
interface exposed by Operating System.

**Asynchronous Operation**: Asynchronous Operations are run to completion
by the Asynchronous Operation Processor in separate kernel threads.

**Completion Dispatcher**: This is responsible for calling back to the application
Completion Handlers when Asynchronous Operations complete. When the Asynchronous
Operation Processor completes an asynchronously initiated operation, the
Completion Dispatcher performs an application callback on its behalf. Usually
delegates the event notification handling to the suitable Completion Handler
according to the type of the event.

**Completion Handler**: This is the interface implemented by application to process
asynchronous event completion events.


Let’s look at how this pattern can be implemented (as a simple echo server) 
using new Java NIO.2 API added in Java 7.

```java
public class ProactorInitiator {
  static int ASYNC_SERVER_PORT = 4333;
 
  public void initiateProactiveServer(int port)
    throws IOException {
 
    final AsynchronousServerSocketChannel listener =
      AsynchronousServerSocketChannel.open().bind(
        new InetSocketAddress(port));
     AcceptCompletionHandler acceptCompletionHandler =
       new AcceptCompletionHandler(listener);
 
     SessionState state = new SessionState();
     listener.accept(state, acceptCompletionHandler);
  }
 
  public static void main(String[] args) {
    try {
       System.out.println("Async server listening on port : " +
         ASYNC_SERVER_PORT);
       new ProactorInitiator().initiateProactiveServer(
         ASYNC_SERVER_PORT);
    } catch (IOException e) {
     e.printStackTrace();
    }
 
    // Sleep indefinitely since otherwise the JVM would terminate
    while (true) {
      try {
        Thread.sleep(Long.MAX_VALUE);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }
}
 
public class AcceptCompletionHandler
  implements
    CompletionHandler<AsynchronousSocketChannel, SessionState> {
 
  private AsynchronousServerSocketChannel listener;
 
  public AcceptCompletionHandler(
    AsynchronousServerSocketChannel listener) {
    this.listener = listener;
  }
 
  @Override
  public void completed(AsynchronousSocketChannel socketChannel,
    SessionState sessionState) {
   // accept the next connection
   SessionState newSessionState = new SessionState();
   listener.accept(newSessionState, this);
 
   // handle this connection
   ByteBuffer inputBuffer = ByteBuffer.allocate(2048);
   ReadCompletionHandler readCompletionHandler =
     new ReadCompletionHandler(socketChannel, inputBuffer);
   socketChannel.read(
     inputBuffer, sessionState, readCompletionHandler);
  }
 
  @Override
  public void failed(Throwable exc, SessionState sessionState) {
   // Handle connection failure...
  }
 
}
 
public class ReadCompletionHandler implements
  CompletionHandler<Integer, SessionState> {
 
   private AsynchronousSocketChannel socketChannel;
   private ByteBuffer inputBuffer;
 
   public ReadCompletionHandler(
     AsynchronousSocketChannel socketChannel,
     ByteBuffer inputBuffer) {
     this.socketChannel = socketChannel;
     this.inputBuffer = inputBuffer;
   }
 
   @Override
   public void completed(
     Integer bytesRead, SessionState sessionState) {
 
     byte[] buffer = new byte[bytesRead];
     inputBuffer.rewind();
     // Rewind the input buffer to read from the beginning
 
     inputBuffer.get(buffer);
     String message = new String(buffer);
 
     System.out.println("Received message from client : " +
       message);
 
     // Echo the message back to client
     WriteCompletionHandler writeCompletionHandler =
       new WriteCompletionHandler(socketChannel);
 
     ByteBuffer outputBuffer = ByteBuffer.wrap(buffer);
 
     socketChannel.write(
       outputBuffer, sessionState, writeCompletionHandler);
  }
 
  @Override
  public void failed(Throwable exc, SessionState attachment) {
    //Handle read failure.....
   }
 
}
 
public class WriteCompletionHandler implements
  CompletionHandler<Integer, SessionState> {
 
  private AsynchronousSocketChannel socketChannel;
 
  public WriteCompletionHandler(
    AsynchronousSocketChannel socketChannel) {
    this.socketChannel = socketChannel;
  }
 
  @Override
  public void completed(
    Integer bytesWritten, SessionState attachment) {
    try {
      socketChannel.close();
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
 
  @Override
  public void failed(Throwable exc, SessionState attachment) {
   // Handle write failure.....
  }
 
}
 
public class SessionState {
 
  private Map<String, String> sessionProps =
    new ConcurrentHashMap<String, String>();
 
   public String getProperty(String key) {
     return sessionProps.get(key);
   }
 
   public void setProperty(String key, String value) {
     sessionProps.put(key, value);
   }
 
}
```

Each type of event completion (accept/ read/ write) is handled by a separate
completion handler implementing CompletionHandler interface (Accept/ Read/
WriteCompletionHandler etc.). The state transitions are managed inside these
connection handlers. Additional SessionState argument can be used to
hold client session specific state across a series of completion events.

## NIO Frameworks (HTTPCore) ##

If you are thinking of implementing a NIO based HTTP server you are in luck.
Apache HTTPCore library provides excellent support for handling HTTP traffic
with NIO. API provides higher level abstractions on top of NIO layer with HTTP
requests handling built in. A minimal non blocking HTTP server implementation
which returns a dummy output for any GET request is given below.

```java
public class NHttpServer {
 
  public void start() throws IOReactorException {
    HttpParams params = new BasicHttpParams();
    // Connection parameters
    params.
      setIntParameter(
        HttpConnectionParams.SO_TIMEOUT, 60000)
     .setIntParameter(
       HttpConnectionParams.SOCKET_BUFFER_SIZE, 8 * 1024)
     .setBooleanParameter(
       HttpConnectionParams.STALE_CONNECTION_CHECK, true)
     .setBooleanParameter(
       HttpConnectionParams.TCP_NODELAY, true);
 
    final DefaultListeningIOReactor ioReactor =
      new DefaultListeningIOReactor(2, params);
    // Spawns an IOReactor having two reactor threads
    // running selectors. Number of threads here is
    // usually matched to the number of processor cores
    // in the system
 
    // Application specific readiness event handler
    ServerHandler handler = new ServerHandler();
 
    final IOEventDispatch ioEventDispatch =
      new DefaultServerIOEventDispatch(handler, params);
    // Default IO event dispatcher encapsulating the
    // event handler
 
    ListenerEndpoint endpoint = ioReactor.listen(
      new InetSocketAddress(4444));
 
    // start the IO reactor in a new separate thread
    Thread t = new Thread(new Runnable() {
      public void run() {
        try {
          System.out.println("Listening in port 4444");
          ioReactor.execute(ioEventDispatch);
        } catch (InterruptedIOException ex) {
          ex.printStackTrace();
        } catch (IOException e) {
          e.printStackTrace();
        } catch (Exception e) {
          e.printStackTrace();
        }
    }
    });
    t.start();
 
    // Wait for the endpoint to become ready,
    // i.e. for the listener to start accepting requests.
    try {
      endpoint.waitFor();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
 
  public static void main(String[] args)
    throws IOReactorException {
    new NHttpServer().start();
  }
 
}
 
public class ServerHandler implements NHttpServiceHandler {
 
 private static final int BUFFER_SIZE = 2048;
 
 private static final String RESPONSE_SOURCE_BUFFER =
 "response-source-buffer";
 
 // the factory to create HTTP responses
 private final HttpResponseFactory responseFactory;
 
 // the HTTP response processor
 private final HttpProcessor httpProcessor;
 
 // the strategy to re-use connections
 private final ConnectionReuseStrategy connStrategy;
 
 // the buffer allocator
 private final ByteBufferAllocator allocator;
 
 public ServerHandler() {
   super();
   this.responseFactory = new DefaultHttpResponseFactory();
   this.httpProcessor = new BasicHttpProcessor();
   this.connStrategy = new DefaultConnectionReuseStrategy();
   this.allocator = new HeapByteBufferAllocator();
 }
 
 @Override
 public void connected(
   NHttpServerConnection nHttpServerConnection) {
   System.out.println("New incoming connection");
 }
 
 @Override
 public void requestReceived(
   NHttpServerConnection nHttpServerConnection) {
 
   HttpRequest request =
     nHttpServerConnection.getHttpRequest();
   if (request instanceof HttpEntityEnclosingRequest) {
     // Handle POST and PUT requests
   } else {
 
     ContentOutputBuffer outputBuffer =
       new SharedOutputBuffer(
         BUFFER_SIZE, nHttpServerConnection, allocator);
 
     HttpContext context =
       nHttpServerConnection.getContext();
     context.setAttribute(
       RESPONSE_SOURCE_BUFFER, outputBuffer);
     OutputStream os =
       new ContentOutputStream(outputBuffer);
 
     // create the default response to this request
     ProtocolVersion httpVersion =
     request.getRequestLine().getProtocolVersion();
     HttpResponse response =
       responseFactory.newHttpResponse(
         httpVersion, HttpStatus.SC_OK,
         nHttpServerConnection.getContext());
 
     // create a basic HttpEntity using the source
     // channel of the response pipe
     BasicHttpEntity entity = new BasicHttpEntity();
     if (httpVersion.greaterEquals(HttpVersion.HTTP_1_1)) {
       entity.setChunked(true);
     }
     response.setEntity(entity);
 
     String method = request.getRequestLine().
       getMethod().toUpperCase();
 
     if (method.equals("GET")) {
       try {
         nHttpServerConnection.suspendInput();
         nHttpServerConnection.submitResponse(response);
         os.write(new String("Hello client..").
           getBytes("UTF-8"));
 
         os.flush();
         os.close();
     } catch (Exception e) {
       e.printStackTrace();
     }
    } // Handle other http methods
   }
 }
 
 @Override
 public void inputReady(
    NHttpServerConnection nHttpServerConnection,
    ContentDecoder contentDecoder) {
    // Handle request enclosed entities here by reading
    // them from the channel
 }
 
 @Override
 public void responseReady(
    NHttpServerConnection nHttpServerConnection) {
 
   try {
     nHttpServerConnection.close();
   } catch (IOException e) {
     e.printStackTrace();
   }
 }
 
 @Override
 public void outputReady(
   NHttpServerConnection nHttpServerConnection,
   ContentEncoder encoder) {
   HttpContext context = nHttpServerConnection.getContext();
   ContentOutputBuffer outBuf =
    (ContentOutputBuffer) context.getAttribute(
      RESPONSE_SOURCE_BUFFER);
 
   try {
     outBuf.produceContent(encoder);
   } catch (IOException e) {
     e.printStackTrace();
   }
 }
 
 @Override
 public void exception(
   NHttpServerConnection nHttpServerConnection,
   IOException e) {
   e.printStackTrace();
 }
 
 @Override
 public void exception(
   NHttpServerConnection nHttpServerConnection,
   HttpException e) {
   e.printStackTrace();
 }
 
 @Override
 public void timeout(
   NHttpServerConnection nHttpServerConnection) {
   try {
     nHttpServerConnection.close();
   } catch (IOException e) {
     e.printStackTrace();
   }
 }
 
 @Override
 public void closed(
   NHttpServerConnection nHttpServerConnection) {
   try {
     nHttpServerConnection.close();
   } catch (IOException e) {
     e.printStackTrace();
   }
 }
 
}
```

IOReactor class will basically wrap the demultiplexer functionality with
ServerHandler implementation handling readiness events.

Apache Synapse (an open source ESB) contains a good implementation of a
NIO based HTTP server in which NIO is used to scaling for a large number
of clients per instance with rather constant memory usage over time.
The implementation also contains good debugging and server statistics
collection mechanisms built in along with Axis2 transport framework integration.
It can be found at [1].

## Conclusion ##

There are several options when it comes to doing I/O which can affect the
scalability and performance of servers. Each of above I/O mechanisms have pros and
cons so that the decisions should be made on expected scalability and performance
characteristics and ease of maintainence of these approaches. This concludes my
somewhat long winded article on I/O. Feel free to provide suggestions,
corrections or comments that you may have. Complete source codes for servers
outlined in the post along with clients can be downloaded from here.

 
## References ##

There were many references I went through in the process. Below are some of the interesting ones.

1. http://www.ibm.com/developerworks/java/library/j-nio2-1/index.html

2. http://www.ibm.com/developerworks/linux/library/l-async/

3. http://lse.sourceforge.net/io/aionotes.txt

4. http://wknight8111.blogspot.com/search/label/AIO

5. http://nick-black.com/dankwiki/index.php/Fast_UNIX_Servers

6. http://today.java.net/pub/a/today/2007/02/13/architecture-of-highly-scalable-nio-server.html

7. Java NIO by Ron Hitchens

8. http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf

9. http://www.cs.wustl.edu/~schmidt/PDF/proactor.pdf

10. http://www.kegel.com/c10k.html
