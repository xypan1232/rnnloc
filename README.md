# rnnloc
In this study, we use two types of features and combine them to construct the model. The first feature type is extracted from a protein–protein interaction network to abstract the relationship between the encoded protein and other proteins. The second type is obtained from gene ontology and biological pathways to indicate the existing functions of the encoded protein. These features are analyzed using some feature selection methods. The final optimum features are adopted to build the model with recurrent neural network as the classification algorithm. Such model yields good performance with Matthews correlation coefficient of 0.844. A decision tree is used as a rule learning classifier to extract decision rules. Although the performance of decision rules is poor, they are valuable in revealing the molecular mechanism of proteins with different subcellular locations. The final analysis confirms the reliability of the extracted rules.

# Dependencies and development enviroment

## Package dependencies
  * <a href=https://github.com/scikit-learn/scikit-learn>sklearn 0.20.0</a> , and also its dependency numpy, pandas and scipy. <br>
  * <a href=https://github.com/scikit-learn-contrib/imbalanced-learn>imbalanced-learn</a> <br>
  * <a href=https://www.tensorflow.org/> TensorFlow 1.10+ </a> <br>
  * Python 3.6 <br>
  
## OS Requirements
This package is supported for *Linux* operating systems. The package has been tested on the following systems: <br>
Linux: Ubuntu 16.04  <br>
  
# 1. Learn node embedding from a protein-protein network using node2vec
1. Download the human protein-protein network from STRING database v9.1, and download the compressed file <a href="http://string91.embl.de/newstring_cgi/show_download_page.pl?UserId=wOOpKXCrcQGf&sessionId=fcg4u2oXFFYd">protein.links.v9.1.txt.gz</a>. Here only human protein-protein interactions are extracted. It needs be transfered to the below described support format. <br>
2. Download the node2vec software from the website <a href="https://snap.stanford.edu/node2vec/">node2vec</a>. you can directly download the source code from <a href="https://github.com/aditya-grover/node2vec">node2vec github </a> to working directory. <br>
3. Run the python script to generate the node embedding: <br>
```python src/main.py --input STRING_9.1_edge.txt --output STRING_9.1_edge_500D.emd --dimensions 500```
<br>
where STRING_9.1_edge.txt is the human protein-protein network, STRING_9.1_edge_500D.emd is the learned embedding for all proteins in the network, and 500 is the specified dimension of the learned embedding. <br>
<br>
Please refer to <a href="https://github.com/aditya-grover/node2vec">node2vec github </a> for more details about how to prepare the input.<br>

### The supported input format is an edgelist: <br>
	node1_id_int node2_id_int
where node1_id_int can be the protein ID number. <br>
<br>
### The output file has *n+1* lines for a graph with *n* vertices.  <br>
The first line has the following format: <br>

	num_of_nodes dim_of_representation

<br>
The next *n* lines are as follows: <br>
	
	node_id dim1 dim2 ... dimd

where dim1, ... , dimd is the *d*-dimensional representation learned by *node2vec*. <br>


# 2. Train a LSTM classifier using learned embedding and erichment features derived from KEGG and GO, including version with Synthetic Minority Over-sampling Technique (SMOTE) and without SMOTE, which is integrated in imbalanced-learn.

In this study, node2loc mainly consists of the following three components: 1) learned embedding from a protein-protein network using node2vec; 2) SMOTE for oversampling minority classes; 3) a LSTM classifier for classifying 16 subcellular locations. Please refer to 2.2 for how to run node2loc for classifying and predicting protein subcellular locations.<br>

Here we provided the learned embedding with 500-D learned from a human protein-protein network. To yield higher performance, you can use Minimum redundancy maximum relevance (<a href="http://home.penglab.com/proj/mRMR/index.htm">mRMR</a>) to reorder the embedding, then train and evaluate each feature subset using IFS with RNN, and select the feature subset with the best performance. <br>

The dataset with 500-D embedding as reprenstations for proteins and subcellular locaitons as labels is given in this repository, including training and test set file. The training file is "train_dataset.zip", and you can decompress it to "train_dataset.csv" that is the embedding of proteins, and "train_sample.txt" that is the protein IDs as sample names. The mapping between label ID and subcellular locations is given in file labelID_to_locations. The test file test_dataset.zip contains the embedding for other proteins not in the benchmakr set and protein names correpsond to the embedding, and we want to predict the locations for them. <br>

You can test node2loc on the uploaded train_dataset.zip using k-fold crossvalidation. <br>
You can also predict the location for the proteins in test_dataset.zip using the trained node2loc model on train_dataset.zip. The output file gives the predicted locations for all proteins in the test set. <br>

## 2.1 Train and test LSTM classifier without SMOTE for oversampling.
1. Train the LSTM classifier without SMOTE for over-sampling:<br>
``` python3 rnn-kfold-run.py -c 16 --datapath subc_human_n2v_GO_KEGG_mRMR_550.arff -e 500 -u 400 -k 10``` <br>
where -c is the number of classes, --datapath is the training file with embedding as features, locations as the labels, -e is the dimension of embedding, -u is number of neurons in the hidden layer of LSTM, k is k-fold cross-validation. This program will evaluate the node2loc using k-fold cross-validation. <br>

## 2.2 Train and test LSTM classifier with SMOTE for oversampling.
1. Train the LSTM classifier with SMOTE for over-sampling:<br>
``` python3 rnn-kfold-smote-run.py -c 16 --datapath subc_human_n2v_GO_KEGG_mRMR_550.arff -e 500 -u 400 -k 10``` <br>
where -c is the number of classes, --datapath is the training file with embedding and enrichment features, locations as the labels, -e is the dimension of embedding, -u is number of neurons in the hidden layer of LSTM, k is k-fold cross-validation. This program will evaluate the node2loc using k-fold cross-validation. <br>

