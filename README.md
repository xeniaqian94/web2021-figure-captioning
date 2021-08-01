# web2021-figure-captioning

Data and supplementary materials for the Web Conference 2021 paper "Generating Accurate Caption Units For Figure Captioning".

### Some Caveats 

While the paper expects to appear in [dl.acm.org](https://dl.acm.org/), currently the final camera ready version is available [here](https://terpconnect.umd.edu/~xinq/Figure_captioning_WWW21.pdf). 

In our view, figure captioning is a visionary problem. Our work on this line is a proof-of-concept of ML capability, based on synthetic figure question answering data availability. 

### Table of Contents

<!--ts-->
   * [Data](#Data)
      * [Quality validation](https://github.com/xeniaqian94/web2021-figure-captioning/blob/main/quality-validation.xlsx)
   * [Model Hyperparameters and Other Design Choices](model-choices.md)
   * [Aggregated Perfect Accuracy](https://github.com/xeniaqian94/web2021-figure-captioning/tree/main/aggregated-perfect-accuracy)
   * [Intellectual Property Note](#intellectual-property-note)
   * [Contacts](#contacts)
<!--te-->


### Data

#### Directory Structure

This directory contains three parts of material.

- Dataset (`DVQA-cap` and `FigureQA-cap`): groundtruth captions for modeling. Converted from `DVQA` and `FigureQA` datasets. Includes the full split `train`, `val`, `test_easy` and `test_hard`. Due to size limit, we provide Google drive link to the `train` split. All splits follow the same schema below.
- `quality-validation.xlsx`: a spreadsheet of quality validatio results. Two co-authors did quality validation on a sample of captions from the `test_hard` split of `captions.json` files, in two dimensions: accuracy and grammar. The sample covers 20 random captions for each type in each dataset.     
- `user-study-12-figures.html` (along with the directory of `user-study-png-output`): the 12 figures for the Google form user study.
- [`aggregated-perfect-accuracy`](https://github.com/xeniaqian94/web2021-figure-captioning/tree/main/aggregated-perfect-accuracy): Calculation of perfect accuracy scores as additional results from Table 3 and 5.

#### `captions.json`

In each split subdirectory, the file `captions.json` contains groundtruth captions and figure metadata that follow our problem formulation, for modeling.

Please download figure images from the original repository of [DVQA](https://github.com/kushalkafle/DVQA_dataset) and [FigureQA](https://github.com/Maluuba/FigureQA) dataset, for figure consistency. 

##### An example caption tuple 
<img src="Example-caption.png" width="500">

##### An example figure metadata tuple
<img src="Example-metadata.png" width="500">


Below code can be used to read JSON objects from `captions.json` files to understand its schema.

    import json

    print("Loading the validation set of converted captions, in JSON, e.g. from DVQA")
    jobject = json.load(open("FigureQA/captions.json", "r"))
    
    print("In this JSON object, the keys are ", jobject.keys())
    print()
    
    print("Total caption count in this validation split", len(jobject['captions']))
    print()
    
    caption_types = set([item['caption_template_fine_grained'] for item in jobject['captions']])
    print("Unique caption types (slight naming difference to Table 1 definition, e.g. horizontal-vertical means figure type)", caption_types)
    print()
    
    print("The metadata for one random figure looks like below (has dynamic dictionary, and bounding box positions)")
    print(jobject['metadata'][list(jobject['metadata'].keys())[0]])
    print()

### Intellectual Property Note

Thank you for readers interested about source implementation. We unfortunately cannot share here due to company policy. 

### Contacts
Please email questions to Xin Qian ([xinq@umd.edu](mailto:xinq@umd.edu)). Thank you!




