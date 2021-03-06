# koa-isomorphic-relay

Koa middleware for isomorphic React + Relay rendering and routing.
This middleware implements basic server side pre-rendering, which is required for isomorphic rendering.

## Components
- `react` + `react-dom` for rendering
- `react-helmet` for document head management
- `relay` for data fetching and management
- `isomorphic-relay` to make Relay isomorphic (see facebook/relay#136 and facebook/relay#558)
- `react-router` for routing
- `react-router-relay` to add support for Relay to the router
- `isomorphic-relay-router` to make Relay Router isomorphic

## Installation
```bash
$ npm install koa-isomorphic-relay
```

## Usage

#### Server side
```javascript
import Koa from 'koa';
import renderServer from 'koa-isomorphic-relay';

const app = new Koa();

// Place this at the end of the middleware stack as it will always render a page or throw an error
app.use(renderServer({
    // URL of a GraphQL server
    graphqlUrl: 'http://localhost:8080/graphql',

    // Routes object from react-router-relay
    routes: routes,

    // Rendering function (allows for use of templating engines for example)
    render: async (reactOutput, preloadedData, helmet) => {
        return `
            <!DOCTYPE>
            <html>
                <head>
                    <meta charset="utf-8">
                    <meta name="viewport" content="width=device-width, initial-scale=1">
                    ${helmet.title.toString()}
                    ${helmet.meta.toString()}
                    ${helmet.link.toString()}
                </head>
                <body>
                    <div id="root">${reactOutput}</div>
                    <script id="preloaded-data" type="application/json">${preloadedData}</script>
                    <script src="/app.js"></script>
                </body>
            </html>
        `;
    }
}));
```

#### Client side
```javascript
import ReactDOM from 'react-dom';
import {browserHistory} from 'react-router';
import IsomorphicRelay from 'isomorphic-relay';
import IsomorphicRouter from 'isomorphic-relay-router';

// Inject preloaded data from the server side
const data = JSON.parse(document.getElementById('preloaded-data').textContent);
IsomorphicRelay.injectPreparedData(data);

// Find the root element
const rootElement = document.getElementById('root');

// Use the same routes object as on the server
ReactDOM.render(
    <IsomorphicRouter.Router routes={routes} history={browserHistory} />,
    rootElement
);
```
