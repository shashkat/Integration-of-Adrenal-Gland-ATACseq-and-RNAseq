(12-18-23)
First, I am gonna download the processed ATACseq data and play around with it.

Types of files in processed folder-

1. bigBed narrowPeak - narrowPeak files in bigBed format
2. bed narrowPeak - narrowPeak files in bed format
3. bigWig - bigWig files
4. bam - bam files
5. bed idr_ranked_peak - idr_ranked_peak files in bed format

In order to open/view the bigBed files, I could have used the UCSC genome browser, Integrative Genomics Viewer (IGV) or bedtools (for usage in command line). I spent some time playing around with the UCSC genome browser. Learnt a bit about it and also discovered an interesting tool called ChromHMM.

ChromHMM is used to make inferences about the histone markers present in the whole genome using multiple histone modifications datasets (which are determined by doing Chip-seq after using antibodies specific to the different marker combinations possible for a histone).

Another similar tool, which also uses Machine Learning (Hidden Markov Model), is Segway. It also performs segmentation of the genome into functional elements (Apparently into more elements like enhancers and promoters, unlike ChromHMM which focuses only on chromatin states), using different types of data like ChromHMM.

Okay, back to UCSC. I learnt a bit about few more tracks which were shown on the default page at UCSC, like OMIM, H3K27Ac Mark (Often Found Near Active Regulatory Elements), Transcription levels, SNPs etc.

I then realized that even in the ENCODE webiste, I could view the bigBed narrowPeak file. I still later opened the bigBed (with only the identified peaks info, so not so interesting) file on UCSC and also the bigWig file (with fold change graphs, so more interesting). 

(12-19-23)

Then, I finally moved on to viewing the files using command line, bedtools.

First, idk why chatGPT suggested me to intersect the bigbed file with the bed file. That intersection would have yielded the common regions in them (the common narrowpeaks in the two files). I looked into the two file formats and it appears that bigbed is a compressed format, but also generally stores more data and hence is recommended for bigger datasets. I viewed its statistics using the bigbedinfo command from ucsc-bigbedinfo package.

Also installed and got a bit used to bigBedSummary package, which can show some simple parameters (coverage, mean depth, min depth, max depth) from a particular, specified range of a bigbed file.

Then, went on to use bedtools intersect command to find the commonalities between the two files (bed and bigbed). Basically, the regions they got signal from are same to what extent (I think they should be pretty much the same) 

Initially, the bedtools intersect command couldnt understand the file format of the bigbed file i was supplying. Hence, I went on to convert it into bed, using bigBedToBed package, which again was installed using conda install -c bioconda ucsc-bigbedtobed (just like bigbedinfo and bigBedSummary).
Turns out that the bigbed file was actually just the bed file in bigbed format (compressed, 14mb vs 19mb). Hence, running the bedtools intersect command basically outputted the contents of the files (same) in terminal.

(12-20-23)

Then moved on to the bigwig files. First downloaded the bigWigSummary package. Also downloaded bigWigInfo package. Realized why the bigwig files are so big: they store the scores (signal p-values in this case, probably negative log of them) for each genomic position, rather than only some of them, which have a high signal, for the case of the bed files.

Also, the bed files with pseudoreplicated peaks have the following column headers (could be wrong too):

    chr   start   end     peak_id   score   strand   signal_value   p-value   q-value   num_of_reads  
eg. chr1  9949	 10686	Peak_100299	 1000	.	        7.65041	   114.29367  112.13363	  508   


Next, moved on to the RNAseq data for Adrenal Gland. Investigated for some time about what was meant by minus/plus strand signal of all/unique reads. It basically means the RNA expression of the genes coming from the plus/minus strand of the original DNA. So the minus strand signal of all reads would mean that only the reads, whose mRNA had come from the minus strand of the original DNA are included in it (and it included reads which map to many parts of the DNA too, meant by all reads rather than unique).

One important learning: Read coverage means the extent to which a particular genomic region has been covered by reads, both in depth and width. Read depth is just the depth component of it.

Okay, so I have played around with the RNAseq files too and now is the time to finally integrate ATACseq with RNAseq.

So what i plan to do is:

- Aim- find good associations between atacseq peaks and genes (independent of distance from genes, as proximity is 
not a necessity). 

