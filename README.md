# flex-sdk-web

[![npm version](https://img.shields.io/npm/v/@cybersource/flex-sdk-web.svg)](https://www.npmjs.com/package/@cybersource/flex-sdk-web)

Storing your customer’s card data dramatically increases your repeat-custom conversion rate, but it can also introduce additional risks along with a [PCI DSS](https://www.pcisecuritystandards.org/) overhead - these are mitigated by tokenizing the card data. CyberSource replaces your customer’s card data with a token that can be used only by you.

Secure Acceptance Flexible Token API provides a secure method for tokenizing card data and reducing your PCI DSS burden. Card data is encrypted on the customer’s device and sent directly to CyberSource, bypassing your systems altogether.

This SDK simplifies the client-side integration and enables:

- Full control of the user experience
- Wide range of browser support
- [SAQ A-EP](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2-SAQ-A_EP-rev1_1.pdf) eligibility

## Table of contents

- [Installation](#installation)
- [How to use](#how-to-use)
- [Example Integration](#example-integration)
- [Browser support](#browser-support)
- [Copyright and license](#copyright-and-license)

## Installation

Install the package with:

```
yarn add @cybersource/flex-sdk-web
# or
npm install @cybersource/flex-sdk-web --save
```

The SDK is exposed as a [UMD](https://github.com/umdjs/umd) providing compatibility with most module loader specs or direct use in the browser via a script tag.

## How to use

### Include in your app

```js
import flex from '@cybersource/flex-sdk-web';

const options = {};

flex.createToken(options, response => {
  // use response
});
```

OR

### Include via a script tag

```html
<html>
  <body>
    <script src="./YOUR_ASSETS_PATH/flex-sdk-web.js"></script>
    <script>
      var options = {};

      FLEX.createToken(options, function(response) {
        // use response
      });
    </script>
  </body>
</html>
```

## Docs

Full documentation is located in the `/docs` folder at the root of the installation directory.

## Example Integration

The SDK is non-prescriptive on your payment flow design, making few assumptions about your development stack. It is easily integrated into any front-end setup. Described below is a minimal example for a web app using full page refreshes and form POSTs.

### Steps:

- For each transaction a new Flex public key should be fetched prior to rendering the checkout page.
  - Refer to the [documentation on how to request a key](https://developer.cybersource.com/api/developer-guides/dita-flex/SAFlexibleToken.html).
- Flex public key is rendered to a JS variable in [JWK](https://tools.ietf.org/html/rfc7517) format.
- On the button click event all payment data is gathered and a token creation call is made using the SDK.
- When the token is created, the value is set on a hidden field and the form is submitted.
- You can use token with follow-on CyberSource services to complete payment (e.g. in the subscription ID field used in the [Simple Order API](https://www.cybersource.com/developers/integration_methods/simple_order_and_soap_toolkit_api/)).

```html
<!-- checkout template -->
<html>
  <body>
    <form id="paymentForm" action="/your/server/endpoint" method="post">
      <input type="text" id="cardNumber" />
      <input type="text" name="securityCode" />
      <input type="text" name="cardExpirationMonth" />
      <input type="text" name="cardExpirationYear" />
      <select name="cardType">
        <option value="001">VISA</option>
        <option value="002">MasterCard</option>
        <option value="003">AmericanExpress</option>
      </select>
      <input type="hidden" name="token" />
      <button type="button" id="payBtn">Pay Now</button>
    </form>
    <script>
      var jwk = {
        /* flex public key in jwk format */
      };
    </script>
    <script src="integration.js"></script>
  </body>
</html>
```

```js
// integration.js
import flex from '@cybersource/flex-sdk-web';

document.querySelector('#payBtn').addEventListener('click', () => {
  const options = {
    kid: jwk.kid,
    keystore: jwk,
    encryptionType: 'RsaOaep', // ensure this matches the encryptionType you specified when creating your key
    cardInfo: {
      cardNumber: document.querySelector('#cardNumber').value,
      cardType: document.querySelector('select[name="cardType"]').value,
      cardExpirationMonth: document.querySelector('input[name="cardExpirationMonth"]').value,
      cardExpirationYear: document.querySelector('input[name="cardExpirationYear"]').value
    }
  };

  flex.createToken(options, response => {
    if (response.error) {
      alert('There has been an error!');
      console.log(response);
      return;
    }

    document.querySelector("input[name='token']").value = response.token;
    document.querySelector('#paymentForm').submit();
  });
});
```

_Note that the SDK can easily be integrated into a single page application, using AJAX for the key fetch and token submission._

## Browser support

- Chrome 37+
- Internet Explorer 11
- Safari 7.1+ \*
- iOS Safari 8+ \*
- Firefox 34+
- Edge 12+

_Note that Safari 10 and below does not support an encryptionType of RsaOaep256_

## Copyright and license

Code and documentation copyright 2017 [CyberSource](http://www.cybersource.com). Released under the CyberSource SDK License Agreement as detailed in `./LICENSE.md`.
