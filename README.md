# StarChild

StarChild is a library for leveraging the interactive theorem proving and SMT checking abilities of F* to verify properties of neural networks.
It currently supports feedforward neural networks using ReLU, sigmoid, and softmax activation functions.
Networks are written as follows:
```fsharp
val layer1 : layer 2 1
let layer1 = { weights    = [[5100.0R]; [1560.0R]]
             ; biases     = [~.5102.0R]
             ; activation = Sigmoid
             }

val model : network 2 1 1
let model = NLast layer1
```
Properties of the model can then be checked using F* assertions.
Let’s check if the model correctly implements the AND gate:
```fsharp
let _ = assert (run_network model [1.0R; 1.0R] == [1.0R])
let _ = assert (run_network model [0.0R; 1.0R] == [0.0R])
let _ = assert (run_network model [1.0R; 0.0R] == [0.0R])
let _ = assert (run_network model [0.0R; 0.0R] == [0.0R])
```
Yes! It correctly implements the AND gate!
However, is it robust?
What do we mean by robust?
That’s generally not an easy question.
For this AND gate, let’s take robustness to mean that if the input is within some epsilon of a 0.0 or 1.0, the gate still works:
```fsharp
let epsilon  = 0.1R
let truthy x = dist x 1.0R <=. epsilon
let falsy  x = dist x 0.0R <=. epsilon

let _ = assert (forall x1 x2. (truthy x1 && truthy x2)
                   ==> (run_network model [x1; x2] == [1.0R]))
let _ = assert (forall x1 x2. (falsy  x1 || falsy  x2)
                   ==> (run_network model [x1; x2] == [0.0R]))
```
Unfortunately, the second assertion fails.
The network we defined is robust around truthy inputs, but it’s not robust around falsy inputs.
Back to the drawing board, I guess?<sup>[1](#unsat)</sup>

StarChild ships with a script which can help you convert a subset of Keras models to F* files.
It’s called `convert.py`, and you invoke it like this:
```bash
python convert.py \
  -i models/Fashion_MNIST_PCA_100_ReLU_64_Softmax_10.h5 \
  -o models/Fashion_MNIST_PCA_100_ReLU_64_Softmax_10.fst
```
Make sure to install the requirements first!<sup>[2](#pip)</sup>

StarChild also comes with two example models, trained on the [Fashion MNIST](https://github.com/zalandoresearch/fashion-mnist) dataset, using the `train_*.py` scripts.
Currently, F* chokes up when type checking the larger of the models.

---

<a name="unsat">1</a>: Is there a choice of weights and bias for which the network is robust around both truthy and falsy values?

<a name="pip">2</a>: Run `pip install -r requirements.txt`.
