---
layout: default
title:  "Coding an Asynchronous Sitemap Generator (With Full Code on Github)"
date:   2024-08-6 10:00:00 +0200
categories: jekyll update
excerpt: "This article explores the creation of an asynchronous sitemap generator in Python.
However, before introducing the tools that will be used, I would like to give a sense of the utility of what has been developed. The first
and foremost aim of my work was experimenting with the <i>asyncio</i> library of Python, which I like because it makes non-linear
programming accessible. The aim of this article is also to discuss the concurrency/parallelism tools available for Python. The reason why I decided to deal with a twofold taks (i.e. presenting an applied project and enter a more general discussion 
about tools), is not to avoid questions like: <i>why is asynchronous programming better that multithreading or multiprocessing in the present case?</i>, 
and ultimately of <i>what are asyncio, multithreading and multiprocessing about?</i> While I acknowledge the fact that this is not the classical approach, in my own view it might eliminate some 
false opinions about the concerned tools, giving a practical context to asynchronous programming, which I believe to be far from obvious."
image: /assets/images/rambosson.jpg
published: true
---

# UNDER REVIEW 

### writing in progess... this is a draft


<p style="text-align: justify">
This article explores the creation of an asynchronous sitemap generator in Python. The code for this project can be found 
<a href="https://github.com/Gabriele-Donato/website-materials-/blob/Scraping/Asynchronous_Sitemap_Generator/Asynchronous_Sitemap_Generator.ipynb">here</a>. 
The first and foremost aim of my work was experimenting with the <i>asyncio</i> library of Python, which I like because it makes non-linear
programming accessible. 
</p>

<p style="text-align: justify">
I would also like to discuss the concurrency/parallelism tools available for Python. The reason why I decided to deal with a twofold taks (i.e. presenting an applied project and enter a more general discussion 
about tools), is not to avoid questions like: <i>why is asynchronous programming better that multithreading or multiprocessing in the present case?</i>, 
and ultimately of <i>what are asyncio, multithreading and multiprocessing about?</i> While I acknowledge the fact that this is not the classical approach, in my own view it might eliminate some 
false opinions about the concerned tools, giving a practical context to asynchronous programming, which I believe to be far from obvious.
</p>

<p style="text-align: justify">However, before introducing the tools that will be used, I would like to give a sense of the utility of what has been developed. </p>

<h2> Why this is a boring interesting project</h2>

<p style="text-align: justify">
A "sitemap" is the representation of the structure of a website: a list of unique links starting from a given page (usually the homepage) and proceeding to increasing depths within the structure of the website.
The depth of a sitemap can be chosen 
arbitrarily, and ideally should include all of the pages of a website. 
However, a sitemap is also a tool useful for website optimization, and therefore it should include primarily the pages that are relevant 
to the issue at hand. For instance, in a shopping website we may be interested in how difficult it is for the user to get from the homepage to the cart, 
or whether it is more difficult to reach man-related products than woman-related ones: in
these cases the depth of those pages with respect to the homepage may be an information of crucial value.
</p>

<p style="text-align: justify">
A good sitemap is also a guide for browsers through what is known as <i>priority</i>: a number that helps indexing the pages depending on the importance given to them by the websmaster. Other useful indications may include the date
of last modification, and the number of visits that a certain page has received. 
</p>

<p style="text-align: justify">
Although it may be an element of particular importance, websites may or may not have sitemaps. If the sitemap is not available, and the information is needed,
scraping is the way to go. However, if the sitemap is available, it should be downloaded from the website since it surely lists what has been deemed important by
the webmaster. 
</p>

<p style="text-align: justify">
From a programming perspective, building a sitemap is a project with its own share of fascination and may be accomplished at different levels of expertise: a simple program that collects 
links from webpages in a synchronous way gets the job done, but it is not scalable, it does not deal with requests overload, it does not question the limits and advantages of the tools
used, and it does not deal with the minimal formalism to present the results.
</p>

