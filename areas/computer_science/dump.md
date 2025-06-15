---
date: 2025-01-05
tags:
  - computer-science
  - to-do
---

# what i learned from software engineering at google
## resource
- https://swizec.com/blog/what-i-learned-from-software-engineering-at-google/
## annotations
- software engineering is programming over time
- if you liked it you should've put a test on it
- you're in trouble when overhead grows faster than engineering. automation helps.
- a small release is easier to manage. a small release is easier to revert. a small release is easier to understand.
- the bigger the change, the harder to figure out which commit or feature broke production. small changes make it obvious.
- you can upgrade a dependency or a piece of code or make a function deprecated. then tell everyone "hey please upgrade".
# non functional requirements for software engineers
## resource
- https://medium.com/@chiragshetty98/non-functional-requirements-for-software-engineers-e413dd33721b
## annotations
- mystery of estimation — on point estimation
- idea is only as good as your ability to present it — effective communication
- act like you own it — take ownership
- perfection is the enemy of progress — embrace tradeoffs
- always ask why — normalise pushback
- ask questions, even if they seem very silly.
- build opinions and express them shamelessly — increase participation
- it's a marathon not a sprint.
	- learn to pace yourself and balance things
- learn to pick your battles — maximise impact
# random thoughts 15 years into software
## resource
- https://roughlywritten.substack.com/p/random-thoughts-15-years-into-software
## annotations
- debuggability is highly underrated. when writing code, you have to think about how it will execute, and how to debug. think logs, audit trails, monitoring, admin tools.
- and in software, there’s always more we can add to a given feature or system. give a best effort, and keep your stakeholders informed of progress and blockers
- aggressively manage scope. related to the above, protect your project’s scope. defensively, as people will often try to add things throughout the project.
	- you don’t have to push back if you don’t want, but be transparent about how it will affect the project delivery and communicate it widely. 
	- offensively, look for things you can cut or, my favorite, look for things that you can ship after launch and push to prioritize those at the end. i love a good “fast follow”.
- action is rewarded. pointing out problems or complaining is not.
- take ownership of your systems. not just *your* code. act like you are personally responsible for the success of the systems you work on, end-to-end (because you are!).
- you are part of a larger organization. 
	- the software your company produces might be the product it sells to make money, but that doesn’t mean your job is the center of the universe. 
	- take time to meet people from other functions (sales, marketing, finance, etc) and learn how they think and work. 
	- you’ll have a much more holistic view of the entire business, and decisions that come down around you will make a lot more sense. 
	- this is true even in the smallest of startups
- you won’t have time to go back and fix technical debt. 
	- do your best to minimize it up front, and prioritize what to address based on what you would be okay living with for the foreseeable future. 
