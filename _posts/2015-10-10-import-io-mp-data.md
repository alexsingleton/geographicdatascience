---
author: Hai Nguyen
layout: post
avatar: hainguyen
title:  import.io - a Swiss Army Knife for Webdata Extraction
comments: true
categories:
- Data Science
- cdrc
---

##Data extraction from webpages
  
Data extraction is an important part of the daily task list of a data scientist. It is very rare (and almost impossible) that we get data in a format suitable for analysis. Many datasets can be available as websites (HTML documents), however, it can take much time and effort to crawl, parse, and transform these HTML documents into useful raw data. There are many HTML parsers available for different programming languages/platforms: HTML Parser (Java), jsoup (Java),  Beautiful Soup (Python), XML (R), etc., yet implementing a parser to extract correct data remains a non-trivial and time-consuming job.

##Data extraction from webpages with _import.io_

_import.io_ was designed to save data scientists' time from writing boring/complex HTML parsers scratch.  This is a web-based/Desktop application that allows you to interactively select which features (HTML/link/text/image) to extract from a webpage and export them as either an IPA (JSON or XML ). Below is a short tutorial on how to extract web data using _import.io_:

<iframe width="560" height="315" src="https://www.youtube.com/embed/pMhj4xk5B-Q" frameborder="0" allowfullscreen></iframe>

##An example - Extracting MP's Social Media accounts and emails
Last week I had to extract MPs' social media account and email for each Local Authority District (LAD). There used to be an API to extract such data (http://www.programmableweb.com/api/find-your-mp-uk-parliament). Unfortunately this API is no longer available and the API link is now redirected to the MPs webpage (http://www.parliament.uk/mps-lords-and-offices/mps/). This webpage contains a list of MPs, with their name, party and constituency.  Click on an MP name, you can go to the MP individual profile where the social media information can be extracted.

#### MPProfile Extractor:

Using _import.io_ desktop app, it is quite straight-forward to create an extractor (namely **MPProfile Extractor**) to look for and extract the social media box (the red box in the image below) for each individual MP:

![example-mp-page](/public/images/example-importio1.png)

Since we are looking for hyperlinks such as "twitter.com/..." or "mailto:abc@gmail.com", we can specify the datatype to be extracted (e.g., as links or images). By doing this, the extractor will filter out non-link data such as the icons or text (e.g., "Website:", "Twitter:") which then make our data cleaner and neater.  Below is the returned data in JSON-format (the output format can be JSON, CSV, or a table for readability):

```javascript
{
    "title": "Ms Diane Abbott MP - UK Parliament",
    "results": [
        {
            "address_as": "Ms Abbott",
            "constituency": "Hackney North and Stoke Newington",
            "emailaddress/_text": "sheyda.m.azar@parliament.uk",
            "socialmedia": [
                "http://www.dianeabbott.org.uk",
                "https://twitter.com/HackneyAbbott"
            ],
            "emailaddress": "mailto:sheyda.m.azar@parliament.uk",
            "socialmedia/_text": [
                "www.dianeabbott.org.uk...",
                "@hackneyabbott"
            ],
            "party": "Labour"
        }
    ],
    "pageUrl": "http://www.parliament.uk/biographies/commons/ms-diane-abbott/172"
}
```

Note that this is the result for only one MP. In this case, the profile of this MP is available on http://www.parliament.uk/biographies/commons/ms-diane-abbott/172 and we have used this URL as the query parameter. For another MP, we just need to change the query parameter to another URL. For example, "http://www.parliament.uk/biographies/commons/debbie-abrahams/4212" returns results for "Debbie Abrahams"

```javascript
"offset": 0,
    "results": [
        {
            "address_as": "Debbie Abrahams",
            "constituency": "Oldham East and Saddleworth",
            "emailaddress/_text": "abrahamsd@parliament.uk",
            "socialmedia": [
                "http://www.debbieabrahams.org.uk/",
                "https://twitter.com/Debbie_abrahams"
            ],
            "emailaddress": "mailto:abrahamsd@parliament.uk",
            "socialmedia/_text": [
                "www.debbieabrahams.org...",
                "@debbie_abrahams"
            ],
            "party": "Labour"
        }
    ],"
```

That's it for a single MP profile. Unluckily we have in total 650 of them. Would it make sense to do this for each of the 650 MPs? Assuming that you can extract data for 1 MP in a minute (running the query with Extractor 1 and save the result in some place), this would take you more than a day of work.

### MPlist Extractor:
_import.io_ has this great facility called "chain API". Recall that we have a list of MPs at http://www.parliament.uk/mps-lords-and-offices/mps (see the image below). The idea is to use the urls of all MPs from this page to feed **MPProfile Extractor**.

![mp-list](/public/images/example-importio2.png)

Again, I used the _import.io_ Desktop app to create this extractor (**MPlist Extractor**).  The result is (in JSON format for readability) a list of all MPs:
```javascript
"results": [
        {
            "LINK/_text": "Abbott, Diane",
            "constituency": "Hackney North and Stoke Newington",
            "LINK": "http://www.parliament.uk/biographies/commons/ms-diane-abbott/172",
            "party": "Labour"
        },
        {
            "LINK/_text": "Abrahams, Debbie",
            "constituency": "Oldham East and Saddleworth",
            "LINK": "http://www.parliament.uk/biographies/commons/debbie-abrahams/4212",
            "party": "Labour"
        },
        {
            "LINK/_text": "Adams, Nigel",
            "constituency": "Selby and Ainsty",
            "LINK": "http://www.parliament.uk/biographies/commons/nigel-adams/4057",
            "party": "Conservative"
        },
        {
            "LINK/_text": "Afriyie, Adam",
            "constituency": "Windsor",
            "LINK": "http://www.parliament.uk/biographies/commons/adam-afriyie/1586",
            "party": "Conservative"
        },
        {
            "LINK/_text": "Ahmed-Sheikh, Tasmina",
            "constituency": "Ochil and South Perthshire",
            "LINK": "http://www.parliament.uk/biographies/commons/ms-tasmina-ahmed-sheikh/4427",
            "party": "Scottish National Party"
        },
        {
            "LINK/_text": "Aldous, Peter",
            "constituency": "Waveney",
            "LINK": "http://www.parliament.uk/biographies/commons/peter-aldous/4069",
            "party": "Conservative"
        },
        {
            "LINK/_text": "Alexander, Heidi",
            "constituency": "Lewisham East",
            "LINK": "http://www.parliament.uk/biographies/commons/heidi-alexander/4038",
            "party": "Labour"
        },
        {
            "LINK/_text": "Ali, Rushanara",
            "constituency": "Bethnal Green and Bow",
            "LINK": "http://www.parliament.uk/biographies/commons/rushanara-ali/4138",
            "party": "Labour"
        },
        ...
```

We can then go back to the **MPprofile Extractor** to specify how we can use this extractor (or an API in _import.io_'s terms) by selecting "Use URLs from another API" (see below).
![api-chain](/public/images/example-importio3.png?dl=0)

That's all done. _import.io_ can automatically extract all MPs' profile page using **MPlist Extractor** and send this list of URLs to **MPprofile Extractor**, which will extract all links  from the Social Media box in each MP profile. Without _import.io_, this task would easily take you a day of work, given that you are already a coder (or hacker/programmer/developer, whatever works), which is not always the case for data scientists. Now with _import.io_ and its powerful yet user-friendly desktop app, you can do this within an hour (or less). 
