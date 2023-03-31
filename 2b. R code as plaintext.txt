#Provide the students NASA metadata available in the JSON
#format. Note that there are many other large scientific research 
#institutions that also publicly release metadata about all of the
#datasets they have, such as CERN Centre europeean pour la recherche nuclaire, 
#which can be found here, using the data from LHCb in 2011 as an example: 

#a. bibliographic records as HTML: https://opendata.cern.ch/record/24501
#                         as JSON: opendata.cern.ch/record/24501/export/json

#b. JSON documentation schema that specifies the meaning of various fields:
#           github.com/cernopendata/opendata.cern.ch/blob/master/cernopendata/jsonschemes/records/record-v1.0.0.json

#ArXiv data is currently available through Kaggle: https://www.kaggle.com/datasets/Cornell-University/arxiv

#Please note that the CERN metadata is just now beginning to be standardized (see page 4
#of this report from 2022: https://cds.cern.ch/record/2834494/files/CERN%20Summer%
#20Programme%20-%20Work%20Project%20Report%20(7)--1847495423.pdf)

#_______________________________________________________________________
#_______________________________________________________________________

#1. Bringing in the data.
#Install and open the package called 'jsonlite.' This package
#allows us to extract the metadata and explore it neatly. 
# Also read in the JSON files from the website, https://data.nasa.gov/data.json,
#and put it into a variable called 'metadata.' (Assumed knowledge: learners know that 
#any name can be used for a variable.)
# Also take a look at names of all the categories of metadata using the 'names'
#command. 

library(jsonlite)
metadata <- fromJSON("https://data.nasa.gov/data.json")
names(metadata$dataset)
#We can see that the names of the categories cover things like [language],
#[publisher], and [theme].


#_______________________________________________________________________

#2. Exploring the data continued.

#We can take a look at what type of variable these categories are,
#by using the 'class' command. For example, here, we are looking at 
#the type for the 'title,' 'description,' and 'keyword' categories. 
#(Assumed knowledge: learners should already know what 'class' is.)
#(For an explanation of these types, see https://www.programiz.com/r/data-types.)

#Direct learners to look at other categories not shown in this example, such
#as 'rights,' 'license,' or 'spatial.'


class(metadata$dataset$title) #This tells us that all of the values under the 
                              #category 'title' are characters.
class(metadata$dataset$description) #This tells us that all the values under 
                                    #the category 'description' are characters.
class(metadata$dataset$keyword) #This tells us that all the values under the 
                                #category 'keyword' are lists, which makes sense
                                # since there can be more than one keyword
                                #for each dataset stored by NASA. 

#_______________________________________________________________________

# 3. Tidying data.

#We should tidy the data before we continue on to do any other analysis to
#explore the relationships between all of NASA's datasets. Those other
#analyses will include  word co-occurrences, tf-idf, and topic modelling.
#Use the package 'dplyr' for this part. When we tidy, what are are
#actually doing is putting the data into a clear table that will 
#work smoothly with R. We use the command 'tibble' which means 'table.' 

library(dplyr)

#Make a new table called 'nasa_title' and put all of the title metadata from
#the JSON into it.
nasa_title <- tibble(id = metadata$dataset$`_id`$`$oid`, title = metadata$dataset$title)
nasa_title

#Make a new table called 'nasa_desc' and put all of the description metadata 
#from the JSON into it.
nasa_desc <- tibble(id = metadata$dataset$`_id`$`$oid`, desc = metadata$dataset$description)
nasa_desc 

#Make a new table called 'nasa_keyword' and put all of the keyword metadata from
#the JSON into it. Remember from step 2, we found out that this variable
#type is different from the others; it is a list and not a character like the 
#previous two we were working with. So we need to use a command called 
#'unnest' to separate the keywords into a tidy table.

library(tidyr)
nasa_keyword <- tibble(id = metadata$dataset$`_id`$`$oid`, keyword = metadata$dataset$keyword) %>%
  unnest(keyword)

nasa_keyword

#We notice that for both the tables called 'nasa_title' and 'nasa_desc',
#they are still messy tables with more than one word per row.
#Let's separate it so we have only one word per row. Use the command
#called 'unnest_tokens' to separate them; plus use the command 'anti_join(stop_
#words)' to remove stopwords, since they don't have a lot of meaning to us.
#Learners should already know what stopwords are.

library(tidytext)
nasa_title <- nasa_title %>% 
  unnest_tokens(word, title) %>% 
  anti_join(stop_words)

nasa_desc <- nasa_desc %>% 
  unnest_tokens(word, desc) %>% 
  anti_join(stop_words)

nasa_title
nasa_desc

#Now the tables are all tidy with one word per row. 

#_______________________________________________________________________


#4. Looking for the most common words in the 'nasa_title' and 'nasa_desc'
#tables.

nasa_title %>% 
  count(word, sort = TRUE) #This tells us the most common words are 'v1.0,' 
                           #'data,' '2,' 'rosetta,' etc.

nasa_desc %>% 
  count(word, sort = TRUE) #This tells us the most most common words are 'data,'
                           #'set,' 'product,' '2,' etc.

#There are still some words that turn up in these searches which are 
#useless, i.e. we should exclude them (they are stopwords). For example,
#we don't really want to see '1,' '2,' 'v1.0,' etc in our search for the
#most common words.

#Therefore we create a custom stopwords list to remove these. You can define
#whatever stopwords you want to remove; you are not limited to what is shown
#in this example.

my_stopwords <- tibble(word = c(as.character(1:10),
                                    "v1.0", "v3.0", "l2", "l3", "l4", "v5.2.0", 
                                    "v003", "v004", "v005", "v006", "v7"))
#Remove those custom stopwords from both datasets.
nasa_title <- nasa_title %>% 
  anti_join(my_stopwords)
nasa_desc <- nasa_desc %>% 
  anti_join(my_stopwords)

nasa_title
nasa_desc


#Now search these two for most common words again, knowing that we've removed 
#all the useless stopwords.

nasa_title %>% 
  count(word, sort = TRUE)

nasa_desc %>% 
  count(word, sort = TRUE)


#End of lesson
#_______________________________________________________________________
#_______________________________________________________________________
