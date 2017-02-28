# Dynatrace Salesforce Commerce Cloud (formerly Demandware) Fastpack for AppMon 6.5


![images_community/download/attachments/215745785/icon.png](/images_community/download/attachments/215745785/icon.png)

This fastpack is for use with Dynatrace Application Monitor Version 6.5. Branch v1.0_AppMon_6.3 is for AppMon 6.3

The Dynatrace FastPack for Salesforce Commerce Cloud provides a preconfigured Dynatrace profile custom tailored to Salesforce Commerce environments. This FastPack contains sensors, a template system profile with measures and business transactions, and dashboards for the Salesforce Commerce platform. Because the Salesforce Commerce integration with UEM leverages CORS, there are a few measures that will likely have to be customized to your environment. 

## Customizations:
Dynatrace for Salesforce Commerce Cloud is only for UEM via [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) as java agents are not allowed on the Salesforce Commerce JVMs. As a result, some of the custom measures and business transactions in the fastpack are dependent User Action names and CSS Selectors specific to the Salesforce Commerce supplied SiteGenesis store. Because the web frontend of a Salesforce Commerce store can be customized, there's no guarantee the foundation of the measures will exist in your store.  However, these can easily be replaced.

### Customization #1: Username Tag

For the Username Tag, this fastpack leverages the CSS Selector ```[class='user-account']@title```, which is stored in a Page Action Metadata measure called Username.  The following Transformation regular expression is run on the result:

```
User: (?!Login \/  Register$)(.*)
```

In the case of SiteGenesis, when a User is not logged in, the value is filled in with 'Login / Register'. The regex used here excludes this state so that visits are not mis-tagged with the default value. If your store displays the username with a different CSS Selector, adjust the particulars in the corresponding sections.

#### Username UEM Configuration:
![Username UEM Configuration](/images_community/download/attachments/215745785/Username_UEM_Configuration.png)

#### Username Meta Measure Configuration
![Username Meta Measure](/images_community/download/attachments/215745785/Username_Meta_Measure.png)

To learn more about metadata measures, please refer to [Dynatrace documentation](https://community.dynatrace.com/community/display/DOCDT63/System+Profile+-+User+Experience#SystemProfile-UserExperience-WebSettings).

### Customization #2: Conversion Goal

The Conversion Goal is a Page/User action used to identify when a user completes an order. Using a conversion goal has many benefits to understanding the conditions under which a user did or did not complete a purchase. While there is only one CSS Selector used, two measures exist as the syntax captured by web vs. mobile web is very different. As with the example in the fastpack, you may find there's a different conversion target for mobile vs. desktop web. You can add multiple conversion goals.

In the Salesforce Commerce Fastpack, we configured the Conversion Goal to be set to a measures called OrderConfirmationNumber and OrderConfirmationMobileWeb. Both of these measures are based on CSS Selector Meta Measures that pick up the order id from the order confirmation page.:


You can change your conversion goal by substituting any other measure you create.

#### Conversion Goal UEM & CSS Selector:
![Conversion Goal UEM & CSS Selector](/images_community/download/attachments/215745785/Conversion.png)

#### Conversion Goal Measure
![Conversion Goal Measure](/images_community/download/attachments/215745785/OrderConfirmationMeasures.png)

### Customization #3: Orders and Revenue
In order to track orders and revenue from orders over time, a User Action business transaction called Orders was created. This is slightly different than standard conversions as a conversion is visit based. If a user makes multiple orders in a single visit, that counts as 1 conversion. However, since I've personally made several orders on commerce sites in a single visit, I thought it would be best to track orders and revenue by the occurence based on User Actions. This way, if multiple orders are made in a single visit, each order and revenue value will be tracked. This Business transaction looks for every occurence of a user landing on the OrderConfirmationPage and tracks the number of occurences and PurchaseValue. These can additionally be split by client type/family, country, User Experience Index of Visit, Operating System or Application

The Business Transaction is reliant on eight measures, two of which you will have to customize to your environment:
- OrderConfirmationPage
- PurchaseValue
- Client Type of Visits
- Country of Visits
- User Experience Index of Visits
- Operating System of Visits
- Client Family of Visits
- Application

#### Orders Business Transaction
![Orders BT](/images_community/download/attachments/215745785/OrdersBT.png)

The two measures you will likely have to customize are:
- OrderConfirmationPage
- PurchaseValue

#### OrderConfirmationPage
This measure is based upon the destination page URI containing the following, based on the SiteGeneisis store
```
orderconfirmation
```
If your order confirmation URI is different, please adjust this.

#### OrderConfirmationPage Measure
![OrderConfirmationPage Measure](/images_community/download/attachments/215745785/OrderConfirmationPageMeasure.png)

#### PurchaseValue

This measure has two parts: 1) custom JavaScript code in your page to capture the order value 2) the measure. 
This measure captures the total value from the order confirmation page. 

- Custom javaScript Code

