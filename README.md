# Vue Stripe

This package provides a convenient wrapper around stripe's [checkout component](https://stripe.com/checkout) for vue.js 2

## Installation

Compile the code using `browserify` + `vueify` or `webpack` + `vue-loader`

```js
npm install vue-stripe
```

Require the script:

```js
import VueStripe from 'vue-stripe'
```

Regsiter the component:

```js
Vue.use(VueStripe)
```

If you want to listen to events require the event bus:

```js
import bus from 'vue-stripe/bus'
```

## Usage

Embed the compnenet in your form as a direct HTML child.

If your are selling a single product this will do:

```html
<form action="/process-payment" method="POST">
 <stripe-checkout
 stripe-key="my-stripe-key"
 product="product"></stripe-checkout>
</form>
```

The `product` object should match at least the bare minimum required by Stripe. E.g:

```js
{
  name:'Moby Dick',
  description:'I love whales',
  amount:100000 // 100$ in cents
}
```

Additional props:

*  `options` Additional options to be merged into the main configuration object (e.g `zipCode:true`)
*  `button` Button text. Default: 'Purchase'

When selling multiple products you can either pass them all to the client (Option A) or, if you are dealing with an espescially large number of products, get the relevant product via ajax (Option B. Requires `vue-resource`>=0.9.0).

Both options require a `product-id` prop, which references the current product id on your instance.
This would normally be a value from a select box or a router param.

Option A:

```html
<form action="/process-payment" method="POST">
 <select v-model="productId">
  <option value="1">Product A</option>
  <option value="2">Product B</option>
  <option value="3">Product C</option>
</select>
<stripe-checkout
stripe-key="my-stripe-key"
:products="products"
:product-id="productId"></stripe-checkout>
</form>
```

```js
{
      data: {
        products:[
          {
          id:1,
          name:'Product A',
          description:'Product A Description',
          amount:100000
        },
        {
          id:2,
          name:'Product B',
          description:'Product B Description',
          amount:50000
        },
        {
           id:3,
           name:'Product C',
           description:'Product C Description',
           amount:60000
        }
      ]
  }
}
```


Option B:

```html
<form action="/process-payment" method="POST">
 <select v-model="productId">
  <option value="1">Product A</option>
  <option value="2">Product B</option>
  <option value="3">Product C</option>
</select>
<stripe-checkout
stripe-key="my-stripe-key"
products-url="/products"
:productId="productId"></stripe-checkout>
</form>
```

Server side Example (Laravel)

```php
Route::get('products', function() {
  $productId = request('productId');

  return Product::find($productId);
});
```

Once the checkout form was submitted the main form will be automatically submitted.
The request will include the `stripeEmail` and `stripeToken` parameters, which will enable you to process the payment and redirect back.

## Events

use the `bus` variable to listen for events. E.g

```js
bus.$on('vue-stripe.error', function(e) {
  console.log(e)
});
```

* `vue-stripe.not-found` Fires off when the selected product was not found
* `vue-stripe.error` Fires off when an invalid response was returned from the server using the `products-url` prop
