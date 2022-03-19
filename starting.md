Scraping is just downloading a webpage and getting the wanted information from it. 
As a start you can scrape the README.md


I'll use khttp for the kotlin implementation because of the ease of use, if you want something company-tier I'd recommend OkHttp.

(**Update**: I have made an okhttp wrapper **for android apps**, check out [NiceHttp](https://github.com/Blatzar/NiceHttp))


# **1. Scraping the Readme** 

**Python**
```python
import requests
url = "https://raw.githubusercontent.com/Blatzar/scraping-tutorial/master/README.md"
response = requests.get(url)
print(response.text)  # Prints the readme
```

**Kotlin**

In build.gradle:
```
repositories {
    mavenCentral()
    jcenter()
    maven { url 'https://jitpack.io' }
}

dependencies {
	// Other dependencies above
	compile group: 'khttp', name: 'khttp', version: '1.0.0'
}
```
In main.kt
```java
fun main() {
    val url = "https://raw.githubusercontent.com/Blatzar/scraping-tutorial/master/README.md"
    val response = khttp.get(url)
    println(response.text)
}
```


# **2. Getting the github project description** 
Scraping is all about getting what you want in a good format you can use to automate stuff.

Start by opening up the developer tools, using

<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>I</kbd> 

or 

<kbd>f12</kbd> 

or 

Right click and press *Inspect*

In here you can look at all the network requests the browser is making and much more, but the important part currently is the HTML displayed. You need to find the HTML responsible for showing the project description, but how? 

Either click the small mouse in the top left of the developer tools or press 

<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>C</kbd> 

This makes your mouse highlight any element you hover over. Press the description to highlight up the element responsible for showing it.

Your HTML will now be focused on something like:


```cshtml
<p class="f4 mt-3">
      Work in progress tutorial for scraping streaming sites
</p>
```

Now there's multiple ways to get the text, but the 2 methods I always use is Regex and CSS selectors. Regex is basically a ctrl+f on steroids, you can search for anything. CSS selectors is a way to parse the HTML like a browser and select an element in it.

## CSS Selectors

The element is a paragraph tag, eg `<p>`, which can be found using the CSS selector: "p".

classes helps to narrow down the CSS selector search, in this case: `class="f4 mt-3"`

This can be represented with 
```css
p.f4.mt-3
```
a dot for every class ([full list of CSS selectors found here](https://www.w3schools.com/cssref/css_selectors.asp))

You can test if this CSS selector works by opening the console tab and typing:

```js
document.querySelectorAll("p.f4.mt-3");
```

This prints:
```java
NodeList [p.f4.mt-3]
```

### **NOTE**: You may not get the same results when scraping from command line, classes and elements are sometimes created by javascript on the site.


**Python**

```python
import requests
from bs4 import BeautifulSoup  # Full documentation at https://www.crummy.com/software/BeautifulSoup/bs4/doc/

url = "https://github.com/Blatzar/scraping-tutorial"
response = requests.get(url)
soup = BeautifulSoup(response.text, 'lxml')
element = soup.select("p.f4.mt-3")  # Using the CSS selector
print(element[0].text.strip())  # Selects the first element, gets the text and strips it (removes starting and ending spaces)
```

**Kotlin**

In build.gradle:
```
repositories {
    mavenCentral()
    jcenter()
    maven { url 'https://jitpack.io' }
}

dependencies {
	// Other dependencies above
	implementation "org.jsoup:jsoup:1.11.3"
	compile group: 'khttp', name: 'khttp', version: '1.0.0'
}
```
In main.kt
```java
fun main() {
    val url = "https://github.com/Blatzar/scraping-tutorial"
    val response = khttp.get(url)
    val soup = Jsoup.parse(response.text)
    val element = soup.select("p.f4.mt-3") // Using the CSS selector
    println(element.text().trim()) // Gets the text and strips it (removes starting and ending spaces)
}
```


## **Regex:**

When working with Regex I highly recommend using https://regex101.com/ (using the python flavor)

Press <kbd>Ctrl</kbd> + <kbd>U</kbd> 

to get the whole site document as text and copy everything

Paste it in the test string in regex101 and try to write an expression to only capture the text you want.

In this case the elements is 

```cshtml
<p class="f4 mt-3">
      Work in progress tutorial for scraping streaming sites
    </p>
```

Maybe we can search for `<p class="f4 mt-3">` (backslashes for ")

```regex
<p class=\"f4 mt-3\">
```

Gives a match, so lets expand the match to all characters between the two brackets ( p>....</ )

Some important tokens for that would be:

`.*?` to indicate everything except a newline any number of times, but take as little as possible

`\s*` to indicate whitespaces except a newline any number of times

`(*expression inside*)` to indicate groups

Which gives:

```regex
<p class=\"f4 mt-3\">\s*(.*)?\s*<
```
**Explained**: 

Any text exactly matching `<p class="f4 mt-3">`

then any number of whitespaces

then any number of any characters (which will be stored in group 1)

then any number of whitespaces

then the text `<`


In code:

**Python**

```python
import requests
import re  # regex

url = "https://github.com/Blatzar/scraping-tutorial"
response = requests.get(url)
description_regex = r"<p class=\"f4 mt-3\">\s*(.*)?\s*<"  # r"" stands for raw, which makes blackslashes work better, used for regexes
description = re.search(description_regex, response.text).groups()[0]
print(description)
```

**Kotlin**
In main.kt
```java
fun main() {
    val url = "https://github.com/Blatzar/scraping-tutorial"
    val response = khttp.get(url)
    val descriptionRegex = Regex("""<p class="f4 mt-3">\s*(.*)?\s*<""")
    val description = descriptionRegex.find(response.text)?.groups?.get(1)?.value
    println(description)
}
```
