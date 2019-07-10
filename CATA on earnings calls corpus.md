
### Introduction & Setup

Idea here is to take a general keyword list (or two) and extract all sentences containing 'em keywords (and key phrases!) from the target corpus of earnings call transcripts, using spacy. 

Next, following what was done previously, we also extract factual vs hypothetical sentences from the corpus. 

Plan is to ultimately build a 2x2 and see if these 4 variables as covariates have any impact on (daily/weekly) stock returns after controlling for other standard covariates. 


```python
## setup chunk
import spacy
nlp = spacy.load('en_core_web_sm')
import time
import re
import pandas as pd
import os 
import feather
```

    C:\Users\20052\AppData\Local\Continuum\anaconda3\lib\importlib\_bootstrap.py:219: RuntimeWarning: cymem.cymem.Pool size changed, may indicate binary incompatibility. Expected 48 from C header, got 64 from PyObject
      return f(*args, **kwds)
    C:\Users\20052\AppData\Local\Continuum\anaconda3\lib\importlib\_bootstrap.py:219: RuntimeWarning: cymem.cymem.Address size changed, may indicate binary incompatibility. Expected 24 from C header, got 40 from PyObject
      return f(*args, **kwds)
    C:\Users\20052\AppData\Local\Continuum\anaconda3\lib\importlib\_bootstrap.py:219: RuntimeWarning: cymem.cymem.Pool size changed, may indicate binary incompatibility. Expected 48 from C header, got 64 from PyObject
      return f(*args, **kwds)
    C:\Users\20052\AppData\Local\Continuum\anaconda3\lib\importlib\_bootstrap.py:219: RuntimeWarning: cymem.cymem.Address size changed, may indicate binary incompatibility. Expected 24 from C header, got 40 from PyObject
      return f(*args, **kwds)
    


```python
## obtain text files list
path0 = 'C:/Users/20052/Dropbox/tech sector data/earnings calls/cleaned_text_transcripts_5/'
files_list = os.listdir(path0)
len(files_list)  # 8313 but not all are earnings calls
files_list1 = [x for x in files_list if bool(re.search("Earnings Call", x))==True ] # 7923 files

# create empty DFs to populate outp with
outp_hypoth_sent_df = pd.DataFrame(columns = ['docName','index', 'modifier', 'sentence', 'tot_sents'])
outp_explor_sent_df = pd.DataFrame(columns = ['docName','sentIndex', 'keywords', 'sentence'])
outp_exploit_sent_df = pd.DataFrame(columns = ['docName','sentIndex', 'keywords', 'sentence'])
```

### Pre-proc of Keyword lists

2 Keyword lists - for exploration and epxloitation firm orientation - are there in my local machine. Both have many words in common, and have bigram phrases as well.

So I use a non-space delimited ";" to tokenize the keyword list and regex to pattern-match each sentence in each document. 

After testing the prototype code, I build a func which I will invoke down the line.

Behold.


```python
# read-in & pre-proc the keyword list
path2 = 'H:\\'
file2 = 'exploration_wordlist.txt'
keywords = open(path2 + file2).read().replace('\n',';')  # note use of delimiter';'
keyword_tokens = re.split(";", keywords)
len(keyword_tokens)  #227
```




    227




```python
# func to split list into single words and key-phrases
def split_keywrds(path3):
  keywords = open(path3).read().replace('\n',';')  # note delimiter';'
  keyword_tokens = re.split(";", keywords)
  key_tokens = []
  key_phrases = []
  for i2 in range(len(keyword_tokens)):
    focal_word = keyword_tokens[i2]
    if (re.search("\s", focal_word)):
      key_phrases.append(focal_word)
    else:
      key_tokens.append(focal_word)
  return key_tokens, key_phrases;

# apply func on both explorn and exploitn lists
path2 = 'H:\\'
explor_path = path2 + "exploration_wordlist.txt"
explor_tokens, explor_phrases = split_keywrds(explor_path) # 0.04 secs

exploit_path = path2 + "exploitation_wordlist.txt"
exploit_tokens, exploit_phrases = split_keywrds(exploit_path)
```

### Func to Exract Qtys of interest

Below I'm directly writing into one big func all the proc that spacy will be needed for, viz. extracting hypotheticals and keyword spfc sentences.

Thereafter we will bundle these into neat panda DF outputs that I can row-bind over docs.

