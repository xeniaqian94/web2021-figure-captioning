# Perfect Accuracy Scores

We re-ran experiments in Table 4 and 6, collected model output, and calculated perfect accuracy as follows.

|    Dataset   |    Split   | # Figures | Avg # captions per figure | Perfect accuracy |
|:------------:|:----------:|:---------:|:-------------------------:|:----------------:|
| FigureQA-cap | validation |   3,000   |           3.946           |       0.926      |
| FigureQA-cap |  test_easy |   3,000   |           4.330           |       0.907      |
| FigureQA-cap |  test_hard |   2,998   |           4.003           |       0.911      |
|   DVQA-cap   | validation |   4,572   |           2.350           |       0.944      |
|   DVQA-cap   |  test_easy |   4,447   |           2.280           |       0.948      |
|   DVQA-cap   |  test_hard |   4,605   |           2.374           |       0.939      |
 

## Sanity Check

The results are largely consistent with Table 2, 3 and 5. 

Reviewers may note that the number of figures (images) is different from Table 2. This is as expected. 

For `test_hard` split of FigureQA-cap, the server log that gives model output has a few rare cases of misplaced lines, that makes number of figures two off from 3,000.

For DVQA-cap, the total number of figures is 5,000. This includes the Title type for post-editing simulation in Section 8. Table 5 results does not include the Title type, therefore has fewer number of figures.

## Example

The script will output stats and aggregated captions for 10 random examples.

Please see [FigureQA-perfect-accuracy-output](https://github.com/anonymous-web2021-sub/data-release/blob/master/aggregated-perfect-accuracy/FigureQA-perfect-accuracy-output) and [DVQA-perfect-accuracy-output](https://github.com/anonymous-web2021-sub/data-release/blob/master/aggregated-perfect-accuracy/DVQA-perfect-accuracy-output).

The server logs of model output are mostly larger than 50MB, we compress at the best effort in [`FigureQA/`](https://github.com/anonymous-web2021-sub/data-release/tree/master/aggregated-perfect-accuracy/FigureQA) and [`DVQA/`](https://github.com/anonymous-web2021-sub/data-release/tree/master/aggregated-perfect-accuracy/DVQA) directories. 


## Script

Below script aggregates captions for the same figure, and calculate the results. 

```
import os
import re
import numpy as np
from collections import defaultdict

# input_folder = "FigureQA" # uncomment to report FigureQA results
input_folder = "DVQA"
input_files = sorted([f for f in os.listdir(input_folder) if f.split(".")[-1] == "out"])

per_figure_captions = defaultdict(lambda: defaultdict(dict))

# Separately report each split for stats and perfect accuracy number
# For each split, do regex match on log line  as "Loader name val_loader epoch ..."
global current_split  # "val_loader" or "test_easy_eval_loader" or "test_hard_eval_loader"
current_split = ""


def next_line(f):
    while True:
        line = next(f).split("\n")[0]

        if "EVALUATING AT BEAM SIZE" not in line:
            return line
    return ""


for f in input_files:
    file_path = os.path.join(input_folder, f)

    units = []
    f = open(file_path, "r")
    if len(file_path.rstrip(".out")) == 0:
        continue
    print(file_path)

    for line in f:
        # Here we find every unique reference (i.e. ground-truth) of a figure, then get a 0/1 match result as value
        # The log outfile contains output for all training epochs, we output the last output at latest timestamp
        #   at the cutoff timestamp, the prediction is generally the most accurate

        if re.findall("Loader name (.*) epoch", line):
            current_split = re.findall("Loader name (.*) epoch", line)[0]

        for image_name in re.findall("Image name:  (.*)", line):
            try:
                line = next_line(f)
                line = next_line(f)
                line = next_line(f)
                ref = re.findall("References .* for this figure in the batch: (.*)", line)[0]

                line = next_line(f)
                line = next_line(f)
                is_match = re.findall("This hypothesis matching one of the references\?  (.*)", line)[0]

                per_figure_captions[current_split][str(image_name)][str(ref)] = int(is_match)
            except:
                # In rare case, the slurm log would have encoding errors in some blocks
                continue

print()
print("For", input_folder)

for split in per_figure_captions:
    print("Current split is ", split)
    print("# figures", len(per_figure_captions[split]))
    print("Avg # of captions figures",
          np.mean([len(per_figure_captions[split][fig]) for fig in per_figure_captions[split]]))
    print("# figures that has perfect match",
          np.mean([(1 if all(list(per_figure_captions[split][fig].values())) else 0) for fig in
                   per_figure_captions[split]]))

print()
print("Random 10 examples of aggregated captions and match (0/1) - \nNote for DVQA, validation has image name of bar_train_0000XXXX.png, test has image name of bar_val_xxxx_0000XXXX.png\n\n")
for split in per_figure_captions:
    threshold = 0
    for fig in per_figure_captions[split]:
        print(fig, per_figure_captions[split][fig])
        threshold += 1
        if threshold == 10:
            break
    print()
```
