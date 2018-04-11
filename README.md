# Density Order Embeddings

This is an implementation of the model in *[Athiwaratkun and Wilson](https://openreview.net/pdf?id=HJCXZQbAZ), Hierarchical Density Order Embeddings, ICLR 2018*.

In this work, we learn Gaussian representations of concepts on hierarchical data (WordNet) using a loss function that penalize the order violation based on truncated divergences. The Gaussian representation reflects the hierarchy of the data via ensapsulation of probability densities. That is, a generic concept such as *entity* corresponds to a broad distribution, encompassing many different specific entities that entail it such as *physical_entity* or *object*.

![wordnetsubset](figs/rn_rep.png?raw=true)


The learned representations capture the hierarchy of the data well and achieves state-of-the-art results for both Hypernym prediction task and the graded lexical entailment task (**HyperLex**).

![hypacctable](figs/hypernym_accuracy.png?raw=true)
![hyperlextable](figs/hyperlex_results.png?raw=true)


The BibTeX entry for the paper is:

```bibtex
@InProceedings{athiwilson_doe_2018,
    author = {Ben Athiwaratkun and Andrew Gordon Wilson},
    title = {Hierarchical Density Order Embeddings},
    booktitle = {ICLR},
    year = {2018}
}
```

Below are the instructions to run the code to train and evaluate the density order embeddings model. Note: this code is partially adapted from https://github.com/ivendrov/order-embeddings-wordnet.

## Install following packages:

- Torch 7 with cutorch, nn, cunn, dpnn, hdf5
- Python 2.7 with NTLK 3.2.1 and Pytorch 0.2

## Prepare the training data

```
python preprocessWordnet.py
th createDatasets.lua
th createDatasets.lua -s2 1 -s4 1
```

## Train the model for hypernym prediction
To train the model and perform prediction on the WordNet relationship prediction test set.

```
th main.lua --epochs 20 --name hyp_prediction --init_normalize --normalize --rep gauss --margin 2000  --varscale 0.00005 --kl_threshold 500 --hyp 1 | tee log/hyp_prediction.log
```
Note that the model files are saved in **modelfiles/hyp_prediction**.

### Hypernym Prediction Test Result
The following plot shows the test accuracies versus training iteration.
![wordnethyp](log/hyp_accuracies.png?raw=true)

The figure can be generated by the following command:
```
python plot_hyp_accuracies.py --pattern log/hyp_prediction.log  --figname log/hyp_accuracies.png
```

## HyperLex

### Train an optimal model for HyperLex
```
th main.lua --epochs 50 --name hyperlex --init_normalize --normalize --rep gauss --margin 2000  --varscale 0.00005 --kl_threshold 500 --hyp 1 --train contrastive_trans_s1-1_s2-1_s4-1 | tee log/hyperlex.log
```

### Evaluate on HyperLex

Download HyperLex dataset.
```
./prepare_hyperlex.sh
```
Evaluate the trained model in **modelfiles/hyperlex** by running:
```
python eval/eval_hyperlex.py --modeldir modelfiles/hyperlex --verbose 1
```
which yields the following output
```
Metric kl Spearman correlation is 0.591162001045 with pvalue 4.96616036411e-204
Metric dot Spearman correlation is 0.0801925229194 with pvalue 0.000188869520576
Metric elk Spearman correlation is 0.271476724097 with pvalue 7.47672809234e-38
Metric reversekl Spearman correlation is -0.106825864228 with pvalue 6.37003701385e-07
```