See below.


```python
## func to process one input doc 'doc0', firm name under 'file1' and outp 3 DFs
def docProc(path0, file1, explor_tokens, explor_phrases, exploit_tokens, exploit_phrases):

  doc0 = open(path0 + file1).read().replace('\n', ' ')

  # clean up doc0
  doc1 = re.sub("\.{3,}",' ',doc0) # drop '...' type formatting seen
  doc1 = re.sub(r'\s{2,}', '', doc1)  # get rid of multiple spaces
  doc1 = re.sub(r'\\x0c', '', doc1)  # get rid of non utf-8 chars

  # testing a keyword detector on sents
  doc = nlp(doc1) # annotate the doc
  sents_list = [sent.orth_ for sent in doc.sents]  # sentence-tokenize into raw text 
  tot_sents = len(sents_list)
  
  # build empty list to populate as DF colms
  explor_sents = []
  explor_sents_index = []
  explor_detected = []

  exploit_sents = []
  exploit_sents_index = []
  exploit_detected = []

  hypoth_sents = []
  hypoth_sents_index = []
  hypoth_mods = []

  # t1 = time.time()
  for i1 in range(len(sents_list)):
    sent0 = sents_list[i1]  
    sent0_ann = nlp(sent0)
    sent0_tokenized = [token.text for token in sent0_ann] 

    # hypoth detection as previously
    morph0 = str([nlp.vocab.morphology.tag_map[token.tag_] for token in sent0_ann])
    morph1 = re.split("},", morph0)
    for i3 in range(len(morph1)):
      if bool(re.search(r'mod', morph1[i3])) == True:
        hypoth_mods.append(sent0_tokenized[i3])
        hypoth_sents_index.append(i1) 
        hypoth_sents.append(sent0)     

    # using set properties to detect keyword intersections
    explor_intersec = set(explor_tokens) & set(sent0_tokenized)
    explor_intersec1 = ','.join(str(s) for s in explor_intersec)
    if (len(explor_intersec) > 0):
      explor_sents.append(sent0)
      explor_sents_index.append(i1)
      explor_detected.append(explor_intersec1)

    # above for exploitn keywrds
    exploit_intersec = set(exploit_tokens) & set(sent0_tokenized)
    exploit_intersec1 = ','.join(str(s) for s in explor_intersec)
    if (len(exploit_intersec) > 0):
      exploit_sents.append(sent0)
      exploit_sents_index.append(i1)
      exploit_detected.append(exploit_intersec1)

    # using regex to pattern match explorn phrases
    for i2 in range(len(explor_phrases)):
      if (re.search(explor_phrases[i2], sent0)):
        explor_sents.append(sent0)
        explor_sents_index.append(i1)
        explor_detected.append(explor_phrases[i2])

    # using regex to pattern match exploitn phrases
    for i3 in range(len(exploit_phrases)):
      if (re.search(exploit_phrases[i3], sent0)):
        exploit_sents.append(sent0)
        exploit_sents_index.append(i1)
        exploit_detected.append(exploit_phrases[i3])
        
  # build DFs now
  doc_name_hypoth = [file1]*len(hypoth_sents)
  tot_sents2 = [tot_sents]*len(hypoth_sents)  
  hypoth_sent_df1 = pd.DataFrame({'docName': doc_name_hypoth, 
                                  'index': hypoth_sents_index,
                                  'modifier':hypoth_mods,
                                  'sentence': hypoth_sents, 
                                  'tot_sents':tot_sents2 })
    
  # for explorn keywrds
  doc_name_explor = [file1]*len(explor_sents_index)  # to include in the DF 
  explor_sent_df1 = pd.DataFrame({'docName':doc_name_explor, 
                                 'sentIndex':explor_sents_index, 
                                 'keywords':explor_detected, 
                                 'sentence':explor_sents })  
    
  # for exploitn keywords
  doc_name_exploit = [file1]*len(exploit_sents_index)  # to include in the DF 
  exploit_sent_df1 = pd.DataFrame({'docName': doc_name_exploit,
                                  'sentIndex': exploit_sents_index, 
                                  'keywords': exploit_detected, 
                                  'sentence': exploit_sents })       
        
  return hypoth_sent_df1, explor_sent_df1, exploit_sent_df1 
```


