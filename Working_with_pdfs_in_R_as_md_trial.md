### Introduction

PDF files are great for multiple reasons like sharing your work with
others, printing in different OS etc but they are very difficult to
manage when it comes to extracting data from PDFs.

Sometimes it takes a significant amount of our time for just manually
copying data from PDF files to Excel or text files. In this article, I
will discuss two R packages:

-   [pdftools](https://cran.r-project.org/web/packages/pdftools/index.html)
    and

-   [tabulizer](https://github.com/ropensci/tabulizer)

for extracting data from pdf files to R work space.

The `pdftools` package is based on ‘libpoppler’ for extracting text,
fonts, attachments and metadata from a PDF file.

Also it supports high quality rendering of PDF documents info PNG, JPEG,
TIFF format, or into raw bitmap vectors for further processing in R.

The `tabulizer` package provides R bindings to the
[Tabula](http://tabula.technology/) java library, which can be used to
computationally extract tables from PDF documents.

The tabulizer package is available on
[GitHub](https://github.com/ropensci/tabulizer). It can be installed
with *install\_github* command from *“ghit”* package. Refer the code
below for installation of both the packages in 64bit Windows R
environment.

Note that tabulizer package uses Tabula java library, so you need prior
installation of 64bit Java in your computer. If you don’t have Java
installed in your computer, then you should download Java setup file
from this [link](https://java.com/en/download/manual.jsp) (Windows
Offline (64-bit) setup file) and install Java before installing
tabulizer package.

    suppressPackageStartupMessages({
      
    # Install pdftools package
    if (!require(pdftools)) {install.packages("pdftools")}; library(pdftools)

    if(!require(data.tree)){install.packages("data.tree")}; library(data.tree) 

    if (!require(knitr)){install.packages("knitr")}; library(knitr)    
      
    # if (!require(devtools)) {install.packages("devtools")}; library(devtools)

    # if (!require(tabulizer)) {install.packages("tabulizer")}; library(tabulizer)
      
    # Install tabulizer from github
    if (!require(webp)) {install.packages("webp")}; library(webp)

       })

    # first set your working directory
    setwd('C:\\Users\\20052\\Dropbox\\teaching related\\CBA teaching\\DC batch 12 CBA\\DC\\Lec 5\\')
    getwd()    # check wd

    ## [1] "C:/Users/20052/Dropbox/teaching related/CBA teaching/DC batch 12 CBA/DC/Lec 5"

### Working with pdf files in R

In this article I’ wi’ll use Alphabet’s Form 10-K compliance filing,
downloadable from this
[link](https://abc.xyz/investor/pdf/20161231_alphabet_10K.pdf)

Plan is to extract all the text from this 98 page pdf page by page. Say
hello to the `pdf_text()` function form pdftools package.

Below, we define the PDF’s local location. Then use the `pdf_text`
function to convert it to text format. Finally, print the first thousand
characters from the 6th page of the pdf, using base R’s `substr()` func.

    # read in file path
    pdf_file_path = './20161231_alphabet_10K.pdf'    # character

    # extract pdf text
    system.time({ txt <- pdf_text(pdf_file_path) })    # 0.58 secs

    ##    user  system elapsed 
    ##    0.36    0.11    0.50

    # First 1000 characters from Page 6 of text
    cat(substr(txt[6], 1, 1000))

    ##  Table of Contents                                                                                             Alphabet Inc.
    ## PART I
    ##  ITEM 1.       BUSINESS
    ## Overview
    ##        As our founders Larry and Sergey wrote in the original founders' letter, "Google is not a conventional company.
    ## We do not intend to become one." That unconventional spirit has been a driving force throughout our history -- inspiring
    ## us to do things like rethink the mobile device ecosystem with Android and map the world with Google Maps. As part
    ## of that, our founders also explained that you could expect us to make "smaller bets in areas that might seem very
    ## speculative or even strange when compared to our current businesses." From the start, the company has always
    ## strived to do more, and to do important and meaningful things with the resources we have.
    ##        Alphabet is a collection of businesses -- the largest of which, of course, is Google. It also includes businesses
    ## that are generally pretty far afield

We can use `pdf_toc()` function to extract table of contents of the PDF
file.

    # Table of contents
    toc <- pdf_toc(pdf_file_path)

    # Show as data.tree
    require(knitr)
    knitr::kable(as.Node(toc, mode="explicit", nameName="title"), format="rst")

===================================================================================================================
levelName  
===================================================================================================================

¦–Cover  
¦–Table of Contents  
¦–Note About Forward-Looking Statements  
¦–Part I  
¦ ¦–Business  
¦ ¦–Risk Factors  
¦ ¦–Unresolved Staff Comments  
¦ ¦–Properties  
¦ ¦–Legal Proceedings  
¦ °–Mine Safety Disclosures  
¦–Part II  
¦ ¦–Market for Registrant s Common Equity, Related Stockholder Matters
and Issuer Purchases of Equity Securities ¦ ¦–Selected Financial Data  
¦ ¦–Management s Discussion and Analysis of Financial Condition and
Results of Operations  
¦ ¦ ¦–Trends in Our Business  
¦ ¦ ¦–Executive Overview of Results  
¦ ¦ ¦–Information about Segments  
¦ ¦ ¦–Revenues  
¦ ¦ ¦–Revenues by Geography  
¦ ¦ ¦–Cost of Revenues  
¦ ¦ ¦–Operating Expenses  
¦ ¦ ¦–Stock-Based Compensation  
¦ ¦ ¦–Other Income (Expense), Net  
¦ ¦ ¦–Provision for Income Taxes  
¦ ¦ ¦–Quarterly Results of Operations  
¦ ¦ ¦–Capital Resources and Liquidity  
¦ ¦ ¦–Contractual Obligations  
¦ ¦ ¦–Off Balance Sheet Arrangements  
¦ ¦ °–Critical Accounting Policies and Estimates  
¦ ¦–Quantitative and Qualitative Disclosures About Market Risk  
¦ ¦–Financial Statements and Supplementary Data  
¦ ¦ ¦–Report of E&Y - Financials  
¦ ¦ ¦–Report of E&Y - COSO  
¦ ¦ ¦–Consolidated Balance Sheets  
¦ ¦ ¦–Consolidated Statements of Income  
¦ ¦ ¦–Consolidated Statements of Comprehensive Income  
¦ ¦ ¦–Consolidated Statements of Stockholders Equity  
¦ ¦ ¦–Consolidated Statements of Cash Flows  
¦ ¦ °–Notes to Consolidated Financial Statements  
¦ ¦ ¦–Note 1. Nature of Operations and Summary of Significant Accounting
Policies  
¦ ¦ ¦–Note 2. Financial Instruments  
¦ ¦ ¦ ¦–Cash, Cash Equivalents, and Marketable Securities  
¦ ¦ ¦ ¦–Securities Lending Program  
¦ ¦ ¦ ¦–Derivative Financial Instruments  
¦ ¦ ¦ °–Offsetting of Derivatives, Securities Lending, and Reverse
Repurchase Agreements  
¦ ¦ ¦–Note 3. Non-Marketable Investments  
¦ ¦ ¦–Note 4. Debt  
¦ ¦ ¦–Note 5. Supplemental Financial Statement Information  
¦ ¦ ¦–Note 6. Acquisitions  
¦ ¦ ¦–Note 7. Collaboration Agreement  
¦ ¦ ¦–Note 8. Goodwill and Other Intangible Assets  
¦ ¦ ¦–Note 9. Discontinued Operations  
¦ ¦ ¦–Note 10. Commitments and Contingencies  
¦ ¦ ¦–Note 11. Net Income Per Share of Class A and Class B Common
Stock  
¦ ¦ ¦–Note 12. Stockholders Equity  
¦ ¦ ¦–Note 13. 401(k) Plans  
¦ ¦ ¦–Note 14. Income Taxes  
¦ ¦ ¦–Note 15. Information about Segments and Geographic Areas  
¦ ¦ °–Note 16. Subsequent Event  
¦ ¦–Changes in and Disagreements With Accountants on Accounting and
Financial Disclosure  
¦ ¦–Controls and Procedures  
¦ °–Other Information  
¦–Part III  
¦ ¦–Directors, Executive Officers and Corporate Governance  
¦ ¦–Executive Compensation  
¦ ¦–Security Ownership of Certain Beneficial Owners and Management and
Related Stockholder Matters  
¦ ¦–Certain Relationships and Related Transactions, and Director
Independence  
¦ °–Principal Accountant Fees and Services  
¦–Part IV  
¦ °–Exhibits, Financial Statement Schedules  
¦ °–Exhibits  
¦–Exhibit 12  
¦–Exhibit 21.01  
¦–Exhibit 23.01  
¦–Exhibit 31.01  
¦–Exhibit 31.02  
°–Exhibit 32.01  
===================================================================================================================

Also we can use `pdf_info()` function to extract meta data associated
with the PDF file.

    # Author, version, etc, Metadata Extraction
    info <- pdf_info(pdf_file_path)
    info

    ## $version
    ## [1] "1.5"
    ## 
    ## $pages
    ## [1] 98
    ## 
    ## $encrypted
    ## [1] FALSE
    ## 
    ## $linearized
    ## [1] FALSE
    ## 
    ## $keys
    ## $keys$Producer
    ## [1] "WebFilings"
    ## 
    ## $keys$Title
    ## [1] "GOOG Q4 2016 10-K"
    ## 
    ## 
    ## $created
    ## [1] "2017-02-23 10:35:00 IST"
    ## 
    ## $modified
    ## [1] "2106-02-07 11:58:15 IST"
    ## 
    ## $metadata
    ## [1] ""
    ## 
    ## $locked
    ## [1] FALSE
    ## 
    ## $attachments
    ## [1] FALSE
    ## 
    ## $layout
    ## [1] "one_column"

### Saving as Images

The wonders continue.

Say we wanna render of PDF files as bitmap arrays. Say hello to the
`pdf_render_page()` function, to render a page of the PDF into a bitmap,
which can be stored as e.g. png or jpeg.

    # renders pdf to bitmap array
    bitmap <- pdf_render_page(pdf_file_path, page = 6)

    # save bitmap image, will write to wd
    png::writePNG(bitmap, "page6.png")
    jpeg::writeJPEG(bitmap, "page6.jpeg", quality = 5)
    webp::write_webp(bitmap, "page6.webp")

You can see the page6.jpeg file in the working directory.

Enough for now.

Sudhir
