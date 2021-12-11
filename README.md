## Requests based scraping tutorial


You want to start scraping? Well this guide will teach you, and not some baby selenium scraping. This guide only uses raw requests and has examples in both python and kotlin. Only basic programming knowlege in one of those languages is required to follow along in the guide.


0. [Starting scraping from zero](https://github.com/Blatzar/scraping-tutorial/blob/master/starting.md)
1. [Properly scraping JSON apis often found on sites](https://github.com/Blatzar/scraping-tutorial/blob/master/using_apis.md)
2. [Evading developer tools detection when scraping](https://github.com/Blatzar/scraping-tutorial/blob/master/devtools_detectors.md)
3. [Why your requests fails and how to fix them](https://github.com/Blatzar/scraping-tutorial/blob/master/disguising_your_scraper.md)

Once you've read and understood the concepts behind scraping take a look at [a provider in CloudStream](https://github.com/LagradOst/CloudStream-3/blob/3a78f41aad93dc5755ce9e105db9ab19287b912a/app/src/main/java/com/lagradost/cloudstream3/movieproviders/VidEmbedProvider.kt). I added tons of comments to make every aspect of writing CloudStream providers clear. Even if you're not planning on contributing to Cloudstream looking at the code may help. 

Take a look at [Thenos](https://github.com/LagradOst/CloudStream-3/blob/3a78f41aad93dc5755ce9e105db9ab19287b912a/app/src/main/java/com/lagradost/cloudstream3/movieproviders/ThenosProvider.kt) for an example of json based scraping in kotlin.