<p style="text-align: justify">
Nowadays, nobody has to build a sitemap generator, or even a custom scraper: this is fun and instructive, but frameworks like <i>scrapy</i> make the process immediately scalable and 
professional. Nonetheless, <i>scrapy</i> itself does not work with magic (even though it might look like so given the impressive performances), but using asynchronous programming. 
The latter will be the core element of the ensuing discussion.
</p>

<p style="text-align: justify">
At this point, and whenever scraping is involved, I would ask the reader to stop and consider the following question: <i>does it really make sense exploring blindly a whole website?</i> Of course, this is 
a rhetorical question, since if we knew where to search we would not need a sitemap. In practice, before designing a sraper of whathever type it is useful to check not only for existing sitemaps, but also for 
APIs (institutional websites generally have them.) Moreover, this is not a trivial question, since tools like <i>scrapy</i> have built-in methods that are so easy to use that might leave small space for a well thought decision.
</p>

<p style="text-align: justify">
Therefore, even if the project is not as exciting as making a news retriever with real time processing (under construction <a href="https://github.com/Gabriele-Donato/website-materials-/tree/DataEngineering">here</a> by the way),
this is the place to start to acquaint onelf with the libraries. Let's put our hands to work!
</p>

<h2>The structure of a Sitemap</h2>

<p style="text-align: justify">
As explained above, sitemaps are a serious thing and a protocol does exist: check <a href="https://www.sitemaps.org/protocol.html">sitemap.org</a>. The following table taken from the website summarizes
the main tag (note that even when the sitemap is present, most are optional):
</p>

<table>
	<tr>
		<th>Attribute</th>
		<th>Required</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>urlset</td>
		<td>yes</td>
		<td>Reference to protocol standard.</td>
	</tr>
	<tr>
		<td>url</td>
		<td>yes</td>
		<td>Parent tag for all the other urls.</td>
	</tr>
	<tr>
		<td>loc</td>
		<td>yes</td>
		<td>Url of the page (children).</td>
	</tr>
	<tr>
		<td>changefreq</td>
		<td>no</td>
		<td>Signals how frequently the page changes.</td>
	</tr>
	<tr>
		<td>priority</td>
		<td>no</td>
		<td>Tells the crawler which pages are considered more important. It is a number from 0.0 t0 1.0, default value: 0.5. Does not affect the page ordering on the website</td>
	</tr>
</table>

<p style="text-align: justify">
A sitemap may have at most 50,000 urls and be no more of 50MB, but it is possible to join multiple sitemaps by specifying an index through the <i>sitemapindex</i> tag.
Moreover, special characters should be escaped and the encoding should be UTF-8. Generally, sitemaps are stored in XML files, and they can be validated through
schemas (e.g. <a href="http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd">here</a>).
</p>

<p>
These problems become relevant when storing the scraped data, therefore will appear again only at the end.
</p>

<h2>Multithreading and multiprocessing</h2>

<p>
My experience with asynchronous programming has developed as a necessity to make more efficient a webscraper during one of my internships. In that case, however, I used multithreding.
For this reason, I would like to start with the clarification of some concepts.
</p>

<p>
Imagine the hypothetical scenario where a robot, called Worker, has to plough the land. When Worker has to accomplish a task, a human loads in it an sd card with the 
instructions. The sd about "agricultural-skills" had been loaded, therefore the land-ploughing activity can be seen as an instance of a corresponding
program (much like a Word window for writing can be seen as an instance of a the Word program.) An instance of a program is called a <b>process</b>.
</p>

<p>
A process is made up of different components. Ploughing is a complex activity: you have to check not to inavertently hit a stone, because you might break your tool and injure yourself;
you have to pay attention to the sourrandings (is the farmer walking by?); and to the work itself (are you maintaining a steady pace, buring the debris and weeds properly? are you
handling the tools in the correct way?) Similarly, a Word instance will have to check orthography and language as you write. These actvities and sub-activities that contribute to an 
instance of a program are called <b>threads</b>.
</p>

