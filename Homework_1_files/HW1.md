# Homework 1: Clustering & Disease Phenotyping

**Name:** Jeffrey Wang

## Conceptual Questions (20 pts)

### 1. (3 pts)

Define a *clinical phenotype*. Why is unsupervised clustering useful for discovering disease subtypes?

**Answer:**
A clinical phenotype is an observable trait or disease expression of a patient caused by the interaction between their genes and the environment. Some examples are symptoms, imaging, and treatment response. Unsupervised clustering aims to define groupings without any pre-defined labels which is useful for discovering disease subtypes because these phenotypes and their interactions with each other may reveal subtypes unbeknownst to current label techniques. Without relying on labels which may introduce human bias, we let the observable feature set differentiate the data into subtypes.

### 2. (3 pts)

Compare k-means vs. hierarchical clustering. What assumptions do they make about cluster shape/structure?

**Answer:**
With k-means clustering, you need to choose some positive integer `k` with the assumption that there are `k` number of groups. Labels are still not needed but this value determines how many centroids the algorithm can decide what a point is assigned to. It also assumes that the shape of these clusters around the centroids are spherical and convex. It is sensitive to initialization of centroids and outliers, however. On the other hand, hierarchical clustering constructs a tree/dendrogram and links points together by some distance calculation method (which also determines its shape). It connects points/clusters one-by-one in levels such that cutting the dendrogram at a chosen height yields `k` clusters.

### 3. (4 pts)

How can dimensionality reduction (PCA) help in analyzing high-dimensional scRNA-seq data?

**Answer:**
PCA is a lossy data compression algorithm that reduces datasets with large amounts of features (like scRNA-seq) into fewer, orthogonal axes that capture variance in the data. This can help remove noise, gene correlation, and with visualizations as you can plot the top two principal components on a 2D plot. 

### 4. (5 pts)

In disease phenotyping using cluster analysis, why is it important to identify clinically recognizable clusters? Put differently: what is/are the problems if the identified clusters are not clinically meaningful?

**Answer:**
Clusters could be exploiting noise or other unassuming issues with the dataset like batch effect. If all experiments done under some condition that isn't the intended, independent variable like location or time of test(s), are clustered together rather than pointing to some clinical relevant hint, you're likely capturing artifacts. These aren't to be trusted due to how irrelevant or untranslatable they are to our research question.

### 5. (5 pts)

Partitioning Around Medoids (PAM) with Gower's distance is designed for clustering mixed data types. Why did K-means with Euclidean distance perform better on the clinical data (with mixed data types) in Lecture 5? Similarly, why did K-means with Euclidean distance outperform K-means with Manhattan distance on the same dataset?

**Answer:**
K-means with Euclidean distance outperformed PAM/Gower because the pre-processing step of the dataset involving standardizing all features and nominal variables being one-hot encoded meant that the space was geometrically consistent. With Gower, that means that ordinal variables lose their degree and results in variables (both clinically relevant and noisy) with equal weighting. Euclidean beat Manhattan because it squares the differences between clusters, rewarding large deviations in features like high inflammatory markers versus summing the absolute difference which involves the contribution of noise too much.

## Coding, Analysis, and Interpretation

### 2b.

Do the tissues separate in PCA space? Explain your observation.

**Answer:**
In the PC1 vs. PC2 plot (`figs/pca_by_tissue.png`), Limb_Muscle separates from the other two tissue-types along PC1, while Pancreas and Spleen separate from each other along PC2. Only a small number of cells overlap, such as a few Pancreas cells scattered into Limb_Muscle and Spleen and a few Limb_Muscle cells scattered into Spleen. So overall, yes, the tissues do separate ok in PCA space. Cells from different organs express different genes, so tissue-type explains a good amount of the variation. PC1 and PC2 only explain 11.1% and 6.2% of the variance though, hence some overlap and there could be improvements to how to separate even further.

### 3d.

How well do clusters align with tissue labels?

**Answer:**
The clusters are not perfect but align well with tissue labels. The best `k` chosen by silhouette score was 6 (silhouette = 0.397), and the clusters had an ARI of 0.604 and NMI of 0.665 against the tissue labels. Cluster 0 is mostly Spleen with some of cluster 3, clusters 1 and 4 are mostly Limb_Muscle with some of 3, and cluster 2 and 5 are mostly Pancreas. With 6 clusters total, k-means is further clustering Pancreas and Limb_Muscle into sub-populations, which likely reflect different cell types within the same organ. Overall, the clusters mainly capture tissue identity, with additional within-tissue cell-type structure.

### 4c.

How does your alternative clustering method compare to k-means clustering?

**Answer:**
I used agglomerative (hierarchical) clustering with Ward linkage on the same 50 PCs, with `k = 6` to match k-means. The two methods perform about the same on the internal metric: the silhouette score is 0.400 for agglomerative versus 0.398 for k-means. K-means aligns better with the tissue labels though (ARI 0.604 and NMI 0.665, versus 0.542 and 0.601 for agglomerative). Both methods split Pancreas and Limb_Muscle into sub-clusters and keep most of the Spleen together, and the two are similar overall because Ward linkage minimizes within-cluster variance just like k-means does. The gap in ARI/NMI likely comes down to how each method draws the boundary on the handful of cells near cluster edges. Agglomerative clustering is deterministic with no random initialization but it does scale worse with the number of cells. I conclude that neither method is significantly better than the other in this context since the silhouette scores are nearly identical.

### 5b.

Are the identified markers similar for different tissues? Explain briefly why they are or are not similar.

**Answer:**
No, the top markers are mostly different across tissues (using the agglomerative clusters). The Pancreas clusters were marked by digestive enzyme genes (*Pnliprp2*, *Pla2g1b*, *Cpa2*) in cluster 4 and islet hormone/secretory genes (*Iapp*, *Chga*, *Pcsk2*) in cluster 5, the Spleen cluster by pan-immune genes (*Cd52*, *Coro1a*, *Rac2*), and the Limb_Muscle clusters by stromal genes (*Dcn*, *Pcolce*, *Igfbp6*) in cluster 1 and endothelial genes (*Cdh5*, *Emcn*, *Egfl7*) in cluster 3. These lists share no genes, because each tissue has its own specialized cell types with their own expression programs. Markers are tied to cell type more than to tissue: two clusters from the same tissue (e.g. acinar vs. endocrine cells in the Pancreas) also had completely different markers. The one exception is cluster 0, the mixed cluster, whose markers (*Lyz2*, *Lyz1*, *Alox5ap*, *Fcer1g*, *Tyrobp*) are macrophage/myeloid genes. Macrophages are found in every organ and express similar genes regardless of tissue, which explains why that cluster contains cells from all three tissues.
