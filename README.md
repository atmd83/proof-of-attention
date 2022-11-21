# POA - <small>Proof Of Attention</small>
This library has been designed to track the metrics needed to calculate the users level of attention on a webpage.
It collects data such as manual user interactions such as scrolling and implicit metrics such as time spent on the page, idle time and time the page has spent in focus.

###### Two methods of collecting data are used, both with their own benefits and drawbacks.

## The 2 approaches to logging attention data.
While there almost certainly be other methods to collect this data, this library focus on
two methods of collection.

1. Batch collection. The metrics once collected by the client application get analysed and bundled into 
a single array of event data. 
This data can then used by the client application however it wants, for example 
logged/displayed to a user or sent to a api to store the analysed data.
   
> This approach is much easier and faster to implement, but offer less flexability and extendability.

2. Stream collection. Events get streamed to an api via http or websocket in real time as those events occur.
There is no storing of these events on the client and no analysis done in real time. 
   
> This is a more advanced approach but offers the full raw data to extend in anyway required. 

#Batch Events
Batching events means collecting and analysing metrics on the client and sending all data
in one go, rather than after each event.

#### Pros/Cons to this approach
> Pros
- Less logic to write in api's
- Easy to implement
- provides a complete journey including start and stop events

> Cons
- Brittle, a page refresh restarts the tracking
- Network outage can cause lost data
- Can't change analytic logic as it's contained in the client library

### Usage
Simply import the package and create a new instance.

```javascript
    import { BatchEvents } from 'proof-of-attention';

    const userAttention = new BatchEvents(); // creating the instance starts the session
    userAttention.stop(); // ends the current session

```

### Options
When creating the collector you can configure it by passing in an object of options:

```javascript
    const options = {
        userId: '', // A unique identifyer for the user you are tracking, can be used to analys logged in users
        log: true || false, // logs data on collection
        onLog: (data) => console.log(data), // function to log if log === true
        onComplete: (data) => {} // function to send complete data to
    };

    const defaults = {
        onComplete: (data) => console.log(data)
    };  

    const userAttention = new BatchEvents(options);
```

#### Example of catured data
The event data captured if you are using batch collections contains data much like the example below:

```javascript
{
    timestamp: '', // timestamp for batch onComplete
    url: '', // the url of the page the event occured
    useragent: '', // useragent of the device you are currently tracking
    userId: '', // A unique identifyer for the user you are tracking (if configured)
    timeOnPage: '', // Total page time (in seconds)
    timeIdle: '', // Total idle time  (in seconds)
    timeUnfocused: '' // Total time the application wasnt the tab in focus  (in seconds)
}
```


#Stream Events
Streaming events means that rather than collecting the data until a point as in batching, we update an API
on each event in real time. 
Because of this no analytics is done on the client, it's all done either in the API
 or stored in a database and analysed by a purpose built service.

#### Pros/Cons to this approach
> Pros
- Full control over analytic logic as contained in api layer
- provides real time events
- provides much more data

> Cons
- A confirmed end time might not happen, i.e. turn pc off.
- More work required in api layer
- Network issues could cause missing data

### Usage
Simply import the package and create a new instance.

```javascript
    import { StreamEvents } from 'proof-of-attention';

    const userAttention = new StreamEvents(); // creating the instance starts the session

```

### Options
When creating the collector you can configure it by passing in an object of options:

```javascript
    const options = {
        type: Http || WebSocket, // Use a http request to send events too OR use a websocket
        sendMethod: Axios.post, // used when http type is selected, can be overridden with any library
        headers: {}, // can be used for auth headers etc
        userId: '', // A unique identifyer for the user you are tracking, can be used to analys logged in users
        log: true || false, // logs data
        onLog: (data) => console.log(data) // function to log if log === true
    };

    const defaults = {
        type: 'http',
        sendMethod: Axios.post,
        headers: {}
    };

const userAttention = new BatchEvents(options);
```


## Events captured
Once the listener has been instantiated it will listen for a whole host of events triggered by the user to detect attention, these events are:
- `click` - A user clicking on the page
- `scroll` - A user scrolling
- `top`  - A user scrolling to the top of the page
- `bottom` - A user scrolling to the bottom of the page
- `start` - collector instantiated/started
- `stop` - calling the stop method
- `unfocus` - The user selecting another tab in their browser
- `focus` - The user selecting our application tab in their browser

> The names of the events above can be used to turn them off/on in the config.

#### Example of catured data
The event data captured if you are using streaming contains data much like the example below:

```javascript
{
    name: '', // This is the event name, matches the events above OR a custom event name
    timestamp: '', // timestamp for the event
    url: '', // the url of the page the event occured
    useragent: '', // useragent of the device you are currently tracking
    userId: '', // A unique identifyer for the user you are tracking (if configured)
}
```
In addition to the above when using batched events you also get extra metrics such as total time on the page, total idle time, total scrolling time etc.
You will also get total time the page spent unloaded i.e. the user has focused on another tab, and your tab is in the background.

> The logic calculations mentioned above are calculated on the client and can't be changed (see pros & cons of approach).


### Turning off default events
If your use case means you *don't* want all default events to be tracked you can add a white list of events to track. This is an array of event names, any event not
listed in the array will be ignored.

```javascript
    const options = {
        events: ['scroll', 'click']
    };

const userAttention = new BatchEvents(options);
```
> N.B. This list is a white list, any events not listed will *not* be tracked.

### Custom Events
Custom events can be tracked by firing the `event` method manually, This can be used for goal tracking (sign up form completed etc) or tracking any 
metric specific to your application.

```javascript
    const event = { 
        name: '', // event name used for grouping events and makes analysis much easier
        ... // any other metadata used for your custom event
    };

    userAttention.event(event);
```


# Full React example
Setting up a basic set up with defaults using the batch collector is pretty simple:

```javascript
import { useEffect } from 'react';
import { BatchEvents } from 'proof-of-attention';

export default () => {
    useEffect(() => {
        const userAttention = new BatchEvents({
            onComplete: (data) => axios.post('/some-endpoint', data); // Do something to store the data
        });
        
        return () => {
            userAttention.stop();
        }
    })
    
    return (
        <>
            /* Your page content here (works best with enough content to scroll). */
        </>
    );
}
```