The Dynatrace JavaScript ADK is already embedded in your UEM code. As long as UEM is deployed in your store, the ADK is available. One thing to note - since UEM is being injected asynchronously, this means that the ADK is asynchronously loaded as well. Be sure to only use the ADK and accompanying js code after the ADK is loaded.  [For more info, please refer to the Dynatrace JavaScript ADK documentation.](https://community.dynatrace.com/community/pages/viewpage.action?title=JavaScript+ADK&spaceKey=DOCDT65)

In the SiteGenesis store, when a order is successfully completed, the user lands on the order confirmation page which lists the order total, among other things. This script works for SiteGenesis, but you may have to modify it based on your store's design. This example leverages jquery. I should also mention that I'm not a js expert, so any improvements on this are welcome in the comments.

In SiteGeneis store, add the following to templates->default->checkout->confirmation->confirmation.isml

```
<script type="text/javascript" >
  window.addEventListener("load", function(){
	   var PurchaseString = $('.payment-amount').text().replace(/[Amount: $]|,/g, '');
		  var PurchaseValue = parseFloat(PurchaseString);
		  dynaTrace.reportValue('PurchaseValue',PurchaseValue);
 });
</script>
```

This script captures the payment-amount text, strips the extraneous characters, converts it to a float and then passes it back to dynatrace with the Dynatrace JS ADK function 'dynaTrace.reportValue'.  The value is passed in a variable called PurchaseValue. Please note, the function also strips out any commas in the numeric value. This is done because for now, dynatrace cannot handle a comma in a numerica value being passed back. Once this issue is resolved, this can all be done with the CSS Selector like used for UserName.   

- the Measure

The PurchaseValue measure is simply a JavaScript ADK Value (User Actions) measure, set to pick up the value of PurchaseValue as passed back from the ADK. 

#### PurchaseValue Measure:
![PurchaseValue Measure](/images_community/download/attachments/215745785/PurchaseValue.png)


### Customization #4: Abandoned Cart
 
The Abandoned Cart measure counts the number of Visits where users added at least one item to their cart but did not purchase. Along with counting how many abandoned carts there are in a given time period, the business transaction also tracks which pages customers are abandoning on, the response time of the exit page, the monetary value of the abandoned cart, the Client Type and Family of the Visits as well as the User Experience Reason of the visit.
 
 The Business Transaction is reliant on nine measures, two of which you will have to customize to your environment:
- Add to Cart User Action Count: a User Action Count measure based on the syntax of your store's Add To Cart button.
- miniCartValue: a measure that evaluates the numeric value of the cart passed to Dynatrace via the JavaScript ADK
- Non Conversion Goal: A conversion goal measure that tracks when users don't convert. This only works when the Conversion Goal is set as in Customization #2
- User Action Response Time: A measure that will track the response time of the Abandoned Cart Visit's Exit page.
- Client Family of Visits: this measure will split the abandoned carts by Browser make (firefox, chrome mobile, etc)
- Client Type of Visits: this measure will split the abandoned carts by browser type of visit (Desktop, mobile, synthetic)
- Visits - Exit Page - BT: a measure that captures the exit page of a visit
- Application: If you define more than one application

#### Abandoned Cart Business Transaction
![AbandonedCart BT](/images_community/download/attachments/215745785/AbandonedCartBT.png)

The two measures you will likely have to customize are:
- Add to Cart User Action Count
- miniCartValue

#### Add to Cart User Action Count
This measure is based upon the button name of Add to Cart. The value the measure is looking for is an action name regular expression:

```
(.*)(?i)add to cart(.*)
```

The regular expression looks for a User Action containing 'add to cart' regardless of capitalization and what precedes or follows it.

#### Add To Cart Measure:
![Add To Cart Measure](/images_community/download/attachments/215745785/AddToCartUserActionCount.png)

#### miniCart Value
This measure has two parts: 1) custom JavaScript code in your page to capture the minicart value 2) the measure. 
This measure also assumes, like in the SiteGenesis store, the minicart value is exposed on every page once a user adds to cart. If your store is setup differently, you'll have to modify to capture the cart value where it's exposed and evaluate the last captured value in your cart.

- Custom javaScript Code

The Dynatrace JavaScript ADK is already embedded in your UEM code. As long as UEM is deployed in your store, the ADK is available. One thing to note - since UEM is being injected asynchronously, this means that the ADK is asynchronously loaded as well. Be sure to only use the ADK and accompanying js code after the ADK is loaded.  [For more info, please refer to the Dynatrace JavaScript ADK documentation.](https://community.dynatrace.com/community/pages/viewpage.action?title=JavaScript+ADK&spaceKey=DOCDT65)

In the SiteGenesis store, when an item is added to the cart, the minicart is available on all of the rest of the pages in the visit. This script works for SiteGenesis, but you may have to modify it based on your store's design. This example leverages jquery. I should also mention that I'm not a js expert, so any improvements on this are welcome in the comments.

In SiteGeneis store, add the following to templates->default->checkout->cart->minicart.isml

```
<script type="text/javascript" >
window.addEventListener("load", function(){
	 if ($('div.mini-cart-subtotals').length > 0) {
 		var miniCartString = $('.mini-cart-subtotals').text().replace(/,|[Order Subtotal]|[\$]|[\n]| ' '/g, '');
 		var miniCartValue = parseFloat(miniCartString);
 		dynaTrace.reportValue('miniCartValue',(miniCartValue));
		}
	});
</script>
```

This script captures the minicart subtotal, strips the extraneous characters, converts it to a float and then passes it back to dynatrace with the Dynatrace JS ADK function 'dynaTrace.reportValue'.  The value is passed in a variable called miniCartValue. Please note, the function also strips out any commas in the numeric value. This is done because for now, dynatrace cannot handle a comma in a numerica value being passed back. Once this issue is resolved, this can all be done with the CSS Selector like used for UserName.   

- the Measure

The miniCartValue measure is simply a JavaScript ADK Value (User Actions) measure, set to pick up the value of miniCartValue as passed back from the ADK. 

#### miniCartValue Measure:
![miniCartValue Measure](/images_community/download/attachments/215745785/AbandonedCartValue.png)


Find further information in the [dynaTrace community](https://community.dynatrace.com/community/display/DL/Salesforce+Commerce+Cloud+%28formerly+Demandware%29+FastPack). 




