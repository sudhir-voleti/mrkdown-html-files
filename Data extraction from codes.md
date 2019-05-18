
## Introduction and Background

A *barcode* is a visual, machine-readable representation of data. The data in general carries information about the object that carries the barcode. 

Traditional barcodes systematically represent data by varying the widths and spacings of parallel lines, and may be referred to as linear or one-dimensional (1D). 

Initially, barcodes were only scanned by special optical scanners called barcode readers but now with the development of application software, devices that could read images, such as smartphones are also being utilized to read them. 

Later, two-dimensional (2D) variants were developed, using rectangles, dots, hexagons and other geometric patterns, called matrix codes or 2D barcodes. 

### Quick Response (QR) codes

QR is a matrix barcode. The data is stored in horizontal and vertical black square grid patterns on a white background. 

In general, QR codes often contain data for a locator, identifier, or tracker that points to a website or application. 

A QR code uses four standardized encoding modes (numeric, alphanumeric, byte/binary, and kanji) to store data efficiently; extensions may also be used.

Barcodes and QR codes are important from a data generation and collection angle. Hence, an open-source based approach to encoding (writing)  and decoding (reading) them finds important place in DC as a course.

Here are some examples:

<img src="https://www.pyimagesearch.com/wp-content/uploads/2018/05/barcode_scanner_output.jpg" alt="Examples of 1-D and 2-D barcodes" title="QR Barcodes" />

In this markdown, we shall read and generate QR and bar codes using Python's pyzbar and __[qrtools](https://www.geeksforgeeks.org/reading-generating-qr-codes-python-using-qrtools/)__.

