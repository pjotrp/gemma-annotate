# Annotation tools for GEMMA

gemma-annotate takes the output of GEMMA and a BED file and annotates
the SNPs that exceed a value threshold. Value can be one of the values
output by GEMMA (column name).

Output is as records listing SNP snpname, chr, position, ann-name,
ann-class and and value.

# Installation

Currently the script depends on Ruby and wget.

# Usage

Here we describe a workflow to get GEMMA output annotated. In the
first step create the GEMMA output file. Next create a BED file
followed by running gemma-annotate.

For more help also try

    gemma-annotate --help

## Running GEMMA

To install and run gemma see the [README](https://github.com/genetics-statistics/GEMMA).
The output should be an assoc.txt file which looks like

    chr rs  ps  n_miss  allele1 allele0 af  logl_H1 l_mle p_lrt
    1 rs3683945 3197400 0 A G 0.443 -1.583931e+03 4.312281e+00  2.085491e-01
    1 rs3707673 3407393 0 G A 0.443 -1.584148e+03 4.310280e+00  2.840289e-01
    1 rs6269442 3492195 0 A G 0.365 -1.584228e+03 4.323805e+00  3.202466e-01
    1 rs6336442 3580634 0 A G 0.443 -1.584127e+03 4.309893e+00  2.756573e-01
    (etc)

Here we are interested in the first 3 columns: chromosome number, SNP
name, SNP position, and the last column containing the LRT. We'll
annotate all SNPs that have a (somewhat arbitrary) p_lrt < 1e-04.

## Creating a BED file for mm10

BED is a [browser extensible data](https://genome.ucsc.edu/FAQ/FAQformat.html) format.

A quick resource for many species is the
[UCSC table browser](https://genome.ucsc.edu/cgi-bin/hgTables) which
contains the same data that the UCSC genome browser uses. For mm10
(GRCm38), select, for example:

    Genome: Mouse
    Assembly: mm10
    Group: gene and gene predictions
    Track: UniProt
    Table: SwissProt
    Output format: BED

and download that file as mm10\_ucsc\_uniprot.bed. You can do it in other
ways (there are many options, e.g. for annotating signal peptides or
predicted genes), but here we use UniProt to give curated annotation
information.

The BED file can be reduced by only including chrom, chromStart,
chromEnd and name. So, for our purposes you only need the first 4 columns which can
be filtered with the Unix 'cut' command

    cut -f 1,2,3,4 ucsc_mm10_uniprot.bed > ucsc_mm10_uniprot_small.bed

An example lives in [./test/data/input](./test/data/input) and the header looks like

    chr1  3216024 3671348 Q5GH67
    chr1  4344602 4352825 P56716
    chr1  4491718 4493406 Q61473-1
    chr1  4776466 4785677 Q9CPR5-1
    chr1  4807913 4845013 P97823-1

Note: gemma-annotate assumes the BED file is *sorted* by CHR+POS.

## Creating an annotation file

Run

    gemma-annotate --eval "p_lrt < 1e-04" \
        --bed test/data/input/ucsc_mm10_uniprot.bed \
        test/data/input/mouse_hs1940_CD8MCH_lmm.assoc.txt

returns

    [{"chr":"1","pos":173365379,"name":"rs13476242","anno":"Q99N28","url":"http://www.uniprot.org/uniprot/Q99N28","gene":"Cadm3","synonyms":"Igsf4b, Necl1, Syncam3, Tsll1"},...

In fact all table columns can be evaluated. This will annotate all SNPs on chr1 that have
an allele 'A'

    ./bin/gemma-annotate --eval "chr == '1' and allele1 == 'A'" \
        --bed test/data/input/ucsc_mm10_uniprot.bed \
        test/data/input/mouse_hs1940_CD8MCH_lmm.assoc.txt

another would be

    ./bin/gemma-annotate --eval "chr == '1' and ps > 182639404 and logl_H1 < 1e-04" \
        --bed test/data/input/ucsc_mm10_uniprot.bed
        test/data/input/mouse_hs1940_CD8MCH_lmm.assoc.txt

the eval is simply a Ruby expression. Go wild!

In each case the output is a JSON record.

If you want to include the value of an evaluated column you can pass
in the --value switch, e.g.

    gemma-annotate --eval "p_lrt < 1e-04" --value p_lrt \
        --bed test/data/input/ucsc_mm10_uniprot.bed \
        test/data/input/mouse_hs1940_CD8MCH_lmm.assoc.txt

returns

    [{"chr":"1","pos":173365379,"name":"rs13476242","p_lrt":2.070492e-05 ... ]

some Ruby magic there!
