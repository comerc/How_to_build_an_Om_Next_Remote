#How to build an Om Next Remote

Notes from a training course [by Malcolm Sparks](https://juxt.pro/people/malcolmsparks.html).

Last January I taught ClojureScript and Om Next to a team of JavaScript engineers in the beautiful city of Cambridge (England). For me, it was my third training course of 2016 but the first I'd ever given to a purely front-end team. It's a strong reflection of how businesses are using ClojureScript to build increasingly sophisticated front-end interfaces for their customers.

##Day 1 - ClojureScript fundamentals

We started the first day with my usual gentle introduction to the language: a brief history of the 300+ years of computing history that preceded it. It felt particularly apt to explain the Turing Machine less than a mile away from where it was conceived. (Of course, Cambridge is important as the home for many other famous discoveries, such as calculus, the electron, the double-helix and the ZX81).

We then delved into ClojureScript proper and began establishing ClojureScript's core elements: literals, data manipulation, vars, functions and sequences. Most of the class hadn't seen ClojureScript nor Clojure before, and many were new to functional programming too. But with some practical exercises, and the willingness of everyone to get stuck in, ask questions and help each other, we made good progress through the material.

We even had time in the afternoon for everyone to run up Tony Kay's excellent [Om Next tutorial](https://github.com/awkay/om-tutorial) and complete the first 2 modules. Being built on Bruce Hauman's [devcards](http://rigsomelight.com/2014/06/03/devcards-taking-interactivity-to-the-next-level.html), this is a quick and gentle way of delving into both Om Next and ClojureScript itself.

Once you have grasped the basic concepts and syntax, reading code is one the best ways to gain familiarity with a new language, and at this stage familiarity builds confidence.

Moreover, the instant feedback of the figwheel-powered development environment makes ClojureScript a lot less daunting to the beginner.

The river Cam

##Day 2 - Om Next fundamentals

On the second day we started out with nothing more than an empty template but one with a working boot-powered build system.

There's no strong reason to use boot over Leiningen/figwheel but boot has the potential to be the more flexible tool, an important consideration when constructing the kinds of build pipelines that are becoming the norm in web front-end development.

We then revised the previous day's material, including the declaration and construction of React components with Om Next. This involves using `defui` for their declaration and `(om/factory ...)` for their construction. We created components more relevant to the team's business domain but otherwise they looked like this:

```
(require '[om.next :as om])

(defui HelloView
  Object
  (render [this] (dom/p nil "Hello World!")))

(def hello-view (om/factory HelloView))

(defui RootView
  Object
  (render [this]
    (dom/div nil
      (dom/h1 nil "Om Next Training Course")
      (hello-view))))
```

Notice how the `RootView` component constructs the `HelloView` sub-component using a factory. This is how a visual hierarchy of components is constructed.

##Understanding Om Next

Om Next introduces some new terminology but like a jigsaw it's easy once you understand where each piece fits.

Let's start with a component. An Om component is responsible for transforming their properties into a (virtual) DOM element and its children. Components declare which properties they need to do this.

For example, a component might need to display a user's name and address. It does so by declaring these properties in a vector like this:

```
[:name :address]
```

The vector is parsed (by a parser) which extracts the properties (`:name` and `:address`).

In the context of the component, the query is added like this:

```
(defui RootView

  static om/IQuery
  (query [this]
    [:name :address])

  Object
  (render [this]
    (dom/div nil
      (dom/h1 nil "Om Next Training Course")
      (hello-view))))
```

A function is called for _each_ property. We provide this function, usually called 'read', the purpose of which is to return the current value of that property. When the property changes value, our component is asked to render itself again. Any differences will be reflected in the browser's DOM automatically by the React library underneath.

That's basically it, but know this:

- Property declarations are more sophisticated than just listing the properties. These property declarations are therefore known as _query expressions_.
- Property values are cached in a local atom called the app-state, but might also exist on remote servers too.

This is basically Om Next.

##Query Expressions

I found it hard to understand Query Expressions until I realised that they are much like SQL. In the '90s I built systems in a product called 'Oracle Forms' which allowed visual components to embed SQL for the data they represented. Oracle Forms would use that SQL to fetch data from the database for me. Same idea in Om Next, only SQL queries which return tabular result sets and replaced by 'query expressions' returning trees. Tree are a much better fit for the hierarchical nature of browser elements in a user interface. Otherwise, same idea.

Query expressions are a recursive structure of 5 ideas, also found in SQL:

- Properties (SELECT name,address)
- Joins (think sub-queries, FROM (SELECT ...))
- Idents (think foreign keys)
- Unions (think UNION)
- Mutations (think INSERT/UPDATE/DELETE)

Of course, query expressions aren't _actually_ SQL, but there are strong conceptual similarities.

##Remotes

You know how I said the read function was called once for each property? Actually it's called _twice_, or more strictly, _once for the local database and once for each remote database_. By default, Om Next thinks there's a single remote database called `:remote` (but you can specify as many as you want)

A read function indicates the involvement of a remote by including a `[remote-key remote-query]` entry in the map it returns. For example:

```
{:value 20
 :remote [:likes]}
```

A full remote query is a composition of all the queries required by calling the read function for each property. Once the full remote query is ready, a send function is called with 2 arguments:

- the query to send to the remote server
- what to do with the result when it comes back (a callback)

##The Butler (Om Next calls him the 'Reconciler')

We continued the 2nd day by discussing the 1990s TV series Jeeves and Wooster.

Jeeves and Wooster

In the series set in the 1920s, Bertie Wooster (played by Hugh Laurie, on the left) is a well-dressed aristocratic playboy ably assisted by his butler Jeeves (played by Stephen Fry, on the right). (btw. both these actors attended Cambridge University).

Every time Wooster tries to do something (beyond his normal routine of attending his club, drinking and hanging out with his peers), he gets into troubling entanglements from which Jeeves has to rescue him. Wooster thinks he's in charge but actually everything is done for him behind the scenes by Jeeves.

Like Wooster, React components work best when they avoid entanglements with the outside world. Without a butler to assist them, components can get into all kinds of trouble. It's much better for everyone if components, like Bertie Wooster, just focus on 'keeping up appearances', leaving everything else to their butler.

In Om Next, the role of Jeeves is filled by 'The Reconciler' (somewhat less P G Wodehouse, more 'Judge Dredd').

The Reconciler

The Reconciler's responsibility is to do all the behind-the-scenes activities that make everything else go smoothly. The Reconciler coordinates the following activities:

- The parsing your query expressions
- Calling your read functions
- Sending your post
- Merging changes with the app state
- Logging
- Cooking, cleaning and washing your clothes

With Om Next, therefore, we have an approach to building React front-ends that is even more opinionated than Om. However, we can still do everything we used to do in the original Om, so flexibility is maintained. Opinionated management of state in Clojure has served us well so far, I think it could work out well for us in Om Next too.

A slide

##Day 3 - An Om Next application

The final day was where everything came together. All the trainees had working systems at this point. The main learning modules for the last day were:

- Building the remote server piece
- Changing the state through transactions (mutations)
- Learning about core.async on the front-end

##Building an Om Next Remote

For me, the standout feature of Om Next is its automatic synchronization of state with remote services across the web.

The ability to build rich and interactive user applications is useful, but hardly something new.

What makes web applications compelling is their ability to synchronize state across the internet, but until now the means of doing this has been ad-hoc, inefficient and laborious for many web developers.

The possibility of remote sync in Om Next naturally falls out of the design of its query protocol. It's the query protocol which allows visual components (views) to explicitly query the state they need to render themselves. Queries are declared ahead of time as data, allowing Om Next to deduce other useful information, such as when to tell components to re-render.

But the protocol itself is location transparent, meaning state can exist both locally and/or remotely, the actual location being an implementation detail provided by the developer.

Enough theory, how do we exploit this great feature?

Firstly, we need to understand that the query protocol must be implemented by a service at the other end of the wire. Ideally, this service will support Transit, favored by Om Next.

Next we should consider the role of a remote service: serving all the data your client might need to display to the user. If all your organization's data is stored in a vast cluster of Datomic databases, you're in good shape. Chances are it isn't. Either way you'll need to build an intermediary service that can serve this data to your Om Next 'clients', securely and efficiently (i.e. not sending data to the client that isn't needed).

Working hard

##Reading the property

Read functions are called for each property, for each remote. By convention we dispatch on the property key. For instance, for a property called `customers/by-id`, we would add the following read method:

```
(defmulti read om/dispatch)

(defmethod read :customers/by-id [env k params]
  (let [st @(:state env)]
    (if-let [[_ v] (find st k)]
      {:value v :remote (:ast env)}
      {:value :not-found})))
```

The read method ensures the entry exists in the state (the value could still be nil, that's why we use `find` to distinguish between these cases). This method is idiomatic and you'll see the same implementation in many Om Next resources.

We should always return a map containing a `:value` entry. But if the property also exists on a remote, we should add an entry mapping the remote to the query we wish to use on it. Here, we use `:remote` because that's the name of the default remote. The `:ast` value is the query snippet we'll send to the serve.

##Sending the request

Once ~~Jeeves~~ the Reconciler gets the hint that the property also exists on a remote it calls the send function you've provided ~~him~~ it.

Our send function uses HTTP to send the request to the remote. Here's my implementation, using the native API of modern browsers.

```
(defn send [m cb]
  (let [xhr (new js/XMLHttpRequest)]
    (.open xhr "POST" "/props")
    (.setRequestHeader xhr "Content-Type" "application/transit+json")
    (.setRequestHeader xhr "Accept" "application/transit+json")
    (.addEventListener
      xhr "load"
      (fn [evt]
        (let [response (t/read (om/reader)
                               (.. evt -currentTarget -responseText))]
          (cb response))))
    (.send xhr (t/write (om/writer) (:remote m)))))
```

You could also use JQuery's AJAX or Google Closure XhrIo to achieve the same if you need to support older browsers.

Also, I've named my endpoint /props to indicate that we're querying for properties, but feel free to change this.

Group

We're going to use the POST method because the request needs a body (our Om Next query), and browsers don't like sending request bodies for GET requests.

It's common in Om Next applications to use Transit as the payload for the request payloads. That's why we set the Content-Type header on the request here:

```
(.setRequestHeader xhr "Content-Type" "application/transit+json")
```

We also want the `application/transit+json` format back from the server, so we need set the `Accept` header otherwise we might get back something else.

```
(.setRequestHeader xhr "Accept" "application/transit+json")
```

Remember I mentioned that the send function gets called with 2 arguments. The first is a map of queries (one for each remote), the second is a callback to ask 'The Reconciler' to merge the response into our local app-state and re-render the affected components. You can see this happen in the line

```
(cb response)
```

(I've omitted to do so but in a production application you should also remember to handle errors appropriately).

##Receiving the request

We'll use my HTTP library **yada** to help build the remote service. One of the design goals of **yada** is to allow you to create re-usable resources that can be parameterized, to discourage the copy-and-paste coding approach prevalent in writing APIs.

```
(require '[yada.yada :refer [resource yada]])

(def transit #{"application/transit+msgpack"
               "application/transit+json;q=0.9"})

(defn om-next-query-resource [parser env]
  (resource
   {:methods
    {:post
     {:consumes transit
      :produces transit
      :response (fn [ctx] (parser env (:body ctx)))}}}))
```

The `om-next-query-resource` is a factory which creates a web resource. The web resource declares a POST method and that it talks Transit. These protocol details can be ignored by users of this resource.

**yada** an HTTP feature called 'content negotiation' which allows clients to dictate the specific Transit format they want to use. The query result will be encoded to the appropriate format, automatically.

Focus

The result of the response function will be a Clojure map, the results of running the Om Next query expression provided in the request body `(:body ctx)`, along with an _environment_. In this example I've passed in the environment. You can put anything in the environment you might need to read your properties, such as database connections, URLs to other data services, etc.

The parser is built in the same way as on the client. As before, we'll need `read` and `mutate` functions.

Here's a read function. As on the client, we use a multi-method to dispatch on the property key.

```
(defmulti readf (fn [env k params] k))

(defmethod readf :customers/by-id [env k params]
  {:value (db/get-customers-by-id (:db env))})
```

Here we read from a database, using a database connection that we'll provide in the environment.

We build the parser as we did on the front-end, with `om/parser`.

Finally we put it all together, firing up an Aleph server on port 3000, and using bidi to route `/props` to our Om Next Remote.

```
(require 'aleph.http
         'bidi.ring
         '[om.next.server :as om]
         '[yada.yada :refer [yada]])

(aleph.http/start-server
  (bidi.ring/make-handler
    ["/props"
      (yada (om-next-query-resource
              (om/parser {:read readf :mutate mutatef})
              {:db (new-database-connection)}))])
 {:port 3000})
``` 

Here I'm pretending there's some `new-database-connection` function that you've provided to create a database connection. The point is, you can put anything you like here, to query any data you might want to supply to your Om Next apps. Yes, Om Next works great with Datomic, but the truth is you can easily integrate any type of data source.

Surprisingly, this is all the code we need to write for the Om Next Remote.

##Serving the application files

To avoid cross-origin issues you might want to serve you HTML and generated JavaScript files with **yada** too. Your bidi routes may then look something like this:

```
(require '[clojure.java.io :as io])

(def routes
  ["/"
    [
      ["app" (yada (io/file "target"))]
      ["props" (yada (om-next-query-resource …))]]])
```

With this ease of integration, and for numerous other advantages not covered here, it does feel like Om Next and **yada** are a great fit for each other. (Disclosure: I'm the author of **yada**)

##Mutations

Once everyone had a working multi-tier system we added the ability to 'like' items in the web-app. This would produce transactions that were recorded by Om Next. Each transaction is given a unique UUID.

We fired up our browser repls and used these UUIDs to travel through time to various states of our application.

##core.async on the front-end

core.async remains a useful library in Om Next, it just isn't necessary for state management. There are still other roles it can play however. I've certainly enjoying employing core.async for writing interactive diagrams in the past.

It turns out that learning core.async in the browser is much easier, and more fun, than on the back-end. The browser gives more opportunity to add visuals to see what is going on. I've also found on courses that it is easy for a beginner to make a mistake with a `go-loop` such that it never terminates and a reboot of the JVM is required. If this happens in the browser, the reboot is simply a matter of hitting the refresh button.

Of course, there are some core.async functions (those ending in a double-bang `!!`) that are only available in a truly multi-threaded environment. However, all the important core.async concepts can be taught in a browser, for me the pros easily outweigh the cons.

We played around with core.async features, introducing `timeout`, `alts!`, `go` and `go-loop`-ы. Then we moved on to transducers and how they can be applied to channels.

After that I spoke about the following core.async extras:

- `mult` and `tap`
- `pub` and `sub`
- `mixer`

Возвращаясь назад к серверному коду мы строим `core.async` каналы в Clojure. Используя `rand-nth`, `timeout`, channel, `go-loop` и `mult` мы строим простую **?news-feed?** который посылает новостные заголовки в канал с постоянными интервалами. Мы хотим что бы это появлялось в клиентской части через Server Sent Events.

С SSE мы не хотим, что бы клиент получал исходное сообщение, но мы хотим переслать копию сообщения каждому клиенту. Для этого исспользутся `mult`. С `mult` мы можем мультиплицировать сообщение в канал для одного или более потребителей.

```
(def mlt (mult ch)]
```

Теперь мы готовы построить **?Server Sent Event?** стрим. Повторюсь, с этим вам может помочь **yada**.

Мы создаем ресурс который который производит **?content-type?** `text/event-stream` это говорит браузеру, что это event stream. Мы так же представить response в виде фенкции (для лучшего контроля) или вернуть `mult` как константа и **yada** позаботиться об остальном.

```
["newsfeed" (yada (resource {:methods
                              {:get
                                {:produces "text/event-stream"
                                 :response mlt}}}))]
```

SSE стрим также может быть использован для  **?communicate transactions?** (изменения стостояния) в Om Next клиенте, называется `side-loading`.

##Заключение

С моей точки зрения, имея много серьездных проектом на Om за последние 2 года, преимушества опыта работы с Om не значительны или отсутствуют вообще. Следовательно не волнуйтесь если считаете что что-то будет упущено, вы можете игнорировать Om и выучить Om Next с нуля так же легко как любой другой.

Единственное сложное понятие в Om Next это **?query expressions?**. Как только вы разберетесь с этим все остальное отностельно легко, просто нужно выучить довольно много новых поняти, и сперва это может ошеломить. Не спешите, проявите упорство и вы поймете что Om Next стояло потраченного времени.
