# gRPC Java Client Side Load Balancer using Consul

This is gRPC Java client-side load balancer, in particular for consul service discovery.
It can also be used for the static service node list.

There are two ways to do load balancing:
- Consul NameResolver Implementation
- Generated Code based

## Consul NameResolver Implementation
Consul NameResolver can be used like this:

    /**
     * Consul NameResolver Usage.
     *
     *
     * @param serviceName consul service name.
     * @param consulHost consul agent host.
     * @param consulPort consul agent port.
     * @param ignoreConsul if true, consul is not used. instead, the static node list will be used.
     * @param hostPorts the static node list, for instance, Arrays.asList("host1:port1", "host2:port2")
     */
    public HelloWorldClientWithNameResolver(String serviceName, 
                                            String consulHost, 
                                            int consulPort, 
                                            boolean ignoreConsul, 
                                            List<String> hostPorts) {

        String consulAddr = "consul://" + consulHost + ":" + consulPort;

        int pauseInSeconds = 5;

        channel = ManagedChannelBuilder
                .forTarget(consulAddr)
                .loadBalancerFactory(RoundRobinLoadBalancerFactory.getInstance())
                .nameResolverFactory(new ConsulNameResolver.ConsulNameResolverProvider(serviceName, pauseInSeconds, ignoreConsul, hostPorts))
                .usePlaintext(true)
                .build();

        blockingStub = GreeterGrpc.newBlockingStub(channel);
    }
    
To use consul service discovery, ignoreConsul is false and hostPorts is null.

and to use static service node list, ignoreConsul is true and hostPorts is the list of static nodes, for instance, Arrays.asList("host1:port1", "host2:port2")


  
### Run Demo
For consul usage, run hello world server docker container.
To register hello world server docker container with the service name to consul, see the consul-registrator or nomad if nomad is your container orchestrator.
    
Run hello world client:

    mvn -e -Dtest=HelloWorldClientWithNameResolverRunner -DserviceName=<service-name> -DconsulHost=localhost -DconsulPort=8500 -DignoreConsul=false test;
    

For static service node list, run hellow world server:

    mvn -e -Dtest=HelloWorldServerRunner test;
    
    
In another console, run hello world client:

    mvn -e -Dtest=HelloWorldClientWithNameResolverRunner -DserviceName=any-service -DconsulHost=localhost -DconsulPort=8500 -DignoreConsul=true -DhostPorts=localhost:50051 test;

, where Property hostPorts is comma separated hostPort list, for instance, host1:port1,host2:port2


## Generated Code based    
By generating java codes with protoc, GrpcLoadBalancer can be used as follows.

### How to use
First, download and install protoc:

    # download protoc compiler.
    ## for linux.
    wget https://github.com/google/protobuf/releases/download/v3.5.1/protoc-3.5.1-linux-x86_64.zip;
 
    ## for windows.
    wget https://github.com/google/protobuf/releases/download/v3.5.1/protoc-3.5.1-win32.zip;


And generate java classes using protoc:

    # run protoc command for linux:
    mvn protobuf:test-compile -DprotocExecutable="/usr/local/bin/protoc"
    mvn protobuf:test-compile-custom -DprotocExecutable="/usr/local/bin/protoc"
            
    # run protoc command for windows:
    mvn protobuf:test-compile -DprotocExecutable="F:/dev/protoc/protoc-3.5.1-win32/bin/protoc.exe"
    mvn protobuf:test-compile-custom -DprotocExecutable="F:/dev/protoc/protoc-3.5.1-win32/bin/protoc.exe"

The class of load balance GrpcLoadBalancer looks like this:

    GrpcLoadBalancer<R, B, A>
    
The generated classes are used to construct GrpcLoadBalancer instance.
First generic type of GrpcLoadBalancer is rpc class, the second is blocking stub class, and the third is async stub class.

In the test directory, the generated classes of hello world proto and test client can be seen.
Let's see GrpcLoadBalancer with the specific generated type classes in the HelloWorldClient class.

    private GrpcLoadBalancer<GreeterGrpc, GreeterGrpc.GreeterBlockingStub, GreeterGrpc.GreeterStub> lb;
    
To construct load balancer using consul service discovery, GrpcLoadBalancer can be constructed like this:
 
    // GrpcLoadBalancer(String serviceName, String consulHost, int consulPort, Class<R> rpcClass)
    lb = new GrpcLoadBalancer<>(serviceName, consulHost, consulPort, GreeterGrpc.class);
    
Or just using static node list:

    // Arrays.asList("host1:port1", "host2:port2")
    // GrpcLoadBalancer(List<String> hostPorts, Class<R> rpcClass)
    lb = new GrpcLoadBalancer<>(hostPorts, GreeterGrpc.class);
 

To get blocking stub to invoke rpc:

    lb.getBlockingStub()
    
Or to get async stub:

    lb.getAsyncStub()
   
Please, see HelloWorldClient class for the details of how to invoke rpc.


### Run Demo
This demo will be run for static service node list.

Run hello world server:

    mvn -e -Dtest=HelloWorldServerRunner test;
    
In another console, run hello world client:

    mvn -e -Dtest=HelloWorldClientRunner test;