<p>
Multiprocessing refers to the <b>parallel</b> computation distributed among
different central processing units (CPUs) in an asymmetric or symmetric fashion. Asymmetric multiprocessing refers to an implementation that 
treats the CPUs as being in a relation of master-slave: one CPU (the master) assigns the tasks to the slaves. On the other hand, symmetric multiprocessing allows the CPUs to communicate through
a shared memory, so that each can self-schedule their tasks. Of course, designing a symmetric multiprocessing systemem is comparatively more 
challenging because one has to manage how the CPUs interact over the shared resources.
</p>

<p>
Multithreading, on the other hand, refers to the creation of a high number of threads within a process to increase its performance by <b>concurrency</b>. Indeed, ploughing is a complext task composed by other complex tasks: determining whether 
the tools are handled in the correct way requires access to both the robot's hand-sensors and to his sight-sensors. Notably, the threads should participate to the same space in the memory:
there would be no point in coordinating eye and hand if the coordinating mechanism is unable to know <i>at the same time</i>  what are the inputs from the two sensors.
</p>

<p>In plainer terms, a multiprocessing robot is one that can plough and sing; a multithreaded robot is one that can use intelligently the time when the hoe is not being used to plough the land, to check where is the 
best spot to land it. More in general, multiprocessing is used with CPU bound tasks, while multithreading is used with input/output (I/O) tasks.
While multiprocessing operates at the hardware level and manages to achieve real <i>parallelism</i> (different task on different CPUs), multithreading
operates at the software level and simulates parallelism by exploiting the waiting times between tasks to run the operations <i>concurrently</i>.  While symmetric multiprocessing involves a shared memory space, 
multithreading involves a shared memory <i>within</i> the process (threads share memory, code and file <i>of</i>the process).
</p>

<p>
In the context of web-scraping there is no point in using an increased computing power (multiprocessing), since the final performance only depends 
from the speed of the websites to load their contents, our bandwidth, and from how we process and store the results.
</p>

<p>
The inadequacies of the multithreading approach are less obvious: indeed, it succeeds in optimising the task concerned. The problem lies precisely in its being <i>multi-</i>functional: in the case 
of scraping, multithreading requires the initialisation of multiple webdrivers (or workers) performing the same scraping task on different pages of a website and on different threads. This implies that 
it is necessary to coordinate the webdrivers so that each one has a different page to scrape (to avoid blocking each other), and that each one has a similar share of work to accomplish (so that no one sits idle
in the process.) This management of work and resources takes away time from the scraping activity. It is also possible to agree on the fact that multithreading, however more efficient than multiprocessing, should be 
less efficient of an equivalent program that performs the same task in a single thread: this is what asynchronous programming is about.
</p>

<p>
A widespread idea about multithreading in Python is that its <i>threading</i> module does not support multithreading because of the Global Interpreter Lock (GIL.) This is a false statement: what the GIL 
does is just preventing parallel execution of multiple threads in the same process, exactly as explained above. When executing I/O tasks the lock gets released, therefore the program written using the library 
is indeed a multithreaded program, it suffices be mindful of the operations that are being carried out. It is possible to bypass the GIL using the <i>multiprocessing</i> module, or building extensions in C. 
</p>

## How is asynchronicity achieved?

This is an interesting topic that I plan to explain more deeply very soon in this paragraph. 

<h2>The project</h2>

<p>
This project is about creating a sitemap generator able to take as input a webpage and return an XML file including the links up to the _nth_ page.
In this project I will be using no more than these libraries:
</p>

<pre><code>
import aiohttp 
import asyncio 
from bs4 import BeautifulSoup 
import xml.etree.ElementTree as ET
</code></pre>

<p>
Considering how many times _requests_ and _BeautifulSoup_ are imported together, the absence of _requests_ might be striking. 
The problem with this library is that it is designed to deal with _synchronous_ requests to websites: it sends a 
request, waits for the response, and then sends another request. On the other hand, _aiohttp_ allows for _non-blocking_ calls: the fact that a request has been sent 
does not prevent other requests from being 
forwarded if no response is available.
</p>
<p>
While I acknowledge that presenting the code without structuring it in classes might enhance its readability, I would like to present a general method that has allowed me to
advance more steadily in coding, and that I cannot but consider the correct and cleanest way to do it.
</p>
<p>
I will give for granted that the reader knows the syntax of asyncio. YouTube is full of valuable materials, I list some that I found useful in the references section.
</p>

