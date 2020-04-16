
## 线性回归Go语言实现
[《动手学习深度学习》](https://zh.d2l.ai/index.html)是一本非常经典的深度学习入门教程。
这个例子是其中第3.2节[《线性回归从零开始》](https://zh.d2l.ai/chapter_deep-learning-basics/linear-regression-scratch.html) 的go语言实现。
目标是拟合函数

```
y = 2*x1 - 3.4*x2 + 4.2
```

这里用到了机器学习框架[gorgonia](https://github.com/gorgonia/gorgonia)。 这个框架的思想类似TensorFlow，通过构建图来实现，每个计算单元是一个节点。


```go
package main

import (
	"fmt"
	"gorgonia.org/gorgonia"
	"gorgonia.org/tensor"
	"log"
	"math/rand"
	"time"
)

// 这是一组线性函数的矩阵形式, y=x1*w1+x2*w2+...+wn*wn+b
// Tx 是mxn的特征矩阵矩阵, m是样本数，n是特征数
// Tw 是nx1的权重矩阵
// b 是偏移量
// Ty 是一个mx1的标签矩阵
func linear(Tx tensor.Tensor, Tw tensor.Tensor, b float64) tensor.Tensor {
	temp, _ := tensor.MatMul(Tx, Tw)
	Ty, _ := tensor.Add(temp, b, tensor.WithReuse(temp))
	return Ty
}

// 生成特征和标签
// Tx：特征
// Ty: 标签
func genSamples(samplesNum int, trueW []float64, trueB float64) (Tx tensor.Tensor, Ty tensor.Tensor) {
	rand.Seed(time.Now().Unix())

	Tw := tensor.New(tensor.WithShape(len(trueW), 1), tensor.WithBacking(trueW[:]))
	// 随机生成特征
	Tx = tensor.New(tensor.Of(tensor.Float64), tensor.WithShape(samplesNum, len(trueW)), tensor.WithBacking(tensor.Random(tensor.Float64, samplesNum*2)))

	// 生成标签
	Ty = linear(Tx, Tw, trueB)

	// 给标签添加噪声
	for i := 0; i < samplesNum; i++ {
		v, _ := Ty.At(i, 0)
		Ty.SetAt(v.(float64) + float64(rand.Int31n(10)+1)/100)
	}
	return
}

// 预测函数
func predictFun(nX, w, b *gorgonia.Node) *gorgonia.Node {
	return gorgonia.Must(gorgonia.Add(gorgonia.Must(gorgonia.Mul(nX, w)), b))
}

// 损失函数
func lossFun(nodePred, nodeY *gorgonia.Node) *gorgonia.Node {
	cost := gorgonia.Must(gorgonia.Square(gorgonia.Must(gorgonia.Sub(nodePred, nodeY))))
	cost = gorgonia.Must(gorgonia.Mean(cost))
	return cost
}

// 构建gorgonia的计算图
func setup(X, Y tensor.Tensor) (nodeW, nodeB *gorgonia.Node, machine gorgonia.VM) {
	g := gorgonia.NewGraph()

	// 将数据转换成图中的节点
	nodeX := gorgonia.NodeFromAny(g, X)
	nodeY := gorgonia.NodeFromAny(g, Y)
	nodeW = gorgonia.NewMatrix(g, gorgonia.Float64, gorgonia.WithShape(2, 1), gorgonia.WithInit(gorgonia.ValuesOf(float64(0))))
	nodeB = gorgonia.NewScalar(g, gorgonia.Float64, gorgonia.WithValue(float64(0)))

	// 预测函数
	nodePred := predictFun(nodeX, nodeW, nodeB)

	// 损失函数
	cost := lossFun(nodePred, nodeY)

	// 梯度
	if _, err := gorgonia.Grad(cost, nodeW, nodeB); err != nil {
		log.Fatalf("Failed to backpropagate: %v", err)
	}

	machine = gorgonia.NewTapeMachine(g, gorgonia.BindDualValues(nodeW, nodeB))
	return nodeW, nodeB, machine
}

// 训练模型
func train(batchSize int, w, b *gorgonia.Node, machine gorgonia.VM, iter int) {
	model := []gorgonia.ValueGrad{w, b}
	stepSize := 0.01
	clip := 5.0
	solver := gorgonia.NewVanillaSolver(gorgonia.WithLearnRate(stepSize), gorgonia.WithBatchSize(float64(batchSize)), gorgonia.WithClip(clip))

	var err error
	for i := 0; i < iter; i++ {
		if err = machine.RunAll(); err != nil {
			fmt.Printf("Error during iteration: %v: %v\n", i, err)
			break
		}

		if err = solver.Step(model); err != nil {
			log.Fatal(err)
		}

		machine.Reset()

		if i%10 == 0 {
			fmt.Printf("%v: %v %v\n", i, w.Value().Data(), b.Value())
		}
	}
}

func main() {
	samplesNum := 10000
	trueW := [2]float64{2, -3.4}
	trueB := float64(4.2)

	Tx, Ty := genSamples(samplesNum, trueW[:], trueB)

	nodeW, nodeB, machine := setup(Tx, Ty)
	defer machine.Close()

	train(5, nodeW, nodeB, machine, 1000)

	fmt.Printf("w = %v\nb = %v\n", nodeW.Value().Data(), nodeB.Value())
}

```

经过训练后输出的w和b如下，跟实际函数参数很接近了,继续增加迭代次数可以更接近目标值。但是跟书中的python实现还是有些差距。可能是参数配置的差异，或者框架的问题。
```
w = [1.9633147377650029 -3.3382087877519835]
b = 4.122876433412727
```

