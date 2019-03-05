
###### IMPORT THE NECESSARY PACKAGES


```python
import io
from tkinter import *
from tkinter import filedialog
from docx import Document
from pdfminer.pdfinterp import PDFResourceManager, PDFPageInterpreter
from pdfminer.converter import TextConverter
from pdfminer.layout import LAParams
from pdfminer.pdfpage import PDFPage
from sklearn.feature_extraction.text import CountVectorizer
import re
import string
import nltk
import math
from collections import Counter
```

###### SELECT THE FILE


```python
root = Tk()
root.filename = filedialog.askopenfilename(initialdir='/python', title="Select file",
                                           filetypes=[("Text Files", "*.txt"),
                                                      ("Docx Files","*.docx"),("all files","*.*")])

def quit():
    global root
    root.destroy()
    
```


```python
quit()
ext = root.filename.split('.')[-1]
```

###### DOCX TO TEXT CONVERTOR


```python
def convertDocxToText(path):
    document = Document(path)
    return "      ".join([para.text for para in document.paragraphs])


```

###### PDF TO TEXT CONVERTOR


```python
def convert_pdf_to_txt(path):
    rsrcmgr = PDFResourceManager()
    retstr = io.StringIO()
    codec = 'utf-8'
    laparams = LAParams()
    device = TextConverter(rsrcmgr, retstr, codec=codec, laparams=laparams)
    fp = open(path, 'rb')
    interpreter = PDFPageInterpreter(rsrcmgr, device)
    password = ""
    maxpages = 0
    caching = True
    pagenos = set()

    for page in PDFPage.get_pages(fp, pagenos, maxpages=maxpages,
                                  password=password,
                                  caching=caching,
                                  check_extractable=True):
        interpreter.process_page(page)

    text = retstr.getvalue()

    fp.close()
    device.close()
    retstr.close()
    return text
```

###### RTF  TO TEXT EXTRACTION


```python
from pyth.plugins.rtf15.reader import Rtf15Reader
from pyth.plugins.plaintext.writer import PlaintextWriter

def convertRtfToText(path):
    doc = Rtf15Reader.read(open(path))
    return PlaintextWriter.write(doc).getvalue()
```


```python
def convertRtfToText(path):
    doc = Rtf15Reader.read(open(path))
    return PlaintextWriter.write(doc).getvalue()
```

###### NOISE REMOVAL


```python
def _remove_noise(input_text):
    words = input_text.split() 
    noise_free_words = [word for word in words if word not in noise_list] 
    noise_free_text = " ".join(noise_free_words) 
    return noise_free_text
```

###### TEXT TO VECTOR CONVERSION


```python
def text_to_vector(text): 
    words = text.split() 
    return Counter(words)
```

###### Cosine Similarity Calculation


```python
def get_cosine(vec1, vec2):
    common = set(vec1.keys()) & set(vec2.keys())
    numerator = sum([vec1[x] * vec2[x] for x in common])

    sum1 = sum([vec1[x]**2 for x in vec1.keys()]) 
    sum2 = sum([vec2[x]**2 for x in vec2.keys()]) 
    denominator = math.sqrt(sum1) * math.sqrt(sum2)
   
    if not denominator:
        return 0.0 
    else:
        return float(numerator) / denominator
```

###### DECISION LOOP


```python
if ext == "docx":
    text=[convertDocxToText(root.filename)]
elif ext == "pdf":
    text=[convert_pdf_to_txt(root.filename)]
elif ext == "rtf":
    text=[convertRtfToText(root.filename)]
else:
    print("Kindly upload **docx**,**pdf**,**rtf** format files")
```


```python
from sklearn.feature_extraction.text import CountVectorizer
# create the transform
# Here, by default the characters will be converted to lowercase
vectorizer = CountVectorizer()

# tokenize and build vocabulary
vectorizer.fit(text)
```




    CountVectorizer(analyzer='word', binary=False, decode_error='strict',
            dtype=<class 'numpy.int64'>, encoding='utf-8', input='content',
            lowercase=True, max_df=1.0, max_features=None, min_df=1,
            ngram_range=(1, 1), preprocessor=None, stop_words=None,
            strip_accents=None, token_pattern='(?u)\\b\\w\\w+\\b',
            tokenizer=None, vocabulary=None)




