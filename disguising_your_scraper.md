<h2 align="center">Disguishing your scrapers</h2>

<p align="center">
If you're writing a <b>Selenium</b> scraper, be aware that your skill level doesn't match the minimum requirements for this page.
</p>

<h3>Why is scraping not appreciated?</h3>

- It obliterates ads and hence, blocks the site revenue.
- It is more than usually used to spam the content serving networks, hence, affecting the server performance.
- It is more than usually also used to steal content off of a site and serve in other.
- Competent scrapers usually look for exploits on site. Among these, the open source scrapers may leak site exploits to a wider audience.

<h3>Why do you need to disguise your scraper?</h3>

Like the above points suggest, scraping is a good act. There are mechanisms to actively kill scrapers and only allow the humans in. You will need to make your scraper's identity as narrow as possible to a browser's identity.

Some sites check the client using headers and on-site javascript challenges. This will result in invalid responses along the status code of 400-499. 

*Keep in mind that there are sites that produce responses without giving out the appropriate status codes.*

<h3>Custom Headers</h3>

Here are some headers you need to check for:

| Header | What's the purpose of this? | What should I change this to? |
| --- | --- | --- |
| `User-Agent` | Specifies your client's name along with the versions. | Probably the user-agent used by your browser. |
| `Referer` | Specifies which site referred the current site. | The url from which **you** obtained the scraping url. |
| `X-Requested-With` | Specifies what caused the request to that site. This is prominent in site's AJAX / API. | Usually: `XMLHttpRequest`, it may vary based on the site's JS |
| `Cookie` | Cookie required to access the site. | Whatever the cookie was when you accessed the site in your normal browser. |
| `Authorization` | Authorization tokens / credentials required for site access. | Correct authorization tokens or credentials for site access. |

Usage of correct headers will give you site content access given you can access it through your web browser.

**Keep in mind that this is only the fraction of what the possible headers can be.**

<h3>Appropriate Libraries</h3>

In Python, `requests` and `httpx` have differences.

```py
>>> import requests, httpx
>>> requests.get("http://www.crunchyroll.com/", headers={"User-Agent": "justfoolingaround/1", "Referer": "https://example.com/"})
<Response [403]>
>>> httpx.get("http://www.crunchyroll.com/", headers={"User-Agent": "justfoolingaround/1", "Referer": "https://example.com/"})
<Response [200 OK]>
```

As we can see, the former response is a 403. This is a forbidden response and generally specifies that the content is not present. The latter however is a 200, OK response. In this response, content is available.

This is the result of varying internal mechanisms. 

The only cons to `httpx` in this case might be the fact that it has fully encoded headers, whilst `requests` does not. This means header keys consisting of non-ASCII characters may not be able to bypass some sites.

<h3>Response handling algorithms</h3>

A session class is an object available in many libraries. This thing is like a house for your outgoing and incoming responses. A well written library has a session class that even accounts for appropriate cookie handling. Meaning, if you ever send a request to a site you need not need to worry about the cookies of that site for the next site you visit.

No matter how cool session classes may be, at the end of the day, they are mere objects. That means, you, as a user can easily change what is within it. (This may require a high understanding of the library and the language)

This is done through inheritance. You inherit a session class and modify what's within.

For example:

```py
class QuiteANoise(httpx.Client):

    def request(self, *args, **kwargs):
        print("Ooh, I got a request with arguments: {!r}, and keyword arguments: {!r}.".format(args, kwargs))
        response = super().request(*args, **kwargs)
        print("That request has a {!r}!".format(response))
        return response
```

In the above inherited session, what we do is *quite noisy*. We announced a request that is about to be sent and a response that was just recieved.

`super`, in Python, allows you to get the class that the current class inherits.

Do not forget to return your `response`, else your program will be dumbfounded since nothing ever gets out of your request!

So, we're going to abuse this fancy technique to effectively bypass some hinderances.

