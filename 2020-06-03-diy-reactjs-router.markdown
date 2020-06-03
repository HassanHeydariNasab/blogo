---
layout: post
title: "DIY reactJS router"
date: 2020-06-03 16:49:00
categories: programming
---

I can't remember when and why I decided to write my own router for reactJS. I want to share my code with you:

1 - Map url to component

url_regex cannot be used as key directly; so I use arbitrary name as key.

```js
const routes = {
  hello_stranger: {
    url_regex: /\/hello\//,
    component: (props) => <div>hello stranger!</div>,
  },
  hello: {
    url_regex: /\/hello\/\?name=(?<user_name>.*)/,
    component: (props) => <div>hello {props.user_name}</div>,
  },
  book: {
    url_regex: /\/books\/(?<book_id>\d*)\//,
    component: (props) => <Book {...props} />,
  },
  books: { url_regex: /\/books\//, component: (props) => <Books {...props} /> },
  home: { url_regex: /\/$/, component: (props) => <Home {...props} /> },
  not_matched: { url_regex: /.*/, component: (props) => <div>Not Found!</div> },
};
```

2 - Match url and find route's name and route's props

Use regex match group for props.

```js
function processed_url(url) {
  let route_name = "not_matched";
  let route_props = {};
  for (let name in routes) {
    let match_result = url.match(routes[name].url_regex);
    if (match_result) {
      route_name = name;
      route_props = match_result.groups;
      break;
    }
  }
  console.log("route matched", route_name, route_props);
  return [route_name, route_props];
}
```

3 - Navigate and store props

Navigate without page refreshes. Also store route's props in order to pass it to the component.

```js
export function navigate(url) {
  const [name, props] = processed_url(url);
  window.history.replaceState({}, name, url);
  store.dispatch(setRouteAction({ name, props }));
}
```

4 - Render the component

Call `navigate` for the first time in order to set route's name and route's props.
Also you can call `navigate` anywhere in your code and enjoy!

```js
function Routes() {
  useEffect(() => {
    navigate(window.location.pathname + window.location.search);
  }, []);
  const routerState = useSelector((state) => state.router);
  return (
    <>{routes[routerState.route.name].component(routerState.route.props)}</>
  );
}

export default Routes;
```