<h2>A worker called SitemapGenerator</h2>
<p>
I would like my scraper to take an input, a webpage, and store it into a file (for the moment I want to name the file a priori, but I might as well extract the name of the website from the webpage.)
Moreover,I would like to be able to control the maximum reachable depth: my webpage is level 0, the links I can reach directly from there are level 1, the links I can reach from level 1 are level 2 with respect to the starting page etc...
It might also be useful to be able to choose how many task my scraper can execute concurrently at any given time (to avoid getting blocked by the website when I send too many requests.)
</p>
<p>
Apart from these general parameters, it might be useful to endow my worker (I use the singular even if I am talking about concurrency because we are in a single thread!) with some pieces of 
useful knowlege. In particular, the only necessary thing is what is the hostname of the starting page (I don't want to end up scraping the whole internet
so it is necessary to check every url.)
</p>
<p>
Moreover, I might want to set a general attribute indicating what is the session that I am using. The <a href="https://docs.aiohttp.org/en/stable/client_reference.html">official documentation</a> recommends using only one session unless 
the scraper connects to a considerable number of servers during its lifetime. Sessions are available also for _requests_ (altough barely used in introductory coding exercises), and are a valuable tool for increasing the 
efficiency of the code through, for instance, connection pooling (if more requests are made to the same server, existing requests are reused instead of creating new ones), and default retry mechanisms. 
</p>
<p>
All of the above results can be embedded in a worker called SitemapGenerator (for the moment the session is simply initialised to None.)
</p>

<pre><code>
class SitemapGenerator:
    def __init__(self, root, filename, max_tasks=10, max_depth=3):
       self.filename = filename
       self.urls = {}
       self.root = root
       self.hostname = urlparse(root).hostname
       self.semaphore = asyncio.Semaphore(max_tasks)
       self.max_depth = max_depth
       self.session = None
</code></pre>

<h2>Defining Plow and Hoe</h2>

<p>
One lesson that I have learned from scraping is that inefficiency can be more productive than efficiency sometimes. 
A webscraper that retrieves 100k links in three seconds, but leaves you hopless and banned from the scraped resource, is 
two-times worse than a scraper that takes a doubled time without further complications. Twice worse because: 1) it does not 
take into account the problem, and therefore it is poorly designed, and 2) because it prevents you to correct your mistake and try again.
</p>
<p>
Therefore, I will tell my scraper not to execute more than _n_ concurrent task, and to wait no more than 10 seconds before concludind that a page cannon be reached.
These two settings potentially slow the program, but make it also more adaptive to difficult websites.
</p>
<p>
The response.status is just a number that indicates whether the page could be reached. It is important to print it in case the page was not available because it may serve to 
understand the nature of the problem. There are various codes: 200 is "successful response." To know more about response codes visit <a href="https://it.wikipedia.org/wiki/Codici_di_stato_HTTP">here</a>.
</p>

<pre><code>
    async def fetch(self, url):
        async with self.semaphore:
            try:
                response = await self.session.get(url, timeout=10)
                if response.status == 200:
                    content = await response.text()
                    return content
                else:
                    print(f"Failed to fetch {url} with status {response.status}")
            except Exception as e:
                print(f"Error fetching {url}: {e}")
            return None
</code></pre>


<p>
Now that the scraper knows how to behave when fetching a page, it should be instructed on what pages to fetch out of all the pages on the internet. This is not enough, it should be able, given an url, maybe 
written in an incorrect or incomplete way, to use its knowledge of the hostname from which it started to reconstruct the correct one.
</p>

<p>
A beautiful library for working with urls is <i>urlparse</i>, able to recognise and intelligently join url parts. In particular, the "normalization" involved in the below function refers only to the last 
<i>elif</i> where the "https" is added in order to make the url valid.
</p>

<pre><code>
    def normalize_url(self, href, current_url):
        if not href:
            return None
        parsed_href = urlparse(href)
        if parsed_href.scheme and parsed_href.netloc:
            return href if parsed_href.netloc == self.hostname else None
        elif href.startswith("//"):
            return f"https:{href}" if self.hostname in href else None
        return urljoin(current_url, href)
