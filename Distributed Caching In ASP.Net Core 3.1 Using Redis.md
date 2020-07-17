Distributed caching is the concept of centralizing a cache such that it is used by multiple servers. During development, caches commonly reside in memory on the developer’s system. This presents a challenge when deploying to multiple servers in a farm. If the cache were to reside in the memory of each of these servers, there would be no way to map keys to values of previous requests, since requests are not guaranteed to end up on the same server in a load-balanced distributed environment.

A distributed cache resolves this problem by acting as a central memory store for caching. It ensures that stateful resources such as user sessions are “sticky”, that is, they are routed to the same server that handled the first request for a user session.

Distributed caching in ASP.Net Core makes use of the interface `IDistributedCache`.

To get started, this article makes some assumptions:

- You have an editor installed (Visual Studio Code or Visual Studio)
- You have Redis installed. You can find installation instructions [here](https://chocolatey.org/packages/redis-64/)
- You are running .Net Core 3.1

With this in mind, let’s dive in.

Create a new ASP.Net core API project.
Build the project.
Install the nuget package `Microsoft.Extensions.Caching.StackExchangeRedis`
Register the package in `Startup.cs` using the code below:

```
// Add Redis distributed cache
services.AddStackExchangeRedisCache(options => options.Configuration = this.Configuration.GetConnectionString("redisServerUrl"));

```

This adds Redis to the application and exposes the `IDistributedCache` interface which can be consumed using dependency injection. Notice that it is at this point we pass our Redis server port number. It is good practice not to hard code this, but to set it as a configurable property via appsettings. Now head over to `appsettings.json` and add the Redis server URL under `ConnectionStrings` section.

```
"ConnectionStrings": {
    "redisServerUrl" :  "localhost:6379"
  },
```

You can find your Redis port number by running `redis-server` command in CMD. By default, it is `localhost:6379`. Now Redis is set up and ready to be used in your application.

Head over to the `WeatherForecastController` and inject `IDistributedCache` via constructor injection.

```
public WeatherForecastController(ILogger<WeatherForecastController> logger, IDistributedCache cache)
{
    _logger = logger;
    _cache = cache;
}

```

`IDistributedCache` interface ensures the presence of essential methods for working with a cache namely:
`GetString()` and `GetStringAsync()` for retrieving string values including JSON strings
`Get()` and `GetAsync()` for retrieving byte array values.
`SetString()` and `SetStringAsync()` for setting string values also including JSON strings
`Set()` and `SetAsync()` for setting byte array values.
In the `Get` action method of `WeatherForecastController` controller, check the cache for the specified cache key, if present, return the result from the cache rather than the DB.

```
// Check if content exists in cache
string cachedWeatherResult = await _cache.GetStringAsync("weatherResult");
if (cachedWeatherResult != null)
{
    return cachedWeatherResult;
}
```

Else, process the request

```
var rng = new Random();
            var weatherResult = Enumerable.Range(1, 5).Select(index => new WeatherForecast
    {
        Date = DateTime.Now.AddDays(index),
        TemperatureC = rng.Next(-20, 55),
        Summary = Summaries[rng.Next(Summaries.Length)]
    })
    .ToArray();
```

save it to cache for subsequent requests.

```
string result = JsonConvert.SerializeObject(weatherResult);
await _cache.SetStringAsync("weatherResult", result);
```

## Cache Expiration

Given our setup, we run the risk of having stale data in our cache. To resolve this, we need to set the absolute expiration and sliding expiration times whenever we add an item to the cache.
Absolute expiration time defines the period a particular item can remain in the cache before it is removed. On the other hand, sliding expiration time refers to the period before a cache item is removed if unused. It is usually smaller than the Absolute expiration time.

Now let us implement this by refactoring our code to avoid repetition. We will add extension methods to `IDistributedCache`.

Create a new folder in the project called `Extensions` and add a class named `CacheExtensions` and add the code below:

```
using Microsoft.Extensions.Caching.Distributed;
```

```
public static async Task<T> GetCacheValueAsync<T>(this IDistributedCache cache, string key) where T : class
{
    string result = await cache.GetStringAsync(key);
    if (String.IsNullOrEmpty(result))
    {
         return null;
    }
    var deserializedObj = JsonConvert.DeserializeObject<T>(result);
    return deserializedObj;
}

public static async Task SetCacheValueAsync<T>(this IDistributedCache cache, string key, T value) where T : class
{
      DistributedCacheEntryOptions cacheEntryOptions = new DistributedCacheEntryOptions();

      // Remove item from cache after duration
      cacheEntryOptions.AbsoluteExpirationRelativeToNow = TimeSpan.FromSeconds(60);

      // Remove item from cache if unsued for the duration
      cacheEntryOptions.SlidingExpiration = TimeSpan.FromSeconds(30);

      string result = value.ToJsonString();

      await cache.SetStringAsync(key, result);
}
```

Note that `SetCacheValueAsync` uses an instance of `DistributedCacheEntryOptions` to set Absolute expiration time and sliding expiration time.

With these extension methods in place, refactor the `WeatherForecastController` to invoke them. The controller should look like this

```
public async Task<IEnumerable<WeatherForecast>> Get()
{
   // Check if content exists in cache
   WeatherForecast[] weatherResult = await _cache.GetCacheValueAsync<WeatherForecast[]>("weatherResult");
   if (weatherResult != null)
   {
        return weatherResult;
   }
   var rng = new Random();
    weatherResult = Enumerable.Range(1, 5).Select(index => new WeatherForecast
   {
        Date = DateTime.Now.AddDays(index),
        TemperatureC = rng.Next(-20, 55),
        Summary = Summaries[rng.Next(Summaries.Length)]
    })
        .ToArray();

    await _cache.SetCacheValueAsync("weatherResult", weatherResult);
    return weatherResult;
}
```

Build and run the application. Ensure Redis CLI is still running in CMD. Open Postman and navigate to the weather forecast URL. Note the time taken to service the request. This first request was not from the cache.

Send the request again and this time, the request time should be less than that of the first request. The data in the response should be from the cache. This can be verified by running the command `keys *` in Redis CLI. If your key is in the cache, congrats, you have been able to implement Redis successfully.

There you have it. Thanks for reading.

Did you spot a typo, an error or want to contribute? [Here's the repo on GitHub](https://github.com/samtimberlan/Blog-Posts/blob/drafts/Distributed%20Caching%20In%20ASP.Net%20Core%203.1%20Using%20Redis.md)
