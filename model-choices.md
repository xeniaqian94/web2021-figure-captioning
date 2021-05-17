### Supplementary Material: Model Hyperparameters and Other Design Choices

The `FigJAM` model mainly follows the model configuration and implementation details from [DVQA](https://openaccess.thecvf.com/content_cvpr_2018/papers/Kafle_DVQA_Understanding_Data_CVPR_2018_paper.pdf) (Kafle et al., 2018). 

#### Pre-processing
For data pre-processing before running all models, we resize figure images in all datasets and their corresponding metadata to be 448 × 448.

#### Encoding
To encode figure image, we fine-tune a pre-trained ResNet-50 up to the second last average pooling layer. The pre-trained ResNet-50 comes from [PyTorch and is trained on ImageNet](https://pytorch.org/hub/pytorch_vision_resnet/). As a result, the output image representation has a size of 14 × 14. 

#### Dyanmic Dictionary and Word Embedding
The `FigJAM` model uses the same approach as the `DVQA` baseline to construct the dynamic dictionary. This design greatly reduces the dictionary size.
The dynamic dictionary has 30 slots. It assigns unique IDs to words that appear only in the metadata. 
Other words that appear in the captions but not in the metadata constitute the static dictionary. 
On the `DVQA-cap` dataset, the dynamic and static dictionary of `FigJAM` model have a total size of 93. On the `FigureQA-cap`, the dynamic and static dictionary have a total size of 72.
Dictionary words are represented by 128-dimensional word embedding. An embedding size of 128 is compatible with the reduced dictionary size due to dynamic dictionary. 

#### Model Architecture 
The auxiliary classifier is a two-layer MLP with an intermediate dimension of 512 and an output dimension that is the same as the dictionary size.
Decoder LSTM has a hidden representation dimension of 256. We use teacher forcing during training and beam search with size 1 during testing.

#### Optimization 
We use the Adam optimizer with a learning rate of 1e-3 and cross-entropy losse. The loss applies to both the auxiliary classifier and the decoder. 
Networks were trained using NVIDIA Titan X graphics card, for 50,000 batches with a batch size of 8.
