# Introduction

All webpages work by doing requests back and forth with the website server. A request is a way to ask the server for some piece of content, like asking a person what their name is. The question can be though of as the url and can look like this:

*GET* https://example.com/name

**which translates to:**

*Question*: what is your name?

----

Scraping is just downloading a webpage and getting the wanted information from it.
Usually the information is not directly available on the page you want, meaning you need to do multiple requests. It may sound confusing, but try to understand these two images by the end:

Your browser works like this:
![Browser request](/images/browser_request.jpg) 


When you scrape you want to replicate what the browser does like this:
![Scraper request](/images/scraper_request.jpg)

Every time you visit a website you make a request to the server, the server responds to the request with a response, containing what the browser asked for. This is like asking your friend to give you a copy of his lecture notes for studying.


Scraping is usually more advanced than that, one request leads to the other in complicated ways you cannot debug. To get the information you want you need to use the information found on one page to visit to the next. To continue the anology with lecture notes, this would be like asking your friend for his lecture notes and in the notes you find the email to the teacher. With the email to the teacher you then ask them a question about the course. 

Scrapers are all about writing code to make that process happen automatically, and it can be done in different ways. You can operate an invisible web browser with code using something like selenium. This is the equivalent of simulating an entire brain to read the lecture notes to find the email, which is very slow. You can also take the content, parse it with a regex to instantly find any emails. This is like making a robot do ctrl+f in the lecture notes to find any emails, much more efficent, but requiring more fine tuning to get what you want.

-------

To demonstrate how requests work we will be scraping the Readme.

I'll use khttp for the kotlin implementation because of the ease of use, if you want something company-tier I'd recommend OkHttp.

(**Update**: I have made an okhttp wrapper **for android apps**, check out [NiceHttp](https://github.com/Blatzar/NiceHttp))


# **1. Scraping the Readme** 

This basically does what the first image does, it asks the web server with a GET request to give the content for the url. The .get(url) implies it is a GET request, and is basically the equivalent to saying "Give me the content for this url". This is used to get stuff like html, images and videos.

There are multiple different request types, but this one is the most important, with the second most important being POST requests which we will look at later.

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

**Shell**
```sh
curl "https://raw.githubusercontent.com/Blatzar/scraping-tutorial/master/README.md"
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

**Shell**
```sh
curl "https://github.com/Blatzar/scraping-tutorial" | tr -d '\n' |
  sed -nE "s/.*<article class=\"markdown-body.*<\/path><\/svg>(.*)<\/article>.*/\1/p" # we use the tr -d '\n' command to delete all new lines from the input, since sed works on a line by line basis
```
*Note*: 
Although there are external libraries which can be used to parse html in shellscripts, such as htmlq and pup, these are often slower at parsing than sed (a built-in stream editor command on unix systems).
This is why using sed with the extended regex flag `-E` is a preferrable way of parsing scraped data when writing shellscripts.


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

**Shell**
Here is an example of how html data can be parsed using sed with the extended regex flag:
```sh
printf 'some html data then data-id="123" other data and title here: title="Foo Bar" and more html\n' |
  sed -nE "s/.*data-id=\"([0-9]*)\".*title=\"([^\"]*)\".*/Title: \2\nID: \1/p" # note that we use .* at the beginning and end of the pattern in order to avoid printing everything that preceeds and follows the actual patterns we are matching
```

# Closing words

Make sure you understand everything here before moving on, this is the absolute fundamentals when it comes to scraping.
Some people come this far, but they do not quite understand how powerful this technique is.
Let's say you have a website which when you click a button opens another website. You want to do what the button press does, but when you look at the html the url to the other site is nowhere to be found. How would you solve it?

If you know a bit of how websites work you might figure out that the is because the button link gets generated by JavaScript. The obvious solution would then be to run the JavaScript and generate the button link. This is both impractical, inefficent and not what was used so far in this guide.

What you should do instead is inspect the button link url and check for something unique. For example if the button url is "https://example.com/click/487a162?key=748" then I would look through the webpage for any instances of "487a162" and "748" and figure out a way to get those strings automatically, because that's everything required to make the link.


The secret to scraping is: You have all information required to make anything your browser does, you just need to figure out how. You almost never need to run some website JavaScript to get what you want. It is like a puzzle on how to get to the next request url, you have all the pieces, you just need to figure out how they fit.

### Next up: [Properly scraping JSON apis](https://github.com/Blatzar/scraping-tutorial/blob/master/using_apis.md)