- Method- Basically use the EM algorithm to find out the associations between atacseq peaks and genes for
which the probability of obtaining the transcript abundances is quite high. Logic is that if a gene is being expressed in high amounts, then its regulatory elements will be more open. Can establish a mathematical relation which will be a function to maximize for the EM algo.

- Potential Problems with Method- Since there are so many genes, there will be a set of regulatory elements very faraway from the gene and completely unrelated to it, which will give very high likelihood of getting the gene 

- A prospective solution- Need data from more experiments so that we can better estimate the relation between a faraway regulatory element and a gene. Not necessarily data from the same tissue. Data from different tissues can also work since the genome is still the same.

- New Method- Use a new mathematical function, which would look like: what associations of the regulatory elements (peaks) with the genes would give the particular expression levels in all the samples (tissue-types) in a most likely fashion? Again, the logic being that more open regulatory regions will be associated with more expressed genes.

That looks like a decent plan to me. So, I am gonna first download data for another tissue, other than Adrenal Gland and implement the code first. It will probably give very bad results. Then, I will extend to more tissue types and see what happens.

I am actually comparing using EM vs Reinforcement Learning in this case. EM works in the following fashion: there are some parameters, which determine the data we obtain. So there is a direction of flow of information. I can probably make an analogy in my case where the parameters would be what peak (regulatory element) is associated with which gene, and the data would be the RNA expression levels for the genes. Somehow, I would then need to calculate that given a set of peaks for a gene and their openness, what would be its expression level. This seems tough to calculate.
Another approach could be Reinforcement Learning
In this case, the model parameters are learnt by going through multiple iterations (generations) and each time getting certain rewards/penalties. The model will learn to best maximize its reward and minimize penalty by making a better set of moves as it learns. In my case, I can draw the analogy like: the allocation of peaks to genes is the set of moves taken by the model. The reward could be the mathematical function I was talking about (can simply be the product of peak score and gene expression initially) and the penalty can be the distance between the peak and the gene. Also need to definitely remove the peaks which are already part of gene initially as they should definitely not be included in the analysis.

(12-21-23)

New thoughts on new day.

Reinforcement Learning may not be the best strategy afterall. If I am planning to just give a score according to a metric for the association of peaks and genes, then I dont need to train a whole RL model for that. I can simply calculate the score for the peaks with genes. Granted that there will be a lot of combinations, but we can do some filtering (eg. just focusing on one chromosome at a time, as rarely regulatory elements in other chromosomes). Then choose the best association out of all. 

Finally, what I have decided to do is that I will just do a standard integration of the RNAseq snd ATACseq data (associating peaks with genes by just smallest distance). Later on, I will collect more data on more tissues and using that data, train an ML model which will be able to predict gene expression levels using the chromatin-hidden info.

So I have done the part of integration of the ATACseq and RNAseq of adrenal gland. Precisely, now we have a dataframe, in which there is information of all the peaks (their start, end points and score), the gene that peak was matched to, and its expression level in TPM. I also drew scatter plot to show the same and it looked something like this for just 1000 peaks:

![gene_TPM vs peak_signal_value](image-1.png)

The graph with genes on the x-axis and their peaks' avg score on y-axis looked like:

![avg_peak_signal_value vs gene_TPM](image-2.png)

After some tweaking into the code to make better scatter plots, I will move on to making the ML model. It will need to take in information from many samples. Here, by samples I mean tissue types, cell types etc. This is because for every gene, how its regulatory regions effect it is different. So in our model, in the training sample, one datapoint will be one sample (and relation between its genes' expression and associated atacseq peaks' score). The expression levels will be the output and the peaks score will be 

A prospective problem in the ML thing that came to my mind is that every datapoint will be huge. And ideally, all of our peaks (input of model) would be affecting all the genes (output of model). And there should be possibly hundreds of such datasets (which obviously I wont be able to make happen as even in encode there are not as many tissue samples for which both atacseq and rnaseq data is there). Hence, to fix this, I can just focus on one small part of the genome (eg. only first million bases of chromosome 1). Then, out of all the samples we will be dealing with, we will have to find out all the possible open regions in this limited region of chromosome 1, by merging those present in all the samples. Then, all these merged peaks (all possible peaks in the region of chromosome 1) will be our input. The output will be the expression levels of all the genes in this region of chromosome 1. 