Namely `hCaptcha`, `reCaptcha` and `Cloudflare`.

```py
"""
This code is completely hypothetical, you probably 
do not have a hCaptcha, reCaptcha and a Cloudflare
bypass. 

This code is a mere reference and may not suffice
your need.
"""
from . import hcaptcha
from . import grecaptcha

import httpx

class YourScraperSession(httpx.Client):

    def request(self, *args, **kwargs):
        
        response = super().request(*args, **kwargs)

        if response.status_code >= 400:

            if hcaptcha.has_cloudflare(response):
                cloudflare_cookie = hcaptcha.cloudflare_clearance_jar(self, response, *args, **kwargs)
                self.cookies.update(cloudflare_cookie)
                return self.request(self, *args, **kwargs)

            # Further methods to bypass something else.
            return self.request(self, *args, **kwargs) # psssssst. RECURSIVE HELL, `return response` is safer


        hcaptcha_sk, type_of = hcaptcha.deduce_sitekey(self, response)
        
        if hcaptcha_sk:
            if type_of == 'hsw':
                token = hcaptcha.get_hsw_token(self, response, hcaptcha_sk)
            else:
                token = hcaptcha.get_hsl_token(self, response, hcaptcha_sk)
            
            setattr(response, 'hcaptcha_token', token)

        recaptcha_sk, type_of = grecaptcha.sitekey_on_site(self, response)

        if recaptcha_sk:
            if isinstance(type_of, int):
                token = grecaptcha.recaptcha_solve(self, response, recaptcha_sk, v=type_of)
            else:
                token = type_of

            setattr(response, 'grecaptcha_token', token)
            
        return response
```

So, let's see what happens here.

Firstly, we check whether the response has a error or not. This is done by checking if the response's status code is **greater than or equal to** 400.

After this, we check if the site has Cloudflare, if the site has Cloudflare, we let the hypothetical function do its magic and give us the bypass cookies. Then after, we update our session class' cookie. Cookie vary across sites but in this case, our hypothetical function will take the session and make it so that the cookie only applies to that site url within and with the correct headers.

After a magical cloudflare bypass (people wish they have this, you will too, probably.), we call the overridden function `.request` again to ensure the response following this will be bypassed to. This is recursion. 

If anything else is required, you should add your own code to execute bypasses so that your responses will be crisp and never error-filled.

Else, we just return the fresh `.request`.

Keep in mind that if you cannot bypass the 400~ error, your responses might end up in a permanent recursive hell, at least in the code above.

To not make your responses never return, you might want to return the non-bypassed response.

The next part mainly focuses on CAPTCHA bypasses and what we do is quite simple. A completed CAPTCHA *usually* always returns a token. 

Returning this token with the response is not a good idea as the entire return type will change. We use a sneaky little function here. Namely `setattr`. What this does is, it sets an attribute of an object.

The algorithm in easier terms is:

Task: Bypass a donkey check with your human.

- Yell "hee~haw". (Prove that you're a donkey, this is how the hypothetical functions work.)
- Be handed the ribbon. (In our case, this is the token.)

Now the problem is, the ribbon is not a human but still needs to come back. How does a normal human do this? Wear the ribbon.

Wearning the ribbon is `setattr`. We can wear the ribbon everywhere. Leg, foot, butt.. you name it. No matter where you put it, you get the ribbon, so just be a bit reasonable with it. Like a decent developer and a decent human, wear the ribbon on the left side of your chest. In the code above, this reasonable place is `<captcha_name>_token`.

Let's get out of this donkey business.

After this reasonable token placement, we get the response back.

This token can now, always be accessed in reasonable places, reasonably.


```py
client = YourScraperSession()

bypassed_response = client.get("https://kwik.cx/f/2oHQioeCvHtx")
print(bypassed_response.hcaptcha_token)
```

Keep in mind that if there is no ribbon/token, there is no way of reasonable way of accessing it.

In any case, this is how you, as a decent developer, handle the response properly.