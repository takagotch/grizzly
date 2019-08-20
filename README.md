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
          HttpContent content = contentBuilder.build();
          System.out.printf("(Client writing: %s)\n", b.toStringContent());
          ctx.write(content);
          try {
            Thread.sleep(2000);
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
        }
        
        ctx.write(request.httpTrailerBuilder().build());
        
        System.out.println("\n");
        
        return ctx.getStopAction();
        
      }
      
      @Override
      public NextAction handleRead(FilterChainContext ctx) throws IOException {
        
        HttpContent c = ctx.getMessage();
        Buffer b = c.getContent();
        if (b.hasRemaining()) {
          sb.append(b.toStringContent());
        }
        
        if (c.isLast()) {
          future.result(sb.toString());
        }
        return ctx.getStopAction();
      }
      
      private HttpRequstPacket createRequest() {
        HttpRequestPacket.Builder builder = HttpRequestPacket.builder();
        builder.method("POST");
        builder.protocol("HTTP/1.1");
        builder.uri("/echo");
        builder.chunked(true);
        HttpRequestPacket packet = builder.build();
        packet.addHeader(Header.Host, HOST_HEADER_VALUE);
        return packet;
        
      }
    }
    
    private static class NonBlockingEchoHandler extends HttpHandler {
      
      @Override
      public void service(final Request request,
          final Response response) throws Exception {
        
        final char[] buf = new char[128];
        final NIOReader in = request.getNIOReader();
        final NIOWriter out = response.getNIOWriter();
        
        response.suspend();
        
        in.notifyAvailable(new ReadHandler() {
          
          @Override
          public void onDataAvailable() throws Exception {
            System.out.printf("[onDataAvailable] echoing %d bytes\n", in.readyData());
            echoAvailableData(in, out, buf);
            in.notifyAvailabe(this);
          }
          
          @Override
          public void onError(Throwable t) {
            System.out.println("[onError]" + t);
            response.resume();
          }
          
          @Override
          public void onError(Throwable t) {
            System.out.println("[onError]" + t);
            response.resume();
          }
          
          @Override
          public void onAllDataRead() throws Exceptoin {
            System.out.printf("[onAllDataRead] length: %d\n", in.readyData());
            try {
              echoAvailableData(in, out, buf);
            } finally {
              try {
                in.close();
              } finally {
              }
              
              response.resume();
            }
          }
        });
      }
      
      private void echoAvailableData(NIOReader in, NIOWriter out, char[] buf)
          throws IOException {
        
        while(in.isReady()) {
          int len = in.read(buf);
          out.write(buf, 0, len);
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


