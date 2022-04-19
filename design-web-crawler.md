# System Design - Web Crawler

SET UP 2022/04/12 UPDATE 2022/04/18

## Functional Requirements

Use Case? Search engine index - A crawler collects web pages to create a local index for search engines. Search engines then can download all the pages to create an index on them to perform faster searches, like Reverse Index Service and Document Service.

Number of Total? 15 billion pages per month.

Content type? HTML pages only. (Sound files, images and videos are not considered here)

Protocol - HTTP.

Storage retention? Store up to five years.

Duplication? Duplicate pages should be ignored & discarded.

## Non-Functional Requirements

Scalability - efficient for parallelization

Robustness

Politeness

Extensibility - new functionality could be added to it and there could be newer media types in the future.

## Estimation

15e9 / 30 / 86400 ~= 6000 pages/second

15e9 x (100 KB + 500 Bytes) = 1.5 TB

A web page has the average size of 100KB and 500 Bytes of metadata. => 1.5 TB per month and 90 PB for five years.

For 70% capacity model, we have 1.5 TB / 70% ~= 2.14 TB

## Steps

The basic algorithm executed by any Web crawler is to take a list of seed URLs as its input and repeatedly execute the following steps.

Pick a URL(Uniform Resource Locator) from the unvisited URL list.

Determine the IP Address of its host-name by using DNS.

Establish a connection to the host to download the corresponding document.

Parse the document contents.

Process the downloaded document, e.g., store it or index its contents, etc.

Extract URLs and add the new URLs to the list of unvisited URLs.

Go back to step one again.

## Infra Components Flow Chart

![image](https://roadtoarchitectcom.files.wordpress.com/2019/03/download.png)

[**Seed URLs**] -> [**URL Frontier**] -> [**HTML Downloader**] -> [**Content Parser**] -> [**Content Seen?**] ↓-> [**Content Storage**]

... -> [**Link Extractor**] -> [**URL Filter**] -> [**URL Seen?**] (loop back to **URL Frontier**)

Seed URLs - We can give some beginning URLs where we can start.

URL Frontier - FIFO queue, it also should prioritize which URLs remained to be crawled/downloaded. We can distribute our URL frontier into multiple servers and on each server we have multiple worker threads performing the crawling tasks.

HTML Downloader - it needs DNS resolver to translate URL to IP address.

Content Parser - parses HTML pages and checks/validates if it is malformed.

Content Seen - checks if a HTML page/document is already stored/crawled before. Most are stored on disk (Content Storage) and the popular ones are in memory. We can calculate a 64-bit checksum (like MD5 or SHA) of every processed document and store it in a database. For every new document, we can compare its checksum to all the previously calculated checksums to see the document has been seen before. Checksum storage: 15 billion distinct web pages * 8 bytes => 120 GB, which can fit into a modern-day server’s memory.

Link Extractor - extracts links from HTML pages. It can have the extensibility where we can add image or video Downloader module (different media types) and Web Monitor module.

URL Filter - filter out pages with noise or ads only or few content, even spam pages.

URL Seen - checks if a URL is already stored. The URL Storage keeps tracks of URLs already visited. Hash tables in-memory cache and URL disk storage are usually used. If this URL has not been seen/visited before, pass it to URL Frontier. We keep an in-memory cache of popular URLs on each host shared by all threads.

## Algorithm

BFS - Breadth first search

We have to deal with two problems

Large volume of web pages - Download the same URL in parallel (avoid sending too many requests to the same hosting server in a short period of time, avoid making distributed denial-of-service (DDoS) attack), issue of lack of the priority/freshness.

Rate of changes on web changes - By the time the crawler is downloading the last page from a site, the page may change.

### Distributed Crawling

My idea:

URL Frontier sends different URLs to different HTML Fetcher(Downloader) servers/machines by using a hash function (It can be consistent hashing: distributing URLs based on hostname). And each server can run multiple threads. Also there are Redis on each server maintaining the sorted sets and a queue. Also each worker thread will maintain its own separate sub-queue.

We want to make sure that at most one worker thread will download documents/content from a web server. And it will not overload the web servers.

Horizontal scaling - keep each servers/machines stateless.

### Timeout

We can set a timeout threshold (a maximal wait time) and keep an exponential back-off retry mechanism. After all failed we can stop and abandon that URL.

## Storage

The majority of URLs are stored in disk. We only maintain buffers in memory for recent queue/dequeue operations. The data in the buffer is periodically written to disk.

## Politeness & RobotsExclusion

Check **Robots.txt** file first which is called Robots Exclusion Protocol, before attempting to crawling a website.

## Follow Up

How to handle new updates in the already crawled URL? Compare the content and set a threshold (10% content diff).

DNS lookup can be a bottleneck and bandwidth should be kept enough to keep high throughput.

## Fault Tolerance

Since we are distributing URLs based on hostname to different hosts by using consistent hashing, we can store these data like URLs, checksum and content on the same host.

Each host will perform checkpoint periodically and dump a snapshot of all the data it is holding onto a remote server. If a server goes down, another server can replace it.

## Reference

[1] System Design Interview Book. Alex Xu. Chapter 9. Design A Web Crawler.

[2] <https://roadtoarchitect.com/2019/04/01/design-web-crawler/>

[3] [如何设计网络爬虫系统？How to Design Web Crawler - System Design EP3 花花酱](https://www.youtube.com/watch?v=_NyVaxEIYGo)
