---
layout: post
title: 'How to re-crawl with Nutch'
---

Nutch allows to crawl a site or a collection of sites. If your objective is to simply crawl the content once, it is fairly easy. But if you want to continuously monitor a site and crawl updates, it can be harder. Harder because the Nutch documentation does not have many details about that.

After a bit of digging, I found that Nutch offers an Adaptive Fetch Schedule class that can be used for that purpose. To understand how this class works, let's recap how Nutch manage crawl.

Nutch maintains a record on file of all the urls that it has encountered while crawling. This record is called the crawl db. Initially, the crawl db is build from a list of urls provided by the user using the inject command. An important concept in Nutch is the generate/fetch/update process. The generate command looks up in the crawl db for all the urls due for fetch and regroup them in a segment. An url is due for fetch if it is either a new url or if it is time to re-crawl it. More on that later. The fetch command will, well, fetch on the web all the urls of the segment. After that, the update command will add the results of the crawling (stored in the segment) into the crawl db. Each url crawled will be updated to indicate the fetch time and the next scheduled fetch. New urls discovered will also be added and marked as not fetched.

By default, Nutch will set the next scheduled fetch of a page to be the fetch time + a constant interval. The default value is 30 days, but it can be changed in the file nutch-site.xml via the <b>db.fetch.interval.default</b> property to whatever value. On a later generate call, if the time has come, the url will be added to a segment and re-crawled. This default behavior can be acceptable if roughly all pages of a site change at approximately the same rhythm. But if the site being crawled contains a lot of pages that almost never change, you would probably want Nutch to visit these pages less often and concentrate on the one that changes frequently. But it is not possible to do that with the default fetch schedule that uses the same constant interval for each url.

Enter the Adaptive Fetch schedule. This fetch schedule will adapt to the rhythm of changes of a page and set the next schedule time accordingly. When a new url is added to the crawl db, it is initially set to be re-fetched at the default interval. The next time the page is visited, the Adaptive Fetch schedule will increase the interval before the next fetch if the page has not changed and decreased it if the page has changed. Note that a maximum and a minimum interval is defined in the configuration. The interval will never be longer than that maximum or smaller than the minimum. So after a while, the pages that changes often will tend to be visited more than the one that does not.
<table border="1">
<tbody>
<tr>
<td><strong>db.fetch.schedule.class</strong></td>
<td>The implementation of fetch schedule</td>
</tr>
<tr>
<td><strong>db.fetch.interval.default</strong></td>
<td>The default number of seconds between re-fetches of a page</td>
</tr>
<tr>
<td><strong>db.fetch.schedule.adaptive.min_interval</strong></td>
<td>The min number of seconds between re-fetches of a page</td>
</tr>
<tr>
<td><strong>db.fetch.schedule.adaptive.max_interval</strong></td>
<td>The max number of seconds between re-fetches of a page</td>
</tr>
<tr>
<td><strong>db.fetch.schedule.adaptive.inc_rate</strong></td>
<td>If a page is unmodified, the interval before the next fetch will be increased by this rate</td>
</tr>
<tr>
<td><strong>db.fetch.schedule.adaptive.dec_rate</strong></td>
<td>If a page is modified, the interval before the next fetch will be decreased by this rate</td>
</tr>
<tr>
<td><strong>db.fetch.schedule.adaptive.sync_delta</strong></td>
<td>If true, try to synchronize with the time of page change by shifting the next fetchTime by a fraction (sync_rate) of the difference between the last modification time, and the last fetch time</td>
</tr>
</tbody>
</table>
If a page was modified, the Adaptive Fetch schedule will store the last fetch time as the last modification time. Nutch will use that information in the If-Modified-Since header of the http request of the next fetch. If the web server supports this and the page has not changed since, it will only returns a 304 code. Note that there is a bug in Nutch 1.0 that prevents this to work properly. I have <a href="https://issues.apache.org/jira/browse/NUTCH-815" target="_self">reported the bug</a> and it will be fixed for Nutch 1.1. You can use the trunk in the meantime.

How does Nutch can detect if a page has changed or not? Each time a page is fetched, Nutch computes a signature for the page. At the next fetch, if the signature is the same (or if a 304 is returned by the web server because of the If-Modified-Since header), Nutch can tell if the page was modified or not. By default the signature of a page is built not only with its content, but also with the http headers returned with the page. So even if the content of a page has not changed, if an http header is not the same (like an etag or a date), the signature changes. To solve that problem, there is the TextProfileSignature class. It is designed to look only at the text content of a page to build the signature. To use it, you need to set the <b>db.signature.class</b> property to <b>org.apache.nutch.crawl.TextProfileSignature</b>.

A word about the setting <strong>db.fetch.schedule.adaptive.sync_delta</strong>. I set it to false for my crawls because I have not been able to really understand what it is good for. As I described earlier, the next fetch time is computed by adding a dynamic interval to the last featch time. But with this setting set to true, the interval is applied to a reference time which is a time located between the last fetch time and the last modification time. If someone can enlighten me about the usefulness of this, please do!
