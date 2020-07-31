# libiop安装

1. git clone https://github.com/scipr-lab/libiop.git
2. cd libiop
3. git submodule init && git submodule update
4. mkdir build && cd build
5. cmake .. 
6. make

# xjsnark使用

1. 首先安装MPS
2. 打开xjsnark的项目, 点击左边栏xjsnark.sandbox, 展开project列表
3. 新建project, 编写class文件(可以参考已有的例子, 但是我觉得还是非常难用)
4. 选择rebuild project, 再点击run class, 生成XXX.arith(描述电路)和XXX.in(描述一个instance和witness的实例)

# jsnark/xjsnark与libsnark和libiop的对接

* libsnark和libiop是ZKP后端, 调用后端所需的输入是: 
   - r1cs_constraint_system<FieldT>的一个对象constraint_system   ====>对应电路
   - r1cs_primary_input<FieldT>的一个对象primary_input     ====>对应statement
   - r1cs_auxiliary_input<FieldT>的一个对象auxiliary_input ====>对应witness

* ZKP前端和ZKP后端连接的关键是需要从XXX.arith和XXX.in重构出上述三个对象, jsnark中提供了一个工具 

## 安装jsnark
   1. git clone --recursive https://github.com/akosba/jsnark.git
   2. cd jsnark/libsnark
   3. git submodule init && git submodule update
   4. mkdir build && cd build && cmake ..
   5. make

* 进入jsnark/jsnark_interface, 可以看到circuitreader.cpp和run_ppzksnark.cpp

## 移花接木

为了调用libiop中的ligero, 我们需要借助circuitreader.cpp中功能完成arith和in到r1cs对象的转换. 
   1. 最高级的做法是完全重构libsnark和libiop的库, 将relation部分独立出来. 简便的方法首先重构出libsnark下的r1cs对象, 
      再利用内存格式转换将其转换为libiop下的r1cs对象(这是因为只有输入输出流是和libsnark和libiop独立的)
   2. 需要做出的改动是将libsnark中relation中r1cs.cpp和variable.cpp中对应的序列化部分抄到libiop中(libiop的该部分被裁剪了), 
      抄的时候需要注意变量名略有区别. 
   3. 现在可以仿照run_ppzksnark.cpp编写run_ligerosnark.cpp了. 

# 如何在BCS转换中加入消息以构造签名, 目前肮脏且不正确的方法如下

## prover部分的调整
在bcs_prover.tcc中void bcs_prover<FieldT>::signal_prover_round_done()中在message_hash后面附加上signed_message
>const hash_digest message_hash = this->compute_message_hash(ended_round, this->prover_messages_) + signed_message;


## verifier部分的调整
在bcs_verifier.tcc中void bcs_verifier<FieldT>::seal_interaction_registrations()中在message_hash后面附件上signed_message
>const hash_digest message_hash = this->compute_message_hash(round, this->transcript_.prover_messages_) + signed_message; 


# rust相比C++的好处
内存管理方便

# front-end 
covert NP relations to circuits friendly to ZKP backend
so far, the format under standardization is R1CS (rank one constraint system)
the existing tools for this task are 

* gadget provided in libsnark
* jsnark/xjsnark
* zinc, zokrates
* provide some useful gadgets, such as range proof, poseidon, then we can easily gather them together


# backend
take as input R1CS, generate the proof
so far, the opensource implementations can be divided into two category: with or without trusted setup

## trusted setup
* libsnark: C++ implementation of several crs-based zk-SNARK, such as Groth16
* bellman: rust implementation of math (finite fields, elliptic curve etc.) and Groth16 
* zexe: rust implementation of math (finite fields, elliptic curve etc.) and Groth16 and more  

## transparent setup
* libiop [https://github.com/scipr-lab/libiop]:
C++ implementations of several public-coin zk-SNARK, such as Ligero, Aurora, Fractal

* dalek cryptography 
Zcash [https://github.com/dalek-cryptography]: 
Fast, safe, pure-rust elliptic curve cryptography, rust implementation of Bulletproofs


SCIPR Lab
Succinct Computational Integrity and Privacy Research
