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


python 

import security as sec
import test_engine as zj_test_engine

_AMOUNT = 50000
_COLUMNS = 200

def normal_store_get():
    """Store one sec data into redis, and updata some attributes, check these attributes

    for the same sec_id, use sec_obj.store() twice:
    {'ssm_id':"single_test1", 'a':9.32, 'b':1.23}
    {'ssm_id':'single_test1', 'a':8.23, 'c':1.19, 'jw':1.23}

    check these attributes, excepted result should be:
    {'ssm_id':'single_test1', 'a':8.23, 'c':1.19, 'jw':1.23, 'b':1.23}
    """
    sec_data = {'ssm_id':"single_V0_test1", 'a':9.32, 'b':1.23,}
    failed_list = sec.Security.store(sec_data)
    if failed_list:
        print 'store failed!'
    else:
        print 'store succeed!'

    print 'check from redis returned:'
    print sec.Security.gets('single_V0_test1')
    print '---------------------------------------'

    sec.Security.store([{'ssm_id':'single_V0_test1', 'a':8.23, 'c':1.19, 'jw':1.23}])
    sec1 = sec.Security.gets('single_V0_test1')
    if sec1:
        print 'existed sec obj become: ',sec1
        print 'modify existed sec object succeed!'
        print '---------------------------------------'
        print '---------------------------------------'
    else:
        print 'modify existed sec obj failed!'

    print '---------------------------------------'
    print '---------------------------------------'
    print '-------store a list of sec data--------'
    sec_datas = [{'ssm_id':'V0batch8286R7', 'a1':7.42, 'b2':8.23, 'pimco':1.33},{'ssm_id':'V0batch2X6R7', 'c1':7.42, 'b2':8.23}]
    failed_list = sec.Security.store(sec_datas)
    if failed_list:
        print 'store failed id list: ',failed_list
    secs = sec.Security.gets(['V0batch8286R7', 'V0batch2X6R7', 'notexsitid'])
    print 'check from redis returns: ',secs
    stored_ids = sec.Security.getall()
    print 'get all:', len(stored_ids)

def very_long_attribute():
    sec_pd = zj_test_engine.gen_sec_data_frame(1, 20000)# generate one sec data with 20000 attributes
    source_data = zj_test_engine.gen_source_data(sec_pd)
    print 'this data have ',len(source_data[0]),' attributes!'
    sec.Security.store(source_data[0])
    sec0 = sec.Security.gets(source_data[0]['ssm_id'])
    print 'check from redis returns data with length: ',len(sec0[0])



def pandas_generate_big_data():
    # get source data
    print '-------------------------------------------------------------------'
    print '-------------------------------------------------------------------'
    print 'pandas starts generate a big amount of test data!!!!!!!!'
    print 'please wait!!!!'
    sec_pd = zj_test_engine.gen_sec_data_frame(_AMOUNT, _COLUMNS)
    print 'the test data: ',sec_pd
    source_data = zj_test_engine.gen_source_data(sec_pd) # returns a list of dict
    print 'pandas has generated ',len(source_data),' number of source data',\
        ' with ',_COLUMNS,' attributes!'
    return source_data


def show_running_time(func):
    # a decorate to count how long a function executed
    import datetime
    def warp(*args):
        start = datetime.datetime.now()
        print 'fuction ',func.__name__, ' starts at ',start
        func(*args)
        end = datetime.datetime.now()
        print 'fuction ',func.__name__, ' ends at ',end
        print 'function running ',(end-start).seconds,' seconds'
    return warp


@show_running_time
def performance_test(source_data):

    print '-------------------------------------------------------------------'
    print '-------------------------------------------------------------------'
    print '-------------------------------------------------------------------'
    print '!!!!!!!!!!!performance test starts now!!!!!!!!!!!!!!!!!!!'
    print 'the first source data : '
    print source_data[0]
    sec.Security.store(source_data)
    print 'data store finish!'




if __name__ == '__main__':

    try:

        # get source data from pandas
        source_data = pandas_generate_big_data()
        import pdb; pdb.set_trace()
        performance_test(source_data)

        #normal_store_get()
        #very_long_attribute()

    except Exception as e:
        print e


v2


import security as sec
import test_engine as zj_test_engine
from datetime import datetime

_AMOUNT = 5000
_COLUMNS = 200

