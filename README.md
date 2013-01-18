# Bioconductor quick reference card

## Install

    source("http://bioconductor.org/biocLite.R")
    biocLite()
    biocLite(c("package1","package2"))

more information at http://www.bioconductor.org/install/

## Annotations

    library(biomaRt)
    ensembl = useMart("ensembl", dataset = "hsapiens_gene_ensembl")
    entrezmap <- getBM(attributes = c("ensembl_gene_id", "entrezgene"), 
    	               filters = "ensembl_gene_id", 
                       values = some.ensembl.genes, 
                       mart = ensembl)

    library(GenomicFeatures)
    library(TxDb.Hsapiens.UCSC.hg19.knownGene)
    txdb <- TxDb.Hsapiens.UCSC.hg19.knownGene
    # or alternatively from biomart
    txdb <- makeTranscriptDbFromBiomart(biomart = "ensembl", dataset = "hsapiens_gene_ensembl")
    GR <- transcripts(txdb)
    EX <- exons(txdb)
    GRList <- transcriptsBy(txdb, by = "gene")

## GenomicRanges

    library(GenomicRanges)
    z <- GRanges("chr1",IRanges(1000001,1001000))
    start(z)
    end(z)
    width(z)
    flank(z, both=TRUE, width=100)

## SummarizedExperiment

    library(GenomicRanges)
    library(Rsamtools)
    fls <- list.files(pattern="*.bam$")
    bamlst <- BamFileList(fls)
    library(rtracklayer)
    gffFile <- "gene_annotation.gtf"
    gff0 <- import(gffFile, asRangedData=FALSE)
    idx <- mcols(gff0)$type == "exon"
    gff <- gff0[idx]
    tx <- split(gff, mcols(gff)$gene_id)
    txhits <- summarizeOverlaps(tx, bamlst)
    assay(txhits)
    colData(txhits)
    rowData(txhits)

## Sequencing/sequence data

    library(Rsamtools)
    which <- GRanges("chr1",IRanges(1000001,1001000))
    what <- c("rname","strand","pos","qwidth","seq")
    param <- ScanBamParam(which=which, what=what)
    reads <- scanBam(bamfile, param=param)
    dnastringset <- scanFa(fastaFile, param=granges)
    # DNAStringSet is defined in the Biostrings package

## RNA-seq analysis

    library(DESeq)
    cds <- newCountDataSet(counts, condition)
    cds <- estimateSizeFactors(cds)
    cds <- estimateDispersions(cds)
    res <- nbinomTest(cds, "A", "B")
    # or with glm
    fit1 <- fitNbinomGLMs(cds, count ~ group + treatment)
    fit0 <- fitNbinomGLMs(cds, count ~ group)
    pvals <- nbinomGLMTest(fit1, fit0)

    library(edgeR)
    y <- DGEList(counts=counts,group=group)
    y <- estimateCommonDisp(y)
    y <- estimateTagwiseDisp(y)
    et <- exactTest(y)
    topTags(et)
    # or with glm
    design <- model.matrix(~group)
    y <- estimateGLMCommonDisp(y,design)
    y <- estimateGLMTrendedDisp(y,design)
    y <- estimateGLMTagwiseDisp(y,design)
    fit <- glmFit(y,design)
    lrt <- glmLRT(fit,coef=2)
    topTags(lrt)

## Expresion set

    library(Biobase)
    data(sample.ExpressionSet)
    e <- sample.ExpressionSet
    exprs(e)
    pData(e)
    fData(e)

## Get GEO dataset

    library(GEOquery)
    e <- getGEO("GSE9514")

## Microarray analysis

    library(affy)
    library(limma)
    phenoData <- read.AnnotatedDataFrame("sample-description.csv")
    eset <- justRMA("/celfile-directory", phenoData=phenoData)
    design <- model.matrix(~ Disease, pData(eset))
    fit <- lmFit(eset, design)
    efit <- eBayes(fit)
    topTable(efit, coef=2)

## Gene ontology set enrichment

    library(org.Hs.eg.db)
    library(GOstats)
    params <- new("GOHyperGParams", geneIds = entrezlist, 
                  universeGeneIds = NULL, annotation ="org.Hs.eg.db", 
                  ontology="BP", pvalueCutoff=.001, conditional=FALSE, 
                  testDirection="over")
    hgOver <- hyperGTest(params)
    hgSummary <- summary(hgOver)

## help within R
   
    ?functionName
    help("functionName")
    help("eSet-class")  # classes need the '-class' on the end
    openVignette("package")
    vignette("topic", "package")
    functionName                # prints source code
    getMethod("method","class") # prints source code for methods