Ensure opencv is installed as well, from __[here if required](https://solarianprogrammer.com/2016/09/17/install-opencv-3-with-python-3-on-windows/)__. 

## Reading QR and BAR codes

Quick note here. See below code. 

The use of *import sys* followed by *! pip install moduleName* ensures the system checks if said module is available in the local libraries, else downloads and pip-installs the same.

I'm assuming you have numpy and opencv already installed.

### Installation


```python
import sys

!pip install pyzbar
```

    Requirement already satisfied: pyzbar in c:\users\20052\appdata\local\continuum\anaconda3\lib\site-packages (0.1.8)
    

### Import required libraries


```python
from __future__ import print_function
import pyzbar.pyzbar as pyzbar
import numpy as np
import cv2
```

### Create necessary functions

We'll write 2 user-defined py functions in this markdown, to:

    1. Decode the code
    2. Display the code


```python
def decode(im) : 
  # Find barcodes and QR codes
  decodedObjects = pyzbar.decode(im)
 
  # Print results
  for obj in decodedObjects:
    print('Type : ', obj.type)
    print('Data : ', obj.data,'\n')
     
  return decodedObjects
```

Above func *decode* takes a QR code image from any source (web, disk, mobile camera feed, grbbed still frames from a video etc.) and decodes it into a zbar object, and prints its main attributes that can then be displayed. 

Below, the display func takes as input the decoded zbar object and displays the result as an encoded QR code in a separate window. I'm using quite a few opencv functions here directly. 


```python
# Display barcode and QR code location  
def display(im, decodedObjects):
 
  # Loop over all decoded objects
  for decodedObject in decodedObjects: 
    points = decodedObject.polygon
 
    # If the points do not form a quad, find convex hull
    if len(points) > 4 : 
      hull = cv2.convexHull(np.array([point for point in points], dtype=np.float32))
      hull = list(map(tuple, np.squeeze(hull)))
    else : 
      hull = points;
     
    # Number of points in the convex hull
    n = len(hull)
 
    # Draw the convext hull
    for j in range(0,n):
      cv2.line(im, hull[j], hull[ (j+1) % n], (255,0,0), 3)
 
  # Display results 
  cv2.imshow("Results", im)

  # Press any key to close the window
  cv2.waitKey(0)
  cv2.destroyAllWindows()
```

### Invoke the functions

Time now to use the funcs above. 

Remember to look for a new window to open after the *display* func. Press any key to close the window. 

Later, we'll use %matplotlib to plot these figs inline inside the notebook.


```python
# Read image, safer to use '/' and not '\\' in path specification for images
path1 = 'D:/audio py files/'
im = cv2.imread(path1 + 'data/qr1.png')
 
decodedObjects = decode(im)
display(im, decodedObjects)
```

    Type :  QRCODE
    Data :  b'http://www.qrstuff.com' 
    
    

## Writing QR and BAR codes

### Installation


```python
import sys

!pip install qrcode
!pip install python-barcode
```

    Requirement already satisfied: qrcode in c:\users\20052\appdata\local\continuum\anaconda3\lib\site-packages (6.1)
    Requirement already satisfied: six in c:\users\20052\appdata\local\continuum\anaconda3\lib\site-packages (from qrcode) (1.11.0)
    Requirement already satisfied: colorama; platform_system == "Windows" in c:\users\20052\appdata\local\continuum\anaconda3\lib\site-packages (from qrcode) (0.3.9)
    Requirement already satisfied: python-barcode in c:\users\20052\appdata\local\continuum\anaconda3\lib\site-packages (0.9.0)
    

### Import required libraries


```python
import qrcode
import barcode
from barcode import generate
from barcode.writer import ImageWriter
```

### Invoke the functions

#### Generate QR code


```python
img = qrcode.make(data = 'Business Analytics')
```


```python
img
```




![png](output_23_0.png)



Note that 'business analytics' is just a piece of text which is now stored in the QR code. Implication is that the QR codes act as **storage formats**, essentially. 

Now, a URL is also just a text string. So it should be possible to create QRs for URLs. And further, get a reader (like our friendly mobile phone camera) to read the QR code's stored URL and thenopen the same in a browser. 

Below, I create a link to my isb.edu fac page and used my mobile to scan it --> open it in the mobile's browser. Note: Mine is iPhone 6S and iOS 11 in which this functionality is automated. Can't speak for Android though I suspect they'll have the same easy functionality.


```python
mySite = qrcode.make(data = "https://www.isb.edu/faculty-research/faculty/directory/voleti-sudhir")
mySite
```




![png](output_25_0.png)



## Working with (1-D) BAR codes

The barcode's invention in 1952 was based on the **Morse code** that was extended to thin and thick bars but took 20+ years to become commercially successful with the advent of Universal Product Code (UPC) scanners in supermarket checkouts starting 1974. 

More generally, tasks generically referred to as Automatic Identification and Data Capture (AIDC) include technologies such as QR codes, barcodes, RFIDs etc. AIDC is defined as: 
> "methods of automatically identifying objects, collecting data about them and entering them into computer systems without human involvement."

There are multiple ways to generate barcode using 'barcode' library based on the output format desired as detailed in __[here](https://pypi.org/project/pyBarcode/)__. Common provided barcode standards that we will use are *EAN-8, EAN-13, EAN-14, UPC-A, JAN, ISBN-10, ISBN-13, ISSN, Code 39, Code 128, and PZN*. Here we would generate the output in png and svg(default) format.


```python
import barcode
name = generate('EAN13', '123748597546546', output= path1 + 'barcode_1')
```


```python
name
```




    'D:/audio py files/barcode_1.svg'



Check the location above and see if the file *barcode_1.svg* has indeed saved there. Open and see in browser.

Next, we'll convert a numeric string to barcode after specifying encoding format (EAN) and display inline via matplotlib.


```python
EAN = barcode.get_barcode_class('ean13')
ean = EAN(u'5901234123457', writer = ImageWriter())
fullname = ean.save(path1 + 'barcode_2') 
```


```python
fullname
```




    'D:/audio py files/barcode_2.png'




```python
import cv2
import matplotlib.pyplot as plt
%matplotlib inline

img=cv2.imread(fullname)
plt.imshow(img, cmap = 'gray')
```




    <matplotlib.image.AxesImage at 0x26638093dd8>




![png](output_33_1.png)


Notice that in both examples above, the barcodes encoded numeric values only. This is the default for the EAN standard. 

However, other encoding standards such as the CODE39 and CODE128 encode alphanumeric and all ASCII characters, respectively. For more detail, __[see this link](https://www.keyence.com/ss/products/auto_id/barcode_lecture/basic/barcode-types/)__.

#### Summary

To summarize, we saw how to read and write barcode files (both 1-D linear barcodes as well as 2-D QR barcodes).

More importantly, we did so within an *open-source workflow* that we can combine with or integrate into other larger workflows in the pursuit of a specific task.

Well, that's all for now, class. Ciao.

Sudhir Voleti

April 2019
