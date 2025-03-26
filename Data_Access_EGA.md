# Instructions on accessing sequencing data and phenotypes from EGA

To access sample information, basic phenotypes and sequencing reads from EGA, you need to follow the instructions provided below:

## Prerequisites
**1**. **To request access to the data**, you will need [to register and validate an account at EGA](https://ega-archive.org/register/). The validation time for new accounts by EGA varies, but it usually takes around 18-24 hours.

 ### :bangbang: IN THE MEANTIME, IF YOU HAVE NOT DONE IT YET:

**2**. Read the [CeDNN data access agreement]([insertlink]). Access to the CeDNN Cohort data will be granted 
to all qualified researchers and will be governed by the provisions laid out in this data access agreement.

**3**. Fill out the [Application Form for Access to CeDNN Data]([https://forms.office.com/e/SRBFqZi0nq]) to indicate that you agree with the CeDNN data access agreement.



## Requesting data via EGA
1. Login to EGA
   
2. Request access to datasets from the [CeDNN cohort]([insertlink]). To do so, press the 'Request Access' button on the page of the dataset of interest ([CeDNN metagenomic sequencing]([insertlink])).

3. After receiving your **EGA Access Request** and filled **Application Form for Access to CeDNN Data**, the Data Access Committee will evaluate your request and grant access to the data. We try to do so within 2 weeks.

If there is no approval and no reply within 3 weeks from the date of your **EGA Access Request**, please email us: i.h.jonkers@umcg.nl.


## Processing data from EGA
:exclamation: Upon receiving access to the data, the following instructions may be helpful, as EGA data formatting can sometimes be complex:

### Metadata (phenotypes):
Upon receiving access to a dataset, you will be able to download the metadata containing phenotypes. Below is an example of parsing these files using R.

```
# necessary libraries:
library(jsonlite)
library(data.table)

# importing the metadata:
DF <- read.table("samples.tsv", header=T, sep='\t', quote = "")

# Extracting the JSON-like strings from the DataFrame
json_strings <- as.character(DF[, "extra_attributes"])

# Parsing each JSON-like string
parsed_json <- lapply(json_strings, function(x) jsonlite::fromJSON(x))

# Reformatting each data frame
result_list <- lapply(parsed_json, function(json_obj) {
  buffer <- as.data.frame(t(as.data.table(json_obj)))
  colnames(buffer) <- buffer[1, ]
  buffer <- buffer[3, , drop = FALSE]  # Keeping it as a data frame
  return(buffer)
})

# Extracted values and columns
combined_df <- do.call(rbind, result_list)

# Resulting data frame
result_df <- cbind(DF, combined_df)

# Remove row names
row.names(result_df) <- NULL

#### To connect the metadata and data, you will need to get the sequencing IDs:
# sequencing IDs:
seq_names <- read.table('sample_file.tsv', sep='\t', header=T)
seq_names$Sequencing_ID <- gsub('_kneaddata_cleaned_pair_..fastq.gz', '', gsub('Pilot_VLP_', '',seq_names$file_name))
seq_names <- seq_names[!duplicated(seq_names$sample_alias),c("sample_alias","Sequencing_ID")]
colnames(seq_names)[1] <- "alias"

# resulting metadata with phenotypes:
metadata <- merge(seq_names, result_df, by='alias')

# change NAs from text to NA
metadata[metadata=="NA"] <- NA

```

### Sequencing data:

To download the sequencing data, you need to use [EGA download client: pyEGA3](https://github.com/EGA-archive/ega-download-client). The pyEGA3 has an extensive README.md on installation and usage.
