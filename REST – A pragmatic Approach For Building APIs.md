“How can a computer thingy be RESTful?” That was the only thought that ran through my mind when I first heard of REST. Even more mind boggling was how it was always called in conjunction with APIs. How I fondly remember those days.

With this article, I will greatly describe REST along with a brief description of an API. Let’s get started.

## What is an API?

An API is easily defined as an Application Programming Interface. Yeah yeah, same old tech-y definition. So, what exactly is an API? The easiest way to think of an API is as anything that delivers a service. Imagine you are getting married. As often the case, you’re already weighed down by a lot of responsibilities: providing the best clothes for yourself and the train, ensuring that bills for things such as reception venue, bridal price and the likes are paid. You definitely are not going to be the same person to cook and serve the guests. These could be handled by others willing to render such services.

Similarly, if we are to create a small weather application that forecasts the weather conditions of hundreds of cities round the world, instead of traveling to every single one of those cities, every single day, with instruments for recording weather reports, we could just get the data from those who already have these reports in their libraries, while focusing on creating our app. That makes sense. We could pay them a token from money which otherwise, could have been used to pay for fares.

## What is REST?

I have always thought the English word “REST” to mean, as one dictionary defined: “A pause for relaxation”. However, I have realized that not all words containing those four letters R-E-S-T actually mean relaxation or a state of inactivity. Some are almost opposite, as in the case of the word “Restive”.

In our case, REST is an abbreviation for Representational State Transfer. It is a mindset or thought process developers follow when building an API. It is not an API.

## A brief history:

REST was brought to birth as a Dissertation or research propounded by Roy Fielding while gaining his Doctorate degree in the year 2000.

At that point in time, the internet was the Rave of the moment. It was just five years since the birth of JavaScript. Microsoft, Netscape and Yahoo were all having a good tussle as to who would gain web dominance.

Furthermore, with the growth of the internet arose a need for distributed servers, that is, computers in different locations, to communicate. The only available solution was a protocol which required developers to write complex, error-prone, hard-to-debug XML documents. That protocol was the Simple Object Access Protocol (SOAP).

## Constraints of REST

**Client – Server:**

This is the most basic constraint of REST. It implies that REST can ONLY be implemented on a client – server architecture. This client is usually a browser used to make requests, whereas, the server is the remote computer holding information to be accessed. This constraint of REST enhances separation of concerns and scalability.

**Code On Demand:**

An optional constraint, signifying the fact that the server can send executable code to be implemented by the client.

**Layered System:**

No matter how many systems process a request, the end response should be the same regardless.

**Statelessness:**

The server does not store client-related information in form of sessions.

**Cache-ability:**

The server should identify which responses are cache-able or non-cache-able. Information that will not easily change over time, should be cached or stored in the browser, so that loading time can be reduced and performance, increased.

**Uniform Interface:**

This generally ensures a standard method for communication – HTTP verbs. It is another fundamental constraint that must be implemented for an API to be RESTful. There are four sub constraints included, they are:

- Every resource should be identified. They should have a name or URI. Let’s reconsider the weather service for our small app. Using a very popular service provider, open weather map, we can get the weather report for London by making a call through this URI:
  api.openweathermap.org/data/2.5/weather?q=London
  This is possible because, the resource we are looking for, weather details for London city, is named “London” in the URI. Therefore, finding reports for other cities could be done just by replacing “London” by the city name in the URI.

- The resources can be manipulated through representations. All the data or information needed by the server to take actions are fully included in the representations transferred by the requests. Operations such as editing and deleting of resources should be carried out with the given information the client has.

- Self-descriptive Messages. Every request should provide all information regarding the action it wants the server to perform.

- Hyper-media As The Engine Of Application State(HATEOAS). Don’t be fooled by this big term. This constraint means that URIs for resource representations are embedded in the representation itself. Assuming our request to the weather API was successful, we could get a simplified response of this form:

```
{
"id":123456,
"name":"London",
"link": [{
"rel":"self", "href":"data/2.5/weather?q=London"
}]
}
```

## URL Design Principles for RESTful Web API

Among the constraints of a REST architectural style, Uniform Interface is the central feature. It doesn’t mean all resources share the same URL, the true meaning is to assign every resource a unique URL, and the design of all URLs follows the same normal form. There are many books written about the rules of RESTful API design if you want to do some research however, here are some rules we will follow:

- The hierarchical relationship must be separated by a forward slash (/). For example, http://localhost/products/shoes

- All words should be lowercase and use hyphens (-) to improve readability. For example, http://localhost/user-comments

- No file extensions. Please note, we are designing URLs for a RESTful API, not for websites, so you won’t see the URLs end with /.jsp /.aspx .html /.php, etc.

- Use a plural noun for collection names or store names. For example, http://localhost/users

- Use a singular noun or identity for specific documents or elements. For example, in the URL “http://localhost/offices/2/employees/tim-udoma“, the ID 2 helps the server identify the office, and the employee name helps the server to retrieve his information.

- Query parameters should be used for filtering. For example, http://localhost/shop/24/products/clothes?gender=male

- Use a verb or verb phrase for controller names. For example, http://localhost/products/search

- Create, Read, Update, Delete (CRUD) operations should not be used in URLs. For example, you will never see the URL http://localhost/get-cloth/10, because the correct design is to send the URL http://localhost/products/clothes within an HTTP GET request to the server, the HTTP request is a self-descriptive message.

There you have it. Thanks for reading.

Did you spot a typo, an error or want to contribute? [Here's the repo on GitHub](https://github.com/samtimberlan/Blog-Posts/blob/master/REST%20%E2%80%93%20A%20pragmatic%20Approach%20For%20Building%20APIs.md)
