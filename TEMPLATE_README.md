<a href="http://www.crownpeak.com" target="_blank">![Crownpeak Logo](images/logo/crownpeak-logo.png?raw=true "Crownpeak Logo")</a>

# Fredhopper Proxy Example
This repository contains a simple proxy for [Crownpeak](https://www.crownpeak.com) Fredhopper using [Node.js](https://nodejs.org/) and the [node-fetch](https://github.com/node-fetch/node-fetch) library.

It forwards requests to a specified Fredhopper target URL while adding Basic Authentication, which Crownpeak requires.

The repository can be deployed as a [Netlify Function](https://docs.netlify.com/functions/overview/). With a few changes, it can also be deployed to other hosting providers, such as in Amazon Web Services as a Lambda Function.

## Use Case

This simple proxy allows client-side communication with Fredhopper’s Query API by wrapping Basic Authentication and forwarding requests to the API.

This repo should be cloned/forked to act as a base that can be extended to provide custom functionality as required.

You should always consider the impact of API requests vs performance and implement caching and other capabilities as required within your implementation.

> Note: Server-to-server integration between Fredhopper’s Query API and your customer experience is always recommended.

## Functionality

The proxy function captures incoming requests and appends Basic Authentication credentials to the request headers. It then forwards the request to a specified Fredhopper target host’s Query API and returns the response to the client.

- A Basic Authentication header is constructed using the provided `API_USERNAME` and `API_PASSWORD` and is included in all requests to the Fredhopper Query API.
- The target URL for the Fredhopper instance is constructed by combining the `TARGET_HOST` with the query parameters passed to the proxy.
- The `node-fetch` library is then used to send the request to the Fredhopper API using the specified method, headers, and body.
- The response status code is checked to determine if the request was successful.
- If successful, it extracts the response body and constructs a response object with the appropriate status code, headers, and body.

## Environment Variables

To run this function locally or deploy it on Netlify, you must set the following environment variables:

| Variable       | Description                                                    |
|----------------|----------------------------------------------------------------|
| `API_USERNAME` | Username for Basic Authentication with Fredhopper’s Query API. |
| `API_PASSWORD` | Password for Basic Authentication with Fredhopper’s Query API. |
| `TARGET_HOST`  | Base URL of the Fredhopper Query API endpoint.                 |

## Usage

This repository is ready to be deployed as a Netlify Function. The easiest way to achieve this is to use [Netlify’s default automatic build process](https://docs.netlify.com/functions/deploy/).

Once deployed, set the `API_USERNAME`, `API_PASSWORD`, and `TARGET_HOST` environment variables via the Netlify console, ensuring they are the correct values for your Fredhopper Query API.

You can send requests to the deployed proxy endpoint using the appropriate query string parameters. The proxy will forward the requests to the Fredhopper Query API and return the response.

#### Example Request:

> GET /.netlify/functions/proxy/?q=product&fh_location=//catalog01/en_US

This request will be proxied to the Fredhopper Query API at the `TARGET_HOST` with the query parameters q=product and fh_location=//catalog01/en_US.

For more details on using the [Fredhopper Query API](https://crownpeak.gitbook.io/product-discovery/fredhopper-integration-guide/fredhopper-integration-guide-1/front-end-integration), see the documentation.

To run the proxy locally, use the following commands:
``` 
npm install 
netlify dev
```

##  Legal Notices

Fredhopper Proxy Example is an example solution subject to the [MIT license](https://github.com/Crownpeak/fhr-client-proxy?tab=MIT-1-ov-file).

This proxy must not be used for any illegal activities or in ways that could harm Crownpeak or Fredhopper's systems. This proxy does not store or process any user data. All data is forwarded directly to the Fredhopper Query API.

Copyright © 2025 Crownpeak Technology, Inc. All rights reserved. Fredhopper is a trademark of Crownpeak Technology, Inc.

## Disclaimer
This document is provided for information purposes only. Crownpeak may change the contents hereof without notice. This document is not warranted to be error-free, nor subject to any other warranties or conditions, whether expressed orally or implied in law, including implied warranties and conditions of merchantability or fitness for a particular purpose. Crownpeak specifically disclaims any liability with respect to this document and no contractual obligations are formed either directly or indirectly by this document. The technologies, functionality, services, and processes described herein are subject to change without notice.
