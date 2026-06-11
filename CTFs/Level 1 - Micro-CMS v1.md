**Goal:** 4 flags

1) Upon entering the site I immediately notice there are 3 hyper links:
	1) Testing
		1) upon entering I'm brought to a static page with an option to go home or edit the page
	2) Markdown Test
	3) Create a New Page
	4) Edit `______` page (*accessible through traversing the other pages*)
2) My first impressions are to check the source html, and also consider the possibility this has something to do with form submission vulnerability as creating and editing pages is done through form submission
3) by running `document.cookie` in the console I notice there are values that are returned 
	1) the key value pairs seem to be gibberish, I don't recognize any of them to be possible flags but I will keep note of this as I go forward as a potential route to try again later
4) noticed failed favicon.ico retrieval in the network, tried similar to last level's trial by adding /favicon.ico to the tail of the link and got nothing usable, just standard 404 response
5) looking at the links for the pages, `page/1` corresponds to the testing page, `page/2` corresponds to markdown testing page, what about other pages?
	1) no `page/3` or 4
	2) there is a link to editing the page `edit/1`
	3) upon entering `edit/1` I find, as aforementioned, form submission. looking at the fields, there's no csrf token and it says "**Markdown is supported but scripts or not**" which must be a huge hint. let's see if editing the page with markdown (or a script just to be sure) is reflected on the final site! 
		1) Markdown is confirmed to work upon entering into the text box
		2) JS doesn't run with `<script>` tag or at least not properly
		3) additional thought: maybe I can use code block JS to trigger code execution instead of a script tag!
		4) trying the standard alert method didn't work with markdown, maybe I need to use script tag within markdown?
		5) weird, when I use the script tags in markdown they reflect as "`<scrubbed> ... </scrubbed>`"

I'm getting stuck, why don't we backtrack and return to roots for a second? Let's look back at the very beginning and investigate from the title alone. The name of the level is Micro-CMS. What does CMS stand for? Probably **Content Management System**. Time for some research on it!
This is a really standard use forms to edit data (some sort of content) and push to save to server. Let's play around more with the markdown xss approach to see what we can get.

## Flag 2:
1) going back to my markdown xss idea, I noticed that putting a script tag in the body wasn't allowed but what about the title?
2) I put `<script>alert(document.cookie)</script>` into the title field and saved it, and the title actually saved as that exactly, no scrubbing, however no alert was actually triggered upon save
3) what if i exit out and to the homepage? a re-render has to happen and now the render may not check as input may be checked but if it isn't sanitized it may run upon re-render since it was stored successfully into the backend
4) it was correct! an alert appeared and I got the second flag: `^FLAG^7c27cf87146d039590fd323c36e41f69913e4fd9ef79149e2aff366c4c26d7c1$FLAG$`

