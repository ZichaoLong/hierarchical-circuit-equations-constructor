# 网表编译

[TOC]

## 预处理

- 预处理后的网表亦可作为原始网表.
- 处理 `include` 逻辑. 补全 `Simulation` 下的 `MasterName,ExternalNodes,InternalNodes,InputParams`. 各仿真 case 预处理方式与复合子电路预处理相同.
- 预处理网表中各复合子电路定义.
  - 预处理节点名, 包括展开所有基本器件的 `ExternGALV`.
  - 预处理输入参数、内蕴参数名.

## 编译

### 可移植规则

可移植规则包含如下信息, 可用于实例化电路规则 `CompositeSubCktRule` 及复合子电路 `CompositeSubCkt`(在仿真模块内定义), 且可序列化:

- `MasterName,ExternalNodes,InternalNodes,InputParams,IntrinsicParams,SubModel,GlobalVarNames,Constants,SubCktsNum` .

- `InstancesIdx,InstancesName,Schematic`.

### 编译可移植规则

输入经过预处理后的网表, 将复合电路及仿真 case 定义编译为可移植规则.

1. [验证复合子电路定义](#Verify-VerifySubCktDef).
2. 将基本器件转换为仿真实际支持的器件名, 例如 `inductor`$\to$`inductor3`.
3. 将所有外接节点、输入参数，由字典改为列表.
4. 将复合电路定义编译为可移植规则: 各外接节点、输入参数索引均采用上述列表中对应的序号.

### 编译并实例化

编译为可移植规则后, 将可移植规则实例化为  `CompositeSubCktRule` 及 `CompositeSubCkt`, 返回所有仿真 case 组成的 `DataFrames`, 包含以下字段

- `CaseName,SimInfo,SimOpt,FreedomNum,SimCkt,CE,AllCktRule`.

## 验证

当前可验证内容包括

1. 复合电路定义的语法正确性.
2. 仿真 case 唯一可解性必要条件.
3. 仿真 case 在 `TRAN` 分析下的 `DAE-Index` 是否大于 `1`.

其余可在编译阶段验证的内容需更多研究, 特别是具有子模型的电路, 可能需要新理论.

### <span id=Verify-VerifySubCktDef>验证复合子电路定义</span>

- 每个实例化语句均可找到定义: `MasterName` 应当属于基本器件或已有定义的复合子电路.
- 复合子电路定义无循环引用.
- 复合子电路定义的语法正确性
  - 实例化引用的 `ExternalNodes` = 定义的 `ExternalNodes+InternalNodes`.
  - 实例化语句的 `ExternalNodes` 引用的 key 与各定义相匹配.
  - 实例化引用的 `InputParams` $\subset$ 定义的 `InputParams+IntrinsicParams+GlobalVariable`.
  - 实例化语句的 `InputParams` 引用的 key 与各定义相匹配.

### 图拓扑分析

输入实例化后的复合子电路, 依据电路图拓扑分析唯一可解性、`DAE-Index`.

#### 验证唯一可解性

参考 [[2]](#najm2010circuit) Chap 2.4.3, Chap 2.5.2.

- 电压型节点的连通性.

- 无电压源环路. `DC` 分析需验证无电压源/电感环路.

- 无电流源割集. `DC` 分析需验证无电流源/电容割集.

#### `TRAN` 分析下的 `DAE-Index`

参考 [[1]](#gunther2005modelling) Chap 2.

- 无电压源/电容环路, `CV-Loop`.

- 无电流源/电感割集, `LI-Cutset`.

#### 验证辅助工具/`API`, 由仿真模块提供

- 复合子电路实例树.
- 节点序号到节点索引的映射.
- 复合子电路实例索引到复合子电路实例的映射.
- 复合子电路实例索引到对应外接节点序号的映射.
- 由于输入参数值与仿真信号值有关, 随仿真迭代会动态变化, 因此无需提供输入参数相关索引信息. 可设置其他开关, 记录输入参数.

基于上述辅助 `API` 可以设置节点电压及 `GALV` 电流, 检视方程满足情况, 各器件偏置信息等.

#### 图拓扑分析注意事项

- `GALV` 及 `OLDC,OLDL` 均不是电压型节点. 需区分电压型节点与其他自由度.
- 电路图不是简单图, 因此验证有无环路时需注意自环以及两节点重边组成的环.
- `MOS` 模型通过添加无穷大电阻达成可解性验证条件.

## 参考文献

<span id=gunther2005modelling>[1] Günther, Michael, Uwe Feldmann, and Jan ter Maten. "Modelling and discretization of circuit problems." *Handbook of numerical analysis* 13 (2005): 523-659.</span>

<span id=najm2010circuit>[2] Najm, Farid N. *Circuit simulation*. John Wiley & Sons, 2010.</span>
