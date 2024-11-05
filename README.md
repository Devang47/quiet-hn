# Quiet HN

[https://courses.calhoun.io/lessons/les_goph_85](https://courses.calhoun.io/lessons/les_goph_85)

## Concurrency

Rather than focusing on how to build this application we are going to look at ways to add both concurrency and caching in order to speed up the application. The first - concurrency - will be explored because it is a common reason to want to check out Go, and it is nice to get a feel for how it works in the language. The second - caching - is important because this is actually one of the easiest and most effective ways to speed up our application, and even outperforms our concurrency changes.

For the first phase of this exercise, explore the existing code and figure out a way to retrieve the stories concurrently. At first just try to get something working, but once you have that try to make sure you meet all of the following criteria:

### 1. Stories MUST retain their original order

When retrieving stories concurrently it is possible that you will get results back for a story in position #5 before getting a response about the #3 story. Regardless of the order you hear back, your stories should always be in the correct order - the same order they are in the current version of the code. The only way a story should change positions when compared to Hacker News is if an earlier story was filtered out, and even then the index should be changed but stories shouldn't swap positions randomly.

### 2. Make sure you ALWAYS print out 30, and only 30, stories

When interacting with the HN API you might retrieve stories that need to be filtered. As a result, you can't just get the first 30 stories and then render your page, but you instead need to make sure you find the first 30 that don't get filtered. Doing this without concurrency is easy, but doing this both with concurrency, and while retaining the original order of the stories, can make it tricky.

The first approach I would recommend is to always get a few extra stories to account for filtered stories. Eg if `numStories` is set to `30`, maybe we should always retrieve `1.25 * 30` stories concurrently to account for filtered stories. We obviously can't _always_ count on this working, but it should work a majority of the time.

After that is working, try to figure out a way to ensure you always get at least 30 stories regardless of how many might be filtered out. As I stated earlier, this can be a little trickier with concurrency and while retaining the original order of the stories so take your time and try a few approaches out. Try to weigh the pros and cons of each approach and see if you can find any edge cases where your solution would be incorrect.

_Note: If you change the value of the `numStories` flag then obviously 30 will be a different value, but the same general rule still applies - always render the correct amount of stories._

If a story is in position #3 on HN, it should be in that same position with your concurrent version of this application. This means that even though your API request for story #4 may finish BEFORE story #3

**Warning:** _Each of these two rules are much easier to implement independently than they are together, so if you get stuck or frustrated don't worry - it isn't exactly a simple problem to approach._

## Caching

In addition to adding concurrency, add caching to the application. Your cache should store the results of the top `numStories` stories so that subsequent web requests don't require additional API calls, but that cache should expire at some point after which time more API calls will be needed to update the cache.

How you implement this is up to you, but you should definitely consider the fact that many web requests can be processed at the same time, so you may need to take race conditions into consideration. A great way to test this is the [-race flag](https://blog.golang.org/race-detector).

## Bonus

Experiment with how many goroutines you use. For instance, some of you will code this by creating a goroutine for each item you are retrieving from the HN API, while others will use maybe a small set of goroutines and have them work through a list of item IDs that need retrieved. Try both approaches and see how they compare.

_Note: You can limit your workers via channels, or with something like the [x/sync/semaphore](https://godoc.org/golang.org/x/sync/semaphore) package._

You can also look into ways to improve your cache. For instance, imagine we have a cache that we invalidate every 15 minutes, at which point we will replace all the values in it when we receive the next web request. This means that the next web request will be slow because it has to wait on us to repopulate the cache. One way to improve this experience is to always keep a valid cache, which can be done by creating the new cache BEFORE the old one expires, then rotating which cache we use. Now if we were to update and rotate the caches every 10 minutes, it is very unlikely that our currently in-use cache will ever exceed the 15 minute deadline and our users won't ever see an noticeable slowdown.