```python
## Test-drive the func on a sample-file
file1 = files_list1[101]  # say, pick a random file
t1 = time.time()
hypoth_sent_df1, explor_sent_df1, exploit_sent_df1 = docProc(path0, file1, explor_tokens, explor_phrases, exploit_tokens, exploit_phrases)
t2 = time.time()
print(t2-t1, " secs.")
```

    5.766014814376831  secs.
    


```python
# view some outputs
print(hypoth_sent_df1.shape)
exploit_sent_df1
```

    (97, 5)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>docName</th>
      <th>sentIndex</th>
      <th>keywords</th>
      <th>sentence</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Activision Blizzard, Inc, Q2 2014 Earnings Cal...</td>
      <td>83</td>
      <td>launch,marketing</td>
      <td>Note this includes heavy launch marketing cost...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Activision Blizzard, Inc, Q2 2014 Earnings Cal...</td>
      <td>118</td>
      <td>marketing</td>
      <td>and we expect effective marketing programs ever.</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Activision Blizzard, Inc, Q2 2014 Earnings Cal...</td>
      <td>146</td>
      <td>version,launching</td>
      <td>Q2 was also a big quarter for Hearthstone with...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Activision Blizzard, Inc, Q2 2014 Earnings Cal...</td>
      <td>147</td>
      <td>launch,version</td>
      <td>That version has been popular, bringing millio...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Activision Blizzard, Inc, Q2 2014 Earnings Cal...</td>
      <td>220</td>
      <td>marketing,development</td>
      <td>Obviously, a lot of development cost borne thi...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Activision Blizzard, Inc, Q2 2014 Earnings Cal...</td>
      <td>238</td>
      <td>launch,marketing,development</td>
      <td>Relating to our guidance though, our guidance ...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Activision Blizzard, Inc, Q2 2014 Earnings Cal...</td>
      <td>289</td>
      <td>optimize</td>
      <td>That said, as you might imagine, we learned a ...</td>
    </tr>
  </tbody>
</table>
</div>



### Func Scaling by looping over docs

Now rubber meets road. Time to try looping above func on a sample corpus.

So thus far we have 3 funcs defined viz. splitKeywrds(), docProc() and buildDF() - the outp of one becoming the input to another. 


```python
# create empty DFs to populate
outp_hypoth_sent_df = pd.DataFrame(columns = ['docName','index', 'modifier', 'sentence'])
outp_explor_sent_df = pd.DataFrame(columns = ['docName','sentIndex', 'keywords', 'sentence'])
outp_exploit_sent_df = pd.DataFrame(columns = ['docName','sentIndex', 'keywords', 'sentence'])

## loop over 20 files 0-19 and time the proc
t0 = time.time()
for ind1 in range(0,9):
    t1 = time.time()  
    file1 = files_list1[ind1]
  
    # invoke docProc on file1 and obtain 3 DFs
    hypoth_sent_df1, explor_sent_df1, exploit_sent_df1 = docProc(path0, file1, explor_tokens, explor_phrases, exploit_tokens, exploit_phrases)
      
    outp_hypoth_sent_df = outp_hypoth_sent_df.append(hypoth_sent_df1)
    outp_explor_sent_df = outp_explor_sent_df.append(explor_sent_df1)
    outp_exploit_sent_df = outp_exploit_sent_df.append(exploit_sent_df1)
    t2 = time.time()
    print(t2 - t1, " secs for file#", ind1)

t3 = time.time()
print(t3-t0, " secs for full proc")

```

    C:\Users\20052\AppData\Local\Continuum\anaconda3\lib\site-packages\pandas\core\frame.py:6201: FutureWarning: Sorting because non-concatenation axis is not aligned. A future version
    of pandas will change to not sort by default.
    
    To accept the future behavior, pass 'sort=True'.
    
    To retain the current behavior and silence the warning, pass sort=False
    
      sort=sort)
    

    9.194547414779663  secs for file# 0
    8.006002187728882  secs for file# 1
    8.896836757659912  secs for file# 2
    7.390631437301636  secs for file# 3
    8.607985734939575  secs for file# 4
    9.603638172149658  secs for file# 5
    8.902062177658081  secs for file# 6
    8.622740745544434  secs for file# 7
    7.812696218490601  secs for file# 8
    77.0723283290863  secs for full proc
    


