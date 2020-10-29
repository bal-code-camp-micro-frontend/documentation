# documentation

## Kwik-E-Mart
Our experiments in the code camp were heavily guided by the "Micro Frontends in Action" 
book (see the example code on github <https://github.com/naltatis/micro-frontends-in-action-code>).

Three independent teams develop the online shop Kwik-E-Mart:
- Product detail page: <https://github.com/bal-code-camp-micro-frontend/kwik-e-detail>
- Product list incl. recommendations and search: <https://github.com/bal-code-camp-micro-frontend/kwik-e-list>
- Checkout incl. mini basket and add/remove buttons: <https://github.com/bal-code-camp-micro-frontend/kwik-e-checkout>   

The first approach was using direct links from list to the detail page. There was no 
checkout yet.  

### v1 Server Side Includes
:link: <https://kwik-e-mart-v1-ssi-proxy.apps.baloise.dev/>

- The list team exposes a recommendations fragment under /l/recommendations/:id
- The detail page includes recommendations via server side includes:
```
 <!--#include virtual="/l/recommendations/12" -->
```
We had to use a workaround to include the text in the dom using thymeleaf:
```
<div class="col s12" th:utext="${product.recommendationFragment}"></div>
```
- We use nginx to perform the ssi in the project <https://github.com/bal-code-camp-micro-frontend/kwik-e-proxy>
```
server {
  listen 8080;
  ssi on;

  location /healthz {
    return 200 'woop woop!';
    add_header Content-Type text/plain;
  }

  location /l/ {
    proxy_pass  https://kwik-e-list.apps.baloise.dev;
  }
  location /d/ {
    proxy_pass  https://kwik-e-detail.apps.baloise.dev;
  }
  location /c/ {
    proxy_pass  https://kwik-e-checkout.apps.baloise.dev;
  }
  location / {
    root /opt/html;
    try_files $uri $uri/ /index.html;
  }
}
```      
With this configuration we can use the same url for all applications. 

### v2 Web Components
:link: <https://kwik-e-mart-v2-webcomponents-proxy.apps.baloise.dev/>

-> Client side composition with web components.

#### Example 1: *Add to cart* button:

Provider team:
```javascript
// /c/components/add-to-cart-button.js

const checkoutShoppingCartTemplate = document.createElement('template');
checkoutShoppingCartTemplate.innerHTML = `
  <link href="h/c/components/css/add-to-cart-button.css" rel="stylesheet" />
  <button>
    <!-- ... -->
  </button>
`;

class CheckoutShoopingCart extends HTMLElement {
  connectedCallback() {
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.appendChild(checkoutShoppingCartTemplate.content.cloneNode(true));
  }
  /* functionality */
}

customElements.define('c-shopping-cart', CheckoutShoopingCart);
```

User team:
```html
<script src="/c/components/add-to-cart-button.js"></script>
<c-add-to-cart-button product-id="1"></c-add-to-cart-button>
```

Resulting DOM:
```html
<c-shopping-cart id="1">
 #shadow-root ==
 <link href="/c/components/css/add-to-cart-button.css" rel="stylesheet">
 <button>
    <!-- ... -->
  </button>
</c-shopping-cart>
```

Conventions to keep things clean:
- Every team has its own prefix (e.g. team checkout: 'c')
- Every team follows the same path structure for includes etc.

Every user of the `c-add-to-cart-button` now only needs to know that the API of this micro frontend component is the attribute `product-id`.

#### Example 2: *Recommendations* view:

Issue: Visually big web components that need some time to initialize cause the web site flicker.
Solution: Use skeletons until the actual content is displayed. The responsibility to provide a skeleton that matches the final content is responsibility of the content team. You can e.g. use SSI to include this skeleton before sending the page to the browser.

```html
<script src="/l/components/recommendations.js"></script>
<l-recommendations product-id="4711">
    <!--#include virtual="/l/skeletons/recommendations" -->
</l-recommendations>
```

#### Example 3: Communication between micro frontends of *one* team

```javascript
// c-add-to-car-button onClick handler:
this.dispatchEvent(new CustomEvent('c:cart:changed', { // notice the 'c' prefix for team checkout
  bubbles: true, // bubble all the way to the window element
  composed: true,
  detail: { productId: this.productId }
}));
```
```javascript
// c-shopping-cart:
connectedCallback() {
  window.addEventListener('c:cart:changed', this.cartChangedEventListener);
}

disconnectedCallback() {
  window.removeEventListener('c:cart:changed', this.cartChangedEventListener);
}

cartChangedEventListener = (e) => {
  // update shopping cart count
}
```

**Important**: Only use your team's *global* events. If you need to need to notify the user of one of your web components, make it part of the web component API (e.g. `<my-component onSomeEvent="usersHandlerFunc()"></my-component>`)

### v3 App Shell as Single Page Application with vanilla js
:link: <https://kwik-e-mart-v3-appshell-proxy.apps.baloise.dev/>

- Wrapped 3 developements as Webcomponents
- "Click Events" Urls propagation from single developements to upper App Shell issue: shadow dom hides the origin element of an event and returns it self
-

### v4 App Shell as Single Page Application with Single Spa
- https://single-spa.js.org/
-



## talks
- [Micro Frontend Architecture - Luca Mezzalira, DAZN](https://www.youtube.com/watch?v=BuRB3djraeM)
- [SE Radio - Episode 422: Michael Geers on Micro Frontends](https://www.se-radio.net/2020/08/episode-422-michael-geers-on-micro-frontends/)
- [Frameworks and webcomponents by Filip Bech Larsen](https://www.youtube.com/watch?v=aJ9vqyWWCOw&list=PLVI0Ut22uwY5n8nKfDZeUb14tNksI4ny4&index=8)

## articles
- [Martin Fowler - Micro Frontends](https://martinfowler.com/articles/micro-frontends.html)
- [micro-frontends.org](https://micro-frontends.org/)

## books
- [Micro Frontends in Action by Michael Geers](https://www.manning.com/books/micro-frontends-in-action)

## tools
- [Single SPA JS](https://single-spa.js.org/)
- [Frint JS](https://github.com/frintjs/frint)
- [mosaic9 Zalando](https://www.mosaic9.org/)
- [Web Components](https://www.youtube.com/watch?v=aJ9vqyWWCOw&list=PLVI0Ut22uwY5n8nKfDZeUb14tNksI4ny4&index=8)
  - [Vue build Web Components](https://cli.vuejs.org/guide/build-targets.html#web-component)
  
