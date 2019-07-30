
## Introduction

Folks,

Recall the neat dependency parsed trees (or linear diagrams) we'd seen with Spacy in Py. One useful NLP output is a linkage map between each word in a sentence and (at least one) other word(s) the so-called 'syntatical head' in the sentence. 

While UDpipe nicely replicates spacy's Py functionality in R (and then goes on to add more of its own), creating dependency parsing visualization was a functionality that only recently got added in. 

With dependency parsing output, we can answer questions like:

- What is the nominal subject of a text? The 'actor' or 'prime mover' in a typical sentence.

- What is the object of a verb? That which is getting acted upon. Could be any entity - person, thing, abstraction etc.

- Which word modifies a noun? Classic POStags like adjectives etc but more too such as is it the subject or the object that acts on a noun, etc.

- What is the linked to negative words? Classic sentiment-an gets more teeth and bite provided we know what we are looking for.

- Which words are compound statements? Modifiers of various types and patterns thereof can well be identified and extracted.

- What are noun phrases, verb phrases in the text? Phrase detection by now should be old hat.

- etc

Running this UDpipe R code below, provides you the dependency relationships among the words of the sentence in the columns 
token_id, head_token_id and dep_rel all nicely in dataframe form. 

The possible values in the field dep_rel are defined [here](https://universaldependencies.org/u/dep/index.html).

Let's head down now to see a small demo in action, then we will functionize it and then test-drive the function on scaled up data. Behold.

### Demonstrating Depcy Viz

Demo borrowed from the [UDpipe develope pages here](http://www.bnosac.be/index.php/blog/93-dependency-parsing-with-udpipe).

I'm further reading in a trained english model from my dropbox location on my local machine. Others may first need to download the same based on instructions from [here](https://github.com/bnosac/udpipe.models.ud).


```R
## setup chunk
if (!require(udpipe)){install.package("udpipe")}; library(udpipe)
if (!require(igraph)){install.package("igraph")}; library(igraph)
if (!require(ggraph)){install.package("ggraph")}; library(ggraph)
if (!require(ggplot2)){install.package("ggplot2")}; library(ggplot2)
if (!require(magrittr)){install.package("magrittr")}; library(magrittr)
```

    Loading required package: udpipe
    Warning message:
    "package 'udpipe' was built under R version 3.5.3"Loading required package: igraph
    Warning message:
    "package 'igraph' was built under R version 3.5.3"
    Attaching package: 'igraph'
    
    The following objects are masked from 'package:stats':
    
        decompose, spectrum
    
    The following object is masked from 'package:base':
    
        union
    
    Loading required package: ggraph
    Loading required package: ggplot2
    Warning message:
    "package 'ggplot2' was built under R version 3.5.3"Loading required package: magrittr
    


```R
# prep demo sample
sent1 <- "The economy is weak but the outlook is bright."  # test sentence
lang_model_path <- "C:\\Users\\20052\\Dropbox\\sw explorations\\R miscell\\udpipe depcyparsing\\"
eng_model_path <- paste0(lang_model_path, "english-ewt-ud-2.4-190531.udpipe")
eng_model <- udpipe_load_model(eng_model_path)   

# apply eng model on demo sample
x <- udpipe(sent1, eng_model)     # x is annotated udpipe object.
head(x)
```


<table>
<caption>A data.frame: 6 × 17</caption>
<thead>
	<tr><th scope=col>doc_id</th><th scope=col>paragraph_id</th><th scope=col>sentence_id</th><th scope=col>sentence</th><th scope=col>start</th><th scope=col>end</th><th scope=col>term_id</th><th scope=col>token_id</th><th scope=col>token</th><th scope=col>lemma</th><th scope=col>upos</th><th scope=col>xpos</th><th scope=col>feats</th><th scope=col>head_token_id</th><th scope=col>dep_rel</th><th scope=col>deps</th><th scope=col>misc</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>doc1</td><td>1</td><td>1</td><td>The economy is weak but the outlook is bright.</td><td> 1</td><td> 3</td><td>1</td><td>1</td><td>The    </td><td>the    </td><td>DET  </td><td>DT </td><td>Definite=Def|PronType=Art                            </td><td>2</td><td>det  </td><td>NA</td><td>NA</td></tr>
	<tr><td>doc1</td><td>1</td><td>1</td><td>The economy is weak but the outlook is bright.</td><td> 5</td><td>11</td><td>2</td><td>2</td><td>economy</td><td>economy</td><td>NOUN </td><td>NN </td><td>Number=Sing                                          </td><td>4</td><td>nsubj</td><td>NA</td><td>NA</td></tr>
	<tr><td>doc1</td><td>1</td><td>1</td><td>The economy is weak but the outlook is bright.</td><td>13</td><td>14</td><td>3</td><td>3</td><td>is     </td><td>be     </td><td>AUX  </td><td>VBZ</td><td>Mood=Ind|Number=Sing|Person=3|Tense=Pres|VerbForm=Fin</td><td>4</td><td>cop  </td><td>NA</td><td>NA</td></tr>
	<tr><td>doc1</td><td>1</td><td>1</td><td>The economy is weak but the outlook is bright.</td><td>16</td><td>19</td><td>4</td><td>4</td><td>weak   </td><td>weak   </td><td>ADJ  </td><td>JJ </td><td>Degree=Pos                                           </td><td>0</td><td>root </td><td>NA</td><td>NA</td></tr>
	<tr><td>doc1</td><td>1</td><td>1</td><td>The economy is weak but the outlook is bright.</td><td>21</td><td>23</td><td>5</td><td>5</td><td>but    </td><td>but    </td><td>CCONJ</td><td>CC </td><td>NA                                                   </td><td>9</td><td>cc   </td><td>NA</td><td>NA</td></tr>
	<tr><td>doc1</td><td>1</td><td>1</td><td>The economy is weak but the outlook is bright.</td><td>25</td><td>27</td><td>6</td><td>6</td><td>the    </td><td>the    </td><td>DET  </td><td>DT </td><td>Definite=Def|PronType=Art                            </td><td>7</td><td>det  </td><td>NA</td><td>NA</td></tr>
</tbody>
</table>



Note the sequntial numbering given to word tokens in the sentence in colm 'token_id'. Note also how the dependency relations between tokens are defined in the 'head_token_id' column.

Below we create a basic function which selects the right columns from the annotation and puts it into a graph.

### Functionizing the Visualization


```R
library(igraph)
library(ggraph)
library(ggplot2)

plot_annotation <- function(x,   # annotated udpipe DF from sentence input
                            sent_id1 = 1,   # sent_id of sent to be evaluated
                            size = 3){   # node label fontsize
  
  # basic input checks
  stopifnot(is.data.frame(x) & all(c("sentence_id", "token_id", "head_token_id", "dep_rel",
                                     "token_id", "token", "lemma", "upos", "xpos", "feats") %in% colnames(x)))
  
  # basic manipulations for getting data prepped
  x <- x[!is.na(x$head_token_id), ]     # drop all NAs
    
  # x <- x[x$sentence_id %in% min(x$sentence_id), ]   # consider only 1 sentence at a time
  x <- x[x$sentence_id == sent_id1, ]   # consider only 1 sentence at a time
  
  # define ggraph edges
  edges <- x[x$head_token_id != 0, c("token_id", "head_token_id", "dep_rel")]   # df subset
  edges$label <- edges$dep_rel
  
  # define graph itself
  g <- graph_from_data_frame(edges,
                             vertices = x[, c("token_id", "token", "lemma", "upos", "xpos", "feats")],
                             directed = TRUE)
  
  # windows()   # open separate display page
  ggraph(g, layout = "linear") +
    
    
    geom_edge_arc(ggplot2::aes(label = dep_rel, vjust = -0.20),
                  arrow = grid::arrow(length = unit(4, 'mm'), ends = "last", type = "closed"),
                  end_cap = ggraph::label_rect("wordswordswords"),
                  label_colour = "red", check_overlap = TRUE, label_size = size) +
    
    geom_node_label(ggplot2::aes(label = token), col = "darkgreen", size = size, fontface = "bold") +
    
    geom_node_text(ggplot2::aes(label = upos), nudge_y = -0.35, size = size) +
    
    theme_graph(base_family = "Arial") +
    labs(title = "udpipe output", subtitle = "tokenisation, parts of speech tagging & dependency relations")
  
}  # func ends with 'g' as outp returned
```


```R
## test-drive the func
suppressWarnings({ plot_annotation(x, sent_id = 1, size = 4) })
```

    Warning message in grid.Call(C_stringMetric, as.graphicsAnnot(x$label)):
    "font family not found in Windows font database"Warning message in grid.Call(C_stringMetric, as.graphicsAnnot(x$label)):
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"


![png](output_5_1.png)


Not bad, eh? 

Time now to encode the above into a single chained workflow and invoke. Behold.


```R
# longer workflow lagao & check
require(magrittr)
sent1 <- "Sudhir Voleti is an Associate Professor of Marketing at the ISB."
suppressWarnings(sent1 %>% udpipe(., eng_model) %>% plot_annotation(., size = 4))
```

    Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"


![png](output_7_1.png)


Last part remaining is scaling this up and trying on a multi-sentence paragraph wherein the output graphs are saved into a list. 

Would help if we're doing this in a shiny app too, for instance.

P.S. Doesn't make sense to do this at more than a sentence level, so scaling up depcy visualizn to large corpora is anyway pointless.

### Saving Graphs into a List


```R
## test-driving an annotated paragraph.

para1 <- "Dependency parsing is an NLP technique which links each word in a sentence to another word in the sentence. 
The latter is called it's syntactical head. These inter-word links can be useful to analyze."
para1_ann <- para1 %>% udpipe(., eng_model) # 0.08 secs to annotate dmall para

graph_list <- vector(mode="list", length = max(para1_ann$sentence_id))
for (i0 in 1:max(para1_ann$sentence_id)){ 
  graph_list[[i0]] <- para1_ann %>% plot_annotation(., sent_id1 = i0, size = 3) }

plot(graph_list[[1]])
```

    Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"


![png](output_9_1.png)



```R
# here's graph for sentence 2
plot(graph_list[[2]])
```

    Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"Warning message in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, :
    "font family not found in Windows font database"


![png](output_10_1.png)


Chalo, signing off here.

Ciao.

Sudhir Voleti.


```R

```
