# how to build an application in parallel

one of the biggest weaknesses of technology companies is effectively designing a development pipeline so that teams can be as productive as possible, relying on each other as much as possible, while at the same time reducing effort and therefore time spent on post-development tasks, chores, bugfixes et al

i've worked on several projects in the past. some were terrible, but those who strived only did because of well-organized processes and tools that empowered engineers

the following works particularly well on projects made from scratch, but might also work on existing teams -- with a little tweaking

## define a strong contract

hold conversations in your team so you can agree on a strong contract (entities, endpoints, protocols and so on) before working on the data layer of the app

that way, both teams could develop the front and back-end in parallel, without worrying about each other's bottlenecks or having to delay product delivery because of implementation inconsistencies

## mock the api

use something like [mirage](https://miragejs.com/) so you can create endpoints on the front-end that replicate the contract defined beforehand. it's a cake walk...

```js
import { createServer } from 'miragejs'

const server = createServer()

server.namespace = '/api'

server.get('/users', {
  users: [
    {
      id: 1,
      name: 'Bob'
    }
  ]
})
```

save this file as `src/setupMockAPI.js`

## ... only while developing

chances are that you're using webpack. as it's a highly customizable module bundler, it's not difficult to create an entry that gets bundle depending on the environment

the simplest form of an entry that gets stripped out while bundling for production is something like

```js
const shouldIncludeMockAPI =
  process.env.NODE_ENV === 'development' || process.env.SHOULD_INCLUDE_MOCK_API

const webpackConfig = {
  entry: [
     shouldIncludeMockAPI && 'src/setupMockAPI.js',
    'src/index.js',
  ].filter(Boolean),
}
```

and like that, you don't need to worry about flags and stripping out helpers while crafting your front-end application. let machines do the hard work for you :-)

## ... and iterate with your teammates

at the end of every iteration (i.e. end of sprint), unite with the back-end team and project manager so you can validate the current evolution of the product

using a mirage feature called [passthrough](https://miragejs.com/docs/getting-started/overview/#passthrough), you can remove mocking for certain routes and let some others get fake data

all you need to do is to generate the production build of the app but including the `SHOULD_INCLUDE_MOCK_API` environment variable, so that the request interception mechanism gets bundled

don't forget to set the real api endpoint, allowing you to consume the available endpoints and mock the ones that do not yet exist

## that's cool... but how about the back-end?

as the back-end team have the contract defined, they can implement and test on their own

but there are some points i'd like to highlight

- don't forget about [cors](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) if your client is served from other origins. that's something you might know in advance so you can make integrations easier;

- don't make assumptions about your clients. do not try to hide routes for mobile apps, for example. i found that this approach constantly led to confusion and more complexity, plus it's easily spoofable because you'd need to rely on headers or some explicitly passed parameter from the client;

- and the client can be whatever they want -- including people with bad intentions. so stay paranoid and do not trust what's coming from the outside world at all;

- rely on test-driven development because you do have the contracts, you do have the inputs, outputs and the resources of the api, so nothing stops you from adding tests from day one

## that's it

go build something awesome. i hope that article helped and please let me know about other tips, or if you disagree with this text! thanks for reading!
