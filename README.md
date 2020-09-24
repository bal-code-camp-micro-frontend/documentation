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
- Client-side composition with web components
- Performance improvement using ssi for the recommendations skeleton 
- Shadow dom hides the origin element of an event and returns it self
- 


## talks
- [Micro Frontend Architecture - Luca Mezzalira, DAZN](https://www.youtube.com/watch?v=BuRB3djraeM)
- [SE Radio - Episode 422: Michael Geers on Micro Frontends](https://www.se-radio.net/2020/08/episode-422-michael-geers-on-micro-frontends/)
- [Frameworks and webcomponents by Filip Bech Larsen](https://www.youtube.com/watch?v=aJ9vqyWWCOw&list=PLVI0Ut22uwY5n8nKfDZeUb14tNksI4ny4&index=8)

## articles
- [Martin Fowler - Micro Frontends](https://martinfowler.com/articles/micro-frontends.html)
- [micro-frontends.org](https://micro-frontends.org/)

## books
- [Micro Frontends in Action by Michael Geers](https://www.manning.com/books/micro-frontends-in-action) (to appear in October)

## tools
- [Single SPA JS](https://single-spa.js.org/)
- [Frint JS](https://github.com/frintjs/frint)
- [mosaic9 Zalando](https://www.mosaic9.org/)
- [Web Components](https://www.youtube.com/watch?v=aJ9vqyWWCOw&list=PLVI0Ut22uwY5n8nKfDZeUb14tNksI4ny4&index=8)
  - [Vue build Web Components](https://cli.vuejs.org/guide/build-targets.html#web-component)
  
