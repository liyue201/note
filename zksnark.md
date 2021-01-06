## 零知识证明过程

### setup阶段

生成key  
circuit + random beacon ==> proving key(保密) + verification key（公开）


###  证明阶段
生成见证  
circuit + privite input ==>   witness  
生成证明  
proving key  + witness +  public input ==> public.json(public input/output) + proof.json  


###  验证证明  
verification key + public.json + proof.json ?= true  


### 参考
https://my.oschina.net/u/3843525/blog/4290560  
https://github.com/iden3/snarkjs
