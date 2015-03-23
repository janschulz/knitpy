---
title: "knitpy: dynamic report generation with python"
author: "Jan Schulz"
date: "12.03.2015"
output:
  pdf_document: default
  word_document: default
  html_document:
    keep_md: yes
---

This is a port of knitr (http://yihui.name/knitr/) and rmarkdown 
(http://rmarkdown.rstudio.com/) to python.

For a complete description of the code format see http://rmarkdown.rstudio.com/ and replace
`{r...}` by `{python ...}` and of course use python code blocks...

## Examples

Here are some examples:

``` None
print("Execute some code chunk and show the result")
```
```
Execute some code chunk and show the result
```

---
title: "knitpy: dynamic report generation with python"
author: "Jan Schulz"
date: "12.03.2015"
output:
  pdf_document: default
  word_document: default
  html_document:
    keep_md: yes
---

This is a port of knitr (http://yihui.name/knitr/) and rmarkdown 
(http://rmarkdown.rstudio.com/) to python.

For a complete description of the code format see http://rmarkdown.rstudio.com/ and replace
`{r...}` by `{python ...}` and of course use python code blocks...

## Examples

Here are some examples:


``` python
print("Execute some code chunk and show the result")
```

```
Execute some code chunk and show the result
```


### Code chunk arguments

You can use different arguments in the codechunk declaration. Using `echo=False` will not show the code but only the result.


```
Only the output will be visible as `echo=False`
```


The next paragraphs explores the code chunk argument `results`. 

If 'hide', knitpy will not display the code's results in the final document. If 'hold', knitpy will delay displaying all output pieces until the end of the chunk. If 'asis', knitpy will pass through results without reformatting them (useful if results return raw HTML, etc.)

`results='hold'` is not yet implemented.


``` python
print("Only the input is displayed, not the output")
```



```
This is formatted as markdown:
**This text** will be bold...
```



**This text** will be bold...

Using `eval=False`, one can hide a code chunk completely: it won't be executed and it won't be shown between this text ...


... and this text here!

You can also not show codeblocks at all, but they will be run (not included codeblock sets `have_run = True`):



``` python
if have_run == True:
    print("'have_run==True': ran the codeblock before this one.")
```

```
'have_run==True': ran the codeblock before this one.
```


### Inline code

You can also include code inline: "m=2" (expected: "m=2") 


### IPython / Jupyter display framework

The display framework is also supported.

Plots will be included as images and included in the document. The filename of the 
plot is derived from the chunk label ("sinus" in this case). The code is not 
shown in this case (`echo=False`).




![](knitpy_overview_files/figure-html/sinus-0.png)


If a html or similar thing is displayed via the IPython display framework, it will be 
included 'as is', meaning that apart from `text/plain`-only output, everything else 
will be included without marking it up as output. 


``` python
# As this all produces no output, it should go into the same input section...
from IPython.core.display import display, HTML
display(HTML("<strong>strong text</strong>"))
```



<strong>strong text</strong>


It even handles pandas.DataFrames:


``` python
import pandas as pd
pd.set_option("display.width", 200) 
s = """This is longer text"""
df = pd.DataFrame({"a":[1,2,3,4,5],"b":[s,"b","c",s,"e"]})
df
```



<div style="max-height:1000px;max-width:1500px;overflow:auto;"><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>a</th><th>b</th></tr></thead><tbody><tr><th>0</th><td> 1</td><td> This is longer text</td></tr><tr><th>1</th><td> 2</td><td> b</td></tr><tr><th>2</th><td> 3</td><td> c</td></tr><tr><th>3</th><td> 4</td><td> This is longer text</td></tr><tr><th>4</th><td> 5</td><td> e</td></tr></tbody></table></div>


`pandas.DataFrame` can be represented as `text/plain` or `text/html`. Unfortunately, pandoc cannot convert `html` to `docx` or `pdf`. If this document is converted into `docx`, the above will be displayed as `text/plain`, which looks like the following:


``` python
pd.set_option("display.notebook_repr_html", False) 
df
```

```
   a                    b
0  1  This is longer text
1  2                    b
2  3                    c
3  4  This is longer text
4  5                    e
```

``` python
# set back the display 
pd.set_option("display.notebook_repr_html", True) 
```


It's possible to get around this limitation by using the [tabulate](https://bitbucket.org/astanin/python-tabulate) package together with `results="asis"`. 


``` python
from tabulate import tabulate
from IPython.core.display import display, Markdown
# either print and use `results="asis"`
print(tabulate(df, list(df.columns), tablefmt="simple"))
```

      a  b
--  ---  -------------------
 0    1  This is longer text
 1    2  b
 2    3  c
 3    4  This is longer text
 4    5  e

``` python
# or use the IPython display framework to publish markdown
Markdown(tabulate(df, list(df.columns), tablefmt="simple"))
```



      a  b
--  ---  -------------------
 0    1  This is longer text
 1    2  b
 2    3  c
 3    4  This is longer text
 4    5  e


Unfortunately, all three versions are not perfect (e.g. all have problems with wide tables in PDF, which spill over the page margin).

### Error handling

Errors in code are shown with a bold error text:


``` python
import sys
print(sys.not_available)
```


**ERROR**: AttributeError: 'module' object has no attribute 'not_available'

```
AttributeError                            Traceback (most recent call last)
<ipython-input-29-a5971246c0f7> in <module>()
----> 1 print(sys.not_available)

AttributeError: 'module' object has no attribute 'not_available'
```



**ERROR**: Code invalid

```
for x in []:
print("No indention...")
```