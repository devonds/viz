# photon

AI-enabled biomedical data visualizer.  Learn how to display data *from* data.

We enable users to ask clinical questions and receive data as a result, similar to Wolfram Alpha for medical records. We have coded charts for specific result types, such as survival curves to line plots, but we would like to support a broader range of questions and resultant output data. Given this, we would like to automatically assign a set of output data to a certain chart type that most clearly provides the information to the end user.

This problem addresses the Edward Tufte-esque methods for how best to visualize data for optimal understanding. In a clinical setting, doctors and clinical researchers have little time to make sense of a mountain of complex information. The visual representation of an individual patient’s case is a challenging one, especially when attempting to answer any number of varied, multifaceted questions. 

This problem has the benefit of limited scope. We don’t need to display data for all conditions, or even all infectious diseases - just coronavirus in this case. While we expect to expand this to a broad array of conditions and applications, the benefit of limiting scope in this case will help us test the model’s ability to reliably answer a relatively dense question set with a wide array of possible data types.

**Contributor**: [Devon Sampson](devonds@gmail.com).

## TODO

- [x] Devon registers for [OS](os.omic.ai).
- [x] Devon signs Nondisclosure Agreement.
- [ ] Profiling visualization dataset uploaded -- 200 samples.
- [ ] Training visualization dataset uploaded -- 1K+ samples.
- [ ] Data2vis model up and running.

## Background

- data2vis: https://github.com/victordibia/data2vis.
- svist4get: https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-019-2706-8.
- iobio: http://iobio.io/.
- Department at UW making great progress in this area: https://idl.cs.washington.edu/.
- Data framework used by data2vis:  https://vega.github.io/vega-lite/.

## Data

Good samples here: https://github.com/vincentarelbundock/Rdatasets/tree/master/csv/Stat2Data.

See the `data/` for project data being committed as it is generated.

### Structure

```
([
    entry_0,
    ...
    entry_n
], viz_category)
```

where `viz_category` is in the set `["circos", "line", "bar", "pie", "sunburst", "table", "paragraph", "histograph", "scatter", "funnel"]` and `entry_*` are JSON objects with identical feature alias (but obviously, potentially different values!) -- an example being `{"num0": "0", "num1": "4", "num2": "80", "str0": "female"}`.

### Privacy

The data provided for the development of this model is highly protected; access to this data is permissable only by the signing of our Nondisclosure Agreement and under no condition can be distributed outside of Omic, Inc.

Software developed on top of this data, however, are openly shareable, so long as they do not immediately pose a risk of the above data policy.

*tl;dr don't share data and be mindful with sharing how you interface with the data.*

## Model

We would look for a model that accepts a data set as input and provides as output a D3 chart. We are currently experimenting with an RNN approach ala data2vis (https://github.com/victordibia/data2vis), and would look for someone to propose a superior approach or extend the existing data2vis model based on clinical and complex molecular use cases that exist in analyzing coronavirus. 

These would include the return of host genome and virus genome data, survival curves, treatment paths, phylogenetic trees, and other COVID-19-specific data visualizations.

### Anticipated Approach
The original idea was to get Data2Vis running as the first step, but is dependent on older versions of tensorflow and the underlying seq2seq model may be overkill for these purposes.  Instead, we'll start with the simplest possible LSTM -based recurrent neural network, and make it in PyTorch. This should provide more flexibility later. We'll still lift a lot of the training data and preprocessing methods from Data2Vis.

Following the example from Data2Vis, the *learning objectives* are
- select a subset of fields to visualize
	- data2vis identifies 6 chart types: area, bar, circle, line, point, tick
- learn differences in data types across fields (numeric, string, temporal, ordinal, categorical etc; 
	- in our case, also types specific to genomic data)
- learn appropriate transformations to apply to a field given it's data type
	- data2vis identifies 3 transforms: aggregate, bin, time-unit

*Training data* We'll draw on two sources to train the model: 
- the same pairs of data (in JSON format) and chart specifications that data2vis used. (4300 examples up-sampled to 215,000 training pairs)
	- This is how data2vis generates these pairs: 
		Our training dataset is constructed from 4300 Vega-Lite visualizations examples, based on 11 distinct datasets. The exam- ples were originally compiled by [52] where the authors use the CompassQL [76] recommendation engine within Voyager2 [77] to generate charts with 1-3 variables, filtered to remove problematic instances. These charts are generated based on heuristics and rules which enumerate, cluster and rank visualizations according to data properties and perceptual principles [77]. While these examples contain a simplified range of transformations and do not encode any interactions, they represent valid Vega-Lite examples and conform to important perceptual principles enforced by rules within Voyager2. These characteristics make the dataset a suitable, low-complexity test bed for benchmarking our model’s performance on the task of learning to generate visualizations given only input data.
		We iteratively generate a source (a single row from the dataset) and target pair (see Figure 3) from each example file. Each example is then sampled 50 times (50 different data rows with the same Vega-Lite specification) resulting in a total of 215,000 pairs which are then used to train our model.
- 200-300 data-visualization pairs from Omic (?)
We can train our model on the larger and more general data set first, then freeze a all but the last hidden layer to train with omic-specific training data

*Preprocessing data*
- From data2vis: "Achieving these objectives using a character based sequence model can be resource intensive. While character based models re- sult in smaller vocabulary size and are more accurate for specialized domains, they also present challenges — a character tokenization strategy requires more units to represent a sequence and requires a large amount of hidden layers as well as parameters to model long term dependencies [8]. To address this issue and scaffold the learning process, we perform a set of transformations. First, we replace string and numeric field names using a short notation —
“str” and “num” in the source sequence (dataset). Next, a similar backward transformation (post processing) is eplicated in the tar- get sequence to maintain consistency in field names
[ ] Devon attempts to replicate this upsampling 

*Coding and model architecture*
- Build a LSTM-based recurrent neural network in PyTorch
[ ] Devon gets up to speed on PyTorch
[ ] Devon investigates LSTM based model architecture and selects one with Luke

### Tier 1

This model will take arbitary input JSON and predict the chart type in the set `["line", "bar", "pie", "table", "histograph", "scatter"]`.  The training data will be transformed by the data2vis set [here](https://github.com/victordibia/data2vis/tree/master/examples).

Metrics for success:
 * Accuracy:  `0.70`.

### Tier 2

TODO

### Tier 3

TODO

### Tier 4 (Boss Level)

Collect and package all references and results.  Publish a paper in a top journal.  Save the princess.
