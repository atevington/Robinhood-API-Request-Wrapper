# Robinhood API Request Wrapper

A wrapper for the (currently private) [Robinhood](https://www.robinhood.com/) REST API. See the [unofficial documentation](https://github.com/sanko/Robinhood).

## Basic Usage

````javascript
// Utils + base request class
const RobinhoodWrapper = require("./path/to/Robinhood-API-Request-Wrapper")

// Extra methods for common tasks
const Utils = RobinhoodWrapper.Utils

// Base request class - methods are a 1:1 mapping of the API
const RobinhoodRequestWrapper = RobinhoodWrapper.Request

// Create an api instance - MFA must be disabled on your account
const api = new RobinhoodRequestWrapper("username", "password")
````

## Samples

````javascript
// List all accounts
api.getAccounts().then(response => {
	console.log(response.results)
})

// List all supported markets
api.getMarkets().then(response => {
	console.log(response.results)
})

// Search for a security - any input or by symbol
api.searchInstruments("Apple").then(response => {
	console.log(response.results)
})

// Get a quote
api.getQuote("AAPL").then(response => {
	console.log(response)
})

// Get multiple quotes
api.getQuote(["AAPL", "FB"]).then(response => {
	console.log(response)
})
````

## Response Links

Many responses contain links to other resources in the API. For example, the response from `getQuote` will contain a property called `instrument` that is a url to the security in detail. This library activates those links so that they can be easily accessed by `$propName`, e.g. `$instrument`.

````javascript
// Get the instrument for the returned quote
api.getQuote("AAPL").then(response => response.$instrument()).then(instrument => {
	console.log(instrument)
})
````

By default, it's assumed that the activated links will be `GET` requests. However, you can override that behavior as follows:

````javascript
// Get the instrument for the returned quote
api.getQuote("AAPL").then(response => response.$instrument("POST")).then(instrument => {
	console.log(instrument)
}, error => {

	// This should throw since the instrument endpoint expects the request method to be GET
	console.log(error)
})
`````

This is also useful for paginated responses, which are accompanied by `previous` and `next` properties (activated as `$previous` and `$next`).

## Utils

The `Utils` export contains some functions that compose several API methods for common tasks.

````javascript
// Robinhood represents accounts as an array, this just grabs the first active one
Utils.getFirstActiveAccount(api).then(account => {
	console.log(account)
})

// Get your position for a given symbol - could resolve with null
Utils.getPositionForSymbol(api, "AAPL").then(position => {
	
	// Note that quantities are returned as strings!
	console.log(position)
})

// Place a good-for-day immediate market order, quantity > 0 to buy, quantity < 0 to sell
// Note that market orders are actually converted to limit orders
// https://support.robinhood.com/hc/en-us/articles/208650386-Order-Types
// So this implementation just sets the limit price at the current bid or ask
Utils.placeOrderAtMarket(api, "AAPL", 1).then(status => {
	console.log(status)
})
````