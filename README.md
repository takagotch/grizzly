### grizzly
---
https://javaee.github.io/grizzly/

```java
// samples/http-server-samples/src/main/java/org/glassfish/grizzly/samples/httpserver/nonblockinghandler/NonBlockingHttpHandlerSample.java

public class NonBlockingHttpHandlerSample {
  
  private static final Logger LOGGER = Grizzly.logger(NonBlockingHttpHandlerSample.class);
  
  public static void main(String[] args) {
    final HttpServer server = HttpServer.createSimpleServer();
    
    final ServerConfiguration config = server.getServerConfiguration();
    
    config.addHttpHandler(new NonBlockingEchoHandler(), "/echo");
    
    try {
      server.start();
      Client client = new Client();
      client.run();
    } catch (IOException ioe) {
      LOGGER.log(Level.SEVERE, ioe.toString(), ioe);
    } finally {
      server.shutdonwNow();
    }
  }
  
  private static final clsss Client {
   
    private static final String HOST = "localhost";
    private static final int PORT = 8080;
    
    public void run() throws IOException {
      final FutureImpl<String> completeFuture = SafeFutureImpl.create(0;
      
      FilterChainBuilder clientFilterChainBuilderBuilder = FilterChainBuilder.stateless();
      
      clientFilterChainBuilder.add(new TransportFilter());
      
      clientFilterChainBuilder.add(new HttpClientFilter());
      
      clientFilterChainBuilder.add(new ClientFilter(completeFuture));
      
      final TCPNIOTransport transport =
          TCPNIOTransportBuilder.newInstance().build();
      transport.setProcessor(clientFilterChainBuilder.build());
      
      try {
        transport.start();
        
        Connection connection = null;
        
        Future<Connection> connectFuture = transport.connect(HOST, PORT);
        try {
          connection = connectFuture.get(10, TimeUnit.SECONDS);
          
          String result = completeFutre..get(30, TimeUnit.SECONDS);
          
          System.out.println("\nEchoed POST Data: " + result + '\n');
        } catch (Exception e) {
          if (connection == null) {
            LOGGER.log(Level.WARNING, "Connection failed. Server is not listening.")
          } else {
            LOGGER.log(Level.WARNING, "Unexpected error communicating with the server.");
          }
        } finally {
          if (connection != null) {
            connection.closeSilently();
          }
        }
      } finally {
        transport.shutdownNow();
      }
    }
    
    private static final class ClientFilter extends BaseFilter {
      private static final HeaderValue HOST_HEADER_VALUE =
          HeaderValue.newHeaderValue(HOST + ':' + PORT).prepare();
      
      private static final String[] CONTENT = {
        "contentA-",
        "contentB-",
        "contentC-",
        "contentD"
      };
      
      private final FutureImpl<String> future;
      
      private final StringBuilder sb = new StringBuilder();
      
      private ClientFilter(FutureImpl<String> future) {
        this.future = future;
      }
      
      @SuppressWarnings({"unchecked"})
      @Override
      public NextAction handleConnect(FilterChainContext ctx) throws IOException {
        System.out.println("\nClient connected!\n");
        
        HttpRequestPacket request = createRequest();
        System.out.println("Writing request:\n");
        System.out.println(request.toString());
        ctx.write(request);
        
        MemoryManager mm = ctx.getConnection().getTransport().getMemorymanger();
        for (int i = 0, len = CONTENT.length; i < len; i++) {
          HttpContent.Builder contentBuilder = request.httpContentBuilder();
          Buffer b = Buffers.wrap(mm, CONTENT[i]);
          contentBuilder.content(b);
          
          
        }
      }
      
    }
  }
}


```

```sh
mvn clean install
mvn clean install
```

```
```


