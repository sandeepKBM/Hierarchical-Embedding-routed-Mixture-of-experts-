# Hierarchical-Embedding-routed-Mixture-of-experts-
(In process of further testing and paper will be drafted by March end)
So the model architecture given in this work is for the cases where a input space is given with few classification labels but if we are to dig through the data we can fnd many more sub classification labels which are all generalized into a smaller set of labels. If we are able to partition the space better we can build models for subsets of data which will lead to higher performance and easier training.

While one can say that MOE is well suited to do this I feel Hierarchical Mixture of Experts models are much more better suited. But one can say that Hierarchical Mixture of Experts is hard to work with and there are all those multitute of issues down there. I hope my work will be able to make this wonderfull algorithm more useful.

My inspiration for this model came from a another paper Hierarchical Routing Mixture of experts (https://arxiv.org/abs/1903.07756) Where they have given a recursive EM algorithm for joint partitioning the input and output space. But this method works only for regression problems as the joint partitioner uses a threshold to split the Y space. 

To extend this case to classification cases I need to first make it work for embeddings. To make it work for embeddings I designed a new method to partition the X & Y spaces for the classification case. the model looks like a tree so i would call the classifiers inside the tree non leaf classifiers and the final ones leaf classifiers.

The method for this model is fairly simple
1. A model is first pretrained & finetuned on the full dataset using some vision model
2. Then the input data is sent to this model and the final embeddings are obtained (output layer is pruned out)
3. For all the data this is done to get embeddings.
4. these embeddings are used in a Deep Embedded clustering to find K clusters (This is determined using a grid search). If all the clusters have a mininum amount of data then various sub models (The same primary model just the pretrained version is used) using these clustered data points are trained for each clustered data.
5. The weighted accuracy for these sub models is taken and a grid search is passed over all possible K values which fulfil the basic conditions. The k with hthe ighest accuracy is chosen.
6. The trained models are discarded and the final clusters are again put through the same process of finding sub clusters for these clusters to again from step 1 to 5. This process goes on till either till all k values have at least a single cluster size below min limit or the max depth is reached.
7. When the final depth is reached leaf final classifier are trained using these sub clusters.

In Cifar 32 I have seen a 1 percent increment in accuracy compare to base model(It is a bare-bones implementation and need several refinements. With some regularisation, augmentations and better non leaf and leaf model training using those LORA training methods should give better trained models and also allow us to go for deeper depths more easily) 

Those non leaf node routers which produce embeddings can also be put through grad lam (https://www.eurasip.org/Proceedings/Eusipco/Eusipco2020/pdfs/0001407.pdf) (It is for getting explainability from those embedding models) and using the data flow path from root to leaf we can also gain a degree of explainability in a post hoc sense which can allow us gain a understanding of what features the model is using for its decision. 

Currently working on testing on patchcamelyon benchmark and also work on reducing memory consumption due to compute constraints faced by me (Colab has its limits) and current implementation uses a lot of RAM so currently making all that more efficient to get comprehensive results and overall better models.

