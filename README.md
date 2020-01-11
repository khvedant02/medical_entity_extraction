# Medical Entity Extraction from Clinical Notes using DNC

## Collaborators
[Deep C. Patel](https://github.com/deepcpatel) and [Akshay Shah](https://github.com/shahakshay11)

## Dataset used
2010 i2b2/VA challenge dataset [5]. 

## Background
The 2010 i2b2/VA challenge task 1 [5] is basically to extract medical entities from unannotated clinical texts. This task can be categorized under the umbrella of classic NLP task known as Named Entity Recognition. Here, the extracted medical entities can be classified into a medical problem, a treatment, or a test. We are attempting to solve this task using Differentiable Neural Computer [1].

## Our approach
Inspiring from the usage of Differentiable Neural Computer (DNC) for the question answering Task [1, 2], we tried to use it for the Medical Entity Extraction task in this course project. The advantage of using it over LSTMs is that, unlike them, DNC keeps Neural Network and Memory separate or in other words Neural Network controller is attached to external memory, which allows larger memory allocation without an increase in the parameters. Moreover, the stored information in the memory faces very little interference from the network operations, thus allowing it to be remembered for a longer time.

## Adapting DNC for Medical Entity Extraction task
As indicated above, DNC was used for question answering task in the original paper [1]. However, it was not performing robustly as pointed out by the authors in [2]. The prime issue suspected was that the DNC might not be using memory units in some cases (not using read vectors) and instead be solely generating output using only the given inputs. This issue was solved by putting the Dropout layer before the final output generation, which allowed both the components to be used equally. Moreover, the authors in [2] perform Layer Norm on the controller output to control high variance in performance during different DNC runs.

To get the information of future tokens in the sequence, the authors in [2] also proposed to use a backward LSTM controller in parallel to DNC. These modifications to DNC showed robust performance during question answering task according to them. Thus, to make our Entity Extraction task robust, we also modified our DNC implementation as recommended in [2].

We approached the Medical Entity Extraction task as a token classification task; the classes being `Problem`, `Treatment` and `Test`. And since we adopted BIO tagging as our labeling convention, specifically our classes are `B-Problem`, `I-Problem`, `B-Test`, `I-Test`, `B-Treatment`, `I-Treatment` and `Other`. Finally, we implemented DNC in PyTorch and programmed it to classify each input tokens into one of these classes.

## Data pre-processing
As such, the data required a moderate amount of preprocessing. We first tokenized each Medical Summary into words and removed noisy characters such as `\n` and more than one space. From the `.con` files, we extracted entities for making true labels. After this step, we converted all the words into word2vec vectors using the pre-trained word2vec embeddings on the PubMed corpus and MeSH RDF from [3]. After that, we divided the data into batches and fed them into the DNC.

## Training
We trained our DNC with a batch size of 10 for 100 epochs using Nvidia GTX 1080 GPU. The training took us approximately 10 hours. To update weights, we calculated Softmax Cross-Entropy loss between predicted output and labels and propagated gradients back to the model for weight updation. Since the DNC is fully differentiable end to end, the backpropagation algorithm can be easily used to update its weights. We used only one read and write head in DNC with a memory size of 128 x 128.

## Results
The results of our model are outlined as follows. To clarify at the beginning, we consider a classification to be True Positive only if all the words in an entity correctly matches with that of the corresponding label (exact match) and hence adds +1 to our True Positive score, else adds +1 to False Negative score.

| Entity Type | Precision | Recall | F1 Score |
|---|---|---|---|
| problem | 0.78 | 0.74 | 0.76 |
| test | 0.85 | 0.62 | 0.72 |
| treatment | 0.83 | 0.62 | 0.71 |

Total entity classification accuracy: **66.76 %**<br/>
Macro average F1 Score: **0.73**

## Future work
(1).​ Use BERT embeddings as word embeddings for the data instead of word2vec.<br/>
(2). Along with word embeddings, add character level embeddings, Parts Of Speech information to input as outlined in [4].

## Final message
We welcome contributions, suggestions or any bug findings in our work.

## References
**[1]**. A. Graves, et. al. Hybrid Computing Using a Neural Network with Dynamic External Memory. Nature 538, 471-476, 2016.<br/>
**[2]**. ​J. Franke, J. Niehues, A. Waibel. Robust and Scalable Differentiable Neural Computer for Question Answering. arXiv preprint arXiv:1807.02658, 2018.<br/>
**[3]**. Y. Zhang, Q. Chen, Z. Yang, H. Lin, Z. Lu. ​BioWordVec, Improving biomedical word embeddings with subword information and MeSH. Scientific Data, 2019.<br/>
**[4]**. J.P.C. Chiu, E. Nichols. Named Entity Recognition with Bidirectional LSTM-CNNs. Transactions of the Association for Computational Linguistics, Volume 4, 2016.<br/>
**[5]**. Ö. Uzuner, B.R. South, S. Shen, et al. 2010 i2b2/VA challenge on concepts, assertions, and relations in clinical text. J Am Med Inform Assoc, 18:552–6, 2011.