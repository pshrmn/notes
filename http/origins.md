# Origins

Resources may make requests for other resources. For example, an HTML document may include an `<img>` tag to display an image.

These requested resources may come from the same origin or from another origin.

Browsers add security restrictions for resource requests to other origins. This is the "same-origin policy".

## Same-origin policy

([same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy))

An origin is specified by a location's `protocol`, `hostname` and `port`.

A script on one page can only access data from another if they share the same origin.

This restriction only applies to some types of data, while others can be freely used across origins. Examples of freely used resources are images (`<img>`), stylesheets (`<link rel="stylesheet">`), scripts (`<script>`), and forms (`<form action>`).

```html
<!-- http://www.example.com -->

<!-- load an image from another origin -->
<img src="http://animals.example.com/horse.png" />

<!-- load a stylesheet from another origin -->
<link rel="stylsheet" href="http://cool.style/index.css" />

<!-- load a script from another origin -->
<script src="https://unpkg.com/react/"></script>

<!-- submit form data to another origin -->
<form action="http://data.example.com/submit">
```

Data access across resources can be managed using [cross-origin resource sharing](./cors.md).

Same-origin policy and CORS isn't a silver bullet against all different origin attacks. Resource requests that aren't restricted by same-origin policy can also be used to make malicious requests. This is known as [cross-site request forgery](./csrf.md).
