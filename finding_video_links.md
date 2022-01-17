# Finding video links

Now you know the basics, enough to scrape most stuff from most sites, but not streaming sites.
Because of the high costs of video hosting the video providers really don't want anyone scraping the video and bypassing the ads.
This is why they often obfuscate, encrypt and hide their links which makes scraping really hard.
Some sites even put V3 Google Captcha on their links to prevent scraping while the majority IP/time/referer lock the video links to prevent sharing.
You will almost never find a plain \<video\> element with a mp4 link.

**This is why you should always scrape the video first when trying to scrape a video hosting site. Sometimes getting the video link can be too hard.**

I will therefore explain how to do more advanced scraping, how to get these video links.

What you want to do is:

1. Find the iFrame/Video host.*
2. Open the iFrame in a separate tab to ease clutter.*
3. Find the video link.
4. Work backwards from the video link to find the source.

* *Step 1 and 2 is not applicable to all sites.* 

Let's explain further:
**Step 1**: Most sites use an iFrame system to show their videos. This is essentially loading a separate page within the page. 
This is most evident in [Gogoanime](https://gogoanime.film/yakusoku-no-neverland-episode-1), link gets updated often, google the name and find their page if link isn't found.
The easiest way of spotting these iframes is looking at the network tab trying to find requests not from the original site. I recommend using the HTML filter.

  ![finding](https://user-images.githubusercontent.com/46196380/149821806-7426ca0f-133f-4722-8e7f-ebae26ea2ef1.png)
  
Once you have found the iFrame, in this case a fembed-hd link open it in another tab and work from there. (**Step 2**)
If you only have the iFrame it is much easier to find the necessary stuff to generate the link since a lot of useless stuff from the original site is filtered out.

**Step 3**: Find the video link. This is often quite easy, either filter all media requests or simply look for a request ending in .m3u8 or .mp4
What this allows you to do is limit exclude many requests (only look at the requests before the video link) and start looking for the link origin (**Step 4**).

  ![video_link](https://user-images.githubusercontent.com/46196380/149821919-f65e2f72-b413-4151-a4a3-db7012e2ed18.png)
  
I usually search for stuff in the video link and see if any text/headers from the preceding requests contain it. 
In this case fvs.io redirected to the mp4 link, now do the same steps for the fvs.io link to follow the request backwards to the origin. Like images are showing.

  
  ![fvs](https://user-images.githubusercontent.com/46196380/149821967-00c01103-5b4a-48dd-be18-e1fdfb967e4c.png)
  
  
  
![fvs_redirector](https://user-images.githubusercontent.com/46196380/149821984-0720addd-40a7-4a9e-a429-fec45ec28901.png)
  
  
  
![complete](https://user-images.githubusercontent.com/46196380/149821989-49b2ba8c-36b1-49a7-a41b-3c69df278a9f.png)

  
  
**NOTE: Some sites use encrypted JS to generate the video links. You need to use the browser debugger to step by step find how the links are generated in that case**

## **What to do when the site uses a captcha?**

You pretty much only have 2 options when that happens:

1. Try to use a fake / no captcha token. Some sites actually doesn't check that the captcha token is valid.
2. Use Webview or some kind of browser in the background to load the site in your stead.
