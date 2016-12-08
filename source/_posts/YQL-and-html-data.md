---
title: YQL and Html data
date: 2016-07-15 13:11:08
description: "This is a project for a local church organization for fetching their event data onto a big display monitor. We are using yahoo's YQL queries to get xml data that is then converted into a JSON array string. This is an app that runs on the browser."
categories:
  - clients
tags:
  - YQL
  - Html
  - coding
---
---
# Table of Contents
1. [Part one: Coding](#Part-one-coding)
2. [The Project: Church events onto a display board](#The-Project-Church-events-onto-a-display-board)
3. [Fetching external html data](#Fetching-external-html-data)
4. [Client side solution](#Client-side-solution)
  1. [Parsing data](#Parsing-data)
  2. [Filtering data](#Filtering-data)
  3. [Refreshing the page](#Refreshing-the-page)
5. [Other possible solutions](#Other-possible-solutions)
6. [Conclusion & important links](#Conclusion-and-Important-links)

# Part one: Coding

This will be in two parts piece with this being the first about the coding aspect of the project. I wanted to start with the coding because the logic was the most important implementation for this customer. The design aspect of it will be discussed on the second posting that will be published sometime at the end of august. But as of now you can already check out the github repository for this project: [here](https://github.com/laurames/karkkilanSeurakunta)

## The Project: Church events onto a display board

I got a request from my local church community about building a script that would fetch event data from their website and display this on a a big TV screen. This was to promote the church events to passerby and anyone who saw the display. Because of the requirements i knew 1) i needed to fetch html data, 2) to parse it and get the data i wanted and 3) to clean it up if needed. And because of the [same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy) there had to be either a service in-between or i had to write my own back-end script to do this. The reason for this is that html data of this kind can not be fetched from front-end to another front-end if they are not from the same origin (URI scheme, hostname or port number). And yes, they where not going to be under the same origin. I didn't have access to the church webpage and i wasn't going to build the application onto their server because i wanted to make this as simple as possible.

#### What i knew of the requirements:
1. Data had to be gotten from their website event data **=** A internet connection
2. This fetched and parsed data did not have to be accessible to anyone **=** Not required to be hosted
3. The information had to be displayed onto a big monitor or TV **=** laptop or any device connected to a TV **->** no need to make anything complicated

Because of these known requirements i knew that the simplest way was to make a script that runs on a browser and locally on a machine. It should use an external service to fetch html data that can then be parsed and be able to refresh itself to keep the page up to date.

#### My implementation:
1. HTML, CSS and javaScript/jQuery
2. YQL service to fetch XML data from external site
3. HTML's own refresh meta-tag

## Fetching external html data

I have to be honest i was quite naive in the beginning thinking that i could just use jQuery's wonderful AJAX methods to fetch me some event data :P Ha isn't so easy! Came to the same [Same-origin policy](#Same-origin_policy) problem as mentioned above. And as mentioned in a wonderful [answer](http://stackoverflow.com/questions/15005500/loading-cross-domain-html-page-with-ajax) on stackoverflow it was simply said that "due to browser security restrictions, most Ajax requests are subject to the same origin policy; the request can not successfully retrieve data from a different domain, subdomain, port, or protocol." I also loved the answers detail on way to overcome this same policy restrictions and as i read i went with YQL because even though it is a third party provider and "using third-party proxies is not a secure practice because they can keep track of your data", but because i am never the less playing with public information it wasn't a concern to me.

I thought about a lot of writing my own back-end script for fetching data from the external [church site](http://www.karkkilanseurakunta.fi/tapahtumat) using php or python, but this became a very complicated way of doing things when i found out about [Yahoo's YQL service](https://developer.yahoo.com/yql/guide/). Also by reading more about cross domain requests and searched hits on stackoverflow i found answers to questions like ['Can Javascript read the source of any web page?'](http://stackoverflow.com/questions/680562/can-javascript-read-the-source-of-any-web-page) and [Loading cross domain html page with AJAX](http://stackoverflow.com/questions/15005500/loading-cross-domain-html-page-with-ajax). Here I came to decide to use YQL.

## Client side solution

I haven't done much SQL so YQL was just as new to me, but the query language is apparently very close to SQL. I went on to reading the documentation, but i also found a former employee of Yahoo's (Christian Heilmann) blog post on the subject [here](https://www.christianheilmann.com/2010/01/10/loading-external-content-with-ajax-using-jquery-and-yql/).

My client side solution was to run a javascript calculating the necessary dates (monday and sunday of the week) for the query string.
``` javaScript
//The current dates for today:
function todaysDates(){
  var today = new Date();
  var dayOfTheWeek = today.getDay();
  var day = today.getDate();
  var month = today.getMonth()+1;
  var year = today.getFullYear();
  $(".year").append(year);
  weekForQuery(today, dayOfTheWeek, day, month, year);
}
//function for getting the monday and sunday in query format:
function weekForQuery(today, dayOfTheWeek, day, month, year){
  var dayOffset;
  var daysToOffsetBy = function(offsetBy){
    var newToday = new Date();
    var offset = newToday.setDate(newToday.getDate() + offsetBy);
    return newToday;
  }
  if(dayOfTheWeek === 1){
    dayOffset = daysToOffsetBy(7);
    startOfWeek = day + '%2F' + month + '%2F' + year;
    endOfWeek = dayOffset.getDate() + '%2F' + (dayOffset.getMonth()+1) + '%2F' + dayOffset.getFullYear();
  }else if(dayOfTheWeek === 0){
    dayOffset = daysToOffsetBy(-7);
    startOfWeek = dayOffset.getDate() + '%2F' + (dayOffset.getMonth()+1) + '%2F' + dayOffset.getFullYear();
    endOfWeek = day + '%2F' + month + '%2F' + year;
  }else{
    var fromMonday = 1-dayOfTheWeek; //negative number
    var fromSunday = 7-dayOfTheWeek; //positive number
    dayOffset = daysToOffsetBy(fromMonday);
    var dayOffsetSunday = daysToOffsetBy(fromSunday);
    startOfWeek = dayOffset.getDate() + '%2F' + (dayOffset.getMonth()+1) + '%2F' + dayOffset.getFullYear();
    endOfWeek = dayOffsetSunday.getDate() + '%2F' + (dayOffsetSunday.getMonth()+1) + '%2F' + dayOffsetSunday.getFullYear();
  }
}
```

### Parsing data

I knew that the only div's i needed where the one's with class name **event-list-wrapper** that contained within it **event-item-list** and in that there was the classes for **event-date**, **event-time** and **event-location**.

Yahoo YQL provided a testing tool for queries so i ran a lot of tries to minimize the returned XML. What i came up with in the end looked something like this:

```
select * from html where url="www.karkkilanseurakunta.fi/tapahtumat/-/haku/0/11/7/2016/_/17/7/2016/week/1#events" and xpath='//div[@class="event-list-wrapper"]/div[contains(@class, "event-item-list")]'
```

The testing tool is [here](https://developer.yahoo.com/yql/console/).

After i have the correct Monday and Sunday of this week i built the query string with the correct format:
``` javaScript
function buildQueryString(startOfWeek, endOfWeek){
  //the query:
  var requestQuery = "https://query.yahooapis.com/v1/public/yql?q=select%20*%20from%20html%20where%20url%3D%22www.karkkilanseurakunta.fi%2Ftapahtumat%2F-%2Fhaku%2F0%2F" + startOfWeek + "%2F_%2F" + endOfWeek + "%2Fweek%2F1%23events%22%20and%20xpath%3D'%2F%2Fdiv%5B%40class%3D%22event-list-wrapper%22%5D%2Fdiv%5Bcontains(%40class%2C%20%22event-item-list%22)%5D'&format=xml&callback=?"
  console.log(requestQuery);
  doAjax(requestQuery);
}
```

And then do the necessary request to the service:
``` javaScript
function doAjax(requestQuery){
  var resultData;
  var dataLenght;

  if(requestQuery.match('^http')){
    $.getJSON(requestQuery, function(data){
      resultData = data;
      dataLenght = resultData.results.length;
      if(dataLenght != 0){
        for(i=0; i<dataLenght; i++){
          var eventData = filterData(resultData.results[i]);
          $('#events').append(eventData);
        }
      }else {
        var errormsg = '<p class="error">Error: could not load the page.</p>';
        $('#events').append(errormsg);
      }
    });
  }else {
    console.log("check the query request");
  }
}
```

### Filtering data

Lastly I filter the JSON string as required:
``` javaScript
function filterData(data){
  //filter away the sharing links and images
  data = data.replace(/<div class="share"[^>]*>[\S\s]*?<\/div>/,'');
  //get the index of the images that don't have their root path
  var indexOfImage = data.indexOf('/seurakunta-theme');
  var rootUrl = "http://www.karkkilanseurakunta.fi";
  //place the root path into the string so that we have the images to display
  var output = [data.slice(0, indexOfImage), rootUrl, data.slice(indexOfImage)].join('');
  return output;
}
```

### Refreshing the page

The page has to be kept up-to date with the latest event news, but it doesn't require refreshing more then once or twice a day. So at first thought about using a javascript for this to calculate the amount of time a page has been inactive and then just reload it, but then found out that there is a very nice meta-tag for this too! Here is the [page](http://stackoverflow.com/questions/4644027/how-to-automatically-reload-a-page-after-a-given-period-of-inactivity) where i found out about this.

```
<meta http-equiv="refresh" content="5" >
//where content ="5" are the seconds that the page will wait until refreshed
```

## Other possible solutions

As explained in the [answer](http://stackoverflow.com/questions/15005500/loading-cross-domain-html-page-with-ajax) on stockoverflow.

> There are some ways to overcome the cross-domain barrier:
> [CORS Proxy Alternatives](http://alternativeto.net/software/cors-proxy/)
> [Ways to circumvent the same-origin policy](http://stackoverflow.com/questions/3076414/ways-to-circumvent-the-same-origin-policy)
> [Breaking The Cross Domain Barrier](http://www.slideshare.net/SlexAxton/breaking-the-cross-domain-barrier)
> There are some plugins that help with cross-domain requests:
> [Cross Domain AJAX Request with YQL and jQuery](http://code.tutsplus.com/tutorials/quick-tip-cross-domain-ajax-request-with-yql-and-jquery--net-10225)
> [Cross-domain requests with jQuery.ajax](http://james.padolsey.com/javascript/cross-domain-requests-with-jquery/)

He wonderfully noted also the bast hand duty way of doing things:

> The best way to overcome this problem, is by creating your own proxy in the back-end, so that your proxy will point to the services in other domains, because in the back-end not exists the same origin policy restriction.

## Conclusion and Important links

The script works and it's very simple, but the design of the event display has to still be negotiated with the client. I will be doing my next post about that publishing it at the end of August.

The important and relevant links:
- [Github: Karkkila church event finder](https://github.com/laurames/karkkilanSeurakunta)
- [YQL documentation](https://developer.yahoo.com/yql/guide/)
- [YQL tester tool](https://developer.yahoo.com/yql/console/)
- [jQuery documentation](https://api.jquery.com/)
- [Christian Heilmann's blog post](https://www.christianheilmann.com/2010/01/10/loading-external-content-with-ajax-using-jquery-and-yql/)
- [Stackoverflow: Loading cross domain html page with AJAX](http://stackoverflow.com/questions/15005500/loading-cross-domain-html-page-with-ajax)
