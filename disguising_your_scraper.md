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

This is the result of varying internal mechanisms. 

The only cons to `httpx` in this case might be the fact that it has fully encoded headers, whilst `requests` does not. This means header keys consisting of non-ASCII characters may not be able to bypass some sites.

<h3>Response handling algorithms</h3>

You may encounter obstructions such as Cloudflare, hCaptcha, ReCaptcha. Assuming the below hypothetical code that you happen to own. You can override the session and deal with these things without having to worry about making any changes in your code per site.

```py
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
            return self.request(self, *args, **kwargs)


        hcaptcha_sk, type_of = hcaptcha.deduce_sitekey(self, response)
        
        if hcaptcha_sk:
            if type_of == 'hsw':
                token = hcaptcha.get_hsw_token(self, response, hcaptcha_sk)
            else:
                token = hcaptcha.get_hsl_token(self, response, hcaptcha_sk)
            
            setattr(response, 'hcaptcha_token', type_of)

        recaptcha_sk, type_of = grecaptcha.sitekey_on_site(self, response)

        if recaptcha_sk:
            if isinstance(type_of, int):
                token = grecaptcha.recaptcha_solve(self, response, recaptcha_sk, v=type_of)
            else:
                token = type_of

            setattr(response, 'grecaptcha_token', type_of)
            
        return response
```
