---
title: gRPC example in Docker
date: 2018-03-19
tags:
- gRPC
- stream algorithm implement
- Docker
categories:
- implement
description: " use gRPC to build the test platform for stream algorithm in Docker"
---
# gRPC example

## protocol buffers;
As we all know, gRPC service is defined using protocol buffers; If you want to know how to define a .proto file, you can go to [protocol buffer Developer Guide]（https://developers.google.com/protocol-buffers/docs/overview）, Here is our stream_algorithm.proto defination:

```

syntax = "proto3";

// Interface exported by the server.
service MyAlgorithmBrain {
    //get the rank of the item in real time
    rpc MyRankOfStream(stream ItemRequest) returns (ItemResponse){}
    //get the structure of the sketch maintained in server
    rpc SketchOfStream(stream ItemRequest) returns (ItemResponse){}
}


message ItemRequest {

    int32 id = 1;  
    int32 value = 2;
    string timestamp = 3;
}

message ItemResponse {
    float rank = 1;
}

```

## Generate gRPC code
Next we need to update the gRPC code used by our application to use the new service definition.
```
python -m grpc_tools.protoc -I../../protos --python_out=. --grpc_python_out=. ../../protos/stream_algorithm.proto

```

This regenerates stream_algorithm_pb2.py which contains our generated request and response classes and stream_algorithm_pb2_grpc.py which contains our generated client and server classes.

## Write the client and the server code

Before we write the client code, we think about how to get the data, in this case, we use the local data, which are stored in the txt, and we read the file, every line is every input data.

the nums.txt is going to be like this:

```

1918,1520882120.0043917
9732,1520882120.01238
2657,1520882120.01238
5164,1520882120.01238
5708,1520882120.01238
4403,1520882120.01238
1870,1520882120.01238
3729,1520882120.01238
9145,1520882120.01238
5544,1520882120.0133848
747,1520882120.0133848
4681,1520882120.0133848
6552,1520882120.0133848
5737,1520882120.0133848
885,1520882120.0133848
1467,1520882120.0133848

```
the first column is the value, the second column is the timestamp, and then we create a function to help us read the data:
```
def read_route_guide_database():
    """Reads the numbers database.

  Returns:
    The full contents of the route guide database as a sequence of
      stream_algorithm_pb2.Features.
  """
    feature_list = []
    file = open("nums.txt", "r")
    count = 0
    for line in file:
        array = line.split(',')
        feature = stream_algorithm_pb2.ItemRequest(
            id=count,
            value=int(array[0]),
            timestamp=array[1]
        )
        count += 1
        feature_list.append(feature)
    return feature_list

```

Next is the client part:
First, create a channel defined by the ip and the port, Next, create the stub based on the channel. Then, recall the stub function, like stub.MyRankOfStream( iter(algorithm_resources.read_route_guide_database())), don't forget the iter before the input data.
```
def run():
    channel = grpc.insecure_channel('localhost:50052')
    stub = stream_algorithm_pb2_grpc.MyAlgorithmBrainStub(channel)
    # print(algorithm_resources.read_route_guide_database())
    response = stub.MyRankOfStream( iter(algorithm_resources.read_route_guide_database()))
    print("get the rank infomation from the server" , response)


```

The server part:
We create the MyRankOfStream function, and import the kll algorithm module, which can store the data distribution in a small space. The response is rank of the last item of the input data.
```
class MyAlgorithmBrainServicer(stream_algorithm_pb2_grpc.MyAlgorithmBrainServicer):
    """docstring for MyAlgorithmBrainServicer."""
    def __init__(self):
        super(MyAlgorithmBrainServicer, self).__init__()
        #self.db = arg

    def MyRankOfStream(self, request_iterator, context):
        # for new_note in request_iterator:
        #     yield new_note.value
        kll_algorithm = kll.KLL(128, 500)
        sum = 0

        for new_note in request_iterator:
            #sum += new_note.value
            kll_algorithm.update(new_note)
            lastItem = new_note
        res = stream_algorithm_pb2.ItemResponse(
            rank = kll_algorithm.rank(lastItem)
        )
        return res
def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    stream_algorithm_pb2_grpc.add_MyAlgorithmBrainServicer_to_server(
        MyAlgorithmBrainServicer(), server)

    server.add_insecure_port('[::]:50052')
    server.start()
    try:
        while True:
            time.sleep(_ONE_DAY_IN_SECONDS)
    except Exception as e:
        server.stop(0)

if __name__ == '__main__':
    serve()

```

If you want run this server in the docker, you can just use this line below:
```
docker run -itd --rm --name server -v "$PWD":/usr/src/myapp -w /usr/src/myapp grpc/python:1.4 python server.py


```
see what kind of containers do we have
```
docker ps -a


```
and see the details of our container, ddd8e8286103 is our container id. and the most important thing we are going to find is the ip address of our container.
```
docker inspect ddd8e8286103

```

we got "IPAddress": "172.17.0.2", so we need to change our client code
```
channel = grpc.insecure_channel('172.17.0.2:50052')

```

The last one is that we `docker attach server` get into the server container, and  `python client.py` to start the client.
