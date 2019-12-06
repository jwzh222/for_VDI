# for_VDI


pickle
pandas has generated  50000  number of source data  with  200  attributes!
store finished, use:  0:01:31.960319
gets finished, use:  0:01:13.711038

jsonpickle
pandas has generated  50000  number of source data  with  200  attributes!
store finished, use:  0:02:25.690226
gets finished, use:  0:00:14.228121

msgpack
pandas has generated  50000  number of source data  with  200  attributes!
store finished, use:  0:00:02.833743
gets finished, use:  0:00:03.301713


c++

# How to use
## Install dependences
### msgpack
$ git clone https://github.com/msgpack/msgpack-c.git  

$ cd msgpack-c  

$ cmake -DMSGPACK_CXX[11|17]=ON .  

$ sudo make install  

### cpp_redis
git clone https://github.com/Cylix/cpp_redis.git  

cd cpp_redis  

git submodule init && git submodule update  

mkdir build && cd build  

cmake .. -DCMAKE_BUILD_TYPE=Release  

make  

make install

## Run test case
g++ security.cpp test.cpp -L/usr/local/lib -lcpp_redis -ltacopie -lmsgpackc -std=c++11   

Tested on macOS10.14,with complier LLVM v10.0.1, it works fine!
(this command may be different depends on your machine and complier.)

    ##TEST case 1:
    // In the morning, Li Lei run some linear regression analysis scripts for the 100K number of securities, and generate 200 attributes for each security.
    // He now wants to store all those data into redis.
    security.store(source_data);

    ##TEST case 2:
    // He wants to get some security from redis
    vector<ssm_id> ssm_ids = {"test_cpp_ssm_id0", "test_cpp_ssm_id1","ssm_id_not_exsit"};
    vector<sec> result ;
    result = security.gets(ssm_ids);
    std::cout<<"check from redis returns: "<<endl;
    security.print_secs(result);

    // gets() also accept parametre via string
    //result = security.gets("test_cpp_ssm_id0");
    //security.print_secs(result);

    ##TEST case 3:
    // Li lei want to update some attribute for one security and add some new attributes, and dont want to affect the others attributes
    sec_data test_cpp_sec_data0 = {
        {"oas", 6.1},{"new_attr1", 1.2}
    };
    sec_data test_cpp_sec_data1 = {
        {"duration", 5.12},{"new_1", 8.8},{"new_2", 9.9}
    };
    sec sec0 ("test_cpp_ssm_id0", test_cpp_sec_data0);
    sec sec1 ("test_cpp_ssm_id1", test_cpp_sec_data1);
    vector<sec> sec_list = {sec0, sec1};

    security.store(sec_list);

    ssm_ids = {"test_cpp_ssm_id0", "test_cpp_ssm_id1"};
    result = security.gets(ssm_ids);
    std::cout<<"after update:------------------------------"<<std::endl;
    security.print_secs(result);