```python
print(outp_hypoth_sent_df.shape)
outp_explor_sent_df
```

    (1059, 5)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>docName</th>
      <th>sentIndex</th>
      <th>keywords</th>
      <th>sentence</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>30</td>
      <td>commercialization,R&amp;D</td>
      <td>We posted record sales and returned record cas...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>58</td>
      <td>innovation</td>
      <td>We continue to experience positive selling pri...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>58</td>
      <td>new product</td>
      <td>We continue to experience positive selling pri...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>59</td>
      <td>developing</td>
      <td>In addition, we've been raising prices in cert...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>83</td>
      <td>developing,developed</td>
      <td>Organic local currency growth was nearly 5% ac...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>89</td>
      <td>R&amp;D</td>
      <td>And R&amp;D, as a percent of sales, rose by 20 bas...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>93</td>
      <td>productivity</td>
      <td>An inherent part of 3M's business model is to ...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>95</td>
      <td>R&amp;D</td>
      <td>We invested the equivalent of 90 basis points ...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>104</td>
      <td>optimized</td>
      <td>As a reminder for those who are new to 3M, we ...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>106</td>
      <td>commercialization,R&amp;D</td>
      <td>Organic growth remains our first priority, so ...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>109</td>
      <td>implement</td>
      <td>We communicated our intent to implement these ...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>127</td>
      <td>production</td>
      <td>We also generated double-digit organic growth ...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>133</td>
      <td>productivity</td>
      <td>Margins were boosted by volume leverage, posit...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>149</td>
      <td>developing</td>
      <td>In developing markets, Health Care grew 10% or...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>159</td>
      <td>productivity</td>
      <td>Volume leverage and improving productivity wer...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>173</td>
      <td>efficiency</td>
      <td>In the first quarter, we realigned and combine...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>175</td>
      <td>maintenance</td>
      <td>This brings a comprehensive array of branding,...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>181</td>
      <td>innovation</td>
      <td>Let's talk about the second lever, investing i...</td>
    </tr>
    <tr>
      <th>18</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>186</td>
      <td>innovation</td>
      <td>And we are expanding our innovation capabiliti...</td>
    </tr>
    <tr>
      <th>19</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>187</td>
      <td>innovation</td>
      <td>We have now built 45 innovation centers around...</td>
    </tr>
    <tr>
      <th>20</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>188</td>
      <td>labs</td>
      <td>Our international labs are also typically staf...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>189</td>
      <td>innovation</td>
      <td>And our commitment to innovation helps 3M recr...</td>
    </tr>
    <tr>
      <th>22</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>190</td>
      <td>launched,labs</td>
      <td>It's notable that 47% of our new products laun...</td>
    </tr>
    <tr>
      <th>23</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>190</td>
      <td>new product</td>
      <td>It's notable that 47% of our new products laun...</td>
    </tr>
    <tr>
      <th>24</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>190</td>
      <td>new products</td>
      <td>It's notable that 47% of our new products laun...</td>
    </tr>
    <tr>
      <th>25</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>193</td>
      <td>implementing,efficient</td>
      <td>We continue to make progress on implementing o...</td>
    </tr>
    <tr>
      <th>26</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>194</td>
      <td>launched</td>
      <td>Most recently, we launched it in one of our Eu...</td>
    </tr>
    <tr>
      <th>27</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>195</td>
      <td>implementation,refine</td>
      <td>We are learning more with each implementation ...</td>
    </tr>
    <tr>
      <th>28</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>197</td>
      <td>developing</td>
      <td>When we last met in January, I talked a bit ab...</td>
    </tr>
    <tr>
      <th>29</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>199</td>
      <td>developing</td>
      <td>We have a deep history in developing nations.</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>195</td>
      <td>commercialization</td>
      <td>I'm very encouraged that we are beginning to s...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>203</td>
      <td>refine</td>
      <td>With each deployment, we are learning more and...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>205</td>
      <td>standardized</td>
      <td>To support our ERP system, we are also in the ...</td>
    </tr>
    <tr>
      <th>18</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>206</td>
      <td>productivity</td>
      <td>Ultimately, this will leverage 3M's size and s...</td>
    </tr>
    <tr>
      <th>19</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>230</td>
      <td>manufacturing</td>
      <td>And when we look upon data, when we look upon ...</td>
    </tr>
    <tr>
      <th>20</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>285</td>
      <td>productivity</td>
      <td>If we net out productivity and investment, it ...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>287</td>
      <td>productivity</td>
      <td>Andrew Burris Obin BofA Merrill Lynch, Researc...</td>
    </tr>
    <tr>
      <th>22</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>291</td>
      <td>productivity</td>
      <td>, there are several things driving that produc...</td>
    </tr>
    <tr>
      <th>23</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>292</td>
      <td>productivity</td>
      <td>With the growth we're seeing, we're seeing imp...</td>
    </tr>
    <tr>
      <th>24</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>293</td>
      <td>productivity</td>
      <td>Our business model is to drive those productiv...</td>
    </tr>
    <tr>
      <th>25</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>299</td>
      <td>productivity</td>
      <td>, I don't see us going back into negative prod...</td>
    </tr>
    <tr>
      <th>26</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>323</td>
      <td>manufacturing</td>
      <td>We have a strong manufacturing footprint in or...</td>
    </tr>
    <tr>
      <th>27</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>324</td>
      <td>research,development</td>
      <td>We have a very capable research and developmen...</td>
    </tr>
    <tr>
      <th>28</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>325</td>
      <td>commercialization</td>
      <td>And we have a commercialization on that we, as...</td>
    </tr>
    <tr>
      <th>29</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>328</td>
      <td>produce,develop</td>
      <td>, know the culture, we're able to develop solu...</td>
    </tr>
    <tr>
      <th>30</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>343</td>
      <td>productivity</td>
      <td>And productivity probably still continuing as ...</td>
    </tr>
    <tr>
      <th>31</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>350</td>
      <td>R&amp;D</td>
      <td>I mean, should we really be thinking at this p...</td>
    </tr>
    <tr>
      <th>32</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>351</td>
      <td>developed</td>
      <td>Or is there pricing pressure in some of the va...</td>
    </tr>
    <tr>
      <th>33</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>355</td>
      <td>developing,developed,manufacturing</td>
      <td>and we are very confident that the solutions t...</td>
    </tr>
    <tr>
      <th>34</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>362</td>
      <td>developing,developed</td>
      <td>And when you look upon our mix, 75% of our bus...</td>
    </tr>
    <tr>
      <th>35</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>363</td>
      <td>new product</td>
      <td>And the growth will come there over time and w...</td>
    </tr>
    <tr>
      <th>36</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>401</td>
      <td>manufacturing</td>
      <td>I think the important thing for us when we loo...</td>
    </tr>
    <tr>
      <th>37</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>404</td>
      <td>research,development</td>
      <td>And as you know, the primary strategy for us i...</td>
    </tr>
    <tr>
      <th>38</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>418</td>
      <td>productivity</td>
      <td>And as you know, that will also then drive pro...</td>
    </tr>
    <tr>
      <th>39</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>456</td>
      <td>implement</td>
      <td>But how should we think about the trajectory a...</td>
    </tr>
    <tr>
      <th>40</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>495</td>
      <td>productivity</td>
      <td>In the context of what I see here as kind of a...</td>
    </tr>
    <tr>
      <th>41</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>499</td>
      <td>efficiency</td>
      <td>Nicholas C. Gangestad Chief Financial Officer ...</td>
    </tr>
    <tr>
      <th>42</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>524</td>
      <td>R&amp;D</td>
      <td>If I just dig into the next layer, looking at ...</td>
    </tr>
    <tr>
      <th>43</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>526</td>
      <td>R&amp;D</td>
      <td>just given the movements we're seeing in SG&amp;A ...</td>
    </tr>
    <tr>
      <th>44</th>
      <td>3M Company, Q3 2014 Earnings Call, Oct 23, 201...</td>
      <td>528</td>
      <td>productivity</td>
      <td>Nigel, what we're seeing on the movement there...</td>
    </tr>
  </tbody>