</code></pre>


<h2>Defining the crawler</h2>

<p>
The next chunk of code constitutes the core of the scraper and it features for the first time the maybe strange <i>async def</i> syntax. It should be noted that the code is made up of two functions: 1) <i>crawl</i>, that defines the 
extraction operations and, 2) <i>run</i>, that initialises an aiohttp session, runs the scraper and calls another function to save the output in an xml file. The saving part will be dealt with at last.
</p>

<p>
It should be noted that the <i>crawl</i> function is recursive: it is called in the last <i>if</i> condition within itself. It works as follows: given a level and a starting url it checks whether the maximum depth has been exceeded by the current level and
whether the url has already been scraped, in these cases the function is not executed. If the url has not been scraped, the page content is retrieved through the <i>fetch</i> function, which returns the html content of the page if
the request is correct (status = 200). The content is parsed with BeautifulSoup and all the links of the page are extracted and saved to the <i>self.urls</i> dictionary (incidentally, they are defragmented before saving.) Then for each link
a series of asynchronous tasks is initialised and awaited, the content of these tasks is nothing but the same <i>crawl</i> function, taking as an argument the scraped link and the level of the page on which the link was found augmented by a unit 
(this allows to categorise the starting link as zero, the links found on that page one, the links on page one as two etc... More pragmatically this means that the links on the starting page need only one click to be reached if one starts at that page and so on).
The taks are gathered and awaited (this is a shortcut to having to await the tasks one by one.)
</p>

<p>
On the other hand, the <i>run</i> function initialises a <i>ClientSession</i> and starts the logic explained above. The session element is functional to an efficient use of the resources and levels up the scraping technology: it 
is endowed with keepalive mechanisms that ensure stable connection between client and server, and manages requests intelligently avoiding to send multiple times the same request.
</p>

<pre><code>
   async def crawl(self, url, level):
        if level > self.max_depth:
            return
        if url in self.urls:
            return
        print(f"Level: {level} / Explore {url}")
        content = await self.fetch(url)
        if content:
            url = urlunparse(urlparse(url)._replace(fragment=''))
            self.urls[url] = level
            soup = BeautifulSoup(content, "html.parser")
            tasks = []
            for link in soup.find_all('a', href=True):
                href = self.normalize_url(link.get('href'), url)
                if href and href not in self.urls:
                    tasks.append(self.crawl(href, level + 1))
            await asyncio.gather(*tasks)

    async def run(self):
        async with aiohttp.ClientSession() as self.session:
            await self.crawl(self.root, 0)
            self.generate_file()
</pre></code>

<h2>Storing the links</h2>

<p>
As explained above, there exist a general codification for storing sitemaps in XML files. These files can be read by pandas in the same way as a .csv or .xlsx sheet. Therefore all the urls are stored as children of the <i>urlset</i>
tag within a <i>loc</i> tag inside of a <i>url</i> tag. The latter also contains the priority measure rounded to two decimals and decreasing with the level (this is optional and just to show how it works.)
</p>

<pre><code>
  def generate_file(self):
        urlsbylevel = {}
        for url, level in self.urls.items():
            if level not in urlsbylevel:
                urlsbylevel[level] = []
            urlsbylevel[level].append(url)

        maxlevel = max(urlsbylevel.keys(), default=0)
        step = 1 / (maxlevel * 2 if maxlevel > 0 else 1)
        root = ET.Element('urlset', xmlns='http://www.sitemaps.org/schemas/sitemap/0.9')

        for level, urls in sorted(urlsbylevel.items()):
            priority = round(1 - step * level, 2)
            for url in urls:
                url_element = ET.SubElement(root, "url")
                ET.SubElement(url_element, "loc").text = url
                ET.SubElement(url_element, "priority").text = str(priority)

        tree = ET.ElementTree(root)
        tree.write(self.filename, encoding="utf-8", xml_declaration=True)
        print(f"Sitemap saved to '{self.filename}'.")
</pre></code>




