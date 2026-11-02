# WEKA: SIMPLE STEP-BY-STEP GUIDE
(Same content as before, rewritten as click-by-click steps. Open **Weka GUI Chooser → Explorer**.)

**Explorer tabs you will use:** Preprocess · Classify · Cluster · Associate · Select attributes · Visualize

---
## STEP 0: If the dataset is only written on the question paper
Weka needs a file. Type it in **Notepad**, save as `data.csv` (Save as type: All files), then in Weka: **Preprocess → Open file → change "Files of type" to CSV → pick data.csv**.

**CSV rules:** first line = column names, then one row per line, values separated by commas, no spaces, no blank lines.
```
Outlook,Temp,Humidity,Windy,Play
Sunny,Hot,High,False,No
Overcast,Hot,High,False,Yes
Rainy,Mild,High,False,Yes
```
**ARFF alternative** (save as `data.arff`):
```
@relation weather
@attribute Outlook {Sunny,Overcast,Rainy}
@attribute Temp {Hot,Mild,Cool}
@attribute Humidity {High,Normal}
@attribute Windy {True,False}
@attribute Play {Yes,No}
@data
Sunny,Hot,High,False,No
Overcast,Hot,High,False,Yes
```
Numeric column → write `@attribute Marks numeric`. Missing value → write `?`.

---
## TASK 1: Preprocessing (clean the data)
1. **Preprocess** tab → **Open file**. Top right shows *Instances* (rows) and *Attributes* (columns).
2. Click a column name on the left → right side shows its statistics (numeric: min, max, mean, StdDev; nominal: counts) and a histogram.
3. Click **Choose** (filter box) → pick a filter → click **Apply**. To undo, click **Undo**.

| Goal | Filter path (Choose → …) |
|---|---|
| Fill missing values (mean/mode) | filters → unsupervised → attribute → **ReplaceMissingValues** |
| Remove duplicates | filters → unsupervised → instance → **RemoveDuplicates** |
| Scale to 0–1 | filters → unsupervised → attribute → **Normalize** |
| Mean 0, std 1 | filters → unsupervised → attribute → **Standardize** |
| Numbers → ranges (bins) | filters → unsupervised → attribute → **Discretize** (click filter name, set `bins` = 3/5/10, OK, Apply) |
| Find outliers | filters → unsupervised → attribute → **InterquartileRange** → Apply (adds columns *Outlier* and *ExtremeValue*) |
| Delete those outlier rows | filters → unsupervised → instance → **RemoveWithValues** → attributeIndex = Outlier column number, nominalIndices = `last` (the "yes" value) → Apply |
| Delete a column | tick its box on the left → **Remove** button |

Note: Weka's InterquartileRange uses factors 3.0 (outlier) and 6.0 (extreme) by default; the textbook rule is 1.5 × IQR.

---
## TASK 2: Visualisation
- **Visualize** tab: grid of scatter plots of every attribute pair, coloured by class. Slide **Jitter** to separate overlapping points. Double-click a plot to enlarge it.
- **Preprocess** tab: the histogram at the bottom right (pick "Class" in the box above it to colour by class) is your bar chart/histogram.

---
## TASK 3: K-means clustering
1. Load data (Preprocess tab). If a class/label column exists, you can leave it; we will ignore it next.
2. **Cluster** tab → **Choose → SimpleKMeans**.
3. **Click on the white box showing "SimpleKMeans"** → set `numClusters` = k (e.g. 3) → OK.
4. Choose a cluster mode:
   - **Use training set** → plain clustering.
   - **Classes to clusters evaluation (Nom) class** → compares clusters to the real class and reports *Incorrectly clustered instances %*.
5. If you don't want a column used (ID/name/class): click **Ignore attributes** → select it → Select.
6. Click **Start**.
7. **Read the output:** *Final cluster centroids* table (columns Full Data, Cluster 0, 1, …) and *Clustered Instances* (how many rows per cluster).
8. Plot it: **right-click the result in "Result list" → Visualize cluster assignments**.

*Missing values are automatically replaced by mean/mode before clustering.*

---
## TASK 4: Naive Bayes classification (Weka)
1. Load data. **Classify** tab → **Choose → classifiers → bayes → NaiveBayes**.
2. Pick a **Test option**:
   - *Use training set*: test on the same data (optimistic).
   - *Cross-validation, Folds 10*: most reliable for small data.
   - *Percentage split 66%*: train 66%, test the rest.
3. Make sure the drop-down above Start shows your class column, e.g. **(Nom) Play**.
4. Click **Start**.
5. **Read the output:**
   - *Correctly Classified Instances* (the accuracy).
   - *Confusion Matrix*: rows = ACTUAL, columns = PREDICTED. Diagonal = correct.
   - *Detailed Accuracy By Class*: TP rate (= Recall), FP rate, Precision, F-Measure.
   - *Kappa*: 1 = perfect, 0 = no better than chance.
6. If **Start is greyed out**: the algorithm cannot handle your column types → apply **Discretize** (Task 1) first.

**Formulas:** Accuracy = (TP+TN)/Total · Recall = TP/(TP+FN) · Precision = TP/(TP+FP) · FP rate = FP/(FP+TN).

---
## TASK 5: Hierarchical clustering (Weka)
1. **Cluster** tab → **Choose → clusterers → HierarchicalClusterer**.
2. Click the white box with its name → set `numClusters` (e.g. 2 or 3) and `linkType` (SINGLE, COMPLETE, AVERAGE, WARD…) → OK.
3. Cluster mode: *Use training set* → **Start**.
4. See the tree: **right-click the result in "Result list" → Visualize tree** (this is the dendrogram).
5. The output text also prints the tree in bracket form and *Clustered Instances*.

---
## TASK 6: Apriori (association rules)
1. Apriori needs **nominal (text) columns**. If a column is numeric, apply **Discretize** first.
2. For market-basket data, each item is a column with values `t` (bought) and `?` (not bought).
3. **Associate** tab → **Choose → Apriori**.
4. Click the white box and set:
   - `lowerBoundMinSupport` = minimum support (0.15 = 15 %)
   - `minMetric` = minimum confidence (0.9 = 90 %)
   - `numRules` = how many rules to show (e.g. 10)
5. Click **Start**.
6. **Read the output:** *Size of set of large itemsets L(1), L(2), …* then *Best rules found*, e.g.
   `outlook=overcast 4 ==> play=yes 4  conf:(1)  lift:(1.56)`
   → left side = IF part, right side = THEN part; the numbers are how many rows; conf (1) = 100 %.

*Weka starts at support 1.0 and lowers it step by step until it finds numRules rules or reaches lowerBoundMinSupport.*

---
## TASK 7: Attribute selection (which columns matter)
**Select attributes** tab → Attribute Evaluator: **InfoGainAttributeEval** → Search method: **Ranker** → **Start**. Top of the list = most useful column; bottom ones (score ≈ 0) can be removed.

---
## PageRank
Weka has no PageRank tool. Use notebook `7_PageRank_No_Damping.ipynb`.

---
## Quick viva answers
- **Weka?** Free Java data-mining tool with preprocessing, classification, clustering, association rules, attribute selection and visualisation.
- **ARFF?** Weka's file format: `@relation`, `@attribute`, `@data`.
- **Why Discretize?** Some algorithms (ID3, Apriori) need categorical data; binning also reduces noise.
- **Why cross-validation?** Honest accuracy: every row is used once for testing.
- **Classes-to-clusters evaluation?** Weka maps each cluster to a class and counts the wrong ones.