</table>
<p>352 rows × 4 columns</p>
</div>




```python
## Trying to feather-save the outp DFs
import feather
print(path2)

t1 = time.time()
feather.write_dataframe(outp_hypoth_sent_df, path2 + "outp_hypoth_sent_df.feather")
feather.write_dataframe(outp_explor_sent_df, path2 + "outp_explor_sent_df.feather")
feather.write_dataframe(outp_exploit_sent_df, path2 + "outp_exploit_sent_df.feather")
t2 = time.time()
print(t2-t1, " secs")

```

    H:\
    0.23176288604736328  secs
    

### Post spacy processing

Time now to build the 2x2 as envisioned. Will require using set ops to find intersections between sentence indices of the same doc. 

Should ideally get 4 colms for each doc with these quantities added in. 

I've decided to go with #sentences rather than their % because the sample I saw suggests the % will be too small to matter much. 

Below, I demo obtaining the 4 colms for 1 doc and then we'll functionize the proc and loop over all docs in the DF tables.


```python
# take one sample doc & subset DFs
docName1 = files_list1[0]

# build logical vectors
is_doc_hypoth = outp_hypoth_sent_df['docName']==docName1
is_doc_explor = outp_explor_sent_df['docName']==docName1
is_doc_exploit = outp_exploit_sent_df['docName']==docName1

# subset the DFs for that doc alone
hypoth_sub_df = outp_hypoth_sent_df[is_doc_hypoth]
explor_sub_df = outp_explor_sent_df[is_doc_explor]
exploit_sub_df = outp_exploit_sent_df[is_doc_exploit]

# build list of factual sents
tot_sents = hypoth_sub_df['tot_sents'][0] 
factual_sents_hypoth = [x for x in range(tot_sents) if x not in set(hypoth_sub_df['index'])]

# find intersects b/w hypoth sents and explor sents
hypoth_explor_intersec = len(set(hypoth_sub_df['index']) & set(explor_sub_df['sentIndex']))
hypoth_exploit_intersec = len(set(hypoth_sub_df['index']) & set(exploit_sub_df['sentIndex']))
factual_explor_intersec = len(set(factual_sents_hypoth) & set(explor_sub_df['sentIndex']))
factual_exploit_intersec = len(set(factual_sents_hypoth) & set(exploit_sub_df['sentIndex']))

print(hypoth_explor_intersec)
print(hypoth_exploit_intersec)
print(factual_explor_intersec)
print(factual_exploit_intersec)
```

    11
    6
    43
    13
    