# debugging svg width x height issue for firefox
## context
- Using the naturalHeight and naturalWidth property of images in a certain script.
- But, for few svgs firefox is returning height x width as `0,0` which is causing discrepancies in further calculations in the script.
## debug
- Tried to replicate the cURL command from the firefox and chrome consoles, found that both of them do not export the same command, they have a difference of the --compressed flag in cURL
- Removed the --compressed file, and stored the compressed file returned from cURL command
- Deciphered that the downloaded file is actually a zlib compressed file, but using the `file` command.
	- Can also use [trid](https://mark0.net/soft-trid-e.html) to detect file type.
- `gunzip` does not unzip zlib files, hence needed some other tool.
- Used openSSL to deflate that zip as -
	- `openssl zlib -d -in test`
- After getting the final file, used `imagemagick identify` to get the geometry or size of the image, which turned out to be expected one.
- On further analysis realized that, imagemagick is using the `viewBox` prperty from svg, to actually get the height and width, whereas firefox is unable to do that.
- Also, explicitly set the `height` and `width` attribute on svg made firefox also get the correct height and width.
## todo
- How to read headers of files ?
	- I used `od -bc <file_name> | head` to get some details around this
# what is @Modifying and @Transactional annotations in java spring boot
## what is it ?
- how to use modifying and transactional ?
## what is transactional annotation ?
- it is used to declare that a single persistent context (aka 1st level cache) needs to be created for this
- hence it is always suggested to add anotation tag on session, since the transactions you do within a session, will usually operate on a single entity continuously
- if transactional annotation is not added at a session level, and is added somewhere else, then that session might contain multiple persistent context, which might lead to faulty data being used and wrong business logic
- resource: https://stackoverflow.com/a/57062549/8152977
## what is modifying annotation ?
- it is used to declare that you will be updating an entity in this transaction
- using modifying without any extra parameters like clearAutomatically, flushAutomatically, will not flush or clear the entity manager.
- so, if you have any pending actions in your entity manager, they will not be executed first, nor will the context in entity manager be cleared.
- adding just clearAutomatically, will clear the entity manager, but that will lead to pending operations in the entity manager being cleared as well
- whereas just adding flushAutomatically, will flush the pending queries from entity manager, but will not clear it
- hence, we need to set both flushAutomatically: true and clearAutomatically: true for the modifying annotaiton to work correctly
- resource: https://stackoverflow.com/a/57062549/8152977
# decorators
## What ?

- Decorators as the name suggests are used to decorate methods on the fly, with some new added functionality
    - In most programming languages that support it, it is done by creating a decorator *(added feature)* that you want some function to have
    - Wrap some function, using the decorator, it is basically passing the said function to decorator
- Different languages can have different barriers around how to pass parameters and how to use the return values, you will have to dig into it based on language and use cases
- But at it’s core, decorator, decorates a function with added functionality
    - It is a wrapper function around current function

## Use Cases

### Sending Metrics Data

```python
# creating a decorator
def send_metrics_deorator(fn):
	def send_metrics():
		create_metric()
		x = fn()
		assign_metric(x)
		send()
	return send_metrics

# using a decorator
@send_metrics_decorator
def start():
	do_something()
return 200
```

### Creating wrapper API

- Most of the times, API is just some endpoint to be hit with some arguments
- [This](https://github.com/tapanpandita/pocket/blob/master/pocket.py) github repo, uses python decorators smartly to make a DRY API wrapper for Pocket using Python

```python
# creating an api decorator
def method_wrapper(fn):
		# @wraps(fn) is a python decorator, to link meta args to original function name
    @wraps(fn)
    def wrapped(self, *args, **kwargs):
        arg_names = list(fn.__code__.co_varnames)
        arg_names.remove('self')
        kwargs.update(dict(zip(arg_names, args)))

        url = self.api_endpoints[fn.__name__]
        payload = dict([
            (k, v) for k, v in kwargs.items()
            if v is not None
        ])
        payload.update(self.get_payload())

        return self.make_request(url, payload)

    return wrapped

# using an api decorator
@method_wrapper
    def get(
        self, state=None, favorite=None, tag=None, contentType=None,
        sort=None, detailType=None, search=None, domain=None, since=None,
        count=None, offset=None
    ):
        '''
        This method allows you to retrieve a user's list. It supports
        retrieving items changed since a specific time to allow for syncing.
        http://getpocket.com/developer/docs/v3/retrieve
        '''
```

## Languages

### Ruby

- Does not have an inbuilt decorator, but it does have `yield` keyword which can be used in similar space
- Here, unlike python we do not add syntactic sugar above the function name, but we use it inside of the function
- Also, you can pass parameters to the original function using `yield`

```ruby
# creating a generator
def send_metrics(args):
	create_metric(args)
	param
	# yield, calls the caller block and passed param to it
	yield param
	assign_metric()
	send_metric()
end

# using a generator
def start():
	send_metrics {|args|
		do_something(args)
	}
	return 200
```

# debug linux audio codecs
## audio
### Problem
- Sony WH-1000XM having really bad sound quality, and couldn't find any config to update the codec being used.
- This has happened in the past, and disabling the `mic` usually solves the issue, as I am able to switch to LDAC, but that did not work here.
### Solutions
#### Trying some new driver
- Tried to follow random advice on the internet, and ended up uninstalling [pipewire ](https://pipewire.org/), this caused my screen to blackout and even on reboot the display manager won't load, and it would showcase as `Exited` as soon as system would boot up.
- `systemctl status display-manager` 
	- Showcased `GNOME Display Manager` has exited
- Looks like this uninstalled some package which were being used for display (video) as well.
- Initally I couldn't even get myself to login, and then I remembered the `login` name, since usually I do not enter it when I login.
- To fix it, I had to run -
	- `sudo apt-get install --reinstall ubuntu-desktop`
- **Lesson learnt, never uninstall or install any package without reading their description.**
#### Messing around with Pipewire and Wireplumber
- [Pipewire](https://pipewire.org/) still not aware about how this works, all I understand is it is a bette audio-video processing server than [PulseAudio](https://www.freedesktop.org/wiki/Software/PulseAudio/), and it uses graph based approach.
	- [Pipewire repo](https://gitlab.freedesktop.org/pipewire/pipewire)
- Playing around with [Wireplumber](https://wiki.archlinux.org/title/WirePlumber) which is a session and policy manager for Pipewire.
- Tried to add custom [configurations](https://pipewire.pages.freedesktop.org/wireplumber/configuration.html) for bluetooth codec as defined [here](https://wiki.archlinux.org/title/WirePlumber#Configuration), but that did not work.
	- Note that while doing this, I created a backup config with `.bkp` extension, just in case something went awry, it is critical to create backups like this.
- This didn't help, so I tried to disable the mic access for WH1000XM4 using some tips defined [here](https://unix.stackexchange.com/questions/677986/disable-audio-source-sink-module-in-pipewire).
	- This led me to get a brief idea about how wireplumber loads config sequentially through each step.
	- https://gitlab.freedesktop.org/pipewire/wireplumber/-/issues/279
- Finally realized that `node.disabled` and `device.disabled` config is not defined for bluetooth scripts, so manually added those, and voila! the mic source was disabled for WH1000XM4, but this still did not solve the issue of changing bluetooth codecs.
- Realized that the issue is actually with Bluetooth Profiles and not codecs, so will look into that some other day.
- Also installed [blueman](https://wiki.archlinux.org/title/Blueman) to help with better bluetooth control.
- Also updated `etc/bluetooth/main.conf` and added `Disabled=Headset` in `[Genera]` config, in an attempt to disable bluetooth profile I think, I do not completely understand this yet, so commented it out again.
	- [Link here](https://askubuntu.com/questions/863930/bluetooth-headset-cant-set-a2dp-high-fidelity-playback-poor-sound-quality)
#### Commands
- `wpctl status`
- `wpctl inspect ID`
- `pactl info`
- `pactl list`
- `WIREPLUMBER_DEBUG=3 wireplumber 2>&1 | tee wireplumber.log`
#### Lessons learned
- It is okay to settle for where you are, than aiming for perfection. 
	- **80-20 rule:** Reaching that perfection will possibly only add around 20% more benefits to your experience, so in all likelihood not worth it. *I understand that this might not be how 80-20 principle works but it satisfies my usecase.*
	- Also, it's important to move forward, rather than circling around the same place trying to get to perfection. Since, that perfection probably won't add a lot of benefits, plus you are going around in circles and not progressing at all.
- Also, take 15 minutes out to learn about some solution a person is suggesting, before implementing it directly.
	- Alternatively do a quick google search and readup top 3 links about it.
		- Or ask chatGPT for top 3 sources, and read up about it.
	- This will help you better understand things, rather than going around in circles.
- Use pomodoro technique to timebox your solutions, so that you don't get trapped in an endless pit trying different commands seeing what fits.
#### To Do
- What are bluetooth profiles and how are they different from bluetooth codecs ?
- How can you edit bluetooth profiles for particular devices ?
- Checkout a possible solution suggested [here](https://askubuntu.com/questions/863930/bluetooth-headset-cant-set-a2dp-high-fidelity-playback-poor-sound-quality).
	- https://variwiki.com/index.php?title=BlueZ5_and_A2DP
	- https://wiki.archlinux.org/title/bluetooth_headset
- https://en.wikipedia.org/wiki/Advanced_Linux_Sound_Architecture
# dependency injection

## what is it ?
- we inject the dependent code into the code that uses it
- example: passing database class / object to another class
## example
- another example being, passing a storage object to attachment handler
- this storage handler can be awss3, sftp, webdav, etc, which implements the class storage with
predfined functions like upload
- can create a storage_factory to generate the storage for company specific code
- we use a factory to decide which preview generator to use based on the attachment type
- it's like those puzzle pieces where you plugin each piece of the puzzle, like scanner, storage,
previewGeneration, etc...
- even if a class has only one subclass (inherited obj) it still makes sense to do dependency
injection, as it makes testing easier
- since when using dependency injection we are defining the interace that an object should be using
## resources
- https://youtu.be/J1f5b4vcxCQ?si=ayp1WlZUMOeL2Cbf 
# elasticsearch boolean queries

## boolean queries
- `filter` and `must_not` are only used for filtering and have no scores attached to it
- `must` and `should` are used for both filtering and scoring
## how are multiple boolean queries combined ?
- start from the top and go on applying filter as you move to next queries
- only exception here is `should` clause, since it acts as logical `OR`
- we need to define the minimum number of terms to match in the `should` clause
- if there are other boolean clauses with `should` clause, like `match` clause, the default matching parameter is `0`
- and hence `should` clause is only used for scoring in those scenarios
- but if `should` is the ONLY clause in a boolean query, the default minimum match changes to `1`
- https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html#bool-min-should-match
# elasticsearch shard segments and ordinals

## shards vs segments
- index in elasticsearch is equivalent to a table in SQL based databases
- indices in elasticsearch are made up of shards. And shards in turn are made up of segments.
	- index > shards > segments
- the number of shards in an index are fixed and are declared during the index initialization phase
- shards are nothing but [lucene](https://lucene.apache.org/core/) indices
- segments are immutable inverted indices that make up a shard
- as documents are indexed, they are added to new segments, which are then made searchable after a refresh operation.
- [refresh](https://opster.com/guides/elasticsearch/glossary/elasticsearch-refresh-interval/): adding new index data to segment and making it searchable
- flush: at first all of the refreshing and adding to segments is done in memory and then f'synced to disk in a process called as 'flush'
- merge: with time the number of segments will increase, and this increase in fragmentation will lead to slower search times. 
	- hence over time, many segments are 'merged' into a single segment, which makes search faster.
- segments are immutable inverted index, during searching for a term in shard, this search is performed on all segments in the shard and then the collective result is returned.
- during each refresh new segment is added, as older segments can't be updated
- hence, even when merging segments, duplicate documents are created to be inserted into new segment, whereas documents from older segment are deleted.
- also when document is updated, older document is updated and newer document is added.
- https://stackoverflow.com/a/15429578/8152977

## global ordinals
- global ordinals are a data structure used to optimise aggregation of keyword based fields.
- here each field is a keyword, and hence it is replaced with a number and placed in a new inverted index known as global ordinal.
- at the end of the search operation, this number is converted back to the keyword
- ideally this global ordinal is built after the first search query is performed on the keyword field.
- but one can eagerly build this global ordinal. by providing `eager_global_ordinal: true` in which case this is prepared during the indexing phase itself
- another way to optimise keyword based aggregation is to use `execution_hint: map`, but this stores data in memory and is suggested only for queries which return a small set of results
	- This setting will compute the aggregations only on the filtered documents, which is ideal if the queries you are running are restrictive (i.e., returning a small number of results) as global_ordinals will compute on all shards regardless of the query.
- https://opster.com/guides/elasticsearch/search-apis/terms-aggregation-on-high-cardinality-fields-in-elasticsearch/

# javascript process events in-order

## handling websocket packages in-order in javascript
- websockets emitted by the browsers are always guaranteed to reach your application in-order since websockets are built on top of TCP.
	- https://stackoverflow.com/questions/11804721/can-websocket-messages-arrive-out-of-order
- but on application level, they can be processed out of order
	- https://stackoverflow.com/questions/54718964/processing-websocket-onmessage-events-in-order
- in my particular scenario, i know what the correct order is, and i want to process the events in those order.
- hence, i created a struct that has multiple locks and keys for each phase of the lifecycle, and added all of them to a map, like -
```javascript
// DefaultMap, which returns a default value for an uninitialized key
// Similar to defaultDict in python
class DefaultMap extends Map {
  constructor(getDefaultValue, ...mapConstructorArgs) {
    super(...mapConstructorArgs);

    if (typeof getDefaultValue !== 'function') {
      throw new Error('getDefaultValue must be a function');
    }

    this.getDefaultValue = getDefaultValue;
  }

  get = (key) => {
    if (!this.has(key)) {
      this.set(key, this.getDefaultValue(key));
    }

    return super.get(key);
  };
};

// RequestLifeCycleHandler handles life cycle of a requestId
// Ideal flow:          requestWillBeSent -> requestPaused -> responseReceived -> loadingFinished / loadingFailed
// ServiceWorker flow:  requestWillBeSent -> responseReceived -> loadingFinished / loadingFailed
class RequestLifeCycleHandler {
  constructor() {
    this.releaseRequest = null;
    this.releaseResponse = null;
    this.checkRequestSent = new Promise((resolve) => (this.releaseRequest = resolve));
    this.checkResponseSent = new Promise((resolve) => (this.releaseResponse = resolve));
  }
}

#requestsLifeCycleHandler = new DefaultMap(() => new RequestLifeCycleHandler());
```
- So for each `requestId`, there will always be a `RequestLifeCycleHandler`
- In my scenario, I only have two locks, and both of them are released, when the particular websocket message is received.
- Example, I have `requestPaused`, which should ideally wait for `requestWillBeSent` .
- Hence, in `requestPaused` I will await on `checkRequestSent` promise which can only be released in `requestWillBeSent` phase

# using dsa at work - to get downtime impact
## problem statement
- recently we had a secret rotation activity at work, which caused all our services to go down.
- thankfully the keys were restored back, and everything was up and running in ~1-2 hours.
- once everything was back to normal, we had to gauge the impact of disruption due to this erroneous secret rotation activity.
- now normally, teams have great logging in place to measure impact of such things like, users affected, builds affected, so on and so forth.
- but since, we are pretty new product our logging is at a very nascent stage and immature. 
- although we could get the users imapcted, since nginx does track all the ips and the response codes, but getting the builds impacted was turning out to be a tall ask.
- for builds, we only had a build data, which showcased started at, finished at, status, etc ...

## solution
- since we knew the impact timelines, all we had to calculate was the number of builds doing some activity during this timeline.
- but it wasn't that straightforward, since we had rotate through 3 sets of secrets during the whole impact timeline, and if some build started and finished off when the same set of keys were there in secrets they would not have faced any issues.
- so, the problem essentially got converted into, finding the number of builds which were overlapping in a certain segment.
- say, there were 3 sets of secrets that were rotated with the below timelines -> 
	- 00:00 -> 13:00 : secret A, B
	- 13:00 -> 13:30 : secret C, D
	- 13:30 -> 14:00 : secret B, C
	- 14:00 -> 00:00 : secret B, A
- now, in this scenario builds created before 13:00 would use secrets A, B to validate, and if they sent any events between 13:00 to 13:30 they would not be authenticated. hence, they were impacted.
- but, if some builds started between 13:00 and ended before 13:30 all the events would be validated.
- so, all we had to do was calculate builds which started in a different segment and finished in a different segment, since both were using different keys, there is guaranteed that some events would not be authenticated and dropped.
- we wrote a simple brute force sql query to calculate this.
- and this saved us big time, by getting the probable impact of the downtime