def test_case():
    # In the morning, Li Lei run some linear regression analysis scripts for the 500K number of securities, and generate 200 attributes for each security.
    # the data was like: [{'ssm_id':'V2778286R7', 'duration':7.42, 'oas':8.23, 'pimco':1.33},{'ssm_id':'SD332X6R7', 'c1':7.42, 'b2':8.23}]
    source_data = pandas_generate_big_data()
    print 'starts testing Security API!!!'
    print '-----------------------------------------------------------'

    #Test case 1:
    #In the moring, Li Lei want to store all these 500K*200 data into redis
    time1 = datetime.now()
    print 'store begins: ',time1.time()
    length = len(source_data)
    if length<200000:
        #sec.Security.store(source_data, protection=False)  #protection=False can make it more fast!!!
        #protection=True means pre-processing will impliemented to protect old data
        #protection=False means data will be stored into redis without a pre-processing,
        #because all the 200 attributes have new value, we don't need to protect old data in this case ,so use protection=False can be faster
        sec.Security.store(source_data)
    else:
        # store 500K data in one time may cause memory allocate error,
        # split data into some group can avoid memory allocate error
        group = 5
        splits = length/group+1
        for i in range(group):
            sec.Security.store(source_data[i*splits:(i+1)*splits])
    time2 = datetime.now()
    print 'store finished, use: ',time2-time1

    #Test case 2:
    #Li lei want to update one attribute for one security, and dont want to affect other attributes
    checkpoint_ssm_id = 'test_V2_ssm_id9'
    sec_data1 = {'ssm_id':checkpoint_ssm_id, 'a1':9.99, 'z99':4.3}
    print 'checkpoint before update :',sec.Security.gets(checkpoint_ssm_id)
    #you don't want to lose other attributes in this case, don't use protection=False
    sec.Security.store(sec_data1)
    print '-----------------------------------------------------------'
    #sec.Security.store(sec_data1,protection=True)  is also OK
    #sec.Security.store(sec_data1,protection=False) is WRONG, this will overwrite other attributes
    print 'checkpoint after update :',sec.Security.gets(checkpoint_ssm_id)

    #Test case 3:
    #get a list of data from redis
    all_ids = sec.Security.getall()
    time1 = datetime.now()
    print 'gets begins: ',time1.time()
    sec_datas = sec.Security.gets(all_ids[:50000])
    #print sec_datas[0]
    time2 = datetime.now()
    print 'gets finished, use: ',time2-time1

def test_update():
    """Store one sec data into redis, and updata some attributes, check these attributes

    for the same sec_id, use sec_obj.store() twice:
    {'ssm_id':"singleV2_test1", 'a':9.32, 'b':1.23}
    {'ssm_id':'singleV2_test1', 'a':8.23, 'c':1.19, 'jw':1.23}

    check these attributes, excepted result should be:
    {'ssm_id':'singleV2_test1', 'a':8.23, 'c':1.19, 'jw':1.23, 'b':1.23}
    """
    sec_data1 = {'ssm_id':"test_V2_singleV2_test2", 'a':9.32, 'b':1.23}
    sec_data2 = {'ssm_id':'test_V2_singleV2_test2', 'a':8.23, 'c':1.19, 'jw':1.23}
    sec_data3 = {'ssm_id':'test_V2_singleV2_test2', 'c':9.99}
    failed_list = sec.Security.store(sec_data1)
    if not failed_list:
        print 'store success!'
        print sec.Security.gets('test_V2_singleV2_test2')
    else:
        print 'store failed!'

    sec.Security.store(sec_data2)
    print sec.Security.gets('test_V2_singleV2_test2')

    sec.Security.store(sec_data3)
    print sec.Security.gets('test_V2_singleV2_test2')

    sec_datas = [{'ssm_id':'test_V2_batch8286R7', 'a1':7.42, 'b2':8.23, 'pimco':1.33},{'ssm_id':'test_V2_batch2X6R7', 'c1':7.42, 'b2':8.23}]
    failed_list = sec.Security.store(sec_datas)
    secs = sec.Security.gets(['test_V2_batch8286R7', 'test_V2_batch2X6R7', 'notexsitid'])
    print secs
    sec_datas = [{'ssm_id':'test_V2_batch8286R7', 'a1':1.2, 'jw':2.33},{'ssm_id':'test_V2_batch2X6R7', 'c1':7.42, 'b5':2.23}]
    failed_list = sec.Security.store(sec_datas)
    secs = sec.Security.gets(['test_V2_batch8286R7', 'test_V2_batch2X6R7', 'notexsitid'])
    print secs

def pandas_generate_big_data():
    # get source data
    print '-------------------------------------------------------------------'
    print '-------------------------------------------------------------------'
    print 'pandas starts generate a big amount of test data!!!!!!!!'
    print 'please wait!!!!'
    sec_pd = zj_test_engine.gen_sec_data_frame(_AMOUNT, _COLUMNS)
    print 'the test data: ',sec_pd
    print 'transforming source data, please wait...'
    source_datas = zj_test_engine.gen_source_data(sec_pd) # returns a list of dict
    print 'test engine has generated ',len(source_datas),' number of source data',\
        ' with ',_COLUMNS,' attributes!'
    return source_datas

def deletes():
    all_ids = sec.Security.getall()
    sec.Security.deletes(all_ids)




if __name__ == '__main__':
    try:
        test_case()
        test_update()
        #deletes()

    except Exception as e:
        print e





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



