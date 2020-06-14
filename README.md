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


# 2. Train a LSTM classifier using learned embedding and enrichment features derived from KEGG and GO with Synthetic Minority Over-sampling Technique (SMOTE), which is integrated in imbalanced-learn.

In this study, rnnloc mainly consists of the following five components: 1) learned embedding from a protein-protein network using node2vec; 
2) enrichment feautres derived from KEGG and GO terms; 3)SMOTE for oversampling minority classes; 4) a LSTM classifier for classifying 16 subcellular locations. 
5) decision tree in scikit-learn for generating classification rules. <br>
Please see below to knwo how to run rnnloc for classifying and predicting protein subcellular locations.<br>

Here we provided the final optmized 550-D feautres from embedding and enrichment features. You can use it to do 10-fold cross-validaiton as done in our paper.

You can test rnnloc on the uploaded subc_human_n2v_GO_KEGG_mRMR_550.arff.gz using 10-fold cross-validation as done in paper. <br>

## Evaluate LSTM classifier with SMOTE for oversampling using 10-fold cross-validation on the final optimized 550-D features.
1. Evaluate LSTM classifier with SMOTE for over-sampling using 10-fold cross-validaiton:<br>
``` python3 rnn-kfold-smote-run.py -c 16 --datapath subc_human_n2v_GO_KEGG_mRMR_550.arff -e 500 -u 400 -k 10``` <br>
where -c is the number of classes, --datapath is the training file with embedding and enrichment features, locations as the labels, -e is the dimension of embedding, -u is number of neurons in the hidden layer of LSTM, k is k-fold cross-validation. This program will evaluate the rnnloc using k-fold cross-validation. <br>

