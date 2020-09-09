# SWIMMER
Statistical Workflow for Identification of Molecular Modulators of ribonucleoprotEins by Random variance modeling

Readme for the Python code for screen hit identification, which I used to run in the Jupyter notebook app under the Anaconda package. I initially wrote the code to work in Python 2 but updated it to work in Python 3 in early 2019.

This readme should be used in conjunction with a sample input raw data file ("before DAPI.csv") as well as the corresponding output files that the code should output (2 files, the analysis itself and the analysis parameters for metadata keeping).

There are 2 documentation files for the code, the first is the present documnet, an overall of the code in plain English. The second contains instructions on how to change parameters in the code for your screening library.

Overall, this pipeline takes raw screen data and outputs the locations of the hit wells (i.e. which library plate, row, column) based on 2 statistical hit identification methods explained in the documentation. I have separate scripts for further analyses such as counterscreening, data representation, etc.


Overall:
1) The code takes as input a .csv file with your raw data arrayed such that the first column contains raw data from screen library plate 1, arrayed out row-by-row. All subsequent library plates from the first replicate are arrayed out in additional columns to the right. Finally, additional biological replicates are arrayed out in further columns to the right. For example, in the sample input file "before DAPI.csv" the first column contains 384 entries from the first 384 well plate of our screening library, arrayed out so that the 24 values from row A are in the first 24 cells, and the next 24 values from row B are in the next 24 cells, and so on. The second column contains 384 entries from the second library plate. Finally, because the library screened had 7 total 384 well plates, the first biological replicate data is in the first 7 columns. The second biological replicate data is in the last 7 columns. *Note that the block of data should have the upper left hand corner in the 5th row, 2nd column.

2) The code performs K nearest neighbors (KNN) missing data imputation to fill in any rare wells that are missing data. The reason for this is that as a first pass analysis of a large screen, I preferred to decrease false negative rate (at the cost of increased false positive rate), because I planned to do subsequent counterscreening to help eliminate false positives. *You can change the parameter for number of nearest neighbors to use to impute the missing value; default is 10.

3) Next, it performs Tukey's median polish and calculation of b scores. *You can change the parameter for number of iterations for the median polish; default is 10.

4) Next, it selects hits based on MAD/median absolute deviation cutoff. For example, hits will be called for values >3 MADs below the median, and SG_enhancers will be called for values >3 MADs above the median. *You can change the parameter for number of deviations as cutoff; default is 3.

5) Next, it selects hits based on RVM/random variance modeling. For example, hits will be called for values below the median with p<0.001, and SG_enhancers will be called for values above the median with p<0.001. *You can change the p cutoff; default is 0.001.

5b) There is some commented out code for controlling FWER/family wise error rate or FDR, but I found this to be too stringent, with >90% of my eventual true hits (as validated in subsequent counterscreens) missing the cutoff. Again, I decided not to use FWER in favor of decreasing false negative rate at the cost of increased false positive rate, preferring to validate in secondary counterscreens. However, the skeleton of the code is there.

6) Finally, the code writes out the analysis and analysis parameters into .csv files. If you take a look at the sample output file "cvb smnpc spectrum reDAPI analysis.csv" and scroll down, you will see that several chunks of intermediate analysis data are output: starting with the raw input data, the next block output (~384 rows down in the csv file) is after KNN imputation, the next is after polish, then data after b-score calculation, and finally the hits and SG_enhancers via both cutoff methods (MAD, RVM). The hits are listed as follows: first column is an arbitrary index, second column is the median b-score for the hit, the third column is the p value for the hit if looking at the RVM hits (the MAD hits don't have this value). The next three rows contain the physical location of the hit on the library plate, given as plate, row (converted to number), column. For example: 3, 2, 17 would mean the hit was located on library plate 3, row B, column 17. Finally, the last columns are the individual raw b-scores for the hit compound across each of the biological replicates. Since my library was screened in biological duplicate, there are 2 raw b-scores for each hit.

6b) There is some additional code at the bottom that I used to confirm that the distribution of variances across the biological replicates are well modeled by an F distribution, which is an assumption made by the RVM method.