```python
vector = vectorizer.transform(text)
```


```python
jd_bow_text = " ".join(vectorizer.vocabulary_.keys())
```


```python
# Sample code to remove noisy words from a text
noise_list = [ "...","a", "about", "above", "above", "across", "after", "afterwards", "again", "against", "all", "almost", "alone", "along", "already", "also","although","always","am","among", "amongst", "amoungst", "amount",  "an", "and", "another", "any","anyhow","anyone","anything","anyway", "anywhere", "are", "around", "as",  "at", "back","be","became", "because","become","becomes", "becoming", "been", "before", "beforehand", "behind", "being", "below", "beside", "besides", "between", "beyond", "bill", "both", "bottom","but", "by", "call", "can", "cannot", "cant", "co", "con", "could", "couldnt", "cry", "de", "describe", "detail", "do", "done", "down", "due", "during", "each", "eg", "eight", "either", "eleven","else", "elsewhere", "empty", "enough", "etc", "even", "ever", "every", "everyone", "everything", "everywhere", "except", "few", "fifteen", "fify", "fill", "find", "fire", "first", "five", "for", "former", "formerly", "forty", "found", "four", "from", "front", "full", "further", "get", "give", "go", "had", "has", "hasnt", "have", "he", "hence", "her", "here", "hereafter", "hereby", "herein", "hereupon", "hers", "herself", "him", "himself", "his", "how", "however", "hundred", "ie", "if", "in", "inc", "indeed", "interest", "into", "is", "it", "its", "itself", "keep", "last", "latter", "latterly", "least", "less", "ltd", "made", "many", "may", "me", "meanwhile", "might", "mill", "mine", "more", "moreover", "most", "mostly", "move", "much", "must", "my", "myself", "name", "namely", "neither", "never", "nevertheless", "next", "nine", "no", "nobody", "none", "noone", "nor", "not", "nothing", "now", "nowhere", "of", "off", "often", "on", "once", "one", "only", "onto", "or", "other", "others", "otherwise", "our", "ours", "ourselves", "out", "over", "own","part", "per", "perhaps", "please", "put", "rather", "re", "same", "see", "seem", "seemed", "seeming", "seems", "serious", "several", "she", "should", "show", "side", "since", "sincere", "six", "sixty", "so", "some", "somehow", "someone", "something", "sometime", "sometimes", "somewhere", "still", "such", "system", "take", "ten", "than", "that", "the", "their", "them", "themselves", "then", "thence", "there", "thereafter", "thereby", "therefore", "therein", "thereupon", "these", "they", "thickv", "thin", "third", "this", "those", "though", "three", "through", "throughout", "thru", "thus", "to", "together", "too", "top", "toward", "towards", "twelve", "twenty", "two", "un", "under", "until", "up", "upon", "us", "very", "via", "was", "we", "well", "were", "what", "whatever", "when", "whence", "whenever", "where", "whereafter", "whereas", "whereby", "wherein", "whereupon", "wherever", "whether", "which", "while", "whither", "who", "whoever", "whole", "whom", "whose", "why", "will", "with", "within", "without", "would", "yet", "you", "your", "yours", "yourself", "yourselves", "the"]

def _remove_noise(input_text):
    words = input_text.split() 
    noise_free_words = [word for word in words if word not in noise_list] 
    noise_free_text = " ".join(noise_free_words) 
    return noise_free_text

job_descriptions = _remove_noise(jd_bow_text)
job_descriptions = text_to_vector(job_descriptions)

```


```python
print('congrats you have the job description ready......!!')
nums = int(input('enter the number of resumes to do cosine similarity:'))
```

    congrats you have the job description ready......!!
    enter the number of resumes:2
    


