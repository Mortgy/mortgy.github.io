---
layout: post
title: MGCacheManager class explained
---


# MGCacheManager

-A tool to manage caches with adding timing for each endpoint when to expire.

Advantages : 
-Improves speed for any Application relaying on HTTP API.
-Managing which endpoints responses to cache.
-Useful for API's which doesn't support ( HTTP Status code 304 ) responses.

# * NOTE *

Recommened to implement this Class just with GET methods.

Response Timing Test

First Run Response Time 1.891051 sec

Second Run Response Time 0.025160 sec

**Workflow**

![Workflow](http://mortgy.com/mortgy/MGCacheManager.png)

#At App Delegate Implementation

{% highlight objective-c %}

 #import "MGCacheManager.h"

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    [MGCacheManager initializeExpiredCachesCleanerTimer];
    return YES;
}

{% endhighlight %}


#Example for Request ( Using AFNetworking )

{% highlight objective-c %}

 + (void)sendGetPayload:(NSDictionary *)parameters
                toPath:(NSString *)path
    withLoadingMessage:(NSString *)loadingMessage
              complete:(void (^)(id JSON))complete{
    
    BOOL cachableButFileNotFound = NO;
    if ([MGCacheManager endPointsContainsEndPoint:path]) {
        NSLog(@"YES Contains path");
        if ([MGCacheManager validateEndPointCacheFileExistanceForEndPoint:path]) {
            NSLog(@"YES Fild Found");

            complete([MGCacheManager loadDataFromCacheForEndPoint:path]);
            return;
        }
        else
        {
            cachableButFileNotFound = YES;
        }
    }
    
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
    [manager.requestSerializer setCachePolicy:NSURLRequestReturnCacheDataElseLoad];
    
    manager.responseSerializer = [AFJSONResponseSerializer serializer];
    manager.requestSerializer = [AFJSONRequestSerializer serializer];
    
    manager.responseSerializer.stringEncoding = NSUTF8StringEncoding;
    
    [manager GET:[NSString stringWithFormat:@"%@%@",kAPIURL,path] parameters:parameters success:^(AFHTTPRequestOperation *operation, id responseObject)
     {
         
         if(complete != nil)
         {
             if (cachableButFileNotFound) {
                 
                 if(complete != nil) complete([MGCacheManager saveAndReturnEndPointResponse:responseObject endPoint:path]);
                 
             }
             else
             {
                 if(complete != nil) complete(responseObject);
             }
         }
         
         
     }failure:^(AFHTTPRequestOperation *operation, NSError *error) {
         
         NSMutableDictionary *params = [[NSMutableDictionary alloc] init];
         [params setValue:[NSString stringWithFormat:@"%ld",(long)[operation.response statusCode]] forKey:@"responseCode"];
         if(complete != nil) complete(params);
         
     }];
}

{% endhighlight %}

In "endPointsToCache.plist" you'll have to follow the same structure while adding your endpoints

{% highlight xml %}

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<array>
	<string>posts</string>
	<string>720</string>
</array>
</plist>

{% endhighlight %}

Index 0 is endpoint path

Index 1 is duration ( 1 = 1 minute )

Any issues or recommendations , please contact me or open a new issue



#Other Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>