```python
# build empty DF for populating 2x2 ke colms
# outp_colm_df = pd.DataFrame(columns = ['docName','hypoth_explor', 
#                                       'hypoth_exploit', 'factual_explor', 
#                                       'factual_exploit', 'tot_sents'])

# build func for post-spacy processing
def buildColms(docName1, outp_hypoth_sent_df, outp_explor_sent_df, outp_exploit_sent_df):
    
    # take one sample doc & subset DFs    
    is_doc_hypoth = outp_hypoth_sent_df['docName']==docName1
    is_doc_explor = outp_explor_sent_df['docName']==docName1
    is_doc_exploit = outp_exploit_sent_df['docName']==docName1

    hypoth_sub_df = outp_hypoth_sent_df[is_doc_hypoth]
    explor_sub_df = outp_explor_sent_df[is_doc_explor]
    exploit_sub_df = outp_exploit_sent_df[is_doc_exploit]

    # build list of factual sents
    tot_sents = hypoth_sub_df['tot_sents'][0]
    factual_sents_hypoth = [x for x in range(tot_sents) if x not in set(hypoth_sub_df['index'])]
    tot_sents1 = [tot_sents]
    
    # find intersects b/w hypoth sents and explor sents
    hypoth_explor_intersec = [len(set(hypoth_sub_df['index']) & set(explor_sub_df['sentIndex']))]
    hypoth_exploit_intersec = [len(set(hypoth_sub_df['index']) & set(exploit_sub_df['sentIndex']))]
    factual_explor_intersec = [len(set(factual_sents_hypoth) & set(explor_sub_df['sentIndex']))]
    factual_exploit_intersec = [len(set(factual_sents_hypoth) & set(exploit_sub_df['sentIndex']))]        
    
    return hypoth_explor_intersec, hypoth_exploit_intersec, factual_explor_intersec, factual_exploit_intersec, tot_sents1
```