```python
for num in range(1,nums+1):
    print('--------------------------------------------------------------------------')
    print('result for RESUME "{}"'.format(num))
    root = Tk()
    root.filename = filedialog.askopenfilename(initialdir='/python', title="Select file",
                                           filetypes=[("all files","*.*")])
    #print(root.filename)

    def quit():
        global root
        root.destroy()
    
    ext = root.filename.split('.')[-1]
    
    if ext == "docx":
        text=[convertDocxToText(root.filename)]
    elif ext == "pdf":
        text=[convert_pdf_to_txt(root.filename)]
    elif ext == "rtf":
        text=[convertRtfToText(root.filename)]
    else:
        print("Kindly upload **docx**,**pdf**,**rtf** format files")
        
    vectorizer = CountVectorizer()
    
    vectorizer.fit(text)
    
    vector = vectorizer.transform(text)
    
    jd_bow_text = " ".join(vectorizer.vocabulary_.keys())
    
    noise_list = [ "...","a", "about", "above", "above", "across", "after", "afterwards", "again", "against", "all", "almost", "alone", "along", "already", "also","although","always","am","among", "amongst", "amoungst", "amount",  "an", "and", "another", "any","anyhow","anyone","anything","anyway", "anywhere", "are", "around", "as",  "at", "back","be","became", "because","become","becomes", "becoming", "been", "before", "beforehand", "behind", "being", "below", "beside", "besides", "between", "beyond", "bill", "both", "bottom","but", "by", "call", "can", "cannot", "cant", "co", "con", "could", "couldnt", "cry", "de", "describe", "detail", "do", "done", "down", "due", "during", "each", "eg", "eight", "either", "eleven","else", "elsewhere", "empty", "enough", "etc", "even", "ever", "every", "everyone", "everything", "everywhere", "except", "few", "fifteen", "fify", "fill", "find", "fire", "first", "five", "for", "former", "formerly", "forty", "found", "four", "from", "front", "full", "further", "get", "give", "go", "had", "has", "hasnt", "have", "he", "hence", "her", "here", "hereafter", "hereby", "herein", "hereupon", "hers", "herself", "him", "himself", "his", "how", "however", "hundred", "ie", "if", "in", "inc", "indeed", "interest", "into", "is", "it", "its", "itself", "keep", "last", "latter", "latterly", "least", "less", "ltd", "made", "many", "may", "me", "meanwhile", "might", "mill", "mine", "more", "moreover", "most", "mostly", "move", "much", "must", "my", "myself", "name", "namely", "neither", "never", "nevertheless", "next", "nine", "no", "nobody", "none", "noone", "nor", "not", "nothing", "now", "nowhere", "of", "off", "often", "on", "once", "one", "only", "onto", "or", "other", "others", "otherwise", "our", "ours", "ourselves", "out", "over", "own","part", "per", "perhaps", "please", "put", "rather", "re", "same", "see", "seem", "seemed", "seeming", "seems", "serious", "several", "she", "should", "show", "side", "since", "sincere", "six", "sixty", "so", "some", "somehow", "someone", "something", "sometime", "sometimes", "somewhere", "still", "such", "system", "take", "ten", "than", "that", "the", "their", "them", "themselves", "then", "thence", "there", "thereafter", "thereby", "therefore", "therein", "thereupon", "these", "they", "thickv", "thin", "third", "this", "those", "though", "three", "through", "throughout", "thru", "thus", "to", "together", "too", "top", "toward", "towards", "twelve", "twenty", "two", "un", "under", "until", "up", "upon", "us", "very", "via", "was", "we", "well", "were", "what", "whatever", "when", "whence", "whenever", "where", "whereafter", "whereas", "whereby", "wherein", "whereupon", "wherever", "whether", "which", "while", "whither", "who", "whoever", "whole", "whom", "whose", "why", "will", "with", "within", "without", "would", "yet", "you", "your", "yours", "yourself", "yourselves", "the"]
    
    job_description = _remove_noise(jd_bow_text)
    
    #print(job_description)
    
    vector = text_to_vector(job_description)
    
    print("THE SIMILARITY FOR THE JOB DESCRIPTION AND THE {} : {}".format(num, get_cosine(job_descriptions,vector)))
```

    --------------------------------------------------------------------------
    result for RESUME "1"
    THE SIMILARITY FOR THE JOB DESCRIPTION AND THE 1 : 0.24045246071971557
    --------------------------------------------------------------------------
    result for RESUME "2"
    THE SIMILARITY FOR THE JOB DESCRIPTION AND THE 2 : 0.10520381062896288
    