```python
# build empty lists for populating & later colm-binding into a DF
docName=[]
hypoth_explor=[]
hypoth_exploit=[]
factual_explor=[]
factual_exploit=[]
tot_sents=[]

## testing the func on 1 doc
docName1 = files_list1[10]
hypoth_explor_intersec, hypoth_exploit_intersec, factual_explor_intersec, factual_exploit_intersec, tot_sents1 = buildColms(docName1, outp_hypoth_sent_df, outp_explor_sent_df, outp_exploit_sent_df)

# append to lists and carry on
docName.append(docName1)
hypoth_explor.extend(hypoth_explor_intersec)
hypoth_exploit.extend(hypoth_exploit_intersec)
factual_explor.extend(factual_explor_intersec)
factual_exploit.extend(factual_exploit_intersec)
tot_sents.extend(tot_sents1)

print(docName)
print(hypoth_explor)
print(hypoth_exploit)
print(factual_explor)
print(factual_exploit)
print(tot_sents)
```

    ['3M Company, Q3 2016 Earnings Call, Oct 25, 2016.txt']
    [7]
    [5]
    [19]
    [10]
    [765]
    


```python
## looping above func over 10 files
t1 = time.time()
for i1 in range(10):
    docName1 = files_list1[i1]
    hypoth_explor_intersec, hypoth_exploit_intersec, factual_explor_intersec, factual_exploit_intersec, tot_sents1 = buildColms(docName1, outp_hypoth_sent_df, outp_explor_sent_df, outp_exploit_sent_df)
    
    # append to lists and carry on
    docName.append(docName1)
    hypoth_explor.extend(hypoth_explor_intersec)
    hypoth_exploit.extend(hypoth_exploit_intersec)
    factual_explor.extend(factual_explor_intersec)
    factual_exploit.extend(factual_exploit_intersec)
    tot_sents.extend(tot_sents1)
    
t2 = time.time()
print(t2-t1, " secs")

print(docName)
print(factual_exploit)
print(tot_sents)
```

    0.10770988464355469  secs
    ['3M Company, Q3 2016 Earnings Call, Oct 25, 2016.txt', '3M Company, Q1 2014 Earnings Call, Apr 24, 2014.txt', '3M Company, Q1 2015 Earnings Call, Apr 23, 2015.txt', '3M Company, Q1 2016 Earnings Call, Apr 26, 2016.txt', '3M Company, Q1 2018 Earnings Call, Apr 24, 2018.txt', '3M Company, Q2 2014 Earnings Call, Jul 24, 2014.txt', '3M Company, Q2 2015 Earnings Call, Jul 23, 2015.txt', '3M Company, Q2 2016 Earnings Call, Jul 26, 2016.txt', '3M Company, Q2 2018 Earnings Call, Jul 24, 2018.txt', '3M Company, Q3 2014 Earnings Call, Oct 23, 2014.txt', '3M Company, Q3 2015 Earnings Call, Oct 22, 2015.txt']
    [10, 13, 9, 20, 12, 13, 14, 19, 10, 23, 16]
    [765, 651, 605, 645, 552, 640, 737, 655, 651, 574, 576]
    

Nice. Takes v little time once we have DFs in place, clearly.

Now let's bind above lists as colms into a DF. Behold.


```python
# bind above colms into DF and view
outp_colm_df = pd.DataFrame({'docName':docName,'hypoth_explor':hypoth_explor, 
                             'hypoth_exploit':hypoth_exploit, 'factual_explor':factual_explor, 
                             'factual_exploit':factual_exploit, 'tot_sents':tot_sents})

print(outp_colm_df.shape)

outp_colm_df.iloc[0:4,]
```

    (11, 6)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>docName</th>
      <th>hypoth_explor</th>
      <th>hypoth_exploit</th>
      <th>factual_explor</th>
      <th>factual_exploit</th>
      <th>tot_sents</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3M Company, Q3 2016 Earnings Call, Oct 25, 201...</td>
      <td>7</td>
      <td>5</td>
      <td>19</td>
      <td>10</td>
      <td>765</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3M Company, Q1 2014 Earnings Call, Apr 24, 201...</td>
      <td>11</td>
      <td>6</td>
      <td>43</td>
      <td>13</td>
      <td>651</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3M Company, Q1 2015 Earnings Call, Apr 23, 201...</td>
      <td>1</td>
      <td>0</td>
      <td>22</td>
      <td>9</td>
      <td>605</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3M Company, Q1 2016 Earnings Call, Apr 26, 201...</td>
      <td>9</td>
      <td>7</td>
      <td>43</td>
      <td>20</td>
      <td>645</td>
    </tr>
  </tbody>
</table>
</div>



Dassit for now.

Will further integrate Y and X variables to above dataframes as required prior to econometric analysis.

Sudhir
July 